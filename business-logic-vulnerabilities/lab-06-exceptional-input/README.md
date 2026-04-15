# Lab 06 — Inconsistent Handling of Exceptional Input

| Field | Details |
|-------|---------|
| **Category** | Business Logic Vulnerabilities |
| **Difficulty** | 🟡 Practitioner |
| **Status** | ✅ Solved |

---

## 🎯 Objective

Access the **admin panel** and delete user `carlos` by registering
with a crafted email address that gets truncated by the server
to end in `@dontwannacry.com`.

---

## 🐛 Vulnerability

The server truncates email addresses to a maximum of **255 characters**
before storing them. By crafting an extremely long email address where
the `@dontwannacry.com` part lands exactly at the end after truncation,
the stored email appears to be a legitimate company email — granting
admin access without actually owning that domain.

---

## 🛠️ Tools Used

- Burp Suite (Proxy + Repeater)
- Browser
- Email client (provided by the lab)

---

## 🔢 Steps

### Step 1 — Open the email client

Click the **Email client** button on the lab page to open
your temporary inbox. Note your email address assigned
by the lab.

![Lab email client showing assigned email address](./01-email-client.png)

---

### Step 2 — Go to registration page and intercept

Turn on **Burp Intercept** and go to the registration page.
Fill in the details but do not submit yet.

![Registration page filled in](./02-registration-page.png)

---

### Step 3 — Craft the malicious email address

You need to craft an email where after the server truncates
it to 255 characters, it ends in `@dontwannacry.com`.

The formula is:
```
[padding]@dontwannacry.com
```

Where padding fills the address so `@dontwannacry.com`
lands exactly at position 255.

Use this Python snippet to generate the payload:
```python
domain = "@dontwannacry.com"
padding = "a" * (255 - len(domain))
crafted = padding + domain + ".YOUR-LAB-EMAIL.web-security-academy.net"
print(crafted)
print(len(crafted))
```

The crafted email looks like:
```
aaaaaaa[...]aaa@dontwannacry.com.YOUR-EMAIL.web-security-academy.net
```

![Python script generating the crafted email](./03-crafted-email-script.png)

---

### Step 4 — Submit registration and intercept in Burp

Submit the registration form. Burp intercepts the POST request:
```
POST /register HTTP/2
csrf=xxx&username=attacker&email=aaa[...]@dontwannacry.com.exploit.net&password=test
```

![Burp intercept showing registration POST with long email](./04-intercept-registration.png)

---

### Step 5 — Send to Repeater

Right-click the intercepted request and click
**Send to Repeater**. Forward the original request.

![Send to Repeater in Burp](./05-send-to-repeater.png)

---

### Step 6 — Confirm registration via email

Go back to the lab email client and click the
confirmation link in the registration email.

![Email client showing confirmation link](./06-email-confirmation.png)

---

### Step 7 — Log in and check stored email

Log in with your registered credentials and go to
**My Account**. The stored email should now show as:
```
aaaaaaa[...]aaa@dontwannacry.com
```

The part after `@dontwannacry.com` has been truncated away.

![My Account showing truncated email ending in @dontwannacry.com](./07-truncated-email-stored.png)

---

### Step 8 — Access the admin panel

Navigate to `/admin` in the browser.
You now have full admin access.

![Admin panel accessible after truncated email](./08-admin-panel.png)

---

### Step 9 — Delete carlos

In the admin panel find user `carlos` and click **Delete**.

![Deleting user carlos from admin panel](./09-delete-carlos.png)

---

### Step 10 — Lab solved

![Green lab solved confirmation banner](./10-lab-solved.png)

---

## 📸 Screenshots Reference

| File | What it shows |
|------|---------------|
| `01-email-client.png` | Lab email client with assigned address |
| `02-registration-page.png` | Registration form filled in |
| `03-crafted-email-script.png` | Python script generating the payload |
| `04-intercept-registration.png` | Burp intercept with long crafted email |
| `05-send-to-repeater.png` | Send to Repeater in Burp |
| `06-email-confirmation.png` | Email client with confirmation link |
| `07-truncated-email-stored.png` | My Account showing @dontwannacry.com |
| `08-admin-panel.png` | /admin page now accessible |
| `09-delete-carlos.png` | Deleting carlos in admin panel |
| `10-lab-solved.png` | Green solved banner |

---

## 🔍 How the Truncation Attack Works

| Stage | Email Value |
|-------|------------|
| Submitted | `aaa[...]@dontwannacry.com.exploit.net` (300+ chars) |
| After server truncation | `aaa[...]@dontwannacry.com` (255 chars) |
| Domain check result | ✅ Passes — ends in @dontwannacry.com |
| Admin access granted | ✅ Yes |

---

## 🏁 Key Takeaway

> Never rely solely on string matching for access control.
> Input truncation can silently change the meaning of a value.
> Always validate the **final stored value**, not just
> the submitted one.

---

## 🛡️ Remediation

- Validate email format **after** any truncation or normalisation
- Reject emails that exceed a reasonable maximum length
  rather than silently truncating them
- Send a verification email to confirm actual ownership
  before granting domain-based access
- Never grant elevated privileges based on email domain alone

---

## 🔗 References

- [PortSwigger: Business Logic Vulnerabilities](https://portswigger.net/web-security/logic-flaws)
- [OWASP: Input Validation](https://owasp.org/www-project-web-security-testing-guide/)
```


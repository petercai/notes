
# 1
The error you're encountering during `poetry install` is related to SSL certificate verification when trying to connect to pypi.org. Here's a breakdown of the issue and how to resolve it:

---

### 🔍 **Error Summary**

```
SSLError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: 
Basic Constraints of CA cert not marked critical (_ssl.c:1032)
```

This means Python's SSL module is rejecting the certificate because the CA (Certificate Authority) certificate doesn't meet strict validation rules.

---

### ✅ **Steps to Fix**

#### 1. **Check System Date and Time**

Ensure your system clock is accurate. SSL certificates can fail if your system time is incorrect.

#### 2. **Update `certifi` Package**

Python uses the `certifi` package for CA certificates. Try updating it:

```Shell
pip install --upgrade certifi
```

#### 3. **Use System Certificates (macOS/Linux)**

If you're on macOS or Linux, you can configure Python to use system certificates:

- **macOS**:
    
    ```Shell
    /Applications/Python\ 3.x/Install\ Certificates.command
    ```
    
- **Linux**: Ensure `ca-certificates` is installed and up to date:
    
    ```Shell
    sudo apt-get update
    sudo apt-get install --reinstall ca-certificates
    ```
    

#### 4. **Set `REQUESTS_CA_BUNDLE` Environment Variable**

If you have a custom CA bundle, set this environment variable:

```Shell
export REQUESTS_CA_BUNDLE=/path/to/your/cacert.pem
```

#### 5. **Corporate Network / Proxy Issues**

If you're behind a corporate proxy or firewall (common in enterprise environments), it might be intercepting SSL traffic with its own certificate. You may need to:

- Add your company’s root certificate to Python’s trusted store.
- Configure `poetry` to use the proxy:```Shell
    poetry config http-basic.pypi username password
    poetry config proxy http://localhost:9000
    ```
    

---

Would you like help checking your proxy settings or verifying your certificate chain?
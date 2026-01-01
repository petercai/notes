You’re very close 👍  
On **Oracle Cloud Infrastructure (OCI)**, opening port **3000** requires **two layers** to allow it. Missing either one = no access.

I’ll show the **exact, minimal steps** for both, assuming:

- You’re using the **default VCN**
    
- Your VM already runs Gogs on `0.0.0.0:3000`
    

---

## 🔐 OCI networking model (important mental model)

Remote access must pass **both**:

1. **VCN Security List** (network firewall)
    
2. **Subnet route association** (already exists by default)
    

👉 Opening Linux firewall alone is **not enough** on OCI.

---

## Step 1️⃣ Open port 3000 in the VCN Security List (REQUIRED)

This is the key step most people miss.

### OCI Console path

1. Log in to **Oracle Cloud Infrastructure Console**
    
2. Go to:
    
    ```
    ☰ Menu → Networking → Virtual Cloud Networks
    ```
    
3. Click your **VCN** (usually named something like `vcn-xxxxx`)
    
4. Click **Security Lists**
    
5. Click the **Security List** attached to your subnet  
    (usually `Default Security List for <VCN name>`)
    

---

### Add an **Ingress Rule**

Click **Add Ingress Rules** and fill:

|Field|Value|
|---|---|
|**Source CIDR**|`0.0.0.0/0`|
|**IP Protocol**|`TCP`|
|**Source Port Range**|_(leave empty)_|
|**Destination Port Range**|`3000`|
|**Description**|`Allow Gogs Web UI`|

✅ Click **Add Ingress Rules**

![Image](_assets/Image.jpg)

![Image](_assets/Image.svg)

![Image](https://blogs.oracle.com/wp-content/uploads/sites/83/2025/10/image7-1.png)

---

## Step 2️⃣ Confirm subnet uses that security list

Usually already correct, but double-check once.

1. VCN → **Subnets**
    
2. Click your subnet (e.g. `public-subnet-xxxx`)
    
3. Under **Security Lists**, confirm it includes:
    
    ```
    Default Security List for <VCN>
    ```
    

If yes → you’re done.

---

## Step 3️⃣ Ensure Gogs is listening publicly

On your VM:

```bash
ss -tulnp | grep 3000
```

You **must** see:

```
0.0.0.0:3000
```

If it shows `127.0.0.1:3000`, Gogs is bound to localhost only → remote access will fail.

---

## Step 4️⃣ Test from your local machine

```bash
curl http://<your-oci-public-ip>:3000
```

Or open in browser:

```
http://<your-oci-public-ip>:3000
```

You should see the **Gogs UI**.

---

## Step 5️⃣ (Optional) Linux firewall check

On Oracle Linux images, **firewalld is usually disabled** by default.

Check:

```bash
sudo systemctl status firewalld
```

If it’s **inactive**, you’re fine.

If active:

```bash
sudo firewall-cmd --add-port=3000/tcp --permanent
sudo firewall-cmd --reload
```

---

## Quick checklist (90% of failures)

If it still doesn’t work, one of these is wrong:

- ❌ VCN ingress rule missing
    
- ❌ Wrong security list attached to subnet
    
- ❌ Gogs bound to `127.0.0.1`
    
- ❌ Using **private IP** instead of **public IP**
    

---

## Security tip (since you’re using SSH-only Git)

Once setup is complete, you may want to **restrict access**:

Instead of:

```
0.0.0.0/0
```

Use **your home IP**:

```
<your-ip>/32
```

This locks the web UI to **you only**.

---

If you want next:

- expose **port 80 without sudo**
    
- bind Gogs to **localhost only** + SSH tunnel
    
- protect Gogs with **OCI Network Security Group (NSG)** instead of Security Lists
    

Say the word and I’ll tailor it to your setup.
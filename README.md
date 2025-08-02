## **Step 1: Download \& Prepare Trojan-Go**

1. **Install prerequisites**:

```bash
sudo apt update
sudo apt install wget unzip -y
```

2. **Download and extract Trojan-Go**:

```bash
wget https://github.com/p4gefau1t/trojan-go/releases/latest/download/trojan-go-linux-amd64.zip
unzip trojan-go-linux-amd64.zip -d trojan-go-setup
cd trojan-go-setup
chmod +x trojan-go
```


## **Step 2: Create Your Trojan-Go Config File**

Inside `trojan-go-setup`:

```bash
nano config.json
```

**Paste this configuration** (matches your trojan:// link):

```json
{
  "run_type": "client",
  "local_addr": "127.0.0.1",
  "local_port": 1080,
  "remote_addr": "addYourDomain",
  "remote_port": addYourPortInPanel,
  "password": [
    "addYourPassword"
  ],
  "ssl": {
    "sni": "addYourSNI",
    "verify": false,
    "verify_hostname": false
  }
}
```

- Save with `Ctrl+O` Enter, then `Ctrl+X`.


## **Step 3: Start Trojan-Go**

In the trojan-go-setup folder:

```bash
./trojan-go -config ./config.json
```

- Leave this terminal open. If you get an error like "bind: address already in use," kill previous Trojan-Go processes:

```bash
sudo fuser -k 1080/tcp
```


## **Step 4: Install and Configure Privoxy (HTTP to SOCKS5 Bridge)**

1. **Install Privoxy**:

```bash
sudo apt install privoxy
```

2. **Configure Privoxy**:

```bash
sudo nano /etc/privoxy/config
```

    - At the **very end**, add (no `#` at front):

```
forward-socks5t   /   127.0.0.1:1080 .
```

    - Save and exit.
3. **Restart Privoxy**:

```bash
sudo systemctl restart privoxy
```

4. **Check Privoxy is running:**

```bash
sudo systemctl status privoxy
```

    - Should see "active (running)".

## **Step 5: System-Wide HTTP/HTTPS Proxy Setup**

1. **Edit `/etc/environment`:**

```bash
sudo nano /etc/environment
```

    - **After** the `# STOP KALI-DEFAULTS CONFIG` block, add:

```
http_proxy="http://127.0.0.1:8118"
https_proxy="http://127.0.0.1:8118"
ftp_proxy="http://127.0.0.1:8118"
no_proxy="localhost,127.0.0.1,::1"
HTTP_PROXY="http://127.0.0.1:8118"
HTTPS_PROXY="http://127.0.0.1:8118"
FTP_PROXY="http://127.0.0.1:8118"
NO_PROXY="localhost,127.0.0.1,::1"
```

    - Save and exit.
2. **Reboot or log out \& in**:
Ensures all new sessions load these proxy variables.

## **Step 6: Make APT Use the Proxy Automatically**

1. **Create APT proxy config:**

```bash
sudo nano /etc/apt/apt.conf.d/99proxy
```

    - Paste:

```
Acquire::http::Proxy "http://127.0.0.1:8118/";
Acquire::https::Proxy "http://127.0.0.1:8118/";
```

    - Save and exit.

## **Step 7: Verify Everything Works**

### **Browser:**

- Set SOCKS5 proxy (`127.0.0.1:1080`, SOCKS v5) in browser network/proxy settings, or
- Set HTTP/HTTPS proxy (`127.0.0.1:8118`) in your browser.


### **Terminal:**

- Run:

```bash
curl http://example.com
wget http://example.com
```

If you see output, it's routed via Privoxy and your Trojan tunnel.


### **APT/Package Manager:**

- ```bash
sudo apt update
```

If you see package lists being updated, APT is using the HTTP proxy correctly.


### **Check Proxy Variables:**

```bash
env | grep -i proxy
```

- Should list your 127.0.0.1:8118 settings.


## **After Every Reboot: Minimal Steps Required**

Just:

```bash
cd ~/trojan-go-setup
./trojan-go -config ./config.json
```

Privoxy and your system-wide proxy/apt configs persist and do NOT need to be reset.


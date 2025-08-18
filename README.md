# 🚀 Trojan-Go Setup on Ubuntu / Debian / Kali

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


***

## **Step 2: Create Trojan-Go Config**

Inside `trojan-go-setup`:

```bash
nano config.json
```

Paste:

```json
{
  "run_type": "client",
  "local_addr": "127.0.0.1",
  "local_port": 1080,
  "remote_addr": "addYourDomain",
  "remote_port": addYourPort,
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

Save with **Ctrl+O**, press **Enter**, then exit with **Ctrl+X**.

***

## **Step 3: Start Trojan-Go**

```bash
./trojan-go -config ./config.json
```

If error `"bind: address already in use"`:

```bash
sudo fuser -k 1080/tcp
```


***

## **Step 4: Install \& Configure Privoxy**

1. **Install Privoxy**:
```bash
sudo apt install privoxy -y
```

2. **Edit config**:
```bash
sudo nano /etc/privoxy/config
```

At the end of the file, add:

```
forward-socks5t   /   127.0.0.1:1080 .
```

3. **Restart Privoxy**:
```bash
sudo systemctl restart privoxy
sudo systemctl enable privoxy
```

4. **Check Privoxy**:
```bash
sudo systemctl status privoxy
```

You should see **active (running)**.

***

## **Step 5: System-Wide Proxy (Environment Variables)**

```bash
sudo nano /etc/environment
```

Add at the bottom:

```bash
http_proxy="http://127.0.0.1:8118"
https_proxy="http://127.0.0.1:8118"
ftp_proxy="http://127.0.0.1:8118"
no_proxy="localhost,127.0.0.1,::1"

HTTP_PROXY="http://127.0.0.1:8118"
HTTPS_PROXY="http://127.0.0.1:8118"
FTP_PROXY="http://127.0.0.1:8118"
NO_PROXY="localhost,127.0.0.1,::1"
```

Log out \& back in, or reboot.

***

## **Step 6: Make APT Use the Proxy**

```bash
sudo nano /etc/apt/apt.conf.d/99proxy
```

Paste:

```
Acquire::http::Proxy "http://127.0.0.1:8118/";
Acquire::https::Proxy "http://127.0.0.1:8118/";
```


***

## **Step 7: Verify**

- **Check proxy variables**:

```bash
env | grep -i proxy
```

- **Test with curl/wget**:

```bash
curl -I http://example.com
wget -O - http://example.com
```

- **Test apt**:

```bash
sudo apt update
```


***

## **After Reboot**

Privoxy + system proxy configs persist automatically.
To reconnect, just restart Trojan-Go:

```bash
cd ~/trojan-go-setup
./trojan-go -config ./config.json
```


***

👉 Happy...


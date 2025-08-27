

# 📌 Falco

### 🔹 Falco چیست؟

Falco یک **ابزار امنیتی Open Source** هست که توسط **Sysdig** ساخته شده.
این ابزار مثل یک **IDS (Intrusion Detection System)** برای لینوکس و کانتینرها عمل می‌کنه و با استفاده از **eBPF** یا **kernel module**، رفتار پردازش‌ها و سیستم‌عامل رو پایش می‌کنه.

---

### 🔹 به چه دردی می‌خوره؟

* مانیتورینگ رفتارهای مشکوک در **Host**, **Docker** و **Kubernetes**
* تشخیص اجرای دستورات غیرمجاز (مثل `nc`, `nmap`, `ping`, `ssh`)
* تشخیص دسترسی به فایل‌های حساس (`/etc/shadow`, `/etc/passwd`)
* نظارت بر network connections
* شناسایی privilege escalation یا اجرای شل در container
* تولید alert برای تیم امنیتی

---

### 🔹 نحوه پیاده‌سازی

Falco رو می‌شه در چند حالت نصب و استفاده کرد:

#### 1. نصب روی Host (Linux)

مستقیم روی سیستم نصب می‌شه:

```bash
curl -s https://falco.org/repo/falcosecurity-packages.asc | sudo apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | sudo tee /etc/apt/sources.list.d/falcosecurity.list
sudo apt-get update -y
sudo apt-get install -y falco
```

بعد از نصب سرویس اجرا می‌شه:

```bash
sudo systemctl start falco
```

#### 2. اجرا در Docker

می‌تونی Falco رو داخل یک container اجرا کنی که Host رو مانیتور کنه:

```bash
docker run -d --name falco \
  --privileged \
  -v /var/run/docker.sock:/host/var/run/docker.sock \
  -v /proc:/host/proc:ro \
  -v /boot:/host/boot:ro \
  -v /lib/modules:/host/lib/modules:ro \
  -v /usr:/host/usr:ro \
  falcosecurity/falco:latest
```

#### 3. در Kubernetes

برای K8s راحت‌ترین روش استفاده از **Helm chart** هست:

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco
```

Falco روی هر Node نصب می‌شه و کانتینرها رو مانیتور می‌کنه.

---

### 🔹 مثال عملی (قانون ping در کانتینر)

#### اضافه کردن قانون جدید

داخل فایل:

```bash
sudo nano /etc/falco/falco_rules.local.yaml
```

قانون:

```yaml
- rule: Detect Ping in Container
  desc: Detect usage of ping command inside a container
  condition: >
    container.id != host and
    proc.name = "ping"
  output: >
    🚨 Ping command executed inside container (user=%user.name container=%container.id image=%container.image.repository:%container.image.tag command=%proc.cmdline)
  priority: WARNING
  tags: [network, container]
```

#### تست

1. اجرای کانتینر:

```bash
docker run -it --rm alpine sh
```

2. نصب و اجرای ping:

```bash
apk add iputils
ping 8.8.8.8
```

3. مشاهده لاگ Falco روی host:

```bash
sudo tail -f /var/log/falco.log
```

خروجی مشابه:

```
🚨 Ping command executed inside container (user=root container=abcdef123456 image=alpine:latest command=ping 8.8.8.8)
```

---

✅ پس Falco ابزاریه برای **امنیت Runtime** که می‌تونه در Host, Docker, Kubernetes رفتار مشکوک رو تشخیص بده و هشدار بده.

---


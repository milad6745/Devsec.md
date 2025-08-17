
---

## 🔎 Docker Bench Security چیه؟

**[Docker Bench for Security](https://github.com/docker/docker-bench-security)** یک ابزار **متن‌باز** هست که توسط خود Docker ساخته شده.
این ابزار یک اسکریپت `bash` هست که روی سرور Docker اجرا میشه و سیستم، داکر دیمون، کانتینرها و تنظیمات امنیتی رو بررسی می‌کنه.

📌 در اصل، Docker Bench Security پیاده‌سازی خودکار **CIS Docker Benchmark** هست.
یعنی بررسی می‌کنه که آیا تنظیمات Docker شما مطابق best practices امنیتی هست یا نه.

---

## ⚙️ چه چیزهایی رو بررسی می‌کنه؟

وقتی اجراش کنی، تست‌هایی مثل این‌ها انجام میده:

* **پیکربندی Docker Daemon** (مثل اینکه TLS فعال هست یا نه)
* **مجوزهای فایل‌ها** (مثلاً `/etc/docker` و `docker.sock`)
* **تنظیمات کانتینرها** (آیا کانتینر با `--privileged` اجرا شده؟
  آیا `--cap-add=SYS_ADMIN` داره؟)
* **ایمیج‌ها** (ایمیج از registry امن هست یا ناشناس؟)
* **شبکه و logging**
* **کرنل security features** مثل AppArmor, Seccomp, SELinux

خروجی تست‌ها به صورت: ✅ Pass | ❌ Warn | ⚠️ Info نمایش داده میشه.

---

## 🚀 نصب و اجرای Docker Bench Security

### ۱. اجرای سریع با Docker

```bash
docker run -it --net host --pid host --userns host \
  --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /etc:/etc:ro \
  -v /usr/bin/docker-containerd:/usr/bin/docker-containerd:ro \
  -v /usr/bin/docker-runc:/usr/bin/docker-runc:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /etc/systemd/system:/etc/systemd/system:ro \
  --label docker_bench_security \
  docker/docker-bench-security
```

### ۲. یا کلون از GitHub

```bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh
```

---

## 🛡️ اسکن کانتینرها و اپلیکیشن‌ها

برای امنیت، معمولاً ترکیب چند ابزار رو با هم استفاده می‌کنن:

1. **Docker Bench Security** → بررسی تنظیمات داکر و هاست
2. **Trivy یا Grype** → اسکن ایمیج‌ها و پکیج‌ها برای CVE

   ```bash
   trivy image myapp:latest
   ```
3. **Dockle** → بررسی امنیتی و best practice در Dockerfile و ایمیج

   ```bash
   dockle myapp:latest
   ```
4. **kube-bench** → اگه تو Kubernetes هستی، معادل همینه برای K8s.

---

📌 خلاصه:

* **Docker Bench Security**: سیستم و داکر دیمونت رو بررسی می‌کنه.
* **Trivy/Dockle**: ایمیج‌ها و کانتینرها رو از نظر آسیب‌پذیری و misconfiguration اسکن می‌کنن.

---


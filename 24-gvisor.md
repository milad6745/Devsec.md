خیلی عالی پرسیدی 👌 بریم مرحله به مرحله درباره‌ی **gVisor** توضیح بدم:

---

## 🧩 gVisor چیه؟

* **gVisor** یک **container sandbox** هست که توسط **Google** ساخته شده.
* هدفش اینه که وقتی یه کانتینر (مثل Docker) اجرا می‌شه، امنیت بیشتری داشته باشه.
* به جای اینکه کانتینر مستقیماً از **Linux Kernel Host** استفاده کنه، gVisor بین کانتینر و کرنل قرار می‌گیره و نقش یه **لایه میانی (User-Space Kernel)** رو بازی می‌کنه.

یعنی چی؟
🔹 کانتینر درخواست سیستم (syscall) می‌ده → gVisor می‌گیره → تصمیم می‌گیره آیا اجازه بده یا نه → اگه اوکی بود به کرنل واقعی می‌فرسته.

---

## 🎯 کار gVisor چیه؟

1. **امنیت بیشتر برای کانتینرها**

   * باعث می‌شه اگه یک کانتینر هک شد، نتونه مستقیم به کرنل دسترسی پیدا کنه.
   * شبیه **VM امنیتی سبک** عمل می‌کنه، ولی خیلی سریع‌تر و کم‌هزینه‌تر از VM.

2. **جداسازی (Isolation)**

   * جداسازی قوی‌تر از namespaces معمولی Docker.
   * حملاتی مثل escape از کانتینر خیلی سخت‌تر می‌شن.

3. **سازگاری با Docker و Kubernetes**

   * به راحتی می‌تونی بگی فلان کانتینر با **gVisor runtime** اجرا بشه.

---

## ⚙️ طرز نصب gVisor

روی لینوکس (مثلاً Ubuntu) می‌تونی اینطوری نصبش کنی:

```bash
# دانلود و نصب gVisor
curl -fsSL https://gvisor.dev/install.sh | sh

# نصب به عنوان یک runtime برای docker
sudo cp runsc /usr/local/bin

# اضافه کردن runtime جدید به Docker
sudo nano /etc/docker/daemon.json
```

داخلش اینو اضافه می‌کنی:

```json
{
  "runtimes": {
    "runsc": {
      "path": "/usr/local/bin/runsc"
    }
  }
}
```

بعد ریستارت Docker:

```bash
sudo systemctl restart docker
```

---

## 🚀 نحوه اجرا

حالا می‌تونی یک کانتینر رو با gVisor اجرا کنی:

```bash
docker run --rm -it --runtime=runsc alpine sh
```

🔹 اینجا `--runtime=runsc` به Docker می‌گه از gVisor استفاده کنه.

---

## 🔍 مثال کاربردی

فرض کن یک سرویس **Nginx** رو می‌خوای اجرا کنی:

```bash
docker run --rm -d --runtime=runsc -p 8080:80 nginx
```

* اگه کسی تونست از Nginx exploit بگیره و بخواد به سیستم دسترسی پیدا کنه،
* با gVisor گیر می‌کنه توی همون sandbox و نمی‌تونه به کرنل یا بقیه کانتینرها دست بزنه.

---

## 📌 جمع‌بندی

* **چی هست؟** → یک sandbox (کرنل مجازی در user-space) برای کانتینرها.
* **کارش چیه؟** → افزایش امنیت و جداسازی کانتینرها با کنترل syscalls.
* **نصب؟** → با نصب `runsc` و اضافه کردن به Docker/Kubernetes.
* **مثال؟** → اجرای Nginx یا Alpine در حالت امن با `--runtime=runsc`.

---

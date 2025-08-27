خیلی خوب شد که پرسیدی 🙌 چون **gVisor** و **Kata Containers** هردو برای **امنیت و ایزوله‌سازی کانتینرها** ساخته شدن، ولی روش کارشون فرق اساسی داره.
بریم مرحله به مرحله:

---

## 🧩 Kata Containers چیه؟

* Kata Containers یه **پروژه اپن‌سورس** هست که کانتینرها رو داخل **ماشین مجازی بسیار سبک (Lightweight VM)** اجرا می‌کنه.
* یعنی به جای اینکه فقط با Namespace و cgroups کانتینر رو ایزوله کنه، برای هر کانتینر یک **MicroVM** ایجاد می‌کنه.
* از تکنولوژی‌هایی مثل **KVM/QEMU** استفاده می‌کنه.

---

## 🔎 تفاوت Kata Containers و gVisor

| ویژگی                       | gVisor                                                          | Kata Containers                                                     |
| --------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------- |
| **روش کار**                 | یک **User-space Kernel** که syscalls رو فیلتر می‌کنه            | اجرای کانتینر داخل یک **VM سبک**                                    |
| **ایزوله‌سازی (Isolation)** | قوی‌تر از Docker معمولی ولی هنوز روی همون کرنل اصلی             | خیلی قوی‌تر چون هر کانتینر کرنل جدا (VM) داره                       |
| **کارایی (Performance)**    | سریع‌تر چون VM نداره                                            | کمی سنگین‌تر چون VM هست                                             |
| **سازگاری (Compatibility)** | بعضی syscalls ساپورت نمی‌شن (ممکنه بعضی اپلیکیشن‌ها کار نکنن)   | چون VM واقعی داره، تقریباً همه چی مثل یک ماشین عادی کار می‌کنه      |
| **مناسب برای**              | اپ‌هایی که به امنیت بالا نیاز دارن ولی lightweight بودن هم مهمه | اپ‌هایی که امنیت/جداسازی خیلی حیاتی دارن (مثلاً multi-tenant cloud) |

---

## ⚙️ نصب Kata Containers

روی لینوکس (مثلاً Ubuntu 20.04) می‌تونی اینطوری نصبش کنی:

```bash
# اضافه کردن مخزن Kata
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/katacontainers:/releases:/xUbuntu_20.04/ /' > /etc/apt/sources.list.d/kata-containers.list"

# آپدیت و نصب
curl -sL http://download.opensuse.org/repositories/home:/katacontainers:/releases:/xUbuntu_20.04/Release.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y kata-runtime kata-proxy kata-shim
```

---

## 🚀 اجرای کانتینر با Kata

بعد از نصب، مثل gVisor باید runtime رو به Docker معرفی کنی.

فایل `/etc/docker/daemon.json` رو باز کن و اضافه کن:

```json
{
  "runtimes": {
    "kata-runtime": {
      "path": "/usr/bin/kata-runtime"
    }
  }
}
```

سپس ریستارت Docker:

```bash
sudo systemctl restart docker
```

حالا می‌تونی کانتینر رو با Kata اجرا کنی:

```bash
docker run -it --rm --runtime=kata-runtime alpine sh
```

---

## 🔍 مثال واقعی

مثلاً اجرای Nginx با Kata:

```bash
docker run -d --runtime=kata-runtime -p 8080:80 nginx
```

* اینجا Nginx درون یک **MicroVM مجزا** بالا میاد.
* اگه کسی تونست از Nginx exploit بگیره، نهایتش فقط VM اون کانتینر رو می‌گیره و به هاست اصلی دسترسی نداره.

---

## 📌 جمع‌بندی

* **gVisor**: سریع‌تر، سبک‌تر، ولی همه اپ‌ها رو ساپورت نمی‌کنه (چون syscalls محدودن).
* **Kata Containers**: ایزوله‌سازی خیلی قوی (تقریباً مثل VM)، سازگار با همه اپ‌ها، ولی مصرف منابع بیشتر از gVisor.

---

🔥 به زبان ساده:

* gVisor = **Sandbox** (سبک ولی محدود).
* Kata = **VM سبک برای هر کانتینر** (خیلی امن ولی سنگین‌تر).

---



## **Syft چیست؟**

* **Syft** یک ابزار **open-source** است برای **تحلیل و شناسایی پکیج‌ها و وابستگی‌ها داخل کانتینرها یا سیستم‌عامل‌ها**.
* خروجی آن شامل: نام پکیج‌ها، نسخه‌ها، و نوع پکیج‌ها (مثل Debian, Python, npm و غیره) است.
* معمولاً همراه با **Anchore** و ابزارهای امنیتی برای بررسی آسیب‌پذیری‌ها استفاده می‌شود.

**کاربرد اصلی:**

* اسکن ایمیج‌های Docker برای پکیج‌های نصب شده
* تولید گزارش JSON یا جدول برای استفاده در CI/CD یا امنیت

---

## **روش نصب و اجرای Syft با Docker**

**چرا با Docker؟**

* چون Syft یک برنامه Go هست و ممکنه روی سیستم تو درست اجرا نشه (مثل خطای `Exec format error`)
* راحت‌ترین روش، استفاده از نسخه Docker آن است.

### **دستور پایه برای اسکن یک Docker Image**

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock anchore/syft:latest my-python-app:latest
```

**توضیح:**

* `docker run` → اجرای یک کانتینر جدید
* `--rm` → پس از اتمام اجرا کانتینر حذف می‌شود
* `-v /var/run/docker.sock:/var/run/docker.sock` → اجازه دسترسی به Docker daemon برای مشاهده ایمیج‌ها
* `anchore/syft:latest` → خود کانتینر Syft
* `my-python-app:latest` → نام ایمیجی که می‌خوای اسکن کنی

---

### **خروجی به فرمت JSON**

برای ذخیره یا پردازش خودکار:

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock anchore/syft:latest my-python-app:latest -o json > report.json
```

* فایل `report.json` شامل تمام پکیج‌ها و نسخه‌ها است.

---

## **روش کار Syft (در پشت صحنه)**

1. Syft ایمیج Docker را باز می‌کند و فایل‌سیستم آن را می‌خواند.
2. لیست پکیج‌ها و وابستگی‌ها را با استفاده از **Databaseهای داخلی برای هر نوع پکیج (Debian, Alpine, Python, npm, …)** تطبیق می‌دهد.
3. خروجی را به صورت جدول یا JSON تولید می‌کند.

---

## **مثال واقعی**

فرض کن ایمیج `my-python-app:latest` شامل این پکیج‌هاست:

* Python 3.11
* Flask 2.3.2
* Redis 5.0

بعد از اجرای Syft:

```
NAME     VERSION   TYPE
python   3.11      python
flask    2.3.2     python
redis    5.0       python
```

و اگر از گزینه `-o json` استفاده کنی، همین اطلاعات داخل JSON ذخیره می‌شود که می‌توان برای اسکن آسیب‌پذیری با Trivy یا Anchore استفاده کرد.

---





---

## ۱. Checkov چیست؟

* یک ابزار **Open Source** برای بررسی **misconfigurations و ضعف‌های امنیتی** در فایل‌های زیر است:

  * Dockerfile
  * Terraform
  * Kubernetes manifests
  * CloudFormation و غیره
* هدف: پیدا کردن مشکلات امنیتی و رعایت best practices قبل از deploy کردن.

---

## ۲. نصب Checkov

### روش پیشنهادی: Python/pip

اگر Python و pip روی سیستم شما نصب است:

```bash
# نصب آخرین نسخه
pip install checkov
```

### روش دیگر: Docker

اگر نمی‌خواهید pip نصب کنید، می‌توانید با Docker استفاده کنید:

```bash
docker run --rm -v $(pwd):/checkov bridgecrew/checkov -f /checkov/Dockerfile
```

* `$(pwd)` مسیر فولدری است که Dockerfile داخل آن است.
* `-f /checkov/Dockerfile` مسیر فایل داخل کانتینر Docker است.

---

## ۳. استفاده از Checkov با Dockerfile

فرض کنید فایل شما `Dockerfile` نام دارد:

```bash
# اسکن یک Dockerfile
checkov -f Dockerfile
```

خروجی شامل موارد زیر خواهد بود:

* هشدارهای امنیتی (مثل استفاده از root، پورت‌های باز، نصب package آسیب‌پذیر)
* توصیه برای تغییر دستورها یا تنظیمات
* severity (شدت) مشکلات

---
تفاوت با ابزارهایی مثل Checkov یا Trivy

Checkov / Trivy / Hadolint → بررسی امنیت کد و کانتینرها (قبل از deploy).

OpenVAS → اسکن امنیتی زنده روی سرورها و شبکه‌ها (بعد از deploy).


## ۴. گزینه‌های مهم Checkov

* اسکن کل فولدر:

```bash
checkov -d .
```

* فقط هشدارهای بحرانی:

```bash
checkov -f Dockerfile --check CKV_DOCKER_1
```

* گزارش خروجی به فرمت JSON یا HTML:

```bash
checkov -f Dockerfile --output json > report.json
checkov -f Dockerfile --output html > report.html
```

---

## ۵. نکات مهم در استفاده

1. همیشه **Dockerfile خود را قبل از deploy اسکن کنید**.
2. ترکیب با **Trivy** و **Hadolint** نتیجه بهتری می‌دهد:

   * Hadolint: سینتکس و best practices
   * Trivy: آسیب‌پذیری‌های packages
   * Checkov: misconfigurations و security rules
3. بهتر است **فایل‌های Dockerfile و Kubernetes manifests را در CI/CD pipeline** چک کنید تا خودکار و مداوم بررسی شوند.

---


---

## ۱. یک Dockerfile ناامن (برای تست)

```dockerfile
# Dockerfile
FROM ubuntu:18.04

# اجرای دستورات با root (ضعف امنیتی)
USER root

# نصب curl بدون pin کردن نسخه و بدون clean کردن cache
RUN apt-get update && apt-get install -y curl

# اجرای اپلیکیشن با دستور مستقیم
CMD ["curl", "http://example.com"]
```

---

## ۲. اجرای Checkov روی این Dockerfile

```bash
checkov -f Dockerfile
```

---

## ۳. خروجی نمونه (شبیه به چیزی که می‌بینید)

```
Checkov reported the following issues:

       _               _
   ___| |__   ___  ___| | _____ _ __
  / __| '_ \ / _ \/ __| |/ / _ \ '__|
 | (__| | | |  __/ (__|   <  __/ |
  \___|_| |_|\___|\___|_|\_\___|_|

dockerfile scan results:

Passed checks: 0, Failed checks: 3, Skipped checks: 0

FAILED for resource: Dockerfile
-------------------------------------------------------------------
Check: CKV_DOCKER_2: Ensure that a USER is specified in the Dockerfile
        FAILED for file: Dockerfile:6
        - Currently running as root

Check: CKV_DOCKER_3: Ensure apt-get install uses --no-install-recommends
        FAILED for file: Dockerfile:9
        - Package installed without --no-install-recommends

Check: CKV_DOCKER_5: Ensure cleanup of apt-get cache
        FAILED for file: Dockerfile:9
        - Missing "rm -rf /var/lib/apt/lists/*" after apt-get
```

---

## ۴. توضیح خروجی

* ❌ **CKV\_DOCKER\_2** → شما از `USER root` استفاده کردید → باید یک user غیر root بسازید.
* ❌ **CKV\_DOCKER\_3** → بهتر است هنگام نصب از `--no-install-recommends` استفاده کنید.
* ❌ **CKV\_DOCKER\_5** → بعد از نصب باید cache پاک شود تا image کوچک‌تر و امن‌تر شود.

---

## ۵. نسخه اصلاح‌شده Dockerfile

```dockerfile
FROM ubuntu:20.04

# ساخت یک یوزر امن
RUN useradd -m appuser

# نصب پکیج‌ها با اصول امنیتی
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# اجرای اپلیکیشن با یوزر غیر root
USER appuser

CMD ["curl", "http://example.com"]
```


---



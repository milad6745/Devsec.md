این یکی از محبوب‌ترین ابزارها برای بررسی **Dockerfile** از نظر **سینتکس** و **Best Practices** هست.

---

## 🐳 Hadolint چیه؟

* یک **Dockerfile Linter** (بررسی‌کننده‌ی کد) هست.
* خطاها و هشدارهای مربوط به:

  * **سینتکس دستورات** (مثلاً املای اشتباه)
  * **بهترین شیوه‌ها** (Best Practices)
  * نکات امنیتی (مثلاً استفاده نکردن از `latest` در image base یا پاک نکردن cache)
    را به شما نشون می‌ده.

---

## 📥 نصب Hadolint

### ۱. روی Linux/macOS (binary)

```bash
# دانلود آخرین نسخه
wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/latest/download/hadolint-Linux-x86_64

# دسترسی اجرا بده
chmod +x /usr/local/bin/hadolint
```

### ۲. با Homebrew (macOS/Linux)

```bash
brew install hadolint
```

### ۳. با Docker (بدون نصب مستقیم)

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

---

## 🚀 استفاده از Hadolint

فرض کن Dockerfile شما اینه:

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl

CMD ["curl", "http://example.com"]
```

حالا اجرا کن:

```bash
hadolint Dockerfile
```

### خروجی نمونه:

```
Dockerfile:1 DL3006 Use specific image version pinning instead of 'latest'
Dockerfile:3 DL3008 Pin versions in apt-get install. Instead of `apt-get install <package>` use `apt-get install <package>=<version>`
Dockerfile:3 DL4006 Set the SHELL option -o pipefail before RUN with a pipe in it
```

---

## 📊 نکات کاربردی

* اگر خواستی بعضی خطاها رو **نادیده بگیری**:

  ```dockerfile
  # hadolint ignore=DL3006,DL4000
  FROM ubuntu:latest
  ```
* خروجی JSON برای CI/CD:

  ```bash
  hadolint Dockerfile -f json
  ```
* می‌تونی اونو توی **GitHub Actions یا GitLab CI** بذاری تا Dockerfileها همیشه قبل از Build بررسی بشن.

---

🔑 خلاصه:
**Hadolint** بهترین ابزار برای گرفتن ایرادهای **سینتکسی** و **Best Practices** در Dockerfile است. نصبش ساده‌ست و هم standalone و هم با Docker قابل استفاده است.

---

 یک **Dockerfile عمداً ناامن و پر از خطاهای رایج** بنویسیم که Hadolint چند تا اخطار و خطا بهت بده.

---

## 🔹 Dockerfile پر از ایراد برای تست Hadolint

```dockerfile
FROM python:latest   # ❌ استفاده از latest

WORKDIR /app

# نصب پکیج‌ها بدون --no-cache-dir
RUN pip install flask requests

# استفاده از apt-get بدون clean کردن cache
RUN apt-get update && apt-get install -y curl

# اجرا با یوزر root
USER root

COPY . .

CMD ["python", "app.py"]
```

---

## 🔹 وقتی با Hadolint اسکن کنی

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

### 🔻 خروجی احتمالی (نمونه)

```
Dockerfile:1 DL3006 warning: Always pin versions in the FROM image, not just rely on `latest`
Dockerfile:5 DL3042 warning: Avoid use of cache directory with pip. Use `pip install --no-cache-dir <package>`
Dockerfile:8 DL3008 warning: Pin versions in apt-get install. Instead of `apt-get install <package>` use `apt-get install <package>=<version>`
Dockerfile:8 DL3009 info: Delete the apt-get lists after installing something
Dockerfile:11 DL3002 warning: Last USER should not be root
```

---

## 🔹 معنی خطاها

* **DL3006** → از `latest` استفاده نکن، همیشه نسخه مشخص کن.
* **DL3042** → توی pip باید `--no-cache-dir` بزنی.
* **DL3008** → موقع نصب با `apt-get install` نسخه رو pin کن.
* **DL3009** → بعد از `apt-get install` باید `rm -rf /var/lib/apt/lists/*` بزنی.
* **DL3002** → نباید یوزر root بمونه.

---

👉 نتیجه: این Dockerfile برای تست عالیه چون حداقل ۴-۵ تا پیام از Hadolint می‌گیری.

---



---

## 🔹 نمونه جزوه کوتاه درباره Dockle

### ۱️⃣ عنوان

**Dockle – Docker Image Security Linter**

---

### ۲️⃣ Dockle چیست؟

* Dockle یک **ابزار متن‌باز** برای بررسی امنیت و **Best Practice** در Docker image‌ها است.
* شبیه یک **linter** برای Docker: فایل‌های Dockerfile و image‌ها را اسکن می‌کند و هشدار می‌دهد.

### ۳️⃣ کارهایی که Dockle انجام می‌دهد

* بررسی اینکه کانتینر با **root** اجرا نمی‌شود.
* بررسی پکیج‌های قدیمی یا آسیب‌پذیر.
* بررسی فایل‌ها و permissionهای ناامن.
* بررسی Health check و سایر استانداردهای امنیتی.

---

### ۴️⃣ نصب Dockle روی لینوکس

#### روش ۱: دانلود مستقیم باینری

```bash
# دانلود آخرین نسخه (مثال برای amd64)
curl -L https://github.com/goodwithtech/dockle/releases/download/v0.4.14/dockle_0.4.14_Linux-64bit.tar.gz -o dockle.tar.gz

# اکسترکت
tar -zxvf dockle.tar.gz

# انتقال به PATH
sudo mv dockle /usr/local/bin/

# بررسی نسخه
dockle --version
```

#### روش ۲: با Go

```bash
go install github.com/goodwithtech/dockle/cmd/dockle@latest
```

> بعد مطمئن شو `$GOPATH/bin` در PATH است.

#### روش ۳: با Docker (بدون نصب مستقیم)

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock goodwithtech/dockle:v0.4.14 <IMAGE_NAME>
```

---

### ۵️⃣ نمونه اجرا

```bash
dockle ubuntu:20.04
```

* خروجی شامل **Warnings** و **Fatal Errors** است که نشان می‌دهد چه مواردی امنیتی رعایت نشده.

---

### ۶️⃣ نکات مهم

* Dockle مکمل ابزارهایی مثل **Trivy** است (که آسیب‌پذیری پکیج‌ها را اسکن می‌کند).
* استفاده در **Pipeline CI/CD** توصیه می‌شود تا قبل از انتشار image، امنیت بررسی شود.

---

💡 می‌توانی همین متن را در فایل `dockle_notes.md` یا Word ذخیره کنی و با مثال‌های بیشتر (مثلاً اسکن imageهای خودت) تکمیل کنی.

---


* مثال 
```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock goodwithtech/dockle:v0.4.14 my-python-app:latest
Unable to find image 'goodwithtech/dockle:v0.4.14' locally
v0.4.14: Pulling from goodwithtech/dockle
7b26c2f269ea: Pull complete
b16330547c19: Pull complete
Digest: sha256:68f7473909b49013f97984e9917fb7edd0c440bf15e38f41449860f8a2680d51
Status: Downloaded newer image for goodwithtech/dockle:v0.4.14
FATAL   - DKL-DI-0005: Clear apt-get caches
        * Use 'rm -rf /var/lib/apt/lists' after 'apt-get install|update' : RUN /bin/sh -c set -eux;     apt-get update;         apt-get install -y --no-install-recommends              ca-certificates             netbase                 tzdata  ;       apt-get dist-clean # buildkit
WARN    - CIS-DI-0001: Create a user for the container
        * Last user should not be root
WARN    - DKL-DI-0006: Avoid latest tag
        * Avoid 'latest' tag
INFO    - CIS-DI-0005: Enable Content trust for Docker
        * export DOCKER_CONTENT_TRUST=1 before docker pull/build
INFO    - CIS-DI-0006: Add HEALTHCHECK instruction to the container image
        * not found HEALTHCHECK statement
INFO    - CIS-DI-0008: Confirm safety of setuid/setgid files
        * setuid file: urwxr-xr-x usr/bin/chsh
        * setgid file: grwxr-xr-x usr/sbin/unix_chkpwd
        * setgid file: grwxr-xr-x usr/bin/expiry
        * setuid file: urwxr-xr-x usr/bin/gpasswd
        * setuid file: urwxr-xr-x usr/bin/chfn
        * setuid file: urwxr-xr-x usr/bin/umount
        * setuid file: urwxr-xr-x usr/bin/newgrp
        * setgid file: grwxr-xr-x usr/bin/chage
        * setuid file: urwxr-xr-x usr/bin/su
        * setuid file: urwxr-xr-x usr/bin/mount
        * setuid file: urwxr-xr-x usr/bin/passwd
INFO    - DKL-LI-0003: Only put necessary files
        * Suspicious directory : app/.git
        * unnecessary file : app/project/Dockerfile
        * unnecessary file : app/Dockerfile

```


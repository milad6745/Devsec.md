 **auditd** ✅
این سرویس توی لینوکس برای **مانیتورینگ و ثبت رخدادهای امنیتی** استفاده میشه (مثل لاگین‌ها، تغییر فایل‌ها، اجرای دستورها و ...).
میتونی باهاش روی **Docker** هم نظارت داشته باشی (مثل: کی container ساخته شد، کی پاک شد، کی image pull شد، ...).

---

## 🔹 مراحل فعال‌سازی auditd برای Docker

### 1. نصب auditd

روی Ubuntu 23.04:

```bash
sudo apt update
sudo apt install auditd audispd-plugins -y
```

فعال‌سازی سرویس:

```bash
sudo systemctl enable auditd
sudo systemctl start auditd
```

---

### 2. اضافه کردن قوانین (audit rules) برای Docker

قوانین audit توی فایل `/etc/audit/rules.d/audit.rules` یا `/etc/audit/rules.d/docker.rules` اضافه میشن.

مثال قوانین پایه برای Docker:

```bash
# نظارت روی اجرای باینری docker
-w /usr/bin/docker -p x -k docker

# نظارت روی فایل‌های پیکربندی docker
-w /etc/docker/ -p wa -k docker

# نظارت روی دایرکتوری lib مربوط به کانتینرها
-w /var/lib/docker/ -p wa -k docker

# نظارت روی سوکت docker (برای اینکه کی وصل میشه)
-w /var/run/docker.sock -p rwxa -k docker
```

بعد از ویرایش فایل:

```bash
sudo augenrules --load
sudo systemctl restart auditd
```

---

### 3. مشاهده لاگ‌های auditd

لاگ‌ها توی فایل زیر ذخیره میشن:

```bash
/var/log/audit/audit.log
```

برای جستجو با کلید `docker`:

```bash
sudo ausearch -k docker
```

---

### 4. تست

یک container ساده بساز:

```bash
docker run --rm alpine echo "hello"
```

بعد توی لاگ‌ها بررسی کن:

```bash
sudo ausearch -k docker | less
```

---

## 🔹 نتیجه

با اینکار:

* هر تغییری در دایرکتوری‌های Docker
* اجرای دستورات docker
* تغییر در تنظیمات
* یا دسترسی به `docker.sock`

همه‌اش توی auditd ثبت میشه ✅

---




`auditd`

یک **Linux Audit Framework** هست و میشه باهاش تقریباً هر فعالیت مهم روی سیستم رو لاگ کرد. مخصوصاً برای مانیتورینگ امنیتی و Forensics عالیه.

---

## 🔹 کارهایی که با auditd میشه کرد

1. **نظارت روی فایل‌ها و دایرکتوری‌ها**

   * می‌تونی تغییرات روی فایل‌های حساس (مثلاً `/etc/passwd` یا `/etc/shadow`) رو لاگ کنی.
   * مثال:

     ```bash
     -w /etc/passwd -p wa -k identity
     -w /etc/shadow -p wa -k auth
     ```

     ➝ هر تغییر روی کاربرها یا پسوردها ثبت میشه.

---

2. **نظارت روی اجرای برنامه‌ها**

   * می‌تونی ببینی چه کسی چه برنامه‌ای رو اجرا کرده.
   * مثال:

     ```bash
     -a always,exit -F arch=b64 -S execve -k exec_log
     ```

     ➝ هر بار یک برنامه اجرا بشه، لاگ می‌گیری.

---

3. **نظارت روی دسترسی به فایل‌ها**

   * می‌تونی بفهمی کی فایل خاصی رو خونده یا نوشته.
   * مثال:

     ```bash
     -w /var/log/secure -p r -k log_access
     ```

---

4. **نظارت روی Network / Sockets**

   * میشه بررسی کرد چه کسی به `docker.sock` یا سوکت‌های دیگه وصل شده.
   * مثال:

     ```bash
     -w /var/run/docker.sock -p rwxa -k docker
     ```

---

5. **نظارت روی تغییرات کرنل و ماژول‌ها**

   * خیلی مهم برای امنیت کرنل.
   * مثال:

     ```bash
     -w /sbin/insmod -p x -k modules
     -w /sbin/rmmod -p x -k modules
     ```

---

6. **مانیتورینگ یوزرها و sudo**

   * بفهمی چه کسی `sudo` زده.
   * مثال:

     ```bash
     -w /etc/sudoers -p wa -k sudo
     ```

---

## 🔹 چه لاگ‌هایی تولید میشه؟

همه چیز میره توی:

```
/var/log/audit/audit.log
```

برای تحلیل می‌تونی استفاده کنی از:

* `ausearch` → جستجوی لاگ‌ها

  ```bash
  ausearch -k docker
  ausearch -m execve
  ```
* `aureport` → گزارش‌های خلاصه

  ```bash
  aureport -x   # اجرای برنامه‌ها
  aureport -f   # فایل‌ها
  aureport -au  # احراز هویت
  ```

---

## 🔹 نمونه‌ی واقعی

فرض کن بخوای بدونی کی وارد `/etc/passwd` شده:

1. قانون اضافه می‌کنی:

   ```bash
   -w /etc/passwd -p wa -k passwd_changes
   ```
2. کسی فایل رو تغییر بده.
3. بررسی می‌کنی:

   ```bash
   ausearch -k passwd_changes
   ```

---

یعنی `auditd` می‌تونه مثل یک **سیستم SIEM محلی** باشه که همه فعالیت‌های مشکوک رو لاگ می‌کنه.

---


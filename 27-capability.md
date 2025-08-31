 توضیح که **capability در لینوکس و Docker** و همچنین دستور **setcap** چی هست و کجا به درد می‌خوره:

---

## 🔹 1. Capability در لینوکس چیه؟

* به طور پیش‌فرض توی لینوکس یا یه پروسه **کاربر عادی (unprivileged)** هست یا **root**.
* کاربر root همه‌چیزو می‌تونه انجام بده.
* برای امنیت بهتر، لینوکس قابلیت‌هایی به اسم **Linux Capabilities** معرفی کرده که سطح دسترسی root رو به بخش‌های کوچیک تقسیم می‌کنه.

مثال از capabilityها:

* `CAP_NET_BIND_SERVICE`: اجازه می‌ده پروسه به پورت‌های کمتر از 1024 bind بشه (مثل پورت 80 یا 443).
* `CAP_SYS_TIME`: اجازه تغییر ساعت سیستم.
* `CAP_CHOWN`: اجازه تغییر مالکیت فایل‌ها.
* `CAP_NET_RAW`: اجازه استفاده از raw socket (مثلاً برای ping).

یعنی به جای اینکه کل دسترسی root رو بدی، فقط همون قابلیت لازم رو می‌دی.

---

## 🔹 2. دستور `setcap`

* برای اضافه یا حذف کردن capability روی **فایل اجرایی** استفاده می‌شه.
* سینتکس کلی:

  ```bash
  sudo setcap CAPABILITY=ep /path/to/binary
  ```

  * `e`: effective
  * `p`: permitted

مثال:
می‌خوای یه برنامهٔ معمولی بتونه روی پورت 80 (که فقط root می‌تونه) listen کنه:

```bash
sudo setcap CAP_NET_BIND_SERVICE=ep /usr/bin/python3
```

حالا هر اسکریپتی که با `/usr/bin/python3` اجرا بشه می‌تونه بدون root روی پورت 80 bind بشه.

---

## 🔹 3. Capability در Docker

* وقتی یه کانتینر Docker می‌سازی، به طور پیش‌فرض یه سری capabilityهای محدود داره.
* می‌تونی با فلگ‌های زیر کنترل کنی:

### اضافه کردن capability:

```bash
docker run --cap-add=NET_ADMIN ubuntu ip addr add 192.168.1.10/24 dev eth0
```

* این کانتینر قابلیت مدیریت شبکه رو می‌گیره.

### حذف کردن capability:

```bash
docker run --cap-drop=NET_RAW ubuntu ping 8.8.8.8
```

* این کانتینر دیگه نمی‌تونه از raw socket استفاده کنه (پس ping کار نمی‌کنه).

---

## 🔹 4. مثال عملی

فرض کن یه وب‌سرور Nginx داخل Docker داری و می‌خوای روی **پورت 80** بالا بیاد، ولی نمی‌خوای بهش کل دسترسی root بدی:

```bash
docker run -d --name web --cap-add=NET_BIND_SERVICE -p 80:80 nginx
```

اینجا فقط capability مربوط به bind شدن به پورت‌های پایین‌تر از 1024 داده می‌شه.

---

## 🔹 5. چرا مهمه؟

* امنیت: به جای اینکه `--privileged` بدی (که خیلی خطرناکه)، فقط capability لازم رو می‌دی.
* مدیریت دقیق دسترسی‌ها.
* جلوگیری از حمله‌هایی که نیاز به دسترسی root دارن.

---

👉 خلاصه:

* **Capabilities** = تکه‌تکه کردن دسترسی root به بخش‌های کوچیک.
* **setcap** = دادن یا گرفتن اون قابلیت‌ها روی باینری.
* **Docker capabilities** = کنترل دقیق دسترسی کانتینرها بدون نیاز به full root.

---

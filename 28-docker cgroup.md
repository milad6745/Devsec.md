خیلی خوب، بریم سراغ **cgroup** توی Docker.
در واقع Docker از cgroup لینوکس برای **محدود کردن منابع** کانتینر استفاده می‌کنه (CPU، RAM، I/O و ...). شما مستقیم cgroup نمی‌سازید؛ بلکه موقع `docker run` یا داخل **Docker Compose** یا **Kubernetes**، تنظیمات منابع رو ست می‌کنید. Docker پشت‌صحنه cgroup رو مدیریت می‌کنه.

---

## 📌 مثال‌ها

### ۱. محدود کردن حافظه (Memory)

```bash
docker run --name test-mem \
  --memory=512m \
  --memory-swap=512m \
  ubuntu:22.04 sleep 1h
```

* `--memory=512m` ⇒ حداکثر RAM = 512MB
* `--memory-swap=512m` ⇒ اجازه استفاده از swap نداره

## 👉 این یعنی کانتینر بیشتر از ۵۱۲ مگابایت حافظه نمی‌تونه مصرف کنه.

### ۲. محدود کردن CPU

```bash
docker run --name test-cpu \
  --cpus="1.5" \
  ubuntu:22.04 sleep 1h
```

* `--cpus="1.5"` ⇒ معادل ۱.۵ هسته CPU

---

### ۳. محدود کردن PID (فرآیندها)

```bash
docker run --name test-pids \
  --pids-limit=100 \
  ubuntu:22.04 sleep 1h
```

* `--pids-limit=100` ⇒ کانتینر نمی‌تونه بیشتر از ۱۰۰ پروسه ایجاد کنه.

---

### ۴. ترکیب چند محدودیت

```bash
docker run --name test-limits \
  --memory=256m \
  --cpus=0.5 \
  --pids-limit=50 \
  nginx:alpine
```

👉 این یعنی:

* حداکثر ۲۵۶ مگ RAM
* نیم هسته CPU
* ۵۰ پروسه

---

## 📌 در Docker Compose

می‌تونید محدودیت‌ها رو در `docker-compose.yml` اعمال کنید (نسخه 3.7 به بعد):

```yaml
version: "3.8"
services:
  web:
    image: nginx:alpine
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
        reservations:
          cpus: "0.25"
          memory: 128M
```

---

## 📌 بررسی cgroup کانتینر

بعد از اجرای کانتینر:

```bash
docker inspect <container_id> | grep -i memory
```

یا مستقیماً برید داخل مسیر cgroup لینوکس:

```bash
cat /sys/fs/cgroup/memory/docker/<container_id>/memory.limit_in_bytes
```

---

👉 به این ترتیب بدون نیاز به دست‌کاری مستقیم cgroup، شما با **پارامترهای Docker** می‌تونید منابع کانتینر رو کاملاً کنترل کنید.

---

می‌خوای برات یه اسکریپت نمونه بنویسم که کانتینر رو بالا بیاره و بعد limitهای cgroupش رو نشون بده؟

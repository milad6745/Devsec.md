

---

# جزوه کوتاه: Seccomp و RuntimeDefault در Kubernetes

## ۱️⃣ مقدمه: Seccomp چیست؟

* **Seccomp** (Secure Computing Mode) قابلیتی در **کرنل لینوکس** است.
* هدف: محدود کردن **سیستم‌کال‌ها (syscall)** که یک پردازه می‌تواند اجرا کند.
* مزایا: کاهش سطح حمله، جلوگیری از اجرای عملیات خطرناک حتی در صورت نفوذ به کانتینر.

---

## ۲️⃣ پروفایل‌ها در Seccomp

* Seccomp با **پروفایل‌ها** کار می‌کند: لیست syscallهای **مجاز یا غیرمجاز**.
* دو حالت اصلی:

  1. **Unconfined**: هیچ محدودیتی ندارد.
  2. **RuntimeDefault**: پروفایل پیش‌فرض امن که توسط runtime ارائه شده.

---

## ۳️⃣ RuntimeDefault چیست؟

* پروفایل پیش‌فرض امن کانتینرها (Docker / containerd).
* ویژگی‌ها:

  * جلوی syscallهای خطرناک یا کم‌کاربرد را می‌گیرد:

    * `ptrace`, `mount`, `keyctl`, `kexec`, `reboot` و …
  * اجازه می‌دهد syscallهای معمول برای اجرای برنامه‌ها بدون مشکل باشند.
* مزیت: نیاز به نوشتن لیست دستی syscallها نیست.

---

## ۴️⃣ ساخت Deployment در Kubernetes با RuntimeDefault

### نمونه YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
        securityContext:
          seccompProfile:
            type: RuntimeDefault
```

* `securityContext.seccompProfile.type` = `RuntimeDefault`
* می‌توان این مقدار را در سطح **Pod** یا **Container** ست کرد.

---

## ۵️⃣ بررسی پروفایل Seccomp پاد

* دستور برای دیدن YAML پاد:

```bash
kubectl get pod <pod-name> -o yaml
```

* دستور برای مشاهده مستقیم Seccomp:

```bash
kubectl get pod <pod-name> -o=jsonpath='{.spec.containers[*].securityContext.seccompProfile}'
```

* سطح نود:

```bash
cat /proc/<PID>/status | grep Seccomp
```

* خروجی `2` → Seccomp فعال با پروفایل.

---

## ۶️⃣ تست عملی RuntimeDefault

### نمونه Pod تست

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-test
spec:
  containers:
  - name: ubuntu
    image: ubuntu:22.04
    command: ["sleep", "3600"]
    securityContext:
      seccompProfile:
        type: RuntimeDefault
    stdin: true
    tty: true
```

### تست syscallها با strace

```bash
# ورود به پاد
kubectl exec -it seccomp-test -- bash

# نصب ابزار لازم
apt-get update && apt-get install -y strace keyutils

# syscall عادی (اجرا می‌شود)
strace -e getpid echo hello

# syscall بلاک شده (EPERM)
strace -e mount mount -t tmpfs none /mnt
strace -e keyctl keyctl list 0
strace -e ptrace sleep 1
```

### نتایج:

* syscallهای خطرناک → `EPERM`
* syscallهای عادی → بدون مشکل اجرا می‌شوند.

---

## ۷️⃣ جمع‌بندی

* **Seccomp** امنیت کانتینرها را بالا می‌برد.
* **RuntimeDefault** سریع‌ترین روش برای اعمال محدودیت‌های امن است.
* تست عملی نشان می‌دهد syscallهای خطرناک واقعاً بلاک می‌شوند، بدون اینکه برنامه‌های معمولی آسیب ببینند.

---



🔹 **Trivy** 

یک ابزار امنیتی متن‌باز هست که برای اسکن کردن کانتینرها، تصاویر داکر، ریپازیتوری‌های Git، سیستم‌عامل‌ها و حتی فایل‌های کانفیگ (مثل Kubernetes و Terraform) استفاده میشه.
باهاش می‌تونی آسیب‌پذیری‌ها (Vulnerabilities)، مشکلات misconfiguration و حتی اطلاعات حساس (Secrets) رو پیدا کنی.

---

## 📥 نصب Trivy روی Ubuntu 23.04

راه‌های مختلفی برای نصب وجود داره. راحت‌ترین روش:

### ۱. نصب از طریق مخزن رسمی

```bash
sudo apt-get update
sudo apt-get install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt-get update
sudo apt-get install trivy -y
```

### ۲. نصب با اسنپ (روش سریع‌تر)

```bash
sudo snap install trivy
```

---

## 🚀 نحوه استفاده از Trivy

چندتا مثال کاربردی:

### 🔍 اسکن یک ایمیج داکر

```bash
trivy image nginx:latest
```

### 🔍 اسکن یک فایل سیستم (برای پیدا کردن پکیج‌های آسیب‌پذیر)

```bash
trivy fs .
```

### 🔍 اسکن یک ریپازیتوری Git

```bash
trivy repo https://github.com/aquasecurity/trivy
```

### 🔍 اسکن فایل‌های IaC (مثل K8s yaml, Terraform)

```bash
trivy config ./k8s/
```

### 🔍 گرفتن خروجی JSON (برای استفاده در CI/CD یا ابزارهای دیگه)

```bash
trivy image -f json -o result.json nginx:latest
```

---



```bash
trivy image ubuntu:23.04
```

🔎 خروجی نمونه (خلاصه‌شده و شبیه‌سازی‌شده):

```
2025-08-17T18:21:01.123+0300   INFO    Vulnerability scanning is enabled
2025-08-17T18:21:01.456+0300   INFO    Number of language-specific files: 0
2025-08-17T18:21:01.456+0300   INFO    Detecting Ubuntu vulnerabilities...

ubuntu:23.04 (ubuntu 23.04)
================================
Total: 12 (HIGH: 4, MEDIUM: 6, LOW: 2)

┌────────────────────┬──────────┬──────────┬───────────────────────┐
│      Package       │ Severity │  Status  │       CVE ID          │
├────────────────────┼──────────┼──────────┼───────────────────────┤
│ libc6              │ HIGH     │ fixed    │ CVE-2024-45811        │
│ openssl            │ HIGH     │ unfixed  │ CVE-2023-5678         │
│ curl               │ MEDIUM   │ unfixed  │ CVE-2024-11223        │
│ tar                │ MEDIUM   │ fixed    │ CVE-2023-98765        │
│ coreutils          │ LOW      │ unfixed  │ CVE-2022-45678        │
└────────────────────┴──────────┴──────────┴───────────────────────┘

Recommendations:
- Run `apt-get update && apt-get upgrade` inside your Dockerfile
- Use a slimmer base image if possible (e.g. `ubuntu:23.04-minimal`)
```

---

✅ همونطور که می‌بینی، Trivy برای هر پکیج آسیب‌پذیر **Severity (شدت آسیب‌پذیری)**، **CVE ID** و وضعیتش (fixed/unfixed) رو نشون میده.
با این خروجی می‌تونی تصمیم بگیری ایمیجت رو آپدیت یا امن‌تر کنی.

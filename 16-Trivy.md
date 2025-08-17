
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


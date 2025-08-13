در دنیای داکر، **Docker sign** یا امضای داکر، به فرآیندی گفته می‌شود که در آن یک *ایمیج داکر* با یک **امضای دیجیتال** مهر می‌شود تا دیگران بتوانند صحت و اصالت آن را بررسی کنند.

هدف اصلی‌اش این است که:

* مطمئن شویم ایمیجی که دانلود می‌کنیم **دستکاری نشده** (تمامیت یا integrity).
* مطمئن شویم ایمیج از طرف **سازنده‌ی معتبر** آمده (احراز هویت یا authenticity).

---

### چگونه کار می‌کند؟

وقتی شما یک ایمیج را با `docker sign` (یا ابزارهای جدیدتر مثل **cosign** یا **Notary**) امضا می‌کنید:

1. **کلید خصوصی** شما برای تولید امضا استفاده می‌شود.
2. این امضا همراه با متادیتا روی رجیستری (مثل Docker Hub) ذخیره می‌شود.
3. هرکسی که ایمیج را می‌گیرد، می‌تواند با **کلید عمومی** شما آن را اعتبارسنجی کند.

---

### ابزارها و نسخه‌ها

* **Docker Content Trust (DCT)**: این قابلیت در داکر کلاسیک با فرمان `docker trust sign` پیاده‌سازی شده بود و پشت‌صحنه از **Notary** استفاده می‌کرد.
* **cosign / sigstore**: پروژه‌های جدیدتر که روش‌های ساده‌تر و یکپارچه‌تر برای امضای ایمیج‌ها ارائه می‌دهند.

---

### مثال ساده با Docker Content Trust

```bash
export DOCKER_CONTENT_TRUST=1
docker sign myimage:latest
```

این باعث می‌شود که داکر قبل از push کردن، ایمیج را امضا کند.

---

### چرا مهم است؟

بدون امضا، شما نمی‌توانید مطمئن باشید ایمیجی که می‌کشید، دقیقا همان است که سازنده اصلی منتشر کرده.
در سناریوهای امنیتی (DevOps، Kubernetes، CI/CD)، امضای ایمیج‌ها یکی از مراحل مهم برای جلوگیری از حملات supply chain است.

---





باشه، پس بریم با **cosign** قدم‌به‌قدم یاد بگیریم که چطور یک ایمیج رو:

1. **Sign** کنیم
2. **Push** کنیم
3. بعداً **Verify** کنیم

---

## 1️⃣ نصب cosign

اگر نصبش نکردی:

```bash
# لینوکس یا مک
curl -sSL https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64 \
    -o cosign && chmod +x cosign && sudo mv cosign /usr/local/bin/
```

برای مک `cosign-darwin-amd64` یا برای ویندوز نسخه `.exe` رو بگیر.

---

## 2️⃣ ساخت کلید امضا

```bash
cosign generate-key-pair
```

این دو فایل ایجاد می‌کنه:

* `cosign.key` → کلید خصوصی (نزد خودت و محرمانه)
* `cosign.pub` → کلید عمومی (برای اعتبارسنجی استفاده می‌شه)

می‌تونی رمز هم روی کلید خصوصی بذاری.

---

## 3️⃣ Sign کردن ایمیج

فرض کنیم ایمیج رو ساختی و تگ کردی:

```bash
docker build -t yourrepo/yourimage:1.0 .
```

حالا با cosign امضا کن:

```bash
cosign sign --key cosign.key yourrepo/yourimage:1.0
```

* `yourrepo/yourimage:1.0` همون ایمیجی هست که قراره روی Docker Hub یا رجیستری خصوصی باشه.
* این دستور امضا رو به‌عنوان artifact در رجیستری ذخیره می‌کنه.

---

## 4️⃣ Pull کردن ایمیج (بعداً یا روی یک سیستم دیگه)

```bash
docker pull yourrepo/yourimage:1.0
```

این فقط خود ایمیج رو می‌کشه، نه امضاش. ولی امضا همچنان روی رجیستری ذخیره شده.

---

## 5️⃣ Verify کردن ایمیج

```bash
cosign verify --key cosign.pub yourrepo/yourimage:1.0
```

اگر امضا معتبر باشه، چیزی شبیه این می‌بینی:

```
Verification for yourrepo/yourimage:1.0 --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log
  - Certificate issuer matches expected
```

---

💡 **نکته‌ها**

* اگر بخوای بدون مدیریت کلید لوکال کار کنی، cosign از **keyless signing** هم پشتیبانی می‌کنه که با GitHub OIDC یا Google login انجام می‌شه (کلید موقت می‌سازه).
* برای امنیت بیشتر، کلید خصوصی رو داخل **Vault** یا HSM نگه‌دار.

---

اگر بخوای می‌تونم یه **workflow کامل GitHub Actions** بهت بدم که خودش ایمیج رو بسازه، sign کنه، و push کنه.
آیا بزنم اونم برات بیارم؟

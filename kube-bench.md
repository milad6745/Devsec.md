

### 🧾 خروجی نمونه کامل kube-bench (بخش‌هایی از آن برای وضوح فشرده شده‌اند):

```
kube-bench version: v0.6.14
Benchmark: CIS Kubernetes Benchmark v1.6.0

------------------------------------------------------------
Section 1: Control Plane Components
------------------------------------------------------------

[INFO] 1.1 - API Server
[PASS] 1.1.1 - Ensure that the --anonymous-auth argument is set to false
[FAIL] 1.1.2 - Ensure that the --basic-auth-file argument is not set
[PASS] 1.1.3 - Ensure that the --insecure-bind-address argument is not set
[WARN] 1.1.4 - Ensure that the --kubelet-https argument is set to true
[FAIL] 1.1.5 - Ensure that the --secure-port argument is not 0
[INFO] 1.1.6 - Ensure that the --profiling argument is set to false

[INFO] 1.2 - Scheduler
[PASS] 1.2.1 - Ensure that the --profiling argument is set to false
[PASS] 1.2.2 - Ensure that the scheduler binds only to localhost

[INFO] 1.3 - Controller Manager
[PASS] 1.3.1 - Ensure that the --terminated-pod-gc-threshold argument is set appropriately
[FAIL] 1.3.2 - Ensure that the --profiling argument is set to false
[WARN] 1.3.3 - Ensure that the --use-service-account-credentials argument is set to true

------------------------------------------------------------
Section 2: Etcd
------------------------------------------------------------

[PASS] 2.1 - Ensure that the --cert-file and --key-file arguments are set
[PASS] 2.2 - Ensure that the --client-cert-auth argument is set to true
[FAIL] 2.3 - Ensure that the --auto-tls argument is not set to true
[PASS] 2.4 - Ensure that the --peer-cert-file and --peer-key-file are set

------------------------------------------------------------
Section 3: Policies
------------------------------------------------------------

[INFO] 3.1 - RBAC and Service Accounts
[PASS] 3.1.1 - Ensure that the --authorization-mode includes RBAC
[PASS] 3.1.2 - Ensure that the --service-account-lookup argument is set to true
[FAIL] 3.1.3 - Ensure that the --authorization-mode does not include AlwaysAllow

------------------------------------------------------------
Section 4: Node Configuration
------------------------------------------------------------

[PASS] 4.1.1 - Ensure that the kubelet service file ownership is set to root:root
[PASS] 4.1.2 - Ensure that the kubelet service file permissions are set to 644 or more restrictive
[WARN] 4.2.1 - Ensure that the --anonymous-auth argument is set to false

------------------------------------------------------------
Section 5: Configuration Files
------------------------------------------------------------

[PASS] 5.1.1 - Ensure that the Kubernetes PKI directory and file ownership is set to root:root
[FAIL] 5.1.2 - Ensure that the Kubernetes PKI directory and file permissions are set to 600 or more restrictive

------------------------------------------------------------
Summary
------------------------------------------------------------

Total Passes  : 18  
Total Failures: 6  
Total Warnings: 4  
Total Infos   : 3

Remediation:
- Review failed and warning checks.
- Update configuration files (e.g., kube-apiserver.yaml, kubelet.service).
- Restart affected services or nodes.
```

---

### 📊 توضیحات کلیدی در خروجی:

* `PASS`: تست با موفقیت انجام شده، تنظیمات امن هستند.
* `FAIL`: نیاز به اصلاح دارد؛ از نظر امنیتی توصیه نمی‌شود.
* `WARN`: هشدار؛ بهتر است بررسی شود ولی حتماً بحرانی نیست.
* `INFO`: فقط اطلاعات، نیاز به اقدام ندارد.

---

### 🎯 در پایان چه کاری باید انجام داد؟

بعد از اجرای kube-bench:

1. فایل تنظیمات اجزای مختلف مانند `kube-apiserver.yaml`, `kubelet.service` و غیره را اصلاح کنید.
2. مستندات CIS را بررسی کنید تا نحوه اصلاح هر مورد `FAIL` را بدانید.
3. سرویس‌ها یا نودهای مرتبط را **ری‌استارت** کنید تا تغییرات اعمال شوند.
4. خروجی را نگه دارید تا در گزارش‌های ممیزی و امنیتی استفاده شود.


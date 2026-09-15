# VPC Terraform + Jenkins Pipeline

## هيكل 

```
vpc-terraform-jenkins/
├── provider.tf              # AWS provider + backend config
├── variables.tf             # تعريف كل الـ variables
├── vpc.tf                   # VPC, Subnets, IGW, Route Table
├── outputs.tf                # outputs (vpc_id, subnet ids)
├── environments/
│   ├── dev.tfvars
│   ├── stg.tfvars
│   └── prod.tfvars
└── Jenkinsfile               # الـ pipeline كامل
```

## خطوات الإعداد على Jenkins

### 1. الـ Credentials
روحي في `Manage Jenkins > Credentials` وضيفي:
- `aws-access-key-id` (Secret text)
- `aws-secret-access-key` (Secret text)

نفس الـ IDs دي مستخدمة في الـ `Jenkinsfile` تحت `environment { }`.

### 2. الـ Terraform CLI على السيرفر
لازم Jenkins يقدر يلاقي أمر `terraform`، يعني إما:
- تتثبته على نفس الـ Jenkins agent، أو
- تستخدمي الـ Terraform plugin من Jenkins وتضيفيه في الـ PATH

### 3. الـ Webhook (الـ Trigger)
في GitHub repo بتاعك:
`Settings > Webhooks > Add webhook`
- Payload URL: `http://<jenkins-server>/github-webhook/`
- Content type: `application/json`
- Event: `Just the push event`

وفي Jenkins نفسه لازم يكون عندك الـ **GitHub plugin** مفعّل، والـ `triggers { githubPush() }` الموجود في الـ Jenkinsfile هو اللي بيخلي الـ pipeline يشتغل أوتوماتيك لما يحصل push.

### 4. البريد الإلكتروني (Mail on Success/Fail)
لازم تظبطي الـ SMTP من:
`Manage Jenkins > System > Extended E-mail Notification` (أو E-mail Notification العادي)
وتحطي الإيميل بتاعك بدل `you@example.com` في الـ `post { }` block.

### 5. الـ Approval Step
الـ stage اسمها `Approve` بتستخدم `input()` — ده اللي بيوقف الـ pipeline بعد الـ `plan` ويستنى حد يضغط **Approve** أو **Abort** يدوي من واجهة Jenkins، بالظبط زي ما موضح في الرسمة.
- لو ضغطتي **Approve** → يكمل على stage الـ `Apply`.
- لو ضغطتي **Abort** (أو الـ input عدى الوقت المحدد) → الـ pipeline بيفشل ويروح على الـ `post { aborted { ... } }` ويبعت إيميل.

## طريقة التشغيل
1. اعملي push للكود على GitHub → الـ webhook يشغل الـ pipeline أوتوماتيك.
2. أو شغليها يدوي من Jenkins واختاري الـ `ENVIRONMENT` (dev/stg/prod) من الـ parameter.
3. استني الـ Plan يخلص → اعملي Approve.
4. الـ Apply يشتغل → تبعتلك رسالة Success أو Fail حسب النتيجة.

## ملاحظات مهمة
- ملف `terraform.tfstate` لازم يتخزن remote (S3 backend) لو هتشغلي المشروع من أكتر من جهاز/agent — الجزء ده متعلق في `provider.tf` (معمول comment حاليًا، فعّليه لما يكون عندك S3 bucket جاهز).
- لو عايزة الـ apply يبقى أوتوماتيك من غير approval على بيئة الـ dev بس، ممكن تعملي شرط `when { expression { params.ENVIRONMENT != 'dev' } }` على stage الـ `Approve`.

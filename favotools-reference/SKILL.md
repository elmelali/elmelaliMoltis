---
name: favotools_reference
description: التوثيق التقني الكامل لمنصة FavoTools — البنية المعمارية، الأدوات الست، قواعد البيانات، وقواعد التطوير.
version: "1.0.0"
tags: [favotools, development, php, mysql, reference]
---

# 🛠️ FavoTools — التوثيق التقني الكامل

## ما هو FavoTools؟
منصة ويب مركزية ونظام متكامل (elmelali.com/favotools) يضم 6 أدوات مستقلة مصممة لتنظيم المعرفة، البحث، والإنتاجية الشخصية للأستاذ إسلام.

## 🏗️ البنية المعمارية (Architecture)

### هيكل المجلدات الإلزامي
```
favotools/
├── assets/          ← CSS, JS, Images مشتركة
├── auth/            ← تسجيل الدخول، إنشاء الحسابات
├── process/         ← معالجة البيانات (CRUD)
├── config/          ← إعدادات الاتصال بقاعدة البيانات
├── includes/        ← Header, Footer, Session Check (مشتركة)
└── tools/           ← المجلد الجوهري — كل أداة في مجلد مستقل
    ├── favolinks/
    ├── srp/
    ├── mynotes/
    ├── favostrategy/
    ├── favoincome/
    └── favoresearch/
```

**قاعدة صارمة:** كل كود جديد يجب أن يحترم هذا الهيكل. لا استثناءات.

## 💻 التقنيات المعتمدة (Tech Stack)
| الطبقة | التقنية | الإصدار |
|--------|---------|---------|
| Frontend | Tailwind CSS | v3.x |
| Components | Flowbite | latest |
| Icons | Font Awesome | latest |
| Rich Text | TinyMCE | latest |
| Select/Tags | Choices.js | latest |
| Backend | PHP | 8.x |
| Database | MySQL | 8.x |
| DB Access | PDO + Prepared Statements | — |
| Direction | RTL support | إلزامي |
| Theme | Dark Mode | إلزامي |

## 🛡️ قواعد الأمان (Security Rules)
- **المصادقة:** نظام مركزي — كل صفحة تتحقق من الجلسة عبر `includes/session_check.php`.
- **قواعد البيانات:** PDO + Prepared Statements فقط — SQL Injection محظور.
- **كلمات المرور:** مشفّرة دائماً (password_hash / password_verify).
- **الصلاحيات:** لا وصول لأي أداة بدون تسجيل دخول.

## 🔧 الأدوات الست (The Six Tools)

### 1. FavoLinks — مدير الروابط
- **الهدف:** حفظ وتنظيم المواقع المفضلة.
- **الميزات:** استخراج تلقائي للعناوين والوصف — تنظيم هرمي (تصنيفات رئيسية/فرعية) — Tags — تمييز المفضلة.
- **قاعدة البيانات:** جدول `links`, `categories`, `tags`, `link_tags`.

### 2. SRP — Scientific Research Prompts
- **الهدف:** حفظ وتنظيم Prompts الذكاء الاصطناعي الأكاديمية.
- **الميزات:** دعم العربية والإنجليزية — تصنيف — تقييم الفعالية — بحث متقدم.
- **قاعدة البيانات:** جدول `prompts`, `categories`, `ratings`.

### 3. MyNotes — مدير الملاحظات
- **الهدف:** مساحة شخصية لتدوين الأفكار والملاحظات.
- **الميزات:** TinyMCE كمحرر — تصنيف هرمي — Tags — عرض مرن.
- **قاعدة البيانات:** جدول `notes`, `categories`, `tags`, `note_tags`.

### 4. FavoStrategy — منظم الاستراتيجيات
- **الهدف:** بناء استراتيجيات العمل وتحويلها لمسارات تنفيذية.
- **الميزات:** جمع الاستراتيجيات — هيكلة إلى خطوات — إنشاء "مشاريع" منها.
- **قاعدة البيانات:** جدول `strategies`, `steps`, `projects`.

### 5. FavoIncome — مدير مصادر الدخل
- **الهدف:** متتبع مالي لإدارة مصادر الدخل وتحليل أدائها.
- **الميزات:** تسجيل المصادر — الإيرادات والنفقات — صافي الأرباح — تقارير مالية بصرية.
- **قاعدة البيانات:** جدول `income_sources`, `transactions`, `reports`.

### 6. FavoResearch — مدير المراجع البحثية
- **الهدف:** أداة أكاديمية لتنظيم المراجع العلمية وإدارة الاقتباسات.
- **الميزات:** أرشفة المراجع (كتب/مقالات) — تتبع حالة القراءة — إدارة المؤلفين — ملاحظات واقتباسات — واجهة مشابهة لـ MyNotes.
- **قاعدة البيانات:** جدول `references`, `authors`, `citations`, `notes`.

## 🔄 نظام التحديث الديناميكي
- الملف `install.php` = لوحة تحكم المطور — تثبيت وتحديث قواعد البيانات لكل مشروع بشكل مستقل.
- كل أداة لها قاعدة بياناتها المستقلة داخل مجلد `tools/`.

## ⚡ قواعد الكود الإلزامية عند التطوير
1. لا تكسر الهيكل المعماري تحت أي ظرف.
2. كل ميزة جديدة = ملف PHP مستقل في `process/` أو مجلد الأداة.
3. الـ CSS المخصص فقط في `assets/css/` — لا inline styles.
4. كل استعلام DB = Prepared Statement.
5. كل صفحة تبدأ بـ `require_once '../includes/session_check.php';`.

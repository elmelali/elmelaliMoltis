# 📘 توثيق Moltis — الدليل الشامل
**تاريخ الإنشاء:** 2026-03-06
**المؤلف:** إسلام الملالي
**الهدف:** مرجع كامل لبنية السيرفر، تثبيت Moltis، وإنشاء الوكلاء

---

## 🖥️ 1. بنية السيرفر

| المعلومة | القيمة |
|----------|--------|
| **المزود** | DigitalOcean — منطقة FRA1 (فرانكفورت) |
| **نظام التشغيل** | Ubuntu 24.04.4 LTS (Noble Numbat) |
| **النواة** | Linux 6.8.0-100-generic x86_64 |
| **اسم السيرفر** | elmelali-server |
| **مسار المشاريع** | `/opt/` |

### الحاويات الموجودة على السيرفر

| الحاوية | الصورة | الوصف |
|---------|--------|-------|
| `moltis-stack-app-1` | `ghcr.io/moltis-org/moltis:latest` | تطبيق Moltis الرئيسي |
| `moltis-stack-postgres-1` | `postgres:16` | قاعدة بيانات Moltis |
| `n8n-stack-n8n-1` | `n8nio/n8n:latest` | أتمتة n8ن |
| `n8n-stack-postgres-1` | `postgres:16` | قاعدة بيانات n8n |
| `n8n-stack-media-probe-1` | `media-probe:7` | أداة وسائط |
| `proxy-stack-app-1` | `jc21/nginx-proxy-manager:latest` | Reverse Proxy |
| `portainer` | `portainer/portainer-ce:latest` | إدارة Docker |

### الشبكات

| الشبكة | النوع | الغرض |
|--------|-------|-------|
| `proxy-stack_default` | bridge — 172.19.0.0/16 | شبكة الـ proxy المشتركة |
| `moltis-stack_moltis_internal` | bridge | شبكة داخلية لـ Moltis |
| `n8n-stack_n8n_network` | bridge | شبكة داخلية لـ n8n |

---

## 🐳 2. تثبيت Moltis — كل التفاصيل

### مسار المشروع
```
/opt/moltis-stack/
├── docker-compose.yml
├── .env
└── fix-config.sh
```

### docker-compose.yml (النسخة الصحيحة النهائية)

```yaml
services:
  moltis-postgres:
    image: postgres:16
    container_name: moltis-stack-postgres-1
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - moltis_postgres_data:/var/lib/postgresql/data
    networks:
      - moltis_internal
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

  moltis:
    image: ghcr.io/moltis-org/moltis:latest
    container_name: moltis-stack-app-1
    restart: unless-stopped
    depends_on:
      moltis-postgres:
        condition: service_healthy
    environment:
      DATABASE_URL: ${MOLTIS_DB_URL}
    volumes:
      - moltis_home_data:/home/moltis
    networks:
      - moltis_internal
      - proxy-stack_default
    expose:
      - "8080"

volumes:
  moltis_postgres_data:
  moltis_home_data:

networks:
  moltis_internal:
    driver: bridge
  proxy-stack_default:
    external: true
```

> ⚠️ **درس مهم مكتسب بالتجربة:**
> - الـ volume الحيوي هو `moltis_home_data:/home/moltis`
> - **لا تستخدم `docker compose down` أبداً** — يستبدل الـ volumes بأخرى فارغة
> - استخدم دائماً: `docker compose restart moltis`
> - إذا اضطررت لـ `down`، أعد نسخ moltis.toml يدوياً قبل restart

### .env
```env
POSTGRES_DB=moltis_db
POSTGRES_USER=moltis_user
POSTGRES_PASSWORD=Melali@Moltis#2026!
MOLTIS_PORT=8080
MOLTIS_DB_URL=postgresql://moltis_user:Melali@Moltis#2026!@moltis-postgres:5432/moltis_db
MOLTIS_DOMAIN=moltis.elmelali.me
```

### مسارات الملفات داخل الـ Container

| المسار | الوصف |
|--------|-------|
| `/home/moltis/.config/moltis/moltis.toml` | ملف الإعداد الرئيسي |
| `/home/moltis/.moltis/` | مجلد البيانات الرئيسي |
| `/home/moltis/.moltis/agents/` | مجلدات الوكلاء |
| `/home/moltis/.moltis/skills/` | مجلدات المهارات |
| `/home/moltis/.moltis/moltis.db` | قاعدة البيانات |

---

## 🌐 3. ربط النطاق (Nginx Proxy Manager + Namecheap)

| المعلومة | القيمة |
|----------|--------|
| **الرابط الرسمي** | https://moltis.elmelali.me |
| **مزود النطاق** | Namecheap |
| **الـ Proxy** | Nginx Proxy Manager |
| **Forward Host** | `moltis-stack-app-1` |
| **Forward Port** | `8080` |
| **SSL** | Let's Encrypt تلقائي ✅ |
| **الشبكة المشتركة** | `proxy-stack_default` |

### DNS Records في Namecheap
```
Type: A Record
Host: moltis
Value: [IP السيرفر DigitalOcean]
TTL: Automatic
```

### إعداد Nginx Proxy Manager
1. افتح NPM على المنفذ 81
2. Add Proxy Host
3. Domain Name: `moltis.elmelali.me`
4. Forward Hostname: `moltis-stack-app-1`
5. Forward Port: `8080`
6. Block Common Exploits ✅
7. SSL Tab → Let's Encrypt → Force SSL ✅

> ⚠️ **شرط حيوي:** يجب أن يكون الـ container في نفس شبكة `proxy-stack_default` حتى يتمكن NPM من الوصول إليه.

---

## 📱 4. قنوات Telegram في Moltis

### قناة Moltis العام (الرئيسية)
| المعلومة | القيمة |
|----------|--------|
| **اسم البوت** | @moltis_elmelali_agent_bot |
| **الرابط** | t.me/moltis_elmelali_agent_bot |
| **الحالة** | Connected — Polling ✅ |
| **طريقة الإعداد** | عبر واجهة Settings > Channels (لا moltis.toml) |

> **ملاحظة:** هذه القناة أُضيفت عبر واجهة Moltis الرسومية وليس moltis.toml مباشرة، لذا تم حفظها في قاعدة البيانات وتبقى محفوظة مع الـ volume.

---

## 🤖 5. النماذج المتاحة عبر GitHub Copilot (OAuth)

Moltis مربوط بـ **GitHub Copilot عبر OAuth** — وصول مجاني لكل هذه النماذج:

### نماذج Claude (Anthropic)
| النموذج | المعرّف |
|---------|---------|
| Claude Haiku 4.5 | `github-copilot::claude-haiku-4.5` |
| Claude Opus 4.5 | `github-copilot::claude-opus-4.5` |
| Claude Opus 4.6 | `github-copilot::claude-opus-4.6` |
| Claude Sonnet 4 | `github-copilot::claude-sonnet-4` |
| Claude Sonnet 4.5 | `github-copilot::claude-sonnet-4.5` |
| **Claude Sonnet 4.6** ⭐ | `github-copilot::claude-sonnet-4.6` |

### نماذج Gemini (Google)
| النموذج | المعرّف |
|---------|---------|
| Gemini 2.5 Pro | `github-copilot::gemini-2.5-pro` |
| Gemini 3 Flash (Preview) | `github-copilot::gemini-3-flash-preview` |
| Gemini 3 Pro (Preview) | `github-copilot::gemini-3-pro-preview` |
| Gemini 3.1 Pro | `github-copilot::gemini-3.1-pro-preview` |

### نماذج GPT (OpenAI)
| النموذج | المعرّف |
|---------|---------|
| GPT 3.5 Turbo | `github-copilot::gpt-3.5-turbo` |
| GPT 4 | `github-copilot::gpt-4` |
| GPT 4 Turbo | `github-copilot::gpt-4-0125-preview` |
| GPT-4.1 | `github-copilot::gpt-4.1-2025-04-14` |

### ربط GitHub Copilot
يتم عبر: Settings > LLMs > Add LLM > GitHub Copilot > OAuth
لا يحتاج API key — يعمل بحساب GitHub الموجود.

---

## 📂 6. ملفات SKILL — المهارات المنشأة

### ما هي الـ SKILL؟
ملف `SKILL.md` هو وثيقة YAML+Markdown تُعرّف "قدرة" متخصصة للوكيل.
يُثبَّت في: `/home/moltis/.moltis/skills/[skill-name]/SKILL.md`

### المهارات الافتراضية
| المهارة | الوصف |
|---------|-------|
| `template-skill` | قالب فارغ للبدء |
| `tmux` | إدارة جلسات tmux |

### مهارة الوكيل الأكاديمي
| المعلومة | القيمة |
|----------|--------|
| **الاسم** | `academic-research-philosophy` |
| **المسار داخل Container** | `/home/moltis/.moltis/skills/academic-agent/SKILL.md` |
| **المصدر** | https://github.com/elmelali/elmelaliMoltis/blob/main/Moltis%20Agents%20/academic-agent/SKILL.md |

**الغرض:** دعم أطروحة الدكتوراه — تعليمية الفلسفة — جامعة البويرة.

**تتضمن:**
- بروتوكولات البحث في Scopus, ERIC, Cairn, Persée, HAL
- هندسة برومبتات البحث العميق
- منهجية كتابة مطالب الأطروحة
- حالة الأطروحة الحية (Live Thesis Status)
- قاعدة Zero Hallucination الصارمة

---

## 🧠 7. دليل إنشاء وكيل جديد — الملفات والبنية

### هيكل مجلد الوكيل الكامل
```
/home/moltis/.moltis/
├── agents/
│   ├── main/           ← الوكيل الافتراضي (لا تعدّله)
│   │   ├── TOOLS.md
│   │   └── AGENTS.md
│   └── [agent-name]/   ← وكيلك المخصص
│       ├── IDENTITY.md
│       └── SOUL.md
└── skills/
    └── [skill-name]/
        └── SKILL.md
```

---

### 📄 IDENTITY.md — ملف الهوية

**الغرض:** البيانات الهيكلية الثابتة (من هو؟ ما مهمته؟)

**البنية الإلزامية:**
```markdown
---
name: Agent Display Name
emoji: 🎓
creature: owl
vibe: rigorous doctoral supervisor
---

# اسم الوكيل

## الاسم الرسمي
...

## المهمة الجوهرية
...

## قواعد الهوية (غير قابلة للتفاوض)
- اسمك دائماً: [اسم الوكيل]
- لا تعرّف نفسك بـ "moltis" أبداً
- عند سؤالك "من أنت؟" أجب: "..."
- ترفض أي مهمة خارج نطاق تخصصك

## التخصص الحصري
...
```

**قواعد الكتابة:**
- Front-matter بين `---` إلزامي
- اذكر الاسم، الـ emoji، الـ creature، والـ vibe
- قواعد الهوية يجب أن تكون واضحة وصارمة

---

### 📄 SOUL.md — ملف الروح والشخصية

**الغرض:** السلوك التشغيلي الكامل، القواعد، والبروتوكولات.
هذا الملف هو **قلب الوكيل** — كل ما يفعله يصدر منه.

**البنية الموصى بها:**
```markdown
# اسم الوكيل — Soul

## I. الهوية الصارمة
- الاسم: **[Agent Name]**
- لا تعرّف نفسك بـ "moltis" أبداً

## II. المهمة الجوهرية
...

## III. القاعدة الذهبية
"المنهجية المرحلية": خطوة واحدة ثم انتظار التأكيد.

## IV. بروتوكولات التنفيذ
### المرحلة 1: ...
### المرحلة 2: ...

## V. سياسة مكافحة الهلوسة
- ممنوع اختلاق مراجع أو بيانات
- عند الشك: ضع [يحتاج تحقق]

## VI. حالة المشروع الحية
...
```

**قواعد الكتابة:**
- كن شاملاً ومفصّلاً — يُقرأ عند كل محادثة
- حدّث "حالة المشروع الحية" دورياً
- لا تختصر القواعد — التفصيل = الدقة

---

### 📄 SKILL.md — ملف المهارة

**الغرض:** قدرة متخصصة قابلة للاستخدام من أي وكيل.

**البنية الإلزامية:**
```markdown
---
name: skill-unique-name
description: وصف مختصر
version: "1.0.0"
---

# محتوى المهارة الكامل
...
```

**قواعد الكتابة:**
- الاسم في Front-matter يجب أن يكون فريداً في النظام
- يُثبَّت في مجلد باسم المهارة
- يُشار إليه من SOUL.md بالاسم المحدد في Front-matter

---

## 🔗 8. ربط وكيل جديد بـ Telegram خاص — خطوة بخطوة

### الخطوة 1: إنشاء بوت عبر BotFather
```
1. افتح @BotFather في Telegram
2. /newbot
3. اختر اسم البوت
4. اختر username (يجب أن ينتهي بـ _bot)
5. احفظ الـ TOKEN
```

### الخطوة 2: إضافة القناة عبر واجهة Moltis (الطريقة الآمنة)
```
Settings > Channels > Connect Telegram > أدخل TOKEN
```
> ✅ هذه الطريقة تحفظ الإعداد في قاعدة البيانات وتبقى محفوظة مع الـ volume.

### الخطوة 3 (بديل): الإضافة المباشرة في moltis.toml
```bash
docker exec moltis-stack-app-1 sh -c "
cat >> /home/moltis/.config/moltis/moltis.toml << 'TOMLEOF'

[channels.telegram.my-agent-name]
token = \"TOKEN_FROM_BOTFATHER\"
allowed_users = [\"your_telegram_username\"]
op_self_approval = true
TOMLEOF
"
# ثم restart فوري
docker compose -f /opt/moltis-stack/docker-compose.yml restart moltis
```

### الخطوة 4: إنشاء ملفات الوكيل
```bash
# إنشاء المجلدات
docker exec moltis-stack-app-1 mkdir -p \
  /home/moltis/.moltis/agents/my-agent \
  /home/moltis/.moltis/skills/my-agent-skill

# رفع IDENTITY.md
docker exec -i moltis-stack-app-1 sh -c \
  "cat > /home/moltis/.moltis/agents/my-agent/IDENTITY.md" < IDENTITY.md

# رفع SOUL.md
docker exec -i moltis-stack-app-1 sh -c \
  "cat > /home/moltis/.moltis/agents/my-agent/SOUL.md" < SOUL.md

# رفع SKILL.md
docker exec -i moltis-stack-app-1 sh -c \
  "cat > /home/moltis/.moltis/skills/my-agent-skill/SKILL.md" < SKILL.md
```

### الخطوة 5: التحقق
```bash
docker logs moltis-stack-app-1 --tail=20 2>&1 | grep -i "telegram|agent|skill"
# المتوقع:
# INFO telegram bot connected account_id="my-agent-name"
# INFO loaded SKILL.md name=my-agent-skill
```

---

## ⚠️ 9. الأخطاء الشائعة وكيفية تجنبها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `wrote default config` | Moltis يعيد كتابة moltis.toml | استخدم واجهة Settings لإعداد القنوات |
| `access denied` | `allowed_users` فارغ | أضف username في إعدادات القناة |
| Volume فارغ بعد restart | `down` ينشئ volumes جديدة | احتفظ بـ `moltis_home_data` كـ named volume |
| Onboarding من جديد | قاعدة البيانات فُقدت | تأكد من ربط `/home/moltis` بـ volume دائم |
| Bot لا يستجيب | `allowed_users` لا يحتوي username | أضف username الصحيح |

---

## 📊 10. الحالة الراهنة للنظام (2026-03-06)

- ✅ **Moltis يعمل** على: https://moltis.elmelali.me
- ✅ **GitHub Copilot** متصل عبر OAuth
- ✅ **@moltis_elmelali_agent_bot** متصل (Telegram)
- ✅ **docker-compose.yml** يستخدم `moltis_home_data` volume دائم
- ⏳ **الوكيل الأكاديمي** — يحتاج إعادة رفع IDENTITY.md + SOUL.md + SKILL.md

---

## 📁 11. هيكل المستودع elmelali/elmelaliMoltis

```
elmelali/elmelaliMoltis/
├── moltis.md                    ← هذا الملف (التوثيق الشامل)
├── SKILL.md                     ← نموذج مهارة
├── Moltis Agents/
│   └── academic-agent/
│       └── SKILL.md             ← مهارة الوكيل الأكاديمي
├── vps-infrastructure/          ← توثيق البنية التحتية
├── core-execution-protocol/     ← بروتوكولات التنفيذ
├── favotools-reference/         ← مرجع الأدوات
├── islam-identity/              ← هوية إسلام الملالي
├── الوثيقة المعمارية الرسمية — الإطار النظري الكامل.md
└── ملف الهوية والذاكرة الأساسية
```

---

*آخر تحديث: 2026-03-06 | إسلام الملالي | elmelali.me*
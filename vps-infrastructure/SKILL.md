---
name: vps_infrastructure
description: التوثيق الكامل للبنية التحتية الرقمية — السيرفر، الشبكة، الـ Stack، وحالة GitHub Student Pack.
version: "1.0.0"
tags: [vps, infrastructure, docker, server, reference]
---

# 🖥️ البنية التحتية الرقمية — elmelali-server

## 1. الخادم الرئيسي
| المواصفة | القيمة |
|----------|--------|
| المزود | DigitalOcean |
| اسم السيرفر | elmelali-server |
| نظام التشغيل | Ubuntu 24.04 LTS x64 |
| RAM | 2 GB |
| CPU | 1 vCPU |
| Storage | 50 GB SSD |
| IP Address | 159.89.1.240 |
| الموقع | Frankfurt, Germany (FRA1) |
| التكلفة | مغطاة بمنحة GitHub حتى فبراير 2027 |

## 2. الأمان والنسخ الاحتياطي
- **النسخ الاحتياطي:** مفعّل — أسبوعي تلقائي.
- **Snapshot:** `elmelali-server-1770665991571` — حجم 7.12 GB — نقطة استرجاع يدوية.
- **SSH Access:** `ssh root@159.89.1.240`
- **المستخدم الموحد:** admin أو البريد الإلكتروني.

## 3. الشبكة والدومين
- **مزود الدومين:** Namecheap (مجاني لسنة).
- **الدومين الرئيسي:** elmelali.me
- **SSL:** Let's Encrypt — تجدد تلقائي.

### النطاقات الفرعية (Subdomains)
| الـ Subdomain | الخدمة | الحالة |
|---------------|--------|-------|
| n8n.elmelali.me | منصة الأتمتة n8n | 🟢 Online |
| moltis.elmelali.me | Moltis AI Agent | 🟢 Online |

## 4. منظومة البرمجيات (Docker Stack)
| الأداة | الوظيفة | الرابط الداخلي | الرابط الخارجي |
|--------|---------|---------------|---------------|
| Docker | محرك الحاويات | — | — |
| Portainer | لوحة تحكم رسومية | IP:9443 | خاص |
| Nginx Proxy Manager | SSL + Reverse Proxy | IP:81 | خاص |
| n8n | منصة الأتمتة | IP:5678 | https://n8n.elmelali.me |
| PostgreSQL (n8n) | قاعدة بيانات n8n | port 5432 | داخلية |
| Moltis App | AI Agent Platform | port 13131 | https://moltis.elmelali.me |
| PostgreSQL (Moltis) | قاعدة بيانات Moltis | port 5432 | داخلية |

## 5. Moltis Stack التفصيلي
```
المسار:    /opt/moltis-stack/
Container: moltis-stack-app-1
Port:      13131 (HTTPS داخلي)
Model:     github-copilot::claude-sonnet-4.6
Telegram:  @moltis_elmelali_agent_bot
Web UI:    https://moltis.elmelali.me
```

### Volumes المهمة
```
moltis-stack_moltis_data  → /data
[hash]                    → /home/moltis/.config/moltis  ← الإعدادات
[hash]                    → /home/moltis/.moltis          ← Skills + Memory
[hash]                    → /var/run/docker.sock
```

**⚠️ قاعدة حرجة:** لا تستخدم `docker compose down` — استخدم `docker compose restart` فقط لتجنب فقدان البيانات.

## 6. أوامر الإدارة الأساسية
```bash
# الدخول للسيرفر
ssh root@159.89.1.240

# إعادة تشغيل Moltis (آمن)
cd /opt/moltis-stack && docker compose restart

# مراقبة الموارد
free -h && docker stats --no-stream

# عرض السكيلز المحملة
docker logs moltis-stack-app-1 2>&1 | grep "loaded SKILL"
```

## 7. حالة GitHub Student Pack
### ✅ مُفعَّل ومستغَل
- **DigitalOcean:** رصيد $200 — مُستهلَك جزء بسيط — يكفي +12 شهراً.
- **Namecheap:** دومين .me مجاني + Domain Privacy.
- **GitHub Copilot Pro:** مفعّل — Claude Sonnet 4.6 + Gemini 3.1 Pro + أكثر.

### 🎁 رصيد استراتيجي متبقٍّ
| الخدمة | القيمة | الأولوية |
|--------|--------|---------|
| Microsoft Azure | $100 + Azure OpenAI | 🔴 عالية — قيد الحل |
| JetBrains | رخصة كاملة | متوسطة |
| Canva Pro | 12 شهراً | متوسطة |
| StreamYard | بث مباشر | منخفضة |
| Typeform | استبيانات | منخفضة |

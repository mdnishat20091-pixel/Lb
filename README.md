# ব্যক্তিগত প্রোভাইডার ম্যানেজার বট (Telegram)

Python 3.12 · aiogram 3 · PostgreSQL (asyncpg) · aiohttp · Railway

## ডিপ্লয় (GitHub → Railway)
1. প্রজেক্টটি GitHub রিপোজিটরিতে আপলোড করুন (`.env` কখনো নয়)।
2. Railway → New Project → Deploy from GitHub repo।
3. একই প্রজেক্টে PostgreSQL যোগ করুন।
4. বট সার্ভিসের Variables-এ নিচের মানগুলো দিন।
5. Deploy করুন। টেবিল ও মাইগ্রেশন প্রথম চালুর সময় স্বয়ংক্রিয়ভাবে তৈরি হয়।

## Environment Variables
| নাম | বিবরণ |
|---|---|
| BOT_TOKEN | BotFather থেকে পাওয়া টোকেন |
| OWNER_ID | আপনার Telegram ইউজার আইডি (সংখ্যা) |
| ALLOWED_USER_IDS | অন্য অনুমোদিত আইডি, কমা দিয়ে আলাদা (সর্বোচ্চ মোট ১০) |
| DATABASE_URL | Railway Postgres-এর `DATABASE_URL` (Reference Variable দিন) |
| ENCRYPTION_KEY | ঐচ্ছিক। Fernet কী। না দিলে BOT_TOKEN থেকে তৈরি হয়। |

ENCRYPTION_KEY তৈরি: `python -c "from cryptography.fernet import Fernet;print(Fernet.generate_key().decode())"`
**সতর্কতা:** পরে এই কী (বা BOT_TOKEN, যদি ENCRYPTION_KEY না থাকে) বদলালে সংরক্ষিত API Key পড়া যাবে না; আবার Key দিতে হবে।
API Key কোনো environment variable-এ লাগে না; বটের ভেতর থেকে দিতে হয়।

## ডাটাবেস সেটআপ
আলাদা কিছু করতে হয় না। `migrations/*.sql` ক্রমানুসারে চলে এবং `schema_migrations` টেবিলে রেকর্ড থাকে।
টেবিল: users, providers, provider_configs, request_logs, settings, audit_logs।

## API কনফিগারেশন গাইড
বটে `/api` → প্রোভাইডার বাছুন → Base URL → API Key → সংযোগ পরীক্ষা → সংরক্ষণ → প্রোভাইডার পেজ থেকে সক্রিয় করুন।

সংযোগ পরীক্ষায় শুধু প্ল্যানে তালিকাভুক্ত একটি এন্ডপয়েন্টে প্রমাণীকৃত GET পাঠানো হয়:
| প্রোভাইডার | হেডার | পরীক্ষার এন্ডপয়েন্ট |
|---|---|---|
| VOLTX SMS | mauthapi | GET /liveaccess |
| DGDNETWORK | X-API-KEY | GET /api/v1/user/checknum |
| MK NETWORK | mknetwork-key | GET /whoami |
| 2EEE7 NETWORK | — | নেই (যাচাই বাকি, নিষ্ক্রিয়) |

ফলাফলের নিয়ম: HTTP 2xx = সংযুক্ত; 401/403 = Key সঠিক নয়; 429 = সীমা অতিক্রম; অন্য 4xx = "নিশ্চিত নয়" (সংযুক্ত দেখানো হয় না); 3xx/5xx/টাইমআউট = ব্যর্থ।

## Known Limitations
- ডকুমেন্টেশন পেজগুলো আমার টুল থেকে পড়া যায়নি (বট-ব্লক/মেইনটেন্যান্স/ত্রুটি)। তাই এন্ডপয়েন্ট ও হেডার শুধু আপনার প্ল্যান থেকে নেওয়া; রেসপন্স বডি পার্স করা হয় না, শুধু HTTP স্ট্যাটাস দেখা হয়।
- DGDNETWORK-এর `checknum` সাধারণত প্যারামিটার চায়; প্যারামিটার ছাড়া 400/422 এলে "নিশ্চিত নয়" দেখাবে। এটি আসল ডকুমেন্টেশন দেখে ঠিক করা দরকার।
- নাম্বার নেওয়ার (`getnum`) ফিচার এই সংস্করণে নেই, কারণ রিকোয়েস্ট ফরম্যাট যাচাই করা যায়নি এবং প্ল্যানের মেনুতেও নেই।
- 2EEE7 NETWORK ডকুমেন্টেশন অসম্পূর্ণ থাকায় নিষ্ক্রিয়; শুধু URL ও Key সংরক্ষণ করা যায়।
- ডার্ক/লাইট থিম: Telegram ক্লায়েন্টের থিম অনুযায়ী নিজে বদলায়; বটে আলাদা থিম বোতাম নেই (অকার্যকর বোতাম এড়াতে)।
- উইজার্ডের অবস্থা মেমরিতে থাকে; রিস্টার্ট হলে চলমান উইজার্ড বাতিল হয়।
- `utils/ui.py` ও `.python-version` অতিরিক্ত সহায়ক ফাইল (প্ল্যানের কাঠামোর বাইরে)।

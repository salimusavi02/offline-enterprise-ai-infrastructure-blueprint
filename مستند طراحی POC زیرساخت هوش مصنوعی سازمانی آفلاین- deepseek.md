# مستند طراحی POC زیرساخت هوش مصنوعی سازمانی آفلاین

**نسخهٔ سند**: ۰.۲ (پیش‌نویس — Proposed / Recommended، نه تصمیم معماری مصوب)
**تاریخ آخرین به‌روزرسانی**: ۱۴۰۵/۰۲/۲۶ (2026-05-26)
**دامنه**: Minimum Viable Offline Enterprise AI Infrastructure POC
**زبان**: فارسی، با حفظ نام‌های فنی انگلیسی
**مبنای طراحی**: Master Prompt — بخش‌های ۴ و ۵ (بر اساس Master Prompt استنباط شد)

---

## فهرست

۱. خلاصهٔ اجرایی
۲. دامنه و فرضیات POC
۳. مبنای معماری
۴. معماری منطقی و لایه‌ها
۵. ارزیابی ابزارها — دقیقاً سه ابزار برای هر لایه
۶. ارزیابی مدل‌ها و پردازش زبان فارسی
۷. ماتریس انتخاب فناوری
۸. فهرست نهایی فناوری‌های منتخب
۹. معماری فیزیکی استقرار
۱۰. جریان داده End-to-End
۱۱. استراتژی Offline / Air-Gapped
۱۲. Security and Access Control
۱۳. وضعیت تأیید نسخه و License
۱۴. پیاده‌سازی Roadmap
۱۵. معیارهای پذیرش
۱۶. ریسک‌ها و محدودیت‌های فنی
۱۷. منابع
۱۸. برآورد منابع سخت‌افزاری

---

## ۱. خلاصهٔ اجرایی

این سند طرح حداقلی (Minimum Viable POC) یک سیستم هوش مصنوعی سازمانی مبتنی بر Document Intelligence و Retrieval-Augmented Generation (RAG) را برای محیط کاملاً آفلاین (Air-Gapped) ارائه می‌دهد. دامنهٔ POC شامل ۲ تا ۳ کاربر آزمایشی، استقرار Single-Node، ورودی صرفاً DOCX و XLSX، اجرای کاملاً محلی LLM و Embedding، و مدیریت سرویس‌ها با Docker Compose است.

تمام ده لایهٔ منطقی معماری حفظ می‌شوند، اما پیاده‌سازی هر لایه متناسب با مقیاس کوچک ساده‌سازی شده است. تعداد Deployable Unitها حداقل نگه داشته شده و از زیرساخت‌های توزیع‌شده و ابزارهای پیچیده اجتناب شده است.

**اصل حاکم بر همهٔ انتخاب‌ها**: ابزارها و نسخه‌های ذکرشده در این سند صرفاً **Proposed / Recommended** هستند و به‌معنای تصویب تصمیم معماری نیستند. هیچ نسخه، License یا قابلیت فنی در این سند حدس زده نشده؛ هر جا اطلاعات کافی نبوده، صریحاً به‌عنوان «نیازمند تأیید» علامت‌گذاری شده است.

---

## ۲. دامنه و فرضیات POC

### ۲.۱. دامنهٔ تأییدشده

| مورد | مقدار |
|---|---|
| تعداد کاربران | ۲ تا ۳ کاربر آزمایشی |
| نوع استقرار | Single-Node |
| محیط اجرا | کاملاً آفلاین / Air-Gapped |
| فرمت ورودی | DOCX و XLSX |
| مدل پردازش | Document Ingestion + Document Intelligence + RAG |
| اجرای LLM و Embedding | کاملاً محلی |
| مدیریت سرویس‌ها | Docker Compose |
| معماری نرم‌افزار | Modular Monolith با مرزبندی مشخص |
| هدف | اثبات امکان‌پذیری فنی و ارزیابی کیفیت اجزا |

### ۲.۲. فرضیات صریح

**فرض A1** — دادهٔ Lake در POC با ساختار bucket/prefix در MinIO پیاده‌سازی می‌شود؛ از Iceberg/Hudi/Delta Lake استفاده نمی‌شود (طبق Master Prompt بخش ۱۵).

**فرض A2** — OpenSearch فقط برای BM25 Lexical Search به‌کار می‌رود؛ قابلیت Vector Search آن فعال نمی‌شود (Vector Search توسط Qdrant انجام می‌شود).

**فرض A3** — LLM و Embedding Runtime از نظر منطقی جدا هستند اما می‌توانند در یک یا دو فرایند سبک اجرا شوند. مدل Embedding و مدل تولید مستقل بارگذاری و مستقل نسخه‌بندی می‌شوند.

**فرض A4** — سه سند مرجع در این گفتگو محتوایشان در دسترس نیست. طرح حاضر بر مبنای اصول و لایه‌های تعریف‌شده در Master Prompt بخش‌های ۴ و ۵ استنباط شده است. هر جا استنباط به‌کار گرفته، با برچسب «بر اساس Master Prompt استنباط شد» مشخص شده است.

### ۲.۳. تصمیمات حداقلی برای سه ابهام

**ابهام ۱ — Chunk Schema**: طرح حداقلی پیشنهادی به‌عنوان «پیشنهادی» ارائه می‌شود، نه الزام معماری. حداقل فیلدها: `chunk_id` (ULID)، `document_id`، `document_version`، `chunk_index`، `content`، `source_type` (word_paragraph / excel_sheet_row)، `location_meta` (page/heading/sheet/row)، `content_hash`.

**ابهام ۲ — معنای Activation**: ساده‌سازی به «فعال‌سازی خودکار پس از اعتبارسنجی ایندکس». یعنی سند پس از تکمیل ایندکس‌گذاری و عبور از `index_validation` به‌صورت خودکار `active` می‌شود. بدون گردش‌کار تأیید.

**ابهام ۳ — پردازش فارسی**: نرمال‌سازی فارسی به‌صورت یک ماژول مستقل منطقی داخل Ingestion Layer پیاده‌سازی می‌شود (نه سرویس جدا).

---

## ۳. مبنای معماری

### ۳.۱. اصول معماری غیرقابل تغییر

- Offline-First / Air-Gapped
- Single-Node Deployment
- Separation of Responsibilities
- Modular Application / Modular Monolith
- Hexagonal Architecture (Ports & Adapters)
- Canonical Data Ownership
- Versioned Processing
- Idempotent Processing
- Rebuildable Search Indexes
- Permission-Aware Retrieval
- Controlled Document Lifecycle
- Observable Processing and Retrieval
- Incremental Evolution

### ۳.۲. تفکیک مسئولیت داده (الزامی)

| مؤلفه | مسئولیت |
|---|---|
| MinIO | Canonical Artifacts and Chunks |
| PostgreSQL | Operational Metadata, Workflow State, ACL, Lineage |
| Qdrant | Derived Vector Index (قابل بازسازی) |
| OpenSearch | Derived Lexical Index (قابل بازسازی) |

**قاعدهٔ الزامی**: PostgreSQL مرجع معتبر ACL است. هیچ محتوای غیرمجاز نباید وارد Reranking، Context Assembly یا LLM Context شود. در صورت عدم امکان تأیید مجوز، سیستم باید **Fail Closed** عمل کند.

---

## ۴. معماری منطقی و لایه‌ها

| لایه | مسئولیت اصلی | ساده‌سازی POC |
|---|---|---|
| L1 Client & Access | UI، احراز هویت، آپلود، نمایش منابع | UI و Auth در یک فرایند سبک |
| L2 API & Application | Upload/Query/Document Management API | ادغام با UI در یک Python Process |
| L3 Domain & Orchestration | Document Lifecycle، Idempotency، Retry | Python + PostgreSQL Worker بدون Broker |
| L4 Ingestion & Processing | DOCX/XLSX Parsing، Normalization، Chunking | python-docx + openpyxl + ماژول نرمال‌سازی فارسی |
| L5 AI & Model Runtime | LLM و Embedding محلی | Ollama + Sentence Transformers |
| L6 Retrieval & Search | Vector + BM25 + Hybrid + RRF + ACL Revalidation | Qdrant + OpenSearch + RRF دستی |
| L7 Data & Persistence | Canonical، Metadata، Derived Index | MinIO + PG + Qdrant + OpenSearch |
| L8 Security & Identity | AuthN/AuthZ، Document ACL، Fail Closed | Auth خودساخته + PG ACL |
| L9 Monitoring & Evaluation | Logging، Tracing، RAG Quality | logging + audit_log + Ragas |
| L10 Deployment & Offline Ops | Docker Compose، Offline Transfer | docker save/load + pip download |

**نکته**: تمام ده لایه در POC حضور دارند؛ هیچ لایه‌ای حذف نشده است. ساده‌سازی صرفاً در روش پیاده‌سازی است، نه در وجود لایه.

---

## ۵. ارزیابی ابزارها — دقیقاً سه ابزار برای هر لایه

### Layer 1 — Client & Access Layer

**مسئولیت**: رابط کاربری حداقلی، ورود و احراز هویت، مدیریت Session، دریافت پرسش و نمایش پاسخ، بارگذاری DOCX/XLSX، نمایش منابع استنادشده.

**تصمیم ساده‌سازی**: UI و Auth در یک فرایند سبک. بدون فریم‌ورک فرانت‌اند مستقل، بدون OAuth Provider، بدون Keycloak.

#### Option 1 — Streamlit (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید از Release رسمی PyPI |
| نقش دقیق | Web UI حداقلی: فرم آپلود، صفحهٔ ورود، پرسش و پاسخ، فهرست منابع |
| قابلیت‌های POC | کامپوننت آپلود فایل، نمایش متن، `st.session_state`، فراخوانی HTTP به Backend |
| مزایا | صفر فرانت‌اند، نصب ساده، تصویر Docker کوچک، هم‌زیستی طبیعی با FastAPI |
| محدودیت‌ها | Routing چندصفحه‌ای ضعیف، تعامل پیچیده دشوار، Auth تولیدی نیازمند پیاده‌سازی دستی |
| پیچیدگی نصب | بسیار کم |
| آفلاین | کاملاً ممکن؛ نصب wheel آفلاین، بدون CDN خارجی |
| وابستگی‌ها | Python 3.10+، اختیاری Pillow/tornado |
| منابع سخت‌افزاری | ~۲۰۰–۴۰۰ مگابایت RAM |
| License | Apache 2.0 (نیازمند تأیید از LICENSE رسمی) |
| مستندات | https://docs.streamlit.io |

#### Option 2 — FastAPI + Jinja2/HTMX (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | API و UI یکجا: FastAPI برای Upload/Query، Jinja2 برای HTML حداقلی، HTMX برای به‌روزرسانی جزئی |
| قابلیت‌های POC | کنترل کامل Auth، آپلود فایل، رندر قالب، هم‌فرایندی با Domain Layer |
| مزایا | معماری شفاف، آزادی در Auth، بدون کانتینر UI جداگانه، ادغام طبیعی با L2 |
| محدودیت‌ها | نیاز به مقدار کمی HTML/CSS، حجم توسعهٔ UI بیشتر از Streamlit |
| پیچیدگی نصب | کم تا متوسط |
| آفلاین | کاملاً 가능 |
| وابستگی‌ها | Python، uvicorn، jinja2، python-multipart |
| منابع سخت‌افزاری | ~۱۵۰–۳۰۰ مگابایت RAM |
| License | MIT (FastAPI)، BSD (Jinja2) — نیازمند تأیید |
| مستندات | https://fastapi.tiangolo.com |

#### Option 3 — Gradio (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | ساخت سریع Chat UI و File Upload |
| قابلیت‌های POC | `ChatInterface`، کامپوننت `File`، مدیریت Session |
| مزایا | سریع‌ترین راه‌اندازی، کامپوننت‌های Chat بالغ |
| محدودیت‌ها | Auth قابل‌سفارشی‌سازی ضعیف، تم پیش‌فرض سنگین، وابستگی‌های زیاد (gradio-client، huggingface-hub)، حجم wheel آلفلاین بزرگ‌تر |
| پیچیدگی نصب | کم |
| آفلاین |possible اما نیاز به تنظیم `HF_HUB_OFFLINE=1` |
| وابستگی‌ها | Python، gradio، huggingface-hub، pydantic |
| منابع سخت‌افزاری | ~۴۰۰–۶۰۰ مگابایت RAM |
| License | Apache 2.0 — نیازمند تأیید |
| مستندات | https://www.gradio.app/docs |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیهٔ Layer 1**: **Streamlit (Option 1)**. نیاز UI در POC بسیار محدود است. ویژگی «صفر فرانت‌اند» Streamlit اجازه می‌دهد زمان توسعه‌دهنده روی RAG و لایهٔ داده متمرکز شود. Auth با یک صفحهٔ login ساده و `st.session_state` برای ۲–۳ کاربر کافی است.

---

### Layer 2 — API & Application Layer

**مسئولیت**: Upload API، Document Management API، Query API، اعتبارسنجی درخواست‌ها، کنترل دسترسی اولیه.

**تصمیم ساده‌سازی**: ادغام با Layer 1 و Domain Layer در یک Python Process (Modular Monolith). بدون کانتینر API جداگانه.

#### Option 1 — FastAPI (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید (صفحهٔ رسمی PyPI) |
| نقش دقیق | تعریف endpointهای `/upload`، `/documents`، `/query`؛ مدل‌های Pydantic برای اعتبارسنجی؛ Dependency Injection برای Auth |
| قابلیت‌های POC | پردازش Async آپلود، مستندات OpenAPI خودکار، اعتبارسنجی Schema با Pydantic، `BackgroundTasks` برای کارهای سبک |
| مزایا | Type-safe، مستندات خودکار، هم‌زیستی با اکوسیستم Python، سبک و پرسرعت |
| محدودیت‌ها | برنامه‌نویسی Async ممکن است برای POC ساده پیچیدگی ایجاد کند؛ `BackgroundTasks` برای کارهای طولانی مناسب نیست (نیاز به Worker چرخشی L3) |
| پیچیدگی نصب | کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | uvicorn، pydantic، python-multipart |
| منابع سخت‌افزاری | ~۱۰۰–۲۰۰ مگابایت RAM |
| License | MIT — نیازمند تأیید |
| مستندات | https://fastapi.tiangolo.com |

#### Option 2 — Flask (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | Routing همگام، ساده و مستقیم |
| قابلیت‌های POC | Routing، پارس درخواست، Blueprint برای ماژول‌بندی |
| مزایا | حداقل پیچیدگی، کمترین منحنی یادگیری، مدل همگام برای جریان ساده شهودی‌تر |
| محدودیت‌ها | بدون اعتبارسنجی/سریال‌سازی داخلی (نیاز به marshmallow یا دستی)؛ پشتیبانی Async ضعیف؛ بدون مستندات خودکار |
| پیچیدگی نصب | بسیار کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | Werkzeug، Jinja2 |
| منابع سخت‌افزاری | ~۸۰–۱۵۰ مگابایت RAM |
| License | BSD-3-Clause — نیازمند تأیید |
| مستندات | https://flask.palletsprojects.com |

#### Option 3 — Litestar (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | فریم‌ورک Async مدرن با DI داخلی، سریال‌سازی، پشتیبانی Auth |
| قابلیت‌های POC | مشابه FastAPI اما با تأکید بیشتر بر DI صریح و عملکرد |
| مزایا | عملکرد بهتر از FastAPI در برخی بنچمارک‌ها، طراحی DI شفاف‌تر |
| محدودیت‌ها | اکوسیستم و جامعه بسیار کوچک‌تر از FastAPI؛ مستندات و مثال‌های آفلاین کمتر؛ راه‌حل‌های قابل مراجعه در مواجهه با مشکل کمتر |
| پیچیدگی نصب | کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | uvicorn، msgspec |
| منابع سخت‌افزاری | مشابه FastAPI |
| License | MIT — نیازمند تأیید |
| مستندات | https://docs.litestar.dev |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیهٔ Layer 2**: **FastAPI (Option 1)**. اعتبارسنجی Pydantic مستقیماً به Schema ورودی Upload/Query نگاشت می‌شود و کد اعتبارسنجی دستی را کاهش می‌دهد. مستندات OpenAPI خودکار در پذیرش POC قابل استفاده است. محدودیت `BackgroundTasks` دقیقاً با Worker چرخشی PostgreSQL در Layer 3 جبران می‌شود.

---

### Layer 3 — Domain & Orchestration Layer

**مسئولیت**: ماشین حالت Document Lifecycle، Idempotency، Retry، کنترل Activation، Reconciliation ساده.

**تصمیم ساده‌سازی**:
- **بدون Message Broker**. جدول `processing_jobs` در PostgreSQL، یک Worker مستقل Python که ردیف‌های `status = pending` را چرخشی پردازش می‌کند.
- **Activation**: سند پس از تکمیل ایندکس و عبور از `index_validation` به‌صورت خودکار `active` می‌شود. بدون گردش‌کار تأیید.
- **Idempotency**: کلید SHA-256 فایل؛ آپلود تکراری `document_id` موجود را برمی‌گرداند.
- **Retry**: Worker در صورت شکست `attempt_count` را افزایش می‌دهد؛ پس از آستانه `failed` می‌شود و دلیل در لاگ ثبت می‌شود.

#### Option 1 — Python + PostgreSQL Polling Worker (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | قابل اعمال نیست (کد خودساخته؛ نسخهٔ psycopg یا SQLAlchemy نیازمند تأیید) |
| نقش دقیق | ماشین حالت: `uploaded → validated → parsed → chunked → indexed → active/failed` |
| قابلیت‌های POC | Idempotency (قید یکتایی SHA-256)، Retry (`attempt_count`)، API وضعیت، اسکریپت Reconciliation دستی |
| مزایا | صفر زیرساخت اضافی، اشکال‌زدایی شهودی، سازگاری حالت با تراکنش PostgreSQL |
| محدودیت‌ها | فاصلهٔ چرخش تأخیر ایجاد می‌کند (۲–۵ ثانیه در POC قابل قبول)؛ برای هم‌زمانی بالا مناسب نیست (در POC بی‌ربط) |
| پیچیدگی نصب | بسیار کم (فقط Python + درایور DB) |
| آفلاین | کاملاً possible |
| وابستگی‌ها | psycopg2-binary یا asyncpg، SQLAlchemy (اختیاری) |
| منابع سخت‌افزاری | فرایند Worker ~۵۰–۱۵۰ مگابایت RAM |
| License | کد خودساخته (بدون مشکل License)؛ License کتابخانه‌های وابسته نیازمند تأیید |
| مستندات | قابل اعمال نیست (پیاده‌سازی داخلی) |

#### Option 2 — Celery + Redis (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | Celery نیازمند تأیید؛ Redis نیازمند تأیید |
| نقش دقیق | صف وظایف توزیع‌شده، جایگزین چرخش |
| قابلیت‌های POC | Retry داخلی، پیگیری وضعیت وظیفه، صف اولویت |
| مزایا | مکانیزم Retry بالغ، مدیریت بصری (Flower) |
| محدودیت‌ها | **افزودن کانتینر اضافی (Redis)**، افزایش حافظه و سطح عملیات؛ در سناریوی ۲–۳ کاربر POC سود محدود |
| پیچیدگی نصب | متوسط |
| آفلاین | possible (انتقال تصویر Redis آفلاین) |
| وابستگی‌ها | Celery، Redis، kombu |
| منابع سخت‌افزاری | Redis ~۵۰–۱۰۰ مگابایت + Celery worker ~۲۰۰ مگابایت |
| License | BSD (Celery)؛ RSALv2/SSPL (Redis نسخهٔ جدید — نیازمند تأیید) |
| مستندات | https://docs.celeryq.dev |

#### Option 3 — APScheduler (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | وظایف زمان‌بندی‌شده درون‌فرایندی، چرخش DB یا فراخوانی مستقیم پردازش |
| قابلیت‌های POC | زمان‌بندی ساده، قابل اجرا در همان فرایند API |
| مزایا | بدون فرایند Worker مستقل، پیکربندی ساده |
| محدودیت‌ها | هم‌فرایندی با API یعنی پردازش مسدودکننده یا نیاز به Thread/Process Pool؛ مدیریت وضعیت کمتر شفاف از چرخش DB؛ پس از Restart وظایف درون حافظه از بین می‌روند |
| پیچیدگی نصب | بسیار کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | اختیاری SQLAlchemyJobStore |
| منابع سخت‌افزاری | تقریباً ناچیز |
| License | MIT — نیازمند تأیید |
| مستندات | https://apscheduler.readthedocs.io |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیهٔ Layer 3**: **Python + PostgreSQL Polling Worker (Option 1)**. Master Prompt صراحتاً راهکارهای سادهٔ مبتنی بر Python و PostgreSQL را توصیه کرده و Message Broker را فقط در صورت ضرورت مجاز دانسته است. در مقیاس POC، یک اسکریپت Worker حدود ۵۰ خطی + جدول وضعیت PostgreSQL نیاز Idempotency و Retry را برآورده می‌کند و هیچ کانتینری اضافه نمی‌کند. وابستگی Redis در Celery برای POC تک‌گره overengineering است.

---

### Layer 4 — Document Ingestion & Processing Layer

**مسئولیت**: دریافت DOCX/XLSX، بررسی فرمت، استخراج متن و ساختار، استخراج جداول و اطلاعات Excel، حفظ اطلاعات Sheet/Row/Column/Cell، استخراج Heading، Normalization (شامل فارسی)، Chunking، تولید Canonical Chunks، ثبت Metadata و Processing Provenance.

**تصمیم ساده‌سازی**:
- **Word**: `python-docx` برای استخراج پاراگراف و استایل Heading؛ یا Docling در صورت آماده‌سازی offline wheel و مدل.
- **Excel**: `openpyxl` برای خواندن sheet/row/column/cell با حفظ `sheet_name`، `row_range`، `column_range` به‌عنوان Chunk Metadata.
- **نرمال‌سازی فارسی**: یک ماژول تابع Python داخل Ingestion (نه سرویس جدا): یکسان‌سازی Arabic/Persian (`ي→ی`، `ك→ک`)، مدیریت ZWNJ، یکسان‌سازی ارقام، حذف tatweel. متن RTL چیدمان خاص نمی‌خواهد (RAG به محتوای متنی توجه دارد).
- **Chunking**: تفکیک بر اساس پاراگراف/بلوک جدول، اندازهٔ هدف ۵۱۲–۱۰۲۴ توکن (تقریب کاراکتری)، overlap ۱۰–۱۵٪.

#### Option 1 — python-docx + openpyxl (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | هر دو نیازمند تأیید (صفحهٔ رسمی PyPI) |
| نقش دقیق | Word: استخراج پاراگراف، استایل، جدول؛ Excel: پیمایش sheet/cell، خواندن مقدار فرمول (`data_only=True`) |
| قابلیت‌های POC | سبک، خالص Python، بدون فرایند خارجی، کاملاً آفلاین |
| مزایا | مصرف منابع بسیار کم، نصب ساده، پشتیبانی پایدار از DOCX/XLSX؛ کنترل کامل توسعه‌دهنده بر منطق استخراج |
| محدودیت‌ها | چیدمان پیچیده (TextBox، Header/Footer به‌صورت پیش‌فرض رد می‌شود)؛ Excel اطلاعات استایل را حفظ نمی‌کند؛ بدون OCR داخلی (POC نیاز ندارد) |
| پیچیدگی نصب | بسیار کم (pip) |
| آفلاین | کاملاً آفلاین |
| وابستگی‌ها | lxml (وابستهٔ docx)، et_xmlfile (وابستهٔ openpyxl) |
| منابع سخت‌افزاری | ناچیز (<۵۰ مگابایت اضافی) |
| License | MIT — نیازمند تأیید |
| مستندات | https://python-docx.readthedocs.io ؛ https://openpyxl.readthedocs.io |

#### Option 2 — Docling (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | **2.149.0** (منبع دست‌دوم PyPI، 2026-11-07)؛ نیازمند تأیید نهایی از PyPI رسمی |
| نقش دقیق | پارس یکپارچهٔ سند: استخراج ساختار DOCX، تشخیص جدول، خروجی Markdown/JSON |
| قابلیت‌های POC | ساختار `DoclingDocument` داخلی، حفظ بهتر ساختار جدول، کمک به chunking |
| مزایا | نگهداری IBM، برای چیدمان پیچیدهٔ DOCX بهتر از python-docx؛ خروجی ساختارمند |
| محدودیت‌ها | **وابستگی سنگین** (ممکن است PyTorch یا مدل ML برای تحلیل چیدمان جلب کند)، حجم wheel آلفلاین بزرگ؛ آماده‌سازی اولیه نیازمند جمع‌آوری وابستگی در محیط متصل به شبکه؛ برای DOCX ساده ممکن است «به‌کار بردن چکش برای شکستن گردو» باشد |
| پیچیدگی نصب | متوسط (زنجیرهٔ وابستگی پیچیده، نیاز به `pip download` کامل wheel آفلاین) |
| آفلاین | از نظر تئوری possible اما **نیازمند آزمون**: تأیید اینکه مدل یا سرویس خارجی اجباری دانلود نمی‌کند |
| وابستگی‌ها | نیازمند تأیید از `pyproject.toml` رسمی (احتمالاً docling-core، Pillow، torch) |
| منابع سخت‌افزاری | هنگام پارس احتمالاً ~۵۰۰ مگابایت–۱ گیگابایت RAM (بسته به بارگذاری مدل) |
| License | MIT (نیازمند تأیید) |
| مستندات | https://github.com/DS4SD/docling |

#### Option 3 — Unstructured (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | **0.27.5** (منبع دست‌دوم GitHub Release)؛ نیازمند تأیید رسمی |
| نقش دقیق | رابط یکپارچهٔ `partition_docx` / `partition_xlsx` برای استخراج عناصر (Title، NarrativeText، Table و غیره) |
| قابلیت‌های POC | برچسب‌گذاری نوع عنصر، خروجی HTML جدول، قابل ترکیب با توابع chunking |
| مزایا | API یکپارچه برای چند فرمت، جامعهٔ فعال، طبقه‌بندی عناصر مفید برای RAG |
| محدودیت‌ها | وابستگی آلفلاین سنگین (شامل داده‌های nltk، احتمالاً دانلود مدل)؛ پشتیبانی XLSX کم‌دقت‌تر از openpyxl؛ در نیاز سادهٔ DOCX/XLSX POC لایهٔ انتزاع اضافه می‌کند |
| پیچیدگی نصب | متوسط |
| آفلاین | possible اما نیاز به پیش‌دانلود داده‌های nltk و مدل‌های spacy (در صورت استفاده)؛ **نیازمند آزمون** |
| وابستگی‌ها | نیازمند تأیید از فایل وابستگی رسمی |
| منابع سخت‌افزاری | ~۳۰۰–۸۰۰ مگابایت RAM (بسته به pipeline) |
| License | Apache 2.0 — نیازمند تأیید |
| مستندات | https://docs.unstructured.io |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیٔ Layer 4**: **python-docx + openpyxl (Option 1)**. ورودی POC صراحتاً به DOCX و XLSX محدود است و هدف «اثبات امکان‌پذیری Document Intelligence» است، نه پردازش چیدمان پیچیده. python-docx و openpyxl منطق استخراج را کاملاً شفاف می‌کنند و هر فیلد Chunk Schema (سطح Heading، محدودهٔ Sheet/Row) قابل کنترل دقیق است. نصب آلفلاین فقط دو wheel خالص Python است، بدون ریسک دانلود مدل. Docling به‌عنوان Option 2 حفظ می‌شود؛ اگر در آزمون مشخص شد python-docx برای DOCX هدف کافی نیست، می‌توان در پایان POC پیاده‌سازی داخلی Ingestion را جایگزین کرد بدون تأثیر بر رابط‌های بالادست و پایین‌دست.

---

### Layer 5 — AI & Model Runtime Layer

**مسئولیت**: اجرای محلی LLM، تولید Embedding، بارگذاری مدل، مدیریت نسخهٔ مدل، پشتیبانی از مدل‌های Quantized.

**تصمیم ساده‌سازی**: یک Runtime برای LLM و یک Runtime (یا همان Runtime) برای Embedding. مدیریت نسخهٔ مدل با نام‌گذاری پوشهٔ artifact و ثبت hash در PostgreSQL. بارگذاری مدل در startup و نگهداری در حافظه.

#### Option 1 — Ollama (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید از صفحهٔ رسمی Release |
| نقش دقیق | اجرای LLM و Embedding با API سازگار با OpenAI؛ مدیریت خودکار مدل‌ها |
| قابلیت‌های POC | پشتیبانی CPU و GPU، مدل‌های quantized (GGUF)، API ساده، امکان `ollama pull` از فایل محلی |
| مزایا | نصب بسیار ساده، مدیریت مدل داخلی، سازگاری با اکثر مدل‌های open |
| محدودیت‌ها | کنترل کم روی پارامترهای پیشرفتهٔ inference؛ برای embedding گزینه‌های محدودتر از Sentence Transformers |
| پیچیدگی نصب | بسیار کم (باینری + تصویر Docker) |
| آفلاین | کاملاً آفلاین؛ مدل‌ها را می‌توان به‌صورت فایل GGUF منتقل کرد |
| وابستگی‌ها | بدون وابستگی Python؛ نیاز به درایور GPU اختیاری |
| منابع سخت‌افزاری | CPU: ۸–۱۶ گیگابایت RAM برای مدل ۷B quantized؛ GPU: ۶–۸ گیگابایت VRAM |
| License | MIT (نیازمند تأیید رسمی) |
| مستندات | https://github.com/ollama/ollama |

#### Option 2 — llama.cpp (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | inference خالص GGUF با کنترل کامل پارامترها؛ سرور HTTP داخلی |
| قابلیت‌های POC | quantized، CPU-first، پشتیبانی GPU اختیاری، embedding با `llama-embedding` |
| مزایا | سبک‌ترین گزینه، کنترل کامل، بدون daemon اضافی |
| محدودیت‌ها | مدیریت مدل دستی؛ API کمتر انتزاعی؛ برای embedding باید جداگانه تنظیم شود |
| پیچیدگی نصب | کم–متوسط (کامپایل یا باینری آماده) |
| آفلاین | کاملاً possible |
| وابستگی‌ها | کامپایلر C++ برای build؛ یا باینری از پیش ساخته |
| منابع سخت‌افزاری | مشابه Ollama، کمی کمتر |
| License | MIT (نیازمند تأیید) |
| مستندات | https://github.com/ggerganov/llama.cpp |

#### Option 3 — vLLM (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | inference با throughput بالا، PagedAttention |
| قابلیت‌های POC | API سازگار با OpenAI، پشتیبانی embedding |
| مزایا | عملکرد بالا در هم‌زمانی، پشتیبانی مدل‌های بزرگ‌تر |
| محدودیت‌ها | **عملاً نیازمند GPU با VRAM کافی**؛ برای ۲–۳ کاربر POC overkill؛ نصب سنگین (PyTorch + CUDA) |
| پیچیدگی نصب | بالا |
| آفلاین | possible اما نیازمند wheelهای CUDA و هماهنگی نسخه |
| وابستگی‌ها | PyTorch، CUDA Toolkit، transformers |
| منابع سخت‌افزاری | حداقل ۱۶ گیگابایت VRAM توصیه‌شده |
| License | Apache 2.0 (نیازمند تأیید) |
| مستندات | https://docs.vllm.ai |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیٔ Layer 5**: **Ollama (Option 1)** برای LLM و **Sentence Transformers (به‌عنوان کتابخانه در همان فرایند Python)** برای Embedding. Ollama ساده‌ترین مسیر آفلاین برای LLM است؛ Sentence Transformers برای embedding کنترل بیشتری روی مدل چندزبانه می‌دهد و نیازی به سرویس جدا ندارد. vLLM فقط در صورت وجود GPU قوی و نیاز به throughput بالا در آینده بررسی شود.

---

### Layer 6 — Retrieval & Search Layer

**مسئولیت**: Vector Search، BM25، Hybrid، RRF، Metadata Filtering، Deduplication، ACL Revalidation، Reranking اختیاری، Context Assembly.

**تصمیم ساده‌سازی**: Reranking حذف می‌شود (یا یک تابع سادهٔ امتیازدهی). RRF به‌صورت تابع خالص Python مستقل پیاده‌سازی می‌شود، نه وابسته به فریم‌ورک. ACL Revalidation **قبل از** Context Assembly و اجباری است.

#### Option 1 — Qdrant + OpenSearch + RRF دستی (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | Qdrant نیازمند تأیید؛ OpenSearch نیازمند تأیید |
| نقش دقیق | Qdrant: vector search؛ OpenSearch: BM25؛ RRF: تابع Python |
| قابلیت‌های POC | فیلتر metadata در Qdrant، جستجوی lexical در OpenSearch، ادغام نتایج با RRF |
| مزایا | مطابق معماری مرجع؛ هر دو قابل اجرا در Docker؛ RRF شفاف و قابل تست |
| محدودیت‌ها | دو سرویس جداگانه = مصرف منابع بیشتر؛ OpenSearch سنگین‌تر از یک موتور سبک |
| پیچیدگی نصب | متوسط |
| آفلاین | کاملاً possible با تصاویر آلفلاین |
| وابستگی‌ها | Java (برای OpenSearch)، Python client برای هر دو |
| منابع سخت‌افزاری | OpenSearch: ۲–۴ گیگابایت RAM؛ Qdrant: ۵۰۰ مگابایت–۱ گیگابایت |
| License | Qdrant: Apache 2.0؛ OpenSearch: Apache 2.0 (نیازمند تأیید) |
| مستندات | https://qdrant.tech/documentation ؛ https://opensearch.org/docs |

#### Option 2 — Qdrant تنها با Sparse Vector (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | Qdrant هم dense و هم sparse vector را پشتیبانی می‌کند؛ می‌توان BM25-like را با sparse vector تقریب زد |
| قابلیت‌های POC | یک سرویس به‌جای دو سرویس؛ hybrid search داخلی |
| مزایا | کاهش مصرف منابع و پیچیدگی؛ یک index به‌جای دو |
| محدودیت‌ها | **انحراف از معماری مرجع** (OpenSearch به‌عنوان lexical index حذف می‌شود)؛ کیفیت BM25 واقعی را ندارد |
| پیچیدگی نصب | کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | فقط Qdrant |
| منابع سخت‌افزاری | ~۱ گیگابایت RAM |
| License | Apache 2.0 |
| مستندات | https://qdrant.tech/documentation |

#### Option 3 — PostgreSQL + pgvector + tsvector (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | pgvector نیازمند تأیید |
| نقش دقیق | vector search با pgvector؛ BM25-like با `tsvector` |
| قابلیت‌های POC | یک دیتابیس به‌جای سه سرویس |
| مزایا | کاهش شدید تعداد سرویس‌ها؛ تراکنش یکپارچه با metadata |
| محدودیت‌ها | **انحراف کامل از معماری مرجع**؛ عملکرد vector در مقیاس بزرگ ضعیف‌تر؛ ACL و vector در یک جا = کوپلینگ بیشتر |
| پیچیدگی نصب | کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | افزونهٔ pgvector |
| منابع سخت‌افزاری | مشابه PostgreSQL |
| License | PostgreSQL License؛ pgvector: PostgreSQL License |
| مستندات | https://github.com/pgvector/pgvector |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیٔ Layer 6**: **Qdrant + OpenSearch + RRF دستی (Option 1)**. Master Prompt صراحتاً Qdrant و OpenSearch را به‌عنوان فناوری‌های پایه تعیین کرده و «هیچ فناوری جایگزینی را بدون مقایسه و توضیح فنی جایگزین فناوری‌های پایه نکنید». گزینه‌های ۲ و ۳ انحراف از baseline هستند و فقط به‌عنوان trade-off ثبت می‌شوند. RRF به‌صورت تابع خالص Python پیاده‌سازی می‌شود تا وابسته به فریم‌ورک نباشد.

---

### Layer 7 — Data & Persistence Layer

**مسئولیت**: نگهداری اسناد اصلی، محتوای استخراج‌شده، Canonical Chunks، Metadata، Document Versioning، Processing State، ACL، Lineage، Index Registry.

**تفکیک مسئولیت (غیرقابل تغییر طبق Master Prompt)**:
- MinIO: Canonical Artifacts و Chunks
- PostgreSQL: Operational Metadata، Workflow State، ACL، Lineage
- Qdrant: Derived Vector Index (قابل بازسازی)
- OpenSearch: Derived Lexical Index (قابل بازسازی)

#### Option 1 — MinIO + PostgreSQL + Qdrant + OpenSearch (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | MinIO Server نیازمند تأیید؛ PostgreSQL نیازمند تأیید |
| نقش دقیق | مطابق baseline |
| قابلیت‌های POC | bucket/prefix برای original/extracted/chunks؛ جدول‌های documents/jobs/acl/lineage در PG |
| مزایا | حفظ کامل تفکیک مسئولیت؛ indexها قابل rebuild |
| محدودیت‌ها | چهار سرویس persistence = مصرف منابع و پیچیدگی عملیاتی |
| پیچیدگی نصب | متوسط |
| آفلاین | کاملاً possible |
| وابستگی‌ها | Docker images |
| منابع سخت‌افزاری | MinIO: ۵۰۰ مگابایت–۱ گیگابایت؛ PostgreSQL: ۵۰۰ مگابایت–۱ گیگابایت |
| License | MinIO: AGPL-3.0 (نیازمند تأیید — این License برای استفادهٔ تجاری حساس است)؛ PostgreSQL: PostgreSQL License |
| مستندات | https://min.io/docs ؛ https://www.postgresql.org/docs |

#### Option 2 — MinIO + PostgreSQL + Qdrant (حذف OpenSearch)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | حذف lexical index مستقل؛ تکیه بر sparse vector در Qdrant |
| قابلیت‌های POC | کاهش یک سرویس |
| مزایا | منابع کمتر |
| محدودیت‌ها | **انحراف از baseline**؛ از دست دادن BM25 واقعی |
| پیچیدگی نصب | کم–متوسط |
| آفلاین | کاملاً possible |
| وابستگی‌ها | Docker images |
| منابع سخت‌افزاری | ~۲ گیگابایت کمتر از گزینه ۱ |
| License | مشابه |
| مستندات | مشابه |

#### Option 3 — MinIO + PostgreSQL تنها (بدون Qdrant و OpenSearch)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | ذخیره‌سازی خام؛ جستجو در POC حذف می‌شود |
| قابلیت‌های POC | فقط ingestion و persistence |
| مزایا | حداقل سرویس |
| محدودیت‌ها | **POC را از RAG به یک آرشیو سند تبدیل می‌کند** — ناقض هدف پروژه |
| پیچیدگی نصب | کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | Docker images |
| منابع سخت‌افزاری | حداقل |
| License | مشابه |
| مستندات | مشابه |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیٔ Layer 7**: **گزینه ۱ (MinIO + PostgreSQL + Qdrant + OpenSearch)**. Master Prompt تفکیک مسئولیت را الزام کرده و indexها باید از Canonical Data بازسازی شوند. گزینه‌های ۲ و ۳ انحراف از baseline هستند و فقط به‌عنوان trade-off ثبت می‌شوند. **هشدار License MinIO (AGPL-3.0)** باید در مرحلهٔ بررسی حقوقی جداگانه تأیید شود؛ در POC داخلی معمولاً مشکلی ایجاد نمی‌کند اما برای آیندهٔ سازمانی حتماً باید بررسی شود.

---

### Layer 8 — Security & Identity Layer

**مسئولیت**: Authentication، Authorization، User/Group Management، Document-Level ACL، Service Access Control، Secret Management، Network Isolation، Secure Configuration.

**قید کلیدی**: PostgreSQL مرجع معتبر ACL است. در صورت عدم امکان تأیید مجوز، **Fail Closed**. محتوای غیرمjaz نباید وارد Reranking / Context Assembly / LLM Context شود.

**تصمیم ساده‌سازی**: بدون Keycloak یا Casdoor. Auth با Dependency Injection در FastAPI: جدول کاربران در PostgreSQL، هش رمز با `argon2-cffi`، Session با Cookie امضاشده یا JWT کوتاه‌مدت. Document-Level ACL در جدول `document_acl` (`document_id`، `user_id`/`group_id`، `permission`). بازبینی اجباری ACL پس از بازیابی و قبل از Context Assembly. Fail Closed: هر خطا یا timeout در کوئری ACL، نتیجهٔ خالی برمی‌گرداند نه دور زدن.

#### Option 1 — احراز هویت خودساخته + PostgreSQL ACL (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | کتابخانه‌های وابسته نیازمند تأیید (argon2-cffi، python-jose یا itsdangerous) |
| نقش دقیق | جدول کاربران، هش رمز، صدور و اعتبارسنجی Session/Token؛ جدول ACL و کوئری |
| قابلیت‌های POC | ۲–۳ کاربر، دو سطح دسترسی (admin/user)، کنترل خواندن/نوشتن در سطح سند، Fail Closed |
| مزایا | صفر کانتینر اضافی، کاملاً قابل کنترل، ادغام مستقیم با مرجع PostgreSQL، بدون منحنی یادگیری IdP |
| محدودیت‌ها | بدون قابلیت SSO/OIDC (POC نیاز ندارد)؛ عملکردهایی مانند بازنشانی رمز باید دستی نوشته شود |
| پیچیدگی نصب | بسیار کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | FastAPI (قبلاً در L2)، argon2-cffi |
| منابع سخت‌افزاری | ناچیز |
| License | کد خودساخته |
| مستندات | قابل اعمال نیست (پیاده‌سازی داخلی) |

#### Option 2 — Casdoor (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | v4.17 (برچسب بررسی اکتبر ۲۰۲۶ در سند مقایسه) |
| نقش دقیق | سرویس IAM مستقل با OIDC/OAuth 2.0، مدیریت کاربر، نقش |
| قابلیت‌های POC | MCP server داخلی، پشتیبانی چند دیتابیس (شامل PostgreSQL)، License از نوع Apache-2.0 |
| مزایا | کامل بودن قابلیت‌ها، رابط چینی، License دوستانه |
| محدودیت‌ها | **افزودن کانتینر و سطح عملیات اضافی**؛ در سناریوی ۲–۳ کاربر POC قابلیت‌ها اضافی است؛ ACL همچنان باید در لایهٔ اپلیکیشن پیاده‌سازی شود (Casdoor ACL در سطح سند را مدیریت نمی‌کند) |
| پیچیدگی نصب | متوسط |
| آفلاین | possible (نیاز به image و DB آلفلاین) |
| وابستگی‌ها | PostgreSQL یا MySQL، Go runtime (داخل کانتینر) |
| منابع سخت‌افزاری | ~۲۰۰–۴۰۰ مگابایت RAM |
| License | Apache-2.0 |
| مستندات | https://casdoor.ai |

#### Option 3 — Keycloak (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | 26.8 (برچسب بررسی اکتبر ۲۰۲۶ در سند مقایسه) |
| نقش دقیق | IdP سازمانی، OIDC/SAML، User Federation، نقش‌های دقیق |
| قابلیت‌های POC | بالغ و پایدار، بزرگ‌ترین اکوسیستم، Apache-2.0 |
| مزایا | آشنایی بالا هنگام مهاجرت آینده به محیط سازمانی |
| محدودیت‌ها | **برای POC به‌شدت overengineering**: مصرف RAM بالای Java/Quarkus (~۱ گیگابایت+)، پیکربندی پیچیده، startup کند؛ بدون MCP داخلی؛ ACL در سطح سند همچنان باید خودساخته باشد |
| پیچیدگی نصب | بالا (از دید POC) |
| آفلاین | possible اما حجم image بزرگ |
| وابستگی‌ها | Java، PostgreSQL/MySQL |
| منابع سخت‌افزاری | ~۱–۲ گیگابایت RAM |
| License | Apache-2.0 |
| مستندات | https://www.keycloak.org |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیٔ Layer 8**: **احراز هویت خودساخته + PostgreSQL ACL (Option 1)**. Master Prompt یک مدل ساده اما واقعی خواسته، نه IAM سازمانی. مرجع Document-Level ACL همان PostgreSQL است؛ افزودن IdP مستقل فقط کانتینر و پیکربندی اضافه می‌کند و کمکی به ACL نمی‌کند. Casdoor به‌عنوان Option 2 حفظ می‌شود؛ اگر POC در آینده به OIDC/SSO نیاز پیدا کرد، می‌توان ارائه‌دهندهٔ Auth را بدون تأثیر بر منطق ACL جایگزین کرد.

---

### Layer 9 — Monitoring, Logging & Evaluation Layer

**مسئولیت**: Application Logging، Processing Logs، Error Tracking، Retrieval Tracing، LLM Performance Measurement، Resource Monitoring، RAG Quality Evaluation، Test Dataset Management.

**تصمیم ساده‌سازی**: بدون Prometheus/Grafana. **Python `logging` (ساختاریافته JSON)** + **جدول `audit_log` در PostgreSQL** برای ثبت رخدادهای کلیدی (پرسش، بازیابی، رد ACL، زمان فراخوانی LLM، خطا). ارزیابی کیفیت RAG با **Ragas** به‌صورت کتابخانهٔ آلفلاین، بدون سرویس مستقل. ردیابی بازیابی در PostgreSQL: برای هر پرسش `query_id`، `chunk_ids` بازیابی‌شده، امتیاز RRF، تعداد توکن ورودی/خروجی LLM، زمان صرف‌شده.

#### Option 1 — Python logging + PostgreSQL audit_log + Ragas (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | Ragas نیازمند تأیید (PyPI رسمی) |
| نقش دقیق | Logging: رخدادهای ساختاریافته؛ audit_log: جدول حسابرسی تغییرناپذیر؛ Ragas: شاخص‌های آفلاین RAG (faithfulness، context precision/recall، answer relevancy) |
| قابلیت‌های POC | ردیابی پرسش، ثبت خطا، اندازه‌گیری خط پایهٔ کیفیت RAG، بدون کانتینر اضافی |
| مزایا | صفر زیرساخت اضافی، Ragas مخصوص RAG و Apache-2.0، ارزیابی قابل اسکریپت‌نویسی |
| محدودیت‌ها | بدون Dashboard زمان‌واقعی؛ Ragas نیاز به آماده‌سازی دستی مجموعه‌دادهٔ آزمون دارد؛ کوئری لاگ با SQL |
| پیچیدگی نصب | بسیار کم |
| آفلاین | کاملاً possible |
| وابستگی‌ها | Python logging (کتابخانهٔ استاندارد)، Ragas (pip) |
| منابع سخت‌افزاری | ناچیز (در زمان ارزیابی CPU کوتاه‌مدت) |
| License | Ragas: Apache-2.0 |
| مستندات | https://docs.ragas.io |

#### Option 2 — Langfuse (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید (GitHub Release) |
| نقش دقیق | ردیابی کل زنجیره، مدیریت prompt، ارزیابی LLM-as-judge، replay نشست |
| قابلیت‌های POC | Self-hosted Docker، License MIT (ماژول‌های هسته پس از ژوئن ۲۰۲۵)، سازگار با OTel |
| مزایا | قابلیت‌های کامل، مناسب توسعهٔ آینده، تجسم خوب |
| محدودیت‌ها | **افزودن کانتینر اضافی (Langfuse server + احتمالاً ClickHouse/Redis)**؛ در سناریوی ۲–۳ کاربر POC هزینهٔ عملیات بالا |
| پیچیدگی نصب | متوسط–بالا |
| آفلاین | possible اما وابسته به اجزای متعدد |
| وابستگی‌ها | PostgreSQL، اختیاری ClickHouse/Redis |
| منابع سخت‌افزاری | ~۵۰۰ مگابایت–۱ گیگابایت RAM |
| License | MIT (ماژول هسته) |
| مستندات | https://langfuse.com/docs |

#### Option 3 — Arize Phoenix (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | ارزیابی relevancy RAG، instrumentation خودکار OpenInference، تشخیص drift |
| قابلیت‌های POC | Self-hosted، توانایی قوی ارزیابی تخصصی RAG |
| مزایا | عمق ارزیابی RAG بیشتر از observability عمومی |
| محدودیت‌ها | Elastic License 2.0 (استاندارد OSI نیست، نیازمند ارزیابی انطباق)؛ بدون prompt management؛ کانتینر اضافی |
| پیچیدگی نصب | متوسط |
| آفلاین | possible |
| وابستگی‌ها | Python، احتمالاً server جداگانه |
| منابع سخت‌افزاری | ~۳۰۰–۶۰۰ مگابایت RAM |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیٔ Layer 9**: **Python logging + PostgreSQL audit_log + Ragas (Option 1)**. Master Prompt صراحتاً «سبک و کم‌هزینه» را در اولویت قرار داده. هدف ارزیابی POC ایجاد خط پایه است، نه داشبورد پایش زمان‌واقعی. شاخص‌های تخصصی RAG در Ragas (faithfulness، context recall و غیره) مستقیماً با نیازهای کمی Acceptance Criteria مطابقت دارند. Langfuse و Phoenix به‌عنوان گزینه‌های آینده حفظ می‌شوند.

---

### Layer 10 — Deployment & Offline Operations Layer

**مسئولیت**: Linux Host، Docker Engine، Docker Compose، Container Images، Offline Image Transfer، Python Dependencies، Model Artifact Management، Configuration، Backup، Restore، Offline Update.

**تصمیم ساده‌سازی**: تک‌گره Linux + Docker Compose. انتقال تصاویر آلفلاین با `docker save`/`docker load`. وابستگی‌های Python با `pip download` در محیط متصل به شبکه و نصب آلفلاین با `pip install --no-index --find-links`. artifact مدل (GGUF، وزن‌های Sentence Transformers) پس از دانلود در محیط متصل به شبکه به‌صورت فایل منتقل و SHA-256 آن تأیید می‌شود. پیکربندی با فایل `.env` + متغیرهای محیطی Docker Compose؛ مقادیر حساس hardcode نمی‌شوند.

#### Option 1 — Docker Compose + docker save/load + pip download (Best Fit for POC)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | Docker Engine / Compose نیازمند تأیید (Release رسمی) |
| نقش دقیق | هماهنگی سرویس، انتقال تصویر آلفلاین، نصب آفلاین وابستگی Python |
| قابلیت‌های POC | شروع/توقف با یک فرمان، ماندگاری volume، health check، تأیید startup آفلاین |
| مزایا | صفر ابزار اضافی، راهکار تعیین‌شده در Master Prompt، فهرست تصاویر قابل حسابرسی |
| محدودیت‌ها | بدون registry داخلی؛ به‌روزرسانی تصویر نیاز به save/load دستی؛ نیاز به ساخت ساختار پوشهٔ `offline-bundle` |
| پیچیدگی نصب | کم (خود نصب Docker) |
| آفلاین | کاملاً possible (هدف طراحی) |
| وابستگی‌ها | Docker Engine، Python pip |
| منابع سخت‌افزاری | Docker daemon ~۱۰۰–۲۰۰ مگابایت |
| License | Docker Engine: Apache-2.0 (moby)؛ Docker Desktop نیازمند توجه به مجوز تجاری |
| مستندات | https://docs.docker.com/compose |

#### Option 2 — Podman + docker-compose (Strong Alternative)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | Runtime کانتینر بدون daemon، پشتیبانی rootless |
| قابلیت‌های POC | سازگار با Compose YAML، save/load تصویر |
| مزایا | بدون daemon، امنیت rootless بهتر |
| محدودیت‌ها | سازگاری با Docker Compose گاهی تفاوت دارد؛ آشنایی کاربر POC ممکن است کمتر باشد |
| پیچیدگی نصب | کم–متوسط |
| آفلاین | completely possible |
| وابستگی‌ها | Podman، podman-compose |
| منابع سخت‌افزاری | مشابه Docker |
| License | Apache-2.0 |
| مستندات | https://podman.io |

#### Option 3 — containerd + nerdctl (Alternative with Specific Trade-offs)

| ویژگی | مقدار |
|---|---|
| نسخهٔ پایدار پیشنهادی | نیازمند تأیید |
| نقش دقیق | Runtime کانتینر سطح پایین، nerdctl رابط CLI |
| قابلیت‌های POC | سبک، سازگار با K8s runtime |
| مزایا | سبک‌ترین، بدون Docker daemon |
| محدودیت‌ها | **بدون معادل Compose** (نیاز به هماهنگی دستی یا اسکریپت؛ بهره‌وری توسعه POC پایین) |
| پیچیدگی نصب | متوسط |
| آفلاین | possible |
| وابستگی‌ها | containerd، nerdctl |
| منابع سخت‌افزاری | کمترین |
| License | Apache-2.0 |
| مستندات | https://github.com/containerd/nerdctl |

**رتبه‌بندی**: Option 1: Best Fit — Option 2: Strong Alternative — Option 3: Alternative with Specific Trade-offs

**توصیٔ Layer 10**: **Docker Compose + docker save/load + pip download (Option 1)**. Master Prompt صراحتاً Docker Compose را به‌عنوان گزینهٔ اول تعیین کرده. عملیات آفلاین POC باید قابل تکرار و مستندپذیر باشد؛ `docker save/load` + `pip download` مستقیم‌ترین و کم‌خطاترین روش است.

---

## ۶. ارزیابی مدل‌ها و پردازش زبان فارسی

### ۶.۱. Embedding — سه گزینه

| گزینه | پشتیبانی فارسی | اجرا محلی | حجم تقریبی | ملاحظات |
|---|---|---|---|---|
| **BAAI/bge-m3** | چندزبانه (شامل فارسی) | Sentence Transformers | ~۲.۲ گیگابایت | پشتیبانی dense/sparse/colbert، کیفیت بازیابی خوب |
| **intfloat/multilingual-e5-large** | چندزبانه (شامل فارسی) | Sentence Transformers | ~۲.۲ گیگابایت | بالغ و پایدار، استفادهٔ گستردهٔ جامعه |
| **sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2** | چندزبانه | Sentence Transformers | ~۴۷۰ مگابایت | سبک، مناسب POC با منابع محدود، کیفیت کمتر از دو گزینهٔ اول |

**توصیٔ Embedding**: **multilingual-e5-large** برای شروع (تعادل کیفیت و پایداری)؛ در صورت کمبود منابع به MiniLM؛ در صورت نیاز به کیفیت بالاتر به bge-m3.

### ۶.۲. LLM — سه گزینه (اجرا محلی، پشتیبانی فارسی)

| گزینه | توانایی فارسی | حجم پس از Quantize | روش اجرا | ملاحظات |
|---|---|---|---|---|
| **Qwen2.5-7B-Instruct** | چندزبانه، فارسی قابل قبول | ~۴–۵ گیگابایت (Q4) | Ollama / llama.cpp | انتشار Alibaba، دادهٔ آموزشی چندزبانهٔ گسترده‌تر |
| **Llama-3.1-8B-Instruct** | چندزبانه، فارسی متوسط | ~۵–۶ گیگابایت (Q4) | Ollama / llama.cpp | انتشار Meta، بالغ‌ترین اکوسیستم |
| **Mistral-7B-Instruct-v0.3** | چندزبانه، فارسی ضعیف‌تر | ~۴–۵ گیگابایت (Q4) | Ollama / llama.cpp | زبان‌های اروپایی قوی، فارسی نیازمند آزمون |

**توصیٔ LLM**: **Qwen2.5-7B-Instruct** (کوانتیزه Q4_K_M، فرمت Ollama). در سطح ۷B پوشش چندزبانهٔ گسترده‌تر و معمولاً پشتیبانی فارسی بهتر از Llama/Mistral هم‌مقیاس (نیازمند آزمون تجربی). در صورت ناکافی بودن کیفیت فارسی، گزینهٔ بعدی Qwen2.5-14B یا پذیرش تمرکز POC بر انگلیسی با پشتیبانی فارسی به‌عنوان قابلیت ثانویه.

### ۶.۳. نرمال‌سازی فارسی (داخل Ingestion Layer)

ماژول مستقل منطقی (نه سرویس جدا) شامل:
- یکسان‌سازی Arabic Yeh (`ي`) → Persian Yeh (`ی`)
- یکسان‌سازی Arabic Kaf (`ك`) → Persian Kaf (`ک`)
- مدیریت/نرمال‌سازی ZWNJ
- یکسان‌سازی ارقام (فارسی → ASCII یا برعکس)
- حذف tatweel (`ـ`)

---

## ۷. ماتریس انتخاب فناوری

| Layer | Option 1 | Option 2 | Option 3 | Recommended POC Tool | Selection Rationale |
|---|---|---|---|---|---|
| L1 Client | Streamlit | FastAPI+Jinja2/HTMX | Gradio | **Streamlit** | صفر فرانت‌اند، آفلاین ساده |
| L2 API | FastAPI | Flask | Litestar | **FastAPI** | Pydantic + OpenAPI |
| L3 Orchestration | Python+PG Worker | Celery+Redis | APScheduler | **Python+PG Worker** | بدون Broker، طبق Master Prompt |
| L4 Ingestion | python-docx+openpyxl | Docling | Unstructured | **python-docx+openpyxl** | کنترل Chunk Schema، سبک |
| L5 AI Runtime | Ollama+ST | llama.cpp | vLLM | **Ollama + Sentence Transformers** | ساده‌ترین مسیر آفلاین |
| L6 Retrieval | Qdrant+OpenSearch+RRF دستی | Qdrant sparse | PG+pgvector | **Qdrant+OpenSearch+RRF دستی** | حفظ baseline |
| L7 Persistence | MinIO+PG+Qdrant+OS | MinIO+PG+Qdrant | MinIO+PG | **MinIO+PG+Qdrant+OS** | حفظ baseline |
| L8 Security | احراز هویت خودساخته+PG ACL | Casdoor | Keycloak | **احراز هویت خودساخته+PG ACL** | ساده و واقعی، بدون بار IdP |
| L9 Monitoring | logging+audit+Ragas | Langfuse | Phoenix | **logging+audit+Ragas** | صفر کانتینر، تخصصی RAG |
| L10 Deployment | Docker Compose | Podman | containerd | **Docker Compose** | تعیین‌شده در Master Prompt |

---

## ۸. فهرست نهایی فناوری‌های منتخب

| ابزار | نقش | لایه | Container / Library | وابستگی‌های اصلی | وضعیت آفلاین |
|---|---|---|---|---|---|
| Streamlit | UI | L1 | Container (ادغام با API) | Python | کاملاً possible |
| FastAPI | API | L2 | Container (ادغام با UI) | Python | کاملاً possible |
| Python + PostgreSQL | Worker هماهنگی | L3 | Library/Script (همان کانتینر) | psycopg | کاملاً possible |
| python-docx + openpyxl | پارس سند | L4 | Library | Python | کاملاً possible |
| Ollama | استنتاج LLM | L5 | Container مستقل | Docker | کاملاً possible (انتقال فایل GGUF) |
| Sentence Transformers | Embedding | L5 | Library (همان کانتینر API) | PyTorch (CPU) | کاملاً possible (انتقال فایل مدل) |
| Qdrant | Vector Search | L6 | Container مستقل | Docker | کاملاً possible |
| OpenSearch | BM25 | L6 | Container مستقل | Java | کاملاً possible |
| MinIO | Object Storage | L7 | Container مستقل | Docker | کاملاً possible |
| PostgreSQL | Metadata/ACL | L7/L8 | Container مستقل | Docker | کاملاً possible |
| argon2-cffi | هش رمز | L8 | Library | Python | کاملاً possible |
| Ragas | ارزیابی RAG | L9 | Library (اجرای آلفلاین) | Python | کاملاً possible |
| Docker Compose | هماهنگی | L10 | Host Tool | Docker Engine | هدف طراحی |

**تعداد کانتینر در POC**: **۶ عدد** (앱، Ollama، Qdrant، OpenSearch، MinIO، PostgreSQL). Worker و Embedding Runtime داخل کانتینر `app` قرار می‌گیرند و کانتینر جداگانه ندارند. مطابق الزام Master Prompt برای «حداقل Deployable Unit».

---

## ۹. معماری فیزیکی استقرار

### نقشهٔ کانتینرها و شبکه

بر اساس تصمیمات قبلی، POC بر روی یک Linux Host با Docker Compose اجرا می‌شود. هدف: **حداقل تعداد کانتینر**، جداسازی منطقی شبکهٔ Frontend و Backend، و عدم دسترسی مستقیم کاربران به دیتابیس‌ها و Search Engineها.

#### جدول کانتینرها

| کانتینر | تصویر (پیشنهادی) | نقش | شبکه | پورت‌های منتشرشده (Host) | Volume |
|---|---|---|---|---|---|
| `app` | ساخت محلی (Python) | Streamlit UI + FastAPI + Worker + Sentence Transformers | `frontend` + `backend` | `8501:8501` (فقط Streamlit) | `app_models` (اختیاری) |
| `ollama` | `ollama/ollama` | LLM Inference | `backend` | بدون انتشار (فقط داخلی) | `ollama_models` |
| `qdrant` | `qdrant/qdrant` | Vector Search | `backend` | بدون انتشار | `qdrant_storage` |
| `opensearch` | `opensearchproject/opensearch` | BM25 Search | `backend` | بدون انتشار | `opensearch_data` |
| `minio` | `minio/minio` | Object Storage | `backend` | بدون انتشار (کنسول اختیاری) | `minio_data` |
| `postgres` | `postgres` | Metadata / ACL / State | `backend` | بدون انتشار | `postgres_data` |

**تعداد کانتینر: ۶ عدد.** این با الزام «حداقل Deployable Unit» مطابقت دارد.

#### منطق شبکه

Docker Compose به‌صورت پیش‌فرض یک شبکهٔ واحد می‌سازد که همهٔ سرویس‌ها به آن متصل می‌شوند. برای POC، **دو شبکهٔ نام‌دار** تعریف می‌شود:

- **`frontend`**: فقط کانتینر `app` به آن متصل است. کاربران فقط از طریق این شبکه به Streamlit دسترسی دارند.
- **`backend`**: کانتینرهای `app`، `ollama`، `qdrant`، `opensearch`، `minio`، `postgres` به آن متصل‌اند. تبادل داده بین لایهٔ اپلیکیشن و سرویس‌های داده فقط از این شبکه انجام می‌شود.

با این ساختار، `qdrant` و `opensearch` و `postgres` و `minio` **هیچ شبکه‌ای با `frontend` مشترک ندارند** و بنابراین از دید کاربران خارجی قابل دسترسی نیستند. این الگو دقیقاً همان چیزی است که Docker Docs به‌عنوان جداسازی سرویس‌ها با شبکه‌های مجزا توصیه می‌کند: سرویسی که شبکهٔ مشترک ندارد، نمی‌تواند با سرویس دیگر ارتباط برقرار کند.

**نکتهٔ امنیتی**: پورت‌های `qdrant`، `opensearch`، `minio` و `postgres` **در فایل Compose منتشر نمی‌شوند** (بدون `ports:`). یعنی حتی از خود Host هم از طریق `localhost:PORT` قابل دسترسی نیستند، مگر از داخل شبکهٔ `backend`. این از یک قاعدهٔ ساده پیروی می‌کند: اگر پورت روی `0.0.0.0` منتشر شود و Host IP عمومی داشته باشد، کانتینر مستقیماً در معرض اینترنت قرار می‌گیرد.

**نکتهٔ `internal: true`**: شبکه `backend` با `internal: true` تعریف شده است، بنابراین هیچ اتصالی به خارج ندارد. این برای POC آفلاین ایده‌آل است، زیرا اگر `app` به هر دو شبکه وصل باشد، ترافیک از `frontend` به `backend` از طریق `app` عبور می‌کند، نه مستقیماً. `internal: true` روی `backend` امن‌تر است و در POC توصیه می‌شود.

#### Volumeها

Volumeهای نام‌دار برای هر سرویس حالت‌دار (stateful) تعریف می‌شوند. مسیر داده‌ها مطابق مستندات رسمی هر تصویر است:

| Volume | مسیر داخل کانتینر | سرویس |
|---|---|---|
| `postgres_data` | `/var/lib/postgresql/data` | PostgreSQL |
| `qdrant_storage` | `/qdrant/storage` | Qdrant |
| `opensearch_data` | `/usr/share/opensearch/data` | OpenSearch |
| `minio_data` | `/data` | MinIO |
| `ollama_models` | `/root/.ollama` | Ollama |

Volumeهای نام‌دار با `docker compose up` به‌صورت خودکار ساخته می‌شوند و پس از Restart یا حذف کانتینر باقی می‌مانند. برای Ollama، نگهداری مدل‌ها در volume الزامی است؛ در غیر این صورت هر بار حذف کانتینر، گیگابایت‌ها دانلود مدل از دست می‌رود.

#### نمونهٔ ساختار `docker-compose.yml` (طرح کلی)

```yaml
services:
  app:
    build: ./app
    networks:
      - frontend
      - backend
    ports:
      - "8501:8501"
    depends_on:
      - postgres
      - qdrant
      - opensearch
      - minio
      - ollama

  ollama:
    image: ollama/ollama
    networks:
      - backend
    volumes:
      - ollama_models:/root/.ollama
    # در صورت وجود GPU: deploy.resources.reservations.devices

  qdrant:
    image: qdrant/qdrant
    networks:
      - backend
    volumes:
      - qdrant_storage:/qdrant/storage

  opensearch:
    image: opensearchproject/opensearch
    networks:
      - backend
    environment:
      - discovery.type=single-node
    volumes:
      - opensearch_data:/usr/share/opensearch/data

  minio:
    image: minio/minio
    command: server /data
    networks:
      - backend
    volumes:
      - minio_data:/data

  postgres:
    image: postgres
    networks:
      - backend
    volumes:
      - postgres_data:/var/lib/postgresql/data

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

volumes:
  postgres_data:
  qdrant_storage:
  opensearch_data:
  minio_data:
  ollama_models:
```

---

## ۱۰. جریان داده End-to-End

### Ingestion Flow

هدف: دریافت DOCX/XLSX، پردازش، ذخیرهٔ Canonical، ایندکس‌گذاری، و فعال‌سازی.

| مرحله | مسئول | خروجی / وضعیت |
|---|---|---|
| ۱. User Upload | Streamlit (`app`) | فایل در حافظهٔ موقت |
| ۲. File Validation | FastAPI (`app`) | بررسی پسوند `.docx`/`.xlsx` و MIME type |
| ۳. SHA-256 Hash | FastAPI (`app`) | هش فایل برای Idempotency |
| ۴. Duplicate Detection | PostgreSQL | اگر `content_hash` موجود باشد، `document_id` موجود برگردانده می‌شود؛ جریان متوقف |
| ۵. Original Persistence | MinIO (`backend`) | `original/{document_id}/{filename}` |
| ۶. Metadata Registration | PostgreSQL | ردیف در `documents` با وضعیت `uploaded` |
| ۷. Job Enqueue | PostgreSQL | ردیف در `processing_jobs` با وضعیت `pending` |
| ۸. Word/Excel Parsing | Worker (`app`) | `python-docx` برای DOCX؛ `openpyxl` برای XLSX |
| ۹. Text/Table Extraction | Worker (`app`) | پاراگراف‌ها با Heading؛ سلول‌های Excel با sheet/row/col |
| ۱۰. Normalization | Worker (`app`) | ماژول نرمال‌سازی فارسی + یکسان‌سازی Unicode |
| ۱۱. Chunking | Worker (`app`) | Chunkهای ۵۱۲–۱۰۲۴ توکن با overlap ۱۰–۱۵٪ |
| ۱۲. Canonical Chunk Persistence | MinIO + PostgreSQL | محتوا در MinIO (`chunks/{document_id}/`)؛ metadata در PG (`chunks` جدول) |
| ۱۳. Embedding Generation | Worker (`app`) | Sentence Transformers (`multilingual-e5-large`) |
| ۱۴. Qdrant Indexing | Qdrant (`backend`) | upsert vectors با `chunk_id` به‌عنوان point ID |
| ۱۵. OpenSearch Indexing | OpenSearch (`backend`) | index سند با فیلد `content` و `chunk_id` |
| ۱۶. Index Validation | Worker (`app`) | شمارش chunkهای ایندکس‌شده در Qdrant و OpenSearch و مقایسه با PG |
| ۱۷. Document Activation | PostgreSQL | `documents.status = 'active'`؛ `processing_jobs.status = 'completed'` |

**نکتهٔ Idempotency**: مرحلهٔ ۴ (Duplicate Detection) تضمین می‌کند که آپلود مجدد یک فایل، پردازش تکراری ایجاد نکند. کلید یکتایی `content_hash` در جدول `documents` تعریف می‌شود.

**نکتهٔ Retry**: اگر هر مرحله از ۸ تا ۱۶ شکست بخورد، Worker `processing_jobs.attempt_count` را افزایش می‌دهد. پس از ۳ تلاش ناموفق، وضعیت `failed` می‌شود و دلیل در `error_log` ثبت می‌شود.

### Query & RAG Flow

هدف: دریافت پرسش، بازیابی ترکیبی، بازبینی ACL، تولید پاسخ با LLM و ارجاع به منابع.

| مرحله | مسئول | جزئیات |
|---|---|---|
| ۱. User Authentication | FastAPI (`app`) | اعتبارسنجی Session/Token از Cookie |
| ۲. Query Submission | Streamlit → FastAPI | متن پرسش |
| ۳. Authorization Context Resolution | PostgreSQL (`backend`) | استخراج `user_id` و نقش‌ها از Session |
| ۴. Query Normalization | FastAPI (`app`) | همان تابع نرمال‌سازی فارسی Ingestion |
| ۵. Query Embedding | Sentence Transformers (`app`) | بردار پرسش |
| ۶. Vector Search | Qdrant (`backend`) | top-K (مثلاً ۲۰) با فیلتر `document_status = active` |
| ۷. BM25 Search | OpenSearch (`backend`) | top-K (مثلاً ۲۰) با فیلتر مشابه |
| ۸. RRF Fusion | FastAPI (`app`) | تابع Python خالص: `score = Σ 1/(k + rank_i)` با `k=60` |
| ۹. ACL Revalidation | PostgreSQL (`backend`) | **اجباری**: برای هر `chunk_id`، بررسی `document_acl`؛ حذف chunkهای غیرمجاز |
| ۱۰. Canonical Chunk Retrieval | MinIO (`backend`) | خواندن محتوای chunkهای مجاز از `chunks/` |
| ۱۱. (Optional Reranking) | FastAPI (`app`) | در POC حذف می‌شود؛ جای آن در کد به‌صورت تابع خالی نگه داشته می‌شود |
| ۱۲. Context Assembly | FastAPI (`app`) | چیدن chunkها بر اساس امتیاز RRF؛ محدودیت توکن |
| ۱۳. Local LLM Inference | Ollama (`backend`) | فراخوانی API با prompt شامل context |
| ۱۴. Response Generation | Ollama → FastAPI | متن پاسخ |
| ۱۵. Source Attribution | FastAPI (`app`) | نگاشت chunkهای استفاده‌شده به `document_id` و `location_meta` |
| ۱۶. Response Delivery | FastAPI → Streamlit | پاسخ + فهرست منابع |
| ۱۷. Audit Logging | PostgreSQL (`backend`) | ثبت `query_id`، `chunk_ids`، زمان LLM، توکن‌ها |

**نکتهٔ حیاتی ACL**: مرحلهٔ ۹ **قبل از** Context Assembly و LLM Inference اجرا می‌شود. اگر کوئری ACL با خطا مواجه شود یا timeout رخ دهد، نتیجهٔ آن **رد کامل** است (Fail Closed)، نه نادیده گرفتن. این با الزام Master Prompt مطابقت دارد: «هیچ محتوای غیرمجاز نباید وارد Reranking، Context Assembly یا LLM Context شود.»

**نکتهٔ RRF**: تابع RRF به‌صورت مستقل در `app` پیاده‌سازی می‌شود و به Qdrant یا OpenSearch وابسته نیست. ورودی آن دو لیست رتبه‌بندی‌شده و خروجی یک لیست ادغام‌شده است.

---

## ۱۱. استراتژی Offline / Air-Gapped و چک‌لیست

### اصل حاکم

کل چرخهٔ حیات POC — از آماده‌سازی تا اجرا و به‌روزرسانی — باید بدون هیچ اتصال خارجی در زمان اجرا ممکن باشد. الگوی عملیاتی از یک رویهٔ اثبات‌شده پیروی می‌کند: **ساخت و جمع‌آوری در ماشین متصل (staging)، انتقال بستهٔ تأییدشده، بارگذاری و اجرا در ماشین ایزوله**.

### مرحلهٔ ۱ — آماده‌سازی در ماشین متصل (Staging)

| کار | جزئیات | خروجی |
|---|---|---|
| ساخت تصویر `app` | `docker build` از سورس | `app:offline` |
| دریافت تصاویر پایه | `postgres`، `qdrant/qdrant`، `opensearchproject/opensearch`، `minio/minio`، `ollama/ollama` | تصاویر در حافظهٔ محلی Docker |
| ذخیرهٔ تصاویر | `docker save` برای هر تصویر | `images/*.tar` |
| جمع‌آوری wheelهای Python | `pip download -r requirements.txt -d wheels/` | `wheels/` |
| دریافت مدل LLM | Ollama: `ollama pull qwen2.5:7b` سپس صادرات فایل‌های مدل | `models/` (blobs + manifests) |
| دریافت مدل Embedding | دانلود وزن‌های `multilingual-e5-large` از Hugging Face | `models/embedding/` |
| محاسبهٔ hash | `sha256sum` برای همهٔ فایل‌های بسته | `SHA256SUMS` |
| ثبت مجوز | کپی متن LICENSE هر پروژه | `licenses/` |

**نکتهٔ Ollama**: انتقال مدل به محیط آلفلاین دو روش دارد: (۱) کپی کامل پوشهٔ `/root/.ollama/models/` شامل `blobs/` و `manifests/`؛ (۲) استفاده از فایل GGUF خام و ساخت مدل با `ollama create`. روش اول برای POC ساده‌تر است چون نیاز به Modelfile ندارد.

### مرحلهٔ ۲ — Transition بسته

بستهٔ کامل (`offline-bundle/`) از طریق رسانهٔ تأییدشده (USB، دیسک خارجی، یا کانال انتقال داخلی) منتقل می‌شود. در مقصد، **قبل از هر اقدامی**، `sha256sum -c SHA256SUMS` اجرا می‌شود. هر فایلی که hash آن مطابقت نداشته باشد، بستهٔ انتقال را نامعتبر می‌کند.

### مرحلهٔ ۳ — بارگذاری در ماشین ایزوله

```bash
# بارگذاری تصاویر
for img in images/*.tar; do docker load -i "$img"; done

# نصب وابستگی‌های Python (در ساخت تصویر app یا داخل کانتینر)
pip install --no-index --find-links=wheels/ -r requirements.txt

# بازیابی مدل Ollama
docker cp models/. ollama_container:/root/.ollama/models/

# بازیابی مدل Embedding
# (volume یا مسیر mount شده برای Sentence Transformers)
```

### مرحلهٔ ۴ — اجرا و تأیید

- `docker compose up -d` با فایل Compose آلفلاین (بدون بخش `build`، فقط `image`).
- بررسی `healthcheck` همهٔ سرویس‌ها.
- **تأیید قطع ارتباط خارجی**: بررسی اینکه هیچ کانتینری به اینترنت دسترسی ندارد. در محیط آلفلاین واقعی این به‌صورت طبیعی برقرار است، اما برای اطمینان می‌توان `backend` شبکه را `internal: true` تعریف کرد.
- اجرای یک چرخهٔ کامل Ingestion + Query برای تأیید عملکرد.

### چک‌لیست نهایی Offline

- [ ] همهٔ تصاویر Docker به‌صورت `.tar` ذخیره و منتقل شده‌اند
- [ ] `SHA256SUMS` قبل از بارگذاری تأیید شده است
- [ ] همهٔ wheelهای Python در `wheels/` موجودند و نصب آفلاین موفق بوده
- [ ] مدل LLM در volume مربوطه بازیابی شده و `ollama list` آن را نشان می‌دهد
- [ ] مدل Embedding در مسیر مورد انتظار قرار دارد و بارگذاری می‌شود
- [ ] فایل `.env` شامل هیچ URL خارجی یا کلید API سرویس ابری نیست
- [ ] `docker compose config` هیچ endpoint خارجی (مانند OTLP) نشان نمی‌دهد
- [ ] پس از Restart، همهٔ سرویس‌ها بدون خطا بالا می‌آیند
- [ ] Backup و Restore حداقل یک بار آزمایش شده‌اند
- [ ] متن LICENSE هر ابزار در بستهٔ `licenses/` ثبت شده است

### به‌روزرسانی آفلاین

هر بار که نسخهٔ یک ابزار یا مدل تغییر می‌کند، **کل چرخهٔ staging → Transition → تأیید hash → بارگذاری** تکرار می‌شود. هیچ مکانیزم به‌روزرسانی خودکار در POC وجود ندارد. برای POC کافی است که این رویه یک بار مستند و آزمایش شود.

---

## ۱۲. Security and Access Control (تفصیلی)

### مدل تهدید POC

POC با ۲–۳ کاربر آزمایشی و در محیط ایزوله اجرا می‌شود. تهدیدات اصلی شامل دسترسی غیرمجاز کاربران به اسناد یکدیگر، نشت محتوای غیرمجاز به context مدل، و رفتار نادرست در صورت خرابی سرویس ACL است. تهدیدات مربوط به حملات اینترنتی گسترده (DDoS، تزریق از راه دور) در محیط air-gapped موضوعیت کمتری دارند، اما برخی اصول (مانند عدم اعتماد به محتوای بازیابی‌شده) همچنان اعمال می‌شوند.

### احراز هویت (Authentication)

- **مدل**: جدول `users` در PostgreSQL با `username`، `password_hash` (argon2id)، `role` (`admin` یا `user`)، `is_active`.
- **Session**: پس از login موفق، یک توکن امضاشده (JWT کوتاه‌مدت یا Cookie با `itsdangerous`) صادر می‌شود. FastAPI Dependency هر درخواست را اعتبارسنجی می‌کند.
- **بدون OAuth/OIDC**: برای ۲–۳ کاربر، IdP خارجی نه لازم است و نه مجاز (افزودن کانتینر اضافی و سطح حمله).

### مجوزدهی (Authorization)

#### دو سطح دسترسی

| نقش | مجوزها |
|---|---|
| `admin` | آپلود سند، تعریف ACL، مشاهدهٔ همهٔ اسناد، اجرای query، مدیریت کاربران |
| `user` | اجرای query، مشاهدهٔ اسناد مجاز خود (طبق `document_acl`) |

#### Document-Level ACL

جدول `document_acl` در PostgreSQL:

| ستون | توضیح |
|---|---|
| `document_id` | ارجاع به `documents.id` |
| `subject_type` | `user` یا `group` |
| `subject_id` | `user_id` یا `group_id` |
| `permission` | `read` (برای POC، فقط read کافی است) |

**مرجع معتبر**: PostgreSQL تنها منبع حقیقت برای ACL است. هیچ cache یا snapshot دیگری از ACL در Qdrant یا OpenSearch ذخیره نمی‌شود (اگرچه می‌توان برای فیلتر اولیه از metadata در Qdrant استفاده کرد، اما **بازبینی نهایی همیشه از PostgreSQL انجام می‌شود**).

### رفتار Fail Closed

این مهم‌ترین اصل امنیتی POC است. در هر نقطه‌ای که تأیید مجوز ممکن نیست، سیستم باید **دسترسی را رد کند**، نه اینکه آن را نادیده بگیرد:

- اگر کوئری ACL به PostgreSQL با timeout یا خطا مواجه شود → نتیجهٔ بازیابی **خالی** برگردانده می‌شود.
- اگر `user_id` در Session نامعتبر یا منقضی باشد → درخواست با خطای 401 رد می‌شود.
- اگر یک chunk در نتیجهٔ جستجو باشد اما در ACL نباشد → آن chunk **حذف** می‌شود و هرگز به context LLM نمی‌رسد.

این رویکرد با توصیه‌های امنیتی RAG مطابقت دارد: بازبینی مجوز باید **پس از بازیابی و قبل از مونتاژ context** انجام شود، و عدم قطعیت باید به رد منجر شود نه اجازه.

### ترتیب اجرای امنیتی در Query Flow

ترتیب دقیق مراحل امنیتی در جریان Query (که قبلاً تعریف شد) حیاتی است:

۱. **احراز هویت** (Session validation) → اگر نامعتبر، توقف.
۲. **استخراج گروه‌های کاربر** از PostgreSQL → اگر خطا، Fail Closed.
۳. **جستجوی برداری و lexical** (بدون فیلتر ACL در این مرحله، یا با فیلتر ساده در Qdrant).
۴. **بازبینی اجباری ACL** برای هر chunk از PostgreSQL → حذف chunkهای غیرمجاز.
۵. **مونتاژ context** فقط از chunkهای تأییدشده.
۶. **فراخوانی LLM**.
۷. **ثبت audit log**.

مرحلهٔ ۴ نباید به هیچ عنوان حذف یا به بعد از مونتاژ context موکول شود.

### امنیت محتوای بازیابی‌شده

محتوای اسناد (DOCX/XLSX) باید به‌عنوان **ورودی غیرقابل اعتماد** تلقی شود. دستورالعمل‌های جاسازی‌شده در یک سند نباید بتوانند مجوزهای ابزار یا فیلترهای امنیتی را تغییر کنند. این با دستهٔ LLM01 (Prompt Injection) در OWASP Top 10 for LLM Applications مطابقت دارد، جایی که تزریق غیرمستقیم از طریق محتوای بازیابی‌شده به‌عنوان یک سطح حملهٔ واقعی شناخته می‌شود. در POC، mitigation حداقلی: LLM فقط برای تولید پاسخ فراخوانی می‌شود و هیچ ابزار یا action خارجی در اختیار ندارد؛ بنابراین حتی اگر محتوای مخرب در context باشد، امکان اجرای action وجود ندارد.

### امنیت شبکه و سرویس‌ها

- `qdrant`، `opensearch`، `minio`، `postgres` هیچ پورت منتشرشده‌ای ندارند و فقط از شبکهٔ `backend` قابل دسترسی‌اند.
- `ollama` نیز فقط در `backend` است و از خارج قابل دسترسی نیست.
- تنها سرویس منتشرشده `app` است (پورت Streamlit).
- در محیط air-gapped، لایهٔ شبکهٔ فیزیکی نیز ایزوله است.

### مدیریت Secret

- رمزهای PostgreSQL، رمز MinIO، و کلید امضای Session در فایل `.env` نگهداری می‌شوند (نه در `docker-compose.yml`).
- `.env` در مخزن کد commit نمی‌شود.
- برای POC، `docker secrets` یا Vault لازم نیست؛ `.env` با مجوز فایل محدود (`chmod 600`) کافی است.

### Audit Logging

جدول `audit_log` در PostgreSQL حداقل شامل این فیلدهاست (بر اساس الگوی توصیه‌شده برای RAG):

| فیلد | توضیح |
|---|---|
| `request_id` | شناسهٔ یکتای درخواست |
| `timestamp` | زمان |
| `user_id` | کاربر |
| `query_hash` | hash پرسش (نه متن خام، برای حفظ حریم خصوصی) |
| `chunk_ids_returned` | chunkهای بازیابی‌شده پس از ACL |
| `chunks_denied_count` | تعداد chunkهای ردشده توسط ACL |
| `llm_response_id` | شناسهٔ پاسخ (اختیاری) |
| `latency_ms` | زمان پاسخ |

هدف: پاسخ به این پرسش که «چه کسی، چه زمانی، چه محتوایی را دید». لاگ‌ها باید به‌عنوان دادهٔ محافظت‌شده ذخیره شوند، زیرا خود پرسش‌ها ممکن است حاوی اطلاعات حساس باشند.

---

## ۱۳. وضعیت تأیید نسخه و License

| ابزار | نسخه — سطح اعتماد | License — سطح اعتماد |
|---|---|---|
| Streamlit | نیازمند تأیید | نیازمند تأیید (Apache 2.0 شناخته‌شده) |
| FastAPI | نیازمند تأیید | نیازمند تأیید (MIT شناخته‌شده) |
| Flask | نیازمند تأیید | نیازمند تأیید |
| Litestar | نیازمند تأیید | نیازمند تأیید |
| Python + PG Worker | وابستگی‌ها نیازمند تأیید | کد خودساخته |
| Celery | نیازمند تأیید | نیازمند تأیید |
| APScheduler | نیازمند تأیید | نیازمند تأیید |
| python-docx | نیازمند تأیید | نیازمند تأیید |
| openpyxl | نیازمند تأیید | نیازمند تأیید |
| Docling | **2.149.0** (دست‌دوم، نیازمند تأیید رسمی) | نیازمند تأیید |
| Unstructured | **0.27.5** (دست‌دوم، نیازمند تأیید رسمی) | نیازمند تأیید |
| Ollama | نیازمند تأیید | نیازمند تأیید |
| llama.cpp | نیازمند تأیید | نیازمند تأیید |
| vLLM | نیازمند تأیید | نیازمند تأیید |
| Qdrant | نیازمند تأیید | نیازمند تأیید |
| OpenSearch | نیازمند تأیید | نیازمند تأیید |
| MinIO | نیازمند تأیید | **AGPL-3.0** (نیازمند بررسی حقوقی) |
| PostgreSQL | نیازمند تأیید | PostgreSQL License |
| Casdoor | v4.17 (برچسب بررسی اکتبر ۲۰۲۶) | Apache-2.0 |
| Keycloak | 26.8 (برچسب بررسی اکتبر ۲۰۲۶) | Apache-2.0 |
| Ragas | نیازمند تأیید | Apache-2.0 |
| Langfuse | نیازمند تأیید | MIT (ماژول هسته) |
| Phoenix | نیازمند تأیید | Elastic License 2.0 (نیازمند بررسی انطباق) |
| Podman | نیازمند تأیید | Apache-2.0 |
| containerd | نیازمند تأیید | Apache-2.0 |

**الزام Phase 1**: برای هر ابزار، ① قفل نسخه از صفحهٔ رسمی Release، ② تأیید License از فایل LICENSE مخزن، ③ دانلود wheel/تصویر/فایل مدل و ثبت SHA-256.

---

## ۱۴. پیاده‌سازی Roadmap

### Phase 1 — آماده‌سازی محیط (تخمین: ۲–۳ روز)

| مورد | جزئیات |
|---|---|
| هدف | آماده‌سازی ماشین staging و ماشین هدف، جمع‌آوری بستهٔ آفلاین |
| ابزارها | Docker Engine، Docker Compose (نسخهٔ ۲.۲۴+ برای پشتیبانی از overlay)، Python 3.10+ |
| پیش‌نیاز | دسترسی به اینترنت در staging؛ ماشین هدف با Linux |
| خروجی قابل تحویل | `offline-bundle/` شامل `images/`، `wheels/`، `models/`، `SHA256SUMS` |
| معیار پذیرش | `sha256sum -c SHA256SUMS` در ماشین هدف موفق باشد؛ `docker compose config` بدون خطا |
| وابستگی | هیچ |

**فعالیت‌های کلیدی**: ساخت تصویر `app`؛ `docker save` همهٔ تصاویر؛ `pip download`؛ دریافت مدل LLM و Embedding؛ محاسبهٔ hash؛ Transition بسته؛ بارگذاری تصاویر در ماشین هدف.

### Phase 2 — زیرساخت اصلی (تخمین: ۱–۲ روز)

| مورد | جزئیات |
|---|---|
| هدف | راه‌اندازی PostgreSQL، MinIO، Qdrant، OpenSearch، Ollama با Compose |
| ابزارها | `docker-compose.yml` آلفلاین |
| پیش‌نیاز | Phase 1 کامل |
| خروجی قابل تحویل | همهٔ کانتینرها در وضعیت healthy؛ اتصال شبکه‌ای بین `app` و سرویس‌ها برقرار |
| معیار پذیرش | `docker compose ps` همه را `Up (healthy)` نشان دهد؛ `pg_isready`، `mc admin info`، `curl qdrant:6333`، `curl opensearch:9200` موفق باشند |
| وابستگی | Phase 1 |

### Phase 3 — Ingestion و ذخیره‌سازی Canonical (تخمین: ۳–۵ روز)

| مورد | جزئیات |
|---|---|
| هدف | پیاده‌سازی آپلود، پارس DOCX/XLSX، نرمال‌سازی فارسی، Chunking، ذخیرهٔ Canonical در MinIO و PostgreSQL |
| ابزارها | `python-docx`، `openpyxl`، ماژول نرمال‌سازی، psycopg، MinIO SDK |
| پیش‌نیاز | Phase 2 |
| خروجی قابل تحویل | endpoint `/upload`؛ worker پردازش؛ جدول‌های `documents`، `chunks`، `processing_jobs`؛ اسکریپت Reconciliation دستی |
| معیار پذیرش | آپلود یک DOCX و یک XLSX موفق؛ chunkها در MinIO و PostgreSQL ثبت شوند؛ آپلود تکراری همان `document_id` را برگرداند (Idempotency) |
| وابستگی | Phase 2 |

### Phase 4 — Indexing (تخمین: ۲–۳ روز)

| مورد | جزئیات |
|---|---|
| هدف | تولید Embedding، ایندکس در Qdrant و OpenSearch، اعتبارسنجی ایندکس، فعال‌سازی سند |
| ابزارها | Sentence Transformers (`multilingual-e5-large`)، Qdrant client، OpenSearch client |
| پیش‌نیاز | Phase 3 |
| خروجی قابل تحویل | worker مرحلهٔ indexing؛ جدول `index_registry`؛ تابع `validate_index` |
| معیار پذیرش | تعداد chunkهای Qdrant و OpenSearch با PostgreSQL مطابقت داشته باشد؛ سند پس از تأیید `active` شود؛ بازسازی کامل ایندکس از Canonical Data ممکن باشد |
| وابستگی | Phase 3 |

### Phase 5 — Retrieval و RAG (تخمین: ۴–۶ روز)

| مورد | جزئیات |
|---|---|
| هدف | پیاده‌سازی Hybrid Search، RRF، بازبینی ACL، مونتاژ context، فراخوانی LLM، ارجاع به منابع |
| ابزارها | Qdrant، OpenSearch، تابع RRF Python، FastAPI، Ollama |
| پیش‌نیاز | Phase 4 |
| خروجی قابل تحویل | endpoint `/query`؛ تابع `acl_revalidate`؛ تابع `rrf_fuse`؛ `context_assembler`؛ اتصال به Ollama |
| معیار پذیرش | پرسش یک کاربر `user` فقط chunkهای مجاز را برگرداند؛ chunk غیرمجاز هرگز در پاسخ ظاهر نشود؛ پاسخ دارای ارجاع به منبع باشد؛ در صورت خطای ACL، پاسخ خالی برگردد (Fail Closed) |
| وابستگی | Phase 4 |

### Phase 6 — ارزیابی و پذیرش (تخمین: ۳–۴ روز)

| مورد | جزئیات |
|---|---|
| هدف | اجرای معیارهای پذیرش، ارزیابی کیفیت RAG، آزمایش امنیت و بازیابی |
| ابزارها | Ragas، مجموعه‌دادهٔ آزمون دستی، اسکریپت‌های تست |
| پیش‌نیاز | Phase 5 |
| خروجی قابل تحویل | گزارش ارزیابی (Recall@K، Precision@K، Faithfulness، زمان پاسخ)؛ گزارش تست امنیتی؛ گزارش Backup/Restore |
| معیار پذیرش | همهٔ معیارهای Acceptance Criteria (بخش ۱۵) برآورده شوند یا انحرافات مستند شوند |
| وابستگی | Phase 5 |

### تخمین کل زمان

**حدود ۱۵–۲۳ روز کاری** برای یک توسعه‌دهندهٔ تمام‌وقت با فرض آشنایی با Python و Docker. این تخمین شامل زمان رفع اشکال و آزمون‌های تکراری نیست؛ در عمل ممکن است ۲۰–۳۰٪ افزایش یابد و به **۱۸–۳۰ روز** برسد. اگر توسعه‌دهنده هم‌زمان روی چند Phase کار کند، زمان کلی کاهش می‌یابد اما ریسک یکپارچگی افزایش می‌یابد.

---

## ۱۵. معیارهای پذیرش (Acceptance Criteria)

### معیارهای پردازش اسناد

| معیار | روش اندازه‌گیری | آستانهٔ پذیرش |
|---|---|---|
| دریافت موفق DOCX | آپلود ۵ فایل DOCX نمونه از طریق `/upload` | ۵/۵ موفق، بدون خطای 5xx |
| دریافت موفق XLSX | آپلود ۵ فایل XLSX نمونه | ۵/۵ موفق |
| استخراج متن Word | مقایسهٔ متن استخراج‌شده با متن مرجع (باز شده در Word) | تطابق ≥ ۹۵٪ کاراکترهای غیرفاصله |
| استخراج داده‌های Excel | مقایسهٔ مقادیر سلول‌های استخراج‌شده با فایل اصلی برای ۳ شیت | تطابق ۱۰۰٪ مقادیر، حفظ `sheet_name` و `row_range` |
| حفظ ساختار جدول | بررسی وجود `location_meta` با `sheet`/`row`/`column` در chunkهای Excel | همهٔ chunkهای Excel دارای این فیلد باشند |
| ثبت Metadata | کوئری SQL: شمارش ردیف‌های `documents`، `chunks`، `processing_jobs` | تعداد مطابق با انتظار؛ وضعیت `active` برای اسناد موفق |
| Duplicate Detection | آپلود مجدد همان فایل؛ بررسی `document_id` برگشتی | همان `document_id`، بدون ایجاد ردیف جدید در `documents` |
| Chunking قابل تکرار | اجرای دوبارهٔ chunking روی همان سند با همان تنظیمات | `content_hash` هر chunk یکسان باشد |

### معیارهای بازیابی (Retrieval)

| معیار | روش اندازه‌گیری | آستانهٔ پذیرش |
|---|---|---|
| Vector Search | پرسش با کلمهٔ کلیدی مشخص؛ بررسی وجود chunk حاوی آن در نتایج Qdrant | chunk هدف در top-10 باشد |
| BM25 Search | همان پرسش؛ بررسی نتایج OpenSearch | chunk هدف در top-10 باشد |
| Hybrid Search | ترکیب دو نتیجه؛ بررسی حضور chunk هدف | chunk هدف در top-5 резулت نهایی RRF |
| RRF | تست واحد تابع RRF با دو لیست رتبه‌بندی مصنوعی | خروجی مطابق فرمول `Σ 1/(k+rank)` با `k=60` |
| Source Attribution | بررسی پاسخ؛ وجود `document_id` و `location_meta` برای هر chunk استفاده‌شده | ≥ ۹۰٪ پاسخ‌های دارای حداقل یک منبع معتبر |
| بازیابی نسخهٔ صحیح سند | آپلود دو نسخه از یک سند؛ پرسش مرتبط با نسخهٔ دوم | فقط chunkهای نسخهٔ فعال برگردانده شوند |

### معیارهای امنیتی

| معیار | روش اندازه‌گیری | آستانهٔ پذیرش |
|---|---|---|
| احراز هویت | درخواست به `/query` بدون Session معتبر | پاسخ 401؛ بدون اجرای query |
| دو سطح دسترسی | کاربر `user` سعی در آپلود سند | پاسخ 403؛ سند آپلود نشود |
| جداسازی دسترسی کاربران | کاربر A سندی آپلود کند که ACL آن فقط B را مجاز می‌داند؛ پرسش توسط A | chunkهای آن سند در نتایج A ظاهر نشوند |
| Fail Closed | شبیه‌سازی قطع PostgreSQL در حین بازبینی ACL | پاسخ خالی برگردد؛ بدون فراخوانی LLM با chunk غیرمجاز |
| عدم ورود محتوای غیرمجاز به Context | بررسی `audit_log` برای `chunks_denied_count` و مقایسه با chunkهای ارسال‌شده به LLM | تعداد chunkهای ارسال‌شده به LLM = تعداد chunkهای مجاز پس از ACL |

### معیارهای LLM

| معیار | روش اندازه‌گیری | آستانهٔ پذیرش |
|---|---|---|
| اجرای کاملاً محلی | قطع کامل شبکهٔ خارجی؛ اجرای query | پاسخ بدون خطای اتصال برگردد |
| پاسخ بر اساس منابع بازیابی‌شده | بررسی پاسخ برای پرسش‌هایی که پاسخ در context موجود است | ≥ ۸۰٪ پاسخ‌ها حاوی اطلاعات از context باشند (بدون توهم محض) |
| نمایش منابع پاسخ | بررسی وجود `document_id` و `location_meta` در پاسخ UI | ≥ ۹۰٪ پاسخ‌ها دارای منبع |
| رفتار در نبود اطلاعات | پرسش دربارهٔ موضوعی که در هیچ سندی نیست | پاسخ صریحاً اعلام کند که اطلاعات کافی ندارد؛ بدون ساخت پاسخ |

### معیارهای عملیاتی

| معیار | روش اندازه‌گیری | آستانهٔ پذیرش |
|---|---|---|
| اجرای کامل بدون اینترنت | `docker compose up -d` در ماشین ایزوله | همهٔ سرویس‌ها healthy؛ بدون خطای دانلود/اتصال |
| Restart موفق | `docker compose restart`؛ بررسی وضعیت پس از ۳۰ ثانیه | همهٔ سرویس‌ها healthy |
| ثبت خطاها | ایجاد خطای عمدی (فایل نامعتبر)؛ بررسی `error_log` | خطا با جزئیات در `processing_jobs.error_log` ثبت شود |
| Retry پردازش ناموفق | شبیه‌سازی شکست در مرحلهٔ parsing؛ بررسی `attempt_count` | افزایش `attempt_count`؛ وضعیت نهایی `failed` پس از ۳ تلاش |
| Rebuild ایندکس | حذف collection در Qdrant و index در OpenSearch؛ اجرای اسکریپت rebuild از Canonical Data | تعداد chunkهای بازسازی‌شده با PostgreSQL مطابقت داشته باشد |
| Backup و Restore | Backup از PostgreSQL و MinIO؛ Restore در محیط تمیز | داده‌ها بازیابی شوند؛ query همچنان کار کند |

### معیارهای کمی RAG (پیشنهادی)

بر اساس متریک‌های موجود در Ragas، معیارهای زیر برای ارزیابی کیفیت پیشنهاد می‌شوند. برای هر معیار، یک مجموعه‌دادهٔ آزمون دستی شامل ۲۰–۳۰ جفت (پرسش، پاسخ مرجع، chunkهای مرتبط) لازم است.

| معیار | تعریف (بر اساس Ragas) | روش | آستانهٔ پیشنهادی POC |
|---|---|---|---|
| **Context Precision** | آیا chunkهای مرتبط در رتبه‌های بالاتر قرار گرفته‌اند؟ | Ragas `context_precision` | ≥ ۰.۶ |
| **Context Recall** | آیا context بازیابی‌شده تمام اطلاعات پاسخ مرجع را پوشش می‌دهد؟ | Ragas `context_recall` | ≥ ۰.۷ |
| **Faithfulness** | آیا پاسخ از context استخراج شده (نه توهم)؟ | Ragas `faithfulness` | ≥ ۰.۸ |
| **Answer Relevancy** | آیا پاسخ به پرسش مربوط است؟ | Ragas `answer_relevancy` | ≥ ۰.۷ |
| **زمان پاسخ** | از ارسال query تا دریافت پاسخ کامل | اندازه‌گیری در `audit_log.latency_ms` | ≤ ۳۰ ثانیه (CPU) / ≤ ۱۰ ثانیه (GPU) |

**نکته**: آستانه‌ها برای POC و با مدل ۷B quantized پیشنهاد شده‌اند. اگر مدل بزرگ‌تر یا GPU استفاده شود، آستانه‌ها قابل افزایش هستند. اگر آستانه‌ها برآورده نشوند، این یک یافتهٔ معتبر برای ارزیابی است، نه شکست POC — هدف POC اندازه‌گیری است، نه رسیدن به کیفیت تولیدی.

---

## ۱۶. ریسک‌ها و محدودیت‌های فنی

### ریسک‌های فنی

| ریسک | احتمال | تأثیر | توضیح | کاهش ریسک |
|---|---|---|---|---|
| **کیفیت پایین فارسی در LLM ۷B** | متوسط | بالا | Qwen2.5-7B ممکن است در پاسخ‌دهی فارسی ضعیف عمل کند. | آزمون تجربی در Phase 5؛ در صورت نیاز ارتقا به Qwen2.5-14B (نیازمند RAM/VRAM بیشتر) یا پذیرش تمرکز بر انگلیسی |
| **حجم بالای مدل Embedding** | کم | متوسط | `multilingual-e5-large` (~۲.۲GB) ممکن است در سناریوی RAM محدود مشکل ایجاد کند. | جایگزینی با `paraphrase-multilingual-MiniLM-L12-v2` (~۴۷۰MB) با کاهش کیفیت |
| **مصرف RAM بالای OpenSearch** | متوسط | متوسط | OpenSearch به‌طور پیش‌فرض heap بزرگ درخواست می‌کند؛ در محیط با RAM محدود ممکن است OOM رخ دهد. | تنظیم `OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m`؛ در صورت تشدید، حذف OpenSearch و استفاده از sparse vector در Qdrant (انحراف از baseline) |
| **پیچیدگی استخراج DOCX با python-docx** | متوسط | کم | python-docx برای Header/Footer، TextBox و برخی عناصر پیچیده محدودیت دارد. | آزمون با اسناد واقعی؛ در صورت نیاز، جایگزینی داخلی با Docling (Option 2 در L4) بدون تغییر رابط‌ها |
| **ناسازگاری نسخه‌های وابستگی** | متوسط | متوسط | زنجیرهٔ وابستگی (PyTorch, Sentence Transformers, FastAPI) ممکن است در محیط آلفلاین با نسخه‌های ناسازگار مواجه شود. | قفل نسخه‌ها در `requirements.txt` با `pip freeze`؛ تست نصب آفلاین قبل از Transition نهایی |
| **رفتار غیرقابل پیش‌بینی LLM در برابر prompt injection** | کم | بالا | محتوای مخرب در DOCX/XLSX ممکن است سعی در تغییر دستورالعمل‌ها کند. | LLM هیچ ابزار یا action خارجی ندارد؛ context با جداکننده‌های صریح مونتاژ می‌شود |

### محدودیت‌های معماری POC

| محدودیت | توضیح | دلیل پذیرش در POC |
|---|---|---|
| **بدون Reranking** | مرحلهٔ بازرتبه‌بندی حذف شده است. | کاهش پیچیدگی و مصرف منابع؛ جای آن در کد به‌صورت تابع خالی نگه داشته می‌شود تا در آینده اضافه شود |
| **بدون HA / Replication** | تک‌گره، بدون افزونگی. | دامنهٔ POC؛ Master Prompt صراحتاً Multi-Node و HA را Out of Scope کرده |
| **بدون Message Broker** | Worker با polling از PostgreSQL استفاده می‌کند. | سادگی؛ تأخیر ۲–۵ ثانیه برای POC قابل قبول است |
| **ACL فقط در سطح سند** | مجوز در سطح chunk یا فیلد تعریف نشده است. | کفایت برای ۲–۳ کاربر؛ پیچیدگی کمتر |
| **بدون OCR** | DOCX/XLSX متنی فرض می‌شوند؛ تصاویر داخل اسناد استخراج نمی‌شوند. | دامنهٔ ورودی POC؛ OCR در Out of Scope |
| **بدون Observability پیشرفته** | بدون Prometheus/Grafana. | Master Prompt ابزار سبک را در اولویت قرار داده |

### محدودیت‌های مرتبط با License

| ابزار | License | ریسک | اقدام |
|---|---|---|---|
| **MinIO** | AGPL-3.0 | در صورت Transition شبکه‌ای سرویس مبتنی بر MinIO، ممکن است الزام انتشار سورس ایجاد شود. | بررسی حقوقی جداگانه؛ در POC داخلی معمولاً مشکلی ندارد؛ برای آیندهٔ سازمانی، بررسی License تجاری یا جایگزین |
| **Arize Phoenix** | Elastic License 2.0 | استاندارد OSI نیست؛ محدودیت در ارائه به‌عنوان سرویس مدیریت‌شده. | در POC استفاده نشده؛ فقط به‌عنوان Option 3 ثبت شده |
| **Redis (در صورت انتخاب Celery)** | RSALv2/SSPL (نسخه‌های جدید) | محدودیت برای ارائه به‌عنوان سرویس. | در POC استفاده نشده؛ گزینهٔ ۲ L3 است |

### ریسک‌های مرتبط با OWASP LLM Top 10

بر اساس OWASP Top 10 for LLM Applications (نسخهٔ ۲۰۲۵)، ریسک‌های زیر برای POC مرتبط‌اند:

| ریسک OWASP | ارتباط با POC | وضعیت در POC |
|---|---|---|
| **LLM01: Prompt Injection** | محتوای DOCX/XLSX ممکن است حاوی دستورالعمل باشد. | LLM بدون ابزار؛ context با جداکننده؛ بدون action خارجی |
| **LLM02: Sensitive Information Disclosure** | نشت اسناد بین کاربران. | ACL اجباری؛ Fail Closed |
| **LLM08: Vector and Embedding Weaknesses** | دسترسی غیرمجاز از طریق بازیابی برداری. | ACL Revalidation پس از جستجو و قبل از context |
| **LLM09: Misinformation** | توهم LLM در پاسخ فارسی. | ارزیابی با Ragas (Faithfulness)؛ نمایش منابع |

---

## ۱۷. منابع

### [۱] مستندات رسمی ابزارهای منتخب

| # | ابزار | منبع | کاربرد در سند |
|---|---|---|---|
| [۱] | Streamlit | https://docs.streamlit.io | L1 — UI |
| [۲] | FastAPI | https://fastapi.tiangolo.com | L2 — API |
| [۳] | Qdrant | https://qdrant.tech/documentation | L6 — Vector Search |
| [۴] | OpenSearch | https://opensearch.org/docs | L6 — BM25 |
| [۵] | MinIO | https://min.io/docs | L7 — Object Storage |
| [۶] | Ollama | https://github.com/ollama/ollama | L5 — LLM Runtime |
| [۷] | PostgreSQL | https://www.postgresql.org/docs | L7/L8 — Metadata/ACL |
| [۸] | python-docx | https://python-docx.readthedocs.io | L4 — DOCX Parsing |
| [۹] | openpyxl | https://openpyxl.readthedocs.io | L4 — XLSX Parsing |
| [۱۰] | Docling | https://github.com/DS4SD/docling | L4 — گزینهٔ جایگزین |
| [۱۱] | Unstructured | https://docs.unstructured.io | L4 — گزینهٔ جایگزین |
| [۱۲] | Ragas | https://docs.ragas.io | L9 — ارزیابی RAG |
| [۱۳] | Langfuse | https://langfuse.com/docs | L9 — گزینهٔ جایگزین |
| [۱۴] | Arize Phoenix | https://docs.arize.com/phoenix | L9 — گزینهٔ جایگزین |
| [۱۵] | Docker Compose | https://docs.docker.com/compose | L10 — Deployment |

### [۱۶] منابع امنیتی

| # | منبع | کاربرد |
|---|---|---|
| [۱۶] | OWASP Top 10 for LLM Applications (2025) | تحلیل ریسک‌های امنیتی POC |
| [۱۷] | OWASP LLM Top 10 — Promptfoo Docs | مرجع تکمیلی |

### [۱۸] منابع مرتبط با Air-Gapped

| # | منبع | کاربرد |
|---|---|---|
| [۱۸] | MinIO Data Recovery | درک رفتار Erasure Coding (فقط برای زمینه؛ POC از تک‌گره استفاده می‌کند) |
| [۱۹] | Ollama Import Guide | رویهٔ Transition مدل آلفلاین |

**یادداشت دربارهٔ منابع**: بسیاری از اطلاعات نسخه و License در این سند از منابع دست‌دوم یا اسناد جمع‌آوری‌شده به‌دست آمده‌اند. **در Phase 1، هر ابزار باید از صفحهٔ رسمی Release و فایل LICENSE مخزن تأیید نهایی شود.** این سند ادعای قطعیت دربارهٔ نسخه یا License ندارد.

---

## ۱۸. برآورد منابع سخت‌افزاری (سه سناریو)

این بخش صرفاً **برآورد مهندسی** است و مبنای انتخاب ابزارها نیست. اعداد بر اساس مستندات رسمی ابزارها و منابع عمومی قابل Transition جمع‌آوری شده‌اند، اما **هیچ سند رسمی واحدی برای ترکیب این پشته وجود ندارد**; بنابراین اعداد باید در Phase 1 با آزمون واقعی روی سخت‌افزار هدف کالیبره شوند.

### پایه‌های برآورد

| مؤلفه | منبع تخمین | ملاحظات |
|---|---|---|
| **Ollama (مدل ۷B Q4)** | حداقل ۸ گیگابایت RAM، ۱۶ گیگابایت توصیه‌شده؛ ۲۰ گیگابایت دیسک حداقل، ۵۰–۱۰۰ گیگابایت توصیه‌شده | مدل Qwen2.5-7B-Instruct Q4 حدود ۴–۵ گیگابایت فایل مدل |
| **OpenSearch (تک‌گره dev/test)** | ۲ هسته CPU، ۴ گیگابایت RAM، ۵۰ گیگابایت دیسک | heap پیش‌فرض باید حداقل ۵۱۲ مگابایت باشد؛ کمتر از آن با پلاگین‌های همراه OOM می‌دهد |
| **Qdrant** | ۴ گیگابایت RAM یا بیشتر، ۴۰ گیگابایت دیسک (توصیهٔ عمومی) | برای POC با حجم دادهٔ کوچک، مصرف واقعی بسیار کمتر است |
| **PostgreSQL** | حداقل ۱ گیگابایت RAM برای شروع؛ `shared_buffers` حدود ۲۵٪ RAM | برای POC با ۲–۳ کاربر و دادهٔ کوچک، مصرف بسیار پایین است |
| **MinIO** | توصیهٔ رسمی برای AIStor حداقل ۲۵۶ گیگابایت است | **این عدد برای استقرار سازمانی است، نه POC**. MinIO در حالت تک‌گره و بار سبک، مصرف بسیار کمتری دارد |
| **Docker Engine + Compose** | ۸ گیگابایت حداقل، ۱۶ گیگابایت توصیه‌شده برای محیط توسعه | شامل overhead خود Docker |

**نکتهٔ مهم دربارهٔ MinIO**: عدد ۲۵۶ گیگابایت در مستندات MinIO مربوط به AIStor سازمانی با هزاران اتصال هم‌زمان است. برای POC با ۲–۳ کاربر و چند صد سند، MinIO در عمل چند صد مگابایت RAM مصرف می‌کند. در برآورد زیر، MinIO با مصرف سبک در نظر گرفته شده است.

### سناریو ۱ — Minimum Viable Configuration

**هدف**: اجرای POC با حداقل منابع ممکن، پذیرش کیفیت پایین‌تر و زمان پاسخ بیشتر.

| مؤلفه | مقدار | توضیح |
|---|---|---|
| **CPU** | ۴ هسته | برای Ollama (CPU inference) + OpenSearch + سایر سرویس‌ها |
| **RAM** | **۱۶ گیگابایت** | حداقل مطلق برای اجرای هم‌زمان Ollama ۷B (۸ گیگابایت) + OpenSearch (۴ گیگابایت) + بقیه (~۴ گیگابایت) |
| **GPU / VRAM** | بدون GPU | Ollama روی CPU اجرا می‌شود؛ inference کند خواهد بود |
| **Disk** | **۱۰۰ گیگابایت SSD** | مدل LLM (~۵ گیگابایت) + مدل Embedding (~۲.۲ گیگابایت) + تصاویر Docker (~۵–۸ گیگابایت) + داده و ایندکس (~۲۰–۳۰ گیگابایت) + فضای آزاد |
| **SSD** | الزامی | Qdrant برای ذخیره‌سازی برداری به SSD توصیه می‌شود |

**قابلیت‌های قابل اجرا**:
- همهٔ ده لایه به‌صورت کامل
- LLM ۷B quantized روی CPU
- Embedding با `multilingual-e5-large` یا در صورت کمبود RAM با `MiniLM`

**محدودیت‌ها**:
- زمان پاسخ LLM روی CPU: احتمالاً **۳۰–۶۰ ثانیه** برای پاسخ‌های متوسط
- OpenSearch heap باید روی ۵۱۲ مگابایت تنظیم شود تا از OOM جلوگیری شود
- هم‌زمانی واقع‌بینانه: **۱ پرسش در یک زمان**؛ پرسش دوم در صف می‌ماند
- فرآیند Ingestion ممکن است با query هم‌زمان تداخل منابع داشته باشد

**مدل‌های قابل استفاده**: Qwen2.5-7B-Instruct Q4؛ Embedding: MiniLM یا e5-large با احتیاط.

---

### سناریو ۲ — Recommended POC Configuration

**هدف**: تعادل بین منابع و قابلیت استفاده؛ زمان پاسخ قابل Accept برای ۲–۳ کاربر.

| مؤلفه | مقدار | توضیح |
|---|---|---|
| **CPU** | ۸ هسته | امکان هم‌زمانی بهتر بین Ollama، OpenSearch و Worker |
| **RAM** | **۳۲ گیگابایت** | فضای کافی برای Ollama (۱۲–۱۶ گیگابایت) + OpenSearch (۴–۸ گیگابایت) + بقیه |
| **GPU / VRAM** | **اختیاری: NVIDIA با ۸ گیگابایت VRAM** | اگر GPU موجود باشد، LLM به‌صورت قابل توجهی سریع‌تر می‌شود؛ بدون GPU نیز کار می‌کند |
| **Disk** | **۲۰۰ گیگابایت SSD** | فضای کافی برای مدل‌های بیشتر، دادهٔ آزمون بزرگ‌تر، و snapshotهای Backup |
| **SSD** | NVMe توصیه‌شده | برای OpenSearch و Qdrant |

**قابلیت‌های قابل اجرا**:
- همان قابلیت‌های سناریو ۱، با کارایی بهتر
- امکان استفاده از Embedding با کیفیت بالاتر (`bge-m3`) بدون فشار RAM
- امکان نگهداری هم‌زمان بیش از یک مدل LLM در Ollama

**محدودیت‌ها**:
- بدون GPU: زمان پاسخ **۱۵–۳۰ ثانیه**
- با GPU ۸ گیگابایت: زمان پاسخ **۵–۱۰ ثانیه** برای پاسخ‌های متوسط
- هم‌زمانی واقع‌بینانه: **۲ پرسش هم‌زمان** (با GPU) یا ۱–۲ پرسش (بدون GPU)

**مدل‌های قابل استفاده**: Qwen2.5-7B-Instruct Q4 (یا Q5 برای کیفیت بالاتر)؛ Embedding: e5-large یا bge-m3.

---

### سناریو ۳ — Higher-Performance POC Configuration

**هدف**: بالاترین کیفیت و سرعت در محدودهٔ POC؛ نزدیک‌ترین تجربه به یک سیستم تولیدی کوچک.

| مؤلفه | مقدار | توضیح |
|---|---|---|
| **CPU** | ۱۲–۱۶ هسته | برای بارهای هم‌زمان Ingestion + Query |
| **RAM** | **۶۴ گیگابایت** | فضای کافی برای مدل بزرگ‌تر (۱۴B) + OpenSearch با heap بزرگ‌تر + cache سیستم‌عامل |
| **GPU / VRAM** | **NVIDIA با ۱۶–۲۴ گیگابایت VRAM** | امکان اجرای Qwen2.5-14B Q4 با سرعت بالا |
| **Disk** | **۵۰۰ گیگابایت NVMe** | فضای کافی برای چند مدل، دادهٔ آزمون گسترده، و snapshotهای متعدد |
| **SSD** | NVMe | برای OpenSearch و Qdrant |

**قابلیت‌های قابل اجرا**:
- همهٔ قابلیت‌های سناریو ۲
- امکان ارتقا به **Qwen2.5-14B-Instruct** (کیفیت فارسی بهتر)
- OpenSearch با heap بزرگ‌تر (۲–۴ گیگابایت) برای ایندکس بزرگ‌تر
- امکان نگهداری چند نسخهٔ مدل برای مقایسه

**محدودیت‌ها**:
- هزینهٔ سخت‌افزاری بالاتر
- پیچیدگی بیشتر در مدیریت GPU (درایور، CUDA، سازگاری Ollama)

**مدل‌های قابل استفاده**: Qwen2.5-14B-Instruct Q4؛ Embedding: bge-m3 یا e5-large.

---

### جدول مقایسهٔ سه سناریو

| معیار | Minimum | Recommended | Higher-Performance |
|---|---|---|---|
| **RAM** | ۱۶ گیگابایت | ۳۲ گیگابایت | ۶۴ گیگابایت |
| **CPU** | ۴ هسته | ۸ هسته | ۱۲–۱۶ هسته |
| **GPU** | بدون | اختیاری (۸ گیگابایت) | توصیه‌شده (۱۶–۲۴ گیگابایت) |
| **Disk** | ۱۰۰ گیگابایت SSD | ۲۰۰ گیگابایت SSD | ۵۰۰ گیگابایت NVMe |
| **مدل LLM** | ۷B Q4 (CPU) | ۷B Q4 (CPU/GPU) | ۷B یا ۱۴B Q4 (GPU) |
| **زمان پاسخ تخمینی** | ۳۰–۶۰ ثانیه | ۱۵–۳۰ (CPU) / ۵–۱۰ (GPU) | ۳–۸ ثانیه (GPU) |
| **هم‌زمانی** | ۱ پرسش | ۱–۲ پرسش | ۲–۳ پرسش |
| **کیفیت فارسی** | پایه | قابل Accept | بهتر (با ۱۴B) |
| **هزینهٔ نسبی** | کم | متوسط | بالا |

---

### توصیٔ نهایی برای POC

با توجه به اینکه:
- هدف POC «اثبات امکان‌پذیری و ارزیابی کیفیت» است، نه «عملکرد تولیدی»
- Master Prompt صراحتاً Hardware-Agnostic بودن را الزام کرده
- هیچ سخت‌افزار مشخصی هنوز تعیین نشده

**توصیه**: سناریو ۲ (Recommended) به‌عنوان **نقطهٔ شروع پیشنهادی** در نظر گرفته شود، با این آگاهی که:
- اگر سخت‌افزار موجود فقط ۱۶ گیگابایت RAM دارد، سناریو ۱ قابل اجراست اما با انتظارات عملکردی پایین‌تر
- اگر GPU موجود باشد، حتی با ۱۶ گیگابایت RAM می‌توان به عملکرد سناریو ۲ نزدیک شد
- سناریو ۳ فقط در صورتی که کیفیت فارسی ۷B در آزمون ناکافی باشد و بودجهٔ سخت‌افزاری اجازه دهد توجیه دارد

**تصمیم نهایی دربارهٔ سخت‌افزار، طبق Master Prompt، به مرحله‌ای مستقل پس از تدوین و بررسی این طرح موکول می‌شود.**

---

## تکمیل سند

با افزودن این بخش، تمام بخش‌های خواسته‌شده در ساختار گزارش (بخش ۱۷ Master Prompt) پوشش داده شدند:

| # | بخش | وضعیت |
|---|---|---|
| ۱ | Executive Summary | ✅ |
| ۲ | POC Scope and Assumptions | ✅ |
| ۳ | Architecture Baseline | ✅ |
| ۴ | Logical Architecture | ✅ |
| ۵ | Tool Evaluation — Three per Layer | ✅ (L1–L10) |
| ۶ | Persian Language and Model Evaluation | ✅ |
| ۷ | Technology Selection Matrix | ✅ |
| ۸ | Proposed POC Architecture | ✅ |
| ۹ | Physical Deployment Architecture | ✅ |
| ۱۰ | End-to-End Data Flow | ✅ |
| ۱۱ | Hardware Requirements and Resource Estimation | ✅ |
| ۱۲ | Offline Deployment Strategy | ✅ |
| ۱۳ | Security and Access Control | ✅ |
| ۱۴ | Implementation Roadmap | ✅ |
| ۱۵ | Acceptance Criteria | ✅ |
| ۱۶ | Risks and Technical Limitations | ✅ |
| ۱۷ | References | ✅ |
| ۱۸ | Final Recommended Technology Stack / Resource Estimation | ✅ |

سند طراحی POC از نظر ساختاری کامل است. تمام انتخاب‌ها در وضعیت **Proposed / Recommended** باقی می‌مانند و پیش از تبدیل شدن به تصمیم معماری مصوب باید بررسی و تأیید شوند.
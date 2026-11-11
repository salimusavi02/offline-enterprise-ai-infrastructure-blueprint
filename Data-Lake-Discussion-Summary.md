# Data Lake Architecture & Integration — Discussion Summary

**Version:** 2.0  
**Status:** Proposed — Pending Review  
**Updated:** 2026-11-10  
**Project:** Offline Enterprise AI Infrastructure Blueprint

---

## 1. هدف و دامنه سند

این سند تصمیمات، اصول و ملاحظات مربوط به طراحی و یکپارچه‌سازی Data Lake با زیرساخت سازمانی Document Intelligence و RAG را ثبت می‌کند.

هدف اصلی، ایجاد یک معماری ذخیره‌سازی On-Premise و Air-Gapped است که بتواند:

- اسناد خام و داده‌های ورودی سازمان را نگهداری کند.
- مصنوعات حاصل از پردازش اسناد را به‌صورت نسخه‌دار ذخیره کند.
- Chunkهای canonical و قابل ردیابی را نگهداری کند.
- داده‌های ساختاریافته و نیمه‌ساختاریافته را در صورت نیاز پشتیبانی کند.
- Metadata، Lineage و وضعیت پردازش را مدیریت کند.
- امکان بازسازی Indexهای جست‌وجو را فراهم کند.
- کنترل دسترسی و الزامات امنیتی سازمان را رعایت کند.

این سند مکمل سند معماری اصلی پروژه است و مسئولیت تعریف کامل Logical Architecture، Physical Deployment و End-to-End Data Flow را بر عهده ندارد.

**سندهای مرجع معماری:**

1. `Enterprise AI Infrastructure_ Document Intelligence & RAG Ecosystem.md`
2. `End-to-End Data Flow.md`

در صورت بروز تعارض، تصمیمات مصوب در اسناد معماری مرجع و ADRهای تأییدشده بر محتوای تاریخی این سند اولویت دارند.

---

## 2. Data Lake چیست و چه نقشی در پروژه دارد؟

Data Lake یک مخزن متمرکز برای نگهداری داده‌های خام و مصنوعات پردازش‌شده در قالب‌های مختلف است.

در این پروژه، Data Lake برای نگهداری موارد زیر استفاده می‌شود:

- Original Documents: فایل‌های اصلی مانند PDF، DOCX، XLSX و سایر فرمت‌های مجاز
- Extracted Content: متن، جدول و محتوای استخراج‌شده
- Normalized Content: محتوای حاصل از نرمال‌سازی
- Canonical Chunks: قطعات مرجع تولیدشده از اسناد
- Processing Artifacts: خروجی‌های میانی و نهایی Pipeline
- Curated Data: داده‌های پالایش‌شده و مصنوعات آماده مصرف در کاربردهای سازمانی

Data Lake جایگزین تمام پایگاه‌های داده و Indexهای سامانه نیست. هر نوع داده باید در محل متناسب با مسئولیت و الگوی دسترسی خود نگهداری شود.

### 2.1 تفاوت Data Lake با Storageهای دیگر

| نوع Storage | ابزار اصلی | مسئولیت |
|---|---|---|
| Data Lake | MinIO | نگهداری فایل‌های اصلی و مصنوعات محتوایی |
| Operational Storage | PostgreSQL | Metadata، وضعیت، ACL، Lineage و Workflow State |
| Vector Index | Qdrant | نگهداری Embedding و داده‌های مورد نیاز Vector Retrieval |
| Lexical Index | OpenSearch | نگهداری متن Index‌شده و ساختار BM25 |

این تفکیک برای جلوگیری از چندگانگی Source of Truth و ایجاد وابستگی نامناسب میان اجزای سامانه ضروری است.

---

## 3. تصمیم معماری Data Lake

### 3.1 ابزار اصلی

**تصمیم مصوب: MinIO**

MinIO به‌عنوان Object Storage اصلی Data Lake انتخاب شده است.

دلایل:

- سازگاری با S3 API
- قابلیت استقرار On-Premise
- امکان اجرای کاملاً آفلاین
- پشتیبانی از Object Versioning و قابلیت‌های مرتبط با مدیریت اشیا
- امکان جداسازی دسترسی و Bucketها
- مناسب بودن برای نگهداری اسناد و مصنوعات حجیم
- استقلال نسبی از موتورهای پردازش و جست‌وجو

استفاده از MinIO به‌معنای آن نیست که تمام داده‌های سامانه باید در آن ذخیره شوند. داده‌های عملیاتی و Indexها همچنان در Storage تخصصی خود نگهداری خواهند شد.

### 3.2 جایگاه در معماری منطقی

Data Lake یک جزء از لایه **Data & Persistence** در معماری منطقی هفت‌لایه پروژه است.

این جزء از طریق Adapterها و Portهای مشخص در اختیار سرویس‌های بالادستی قرار می‌گیرد.

Data Lake نباید به‌عنوان یک لایه منطقی مستقل و خارج از معماری اصلی تعریف شود.

### 3.3 اصل استقلال از فناوری

سرویس‌های دامنه نباید مستقیماً به API اختصاصی MinIO وابسته باشند.

دسترسی به Object Storage باید از طریق یک قرارداد انتزاعی مانند `ObjectStoragePort` انجام شود.

در پیاده‌سازی فعلی، MinIO Adapter این قرارداد را پیاده‌سازی خواهد کرد.

---

## 4. ساختار منطقی Data Lake

Data Lake به سه Zone منطقی تقسیم می‌شود.

### 4.1 Raw Zone

**هدف:** نگهداری نسخه اصلی داده‌های ورودی.

محتوا:

- فایل اصلی اسناد
- فایل‌های ورودی از منابع سازمانی
- Snapshot یا Export خام از منابع داده
- داده‌های اولیه قبل از پردازش

اصول:

- حفظ Original Binary
- عدم اعمال Normalization یا تغییر محتوایی روی فایل اصلی
- ثبت شناسه منبع و نسخه سند
- قابلیت ردیابی به درخواست یا فرآیند Ingestion
- اعمال سیاست Retention و کنترل دسترسی
- استفاده از Hash برای تشخیص تغییر و Duplicate

Raw Zone باید مرجع بازیابی محتوای اولیه باشد.

### 4.2 Processed Zone

**هدف:** نگهداری مصنوعات حاصل از پردازش و نسخه‌های canonical محتوا.

محتوا:

- Extracted Text
- Normalized Text
- Canonical Chunks
- Chunk Manifests
- Processing Metadata
- خروجی‌های ساختاریافته حاصل از Parsing
- مصنوعات مورد نیاز برای بازسازی Indexها

**تصمیم مصوب:** نسخه مرجع Chunkها در Processed Zone نگهداری می‌شود.

Qdrant و OpenSearch نباید تنها محل نگهداری محتوای مرجع Chunk باشند.

PostgreSQL نیز متن کامل Chunk را به‌عنوان محل نگهداری اصلی آن ذخیره نمی‌کند؛ مسئولیت PostgreSQL نگهداری Metadata، شناسه‌ها، وضعیت‌ها و Lineage است.

### 4.3 Curated Zone

**هدف:** نگهداری داده‌های پالایش‌شده و آماده مصرف در کاربردهای بالادستی.

نمونه‌ها:

- Entity-Centric Documents
- داده‌های تجمیع‌شده سازمانی
- مصنوعات آماده تحلیل
- خروجی‌های پردازش‌شده برای کاربردهای خاص

استفاده از Curated Zone برای Entity-Centric RAG یک قابلیت توسعه‌ای است و جزئیات آن باید در چارچوب نیازمندی‌های واقعی و تصمیمات بعدی طراحی شود.

وجود این Zone به‌معنای الزام به پیاده‌سازی فوری Data Warehouse یا Lakehouse نیست.

---

## 5. ساختار پیشنهادی Object Key

ساختار زیر یک قرارداد منطقی پیشنهادی برای نام‌گذاری Objectها است:

```text
data-lake/
├── raw/
│   └── {source_id}/
│       └── {document_id}/
│           └── {document_version_id}/
│               └── original
│
├── processed/
│   └── {document_id}/
│       └── {document_version_id}/
│           ├── extracted/
│           ├── normalized/
│           ├── chunks/
│           └── manifests/
│
└── curated/
    └── {dataset_or_entity_type}/
        └── {artifact_version}/
```

این ساختار بیانگر Namespace منطقی است؛ الزاماً به‌معنای ایجاد دایرکتوری واقعی در Object Storage نیست.

### 5.1 اصول نام‌گذاری

- استفاده از شناسه‌های پایدار به‌جای نام فایل به‌عنوان کلید اصلی
- جلوگیری از وابستگی ساختار ذخیره‌سازی به نام‌های فارسی یا کاراکترهای خاص
- ثبت Original Filename در Metadata
- استفاده از UTF-8 برای محتوای متنی و Metadata
- نسخه‌بندی مستقل Document، Artifact و Processing Pipeline
- پرهیز از قرار دادن اطلاعات حساس یا شخصی در Object Key

### 5.2 Object Metadata

در صورت نیاز، Metadata مربوط به Object می‌تواند شامل موارد زیر باشد:

- `document_id`
- `document_version_id`
- `artifact_id`
- `artifact_type`
- `content_hash`
- `schema_version`
- `pipeline_run_id`
- `created_at`

Metadata موجود در Object Storage جایگزین PostgreSQL Catalog نیست.

---

## 6. مرزبندی Source of Truth

در این معماری یک Source of Truth واحد برای تمام انواع داده وجود ندارد. هر بخش مالک مرجع داده مربوط به خود است.

| نوع داده | مرجع اصلی | توضیح |
|---|---|---|
| Original Binary | MinIO / Raw | نسخه اصلی ورودی |
| Extracted Content | MinIO / Processed | خروجی Parsing |
| Normalized Content | MinIO / Processed | خروجی Normalization |
| Canonical Chunks | MinIO / Processed | مرجع محتوای Chunk |
| Chunk Metadata | PostgreSQL | شناسه، موقعیت، نسخه و ارتباطات |
| Document Metadata | PostgreSQL | مشخصات و نسخه‌های سند |
| ACL و Access Policy | PostgreSQL | مرجع مجوزهای عملیاتی |
| Pipeline State | PostgreSQL | وضعیت و تاریخچه پردازش |
| Data Lineage | PostgreSQL | روابط میان منابع و مصنوعات |
| Vector Embeddings | Qdrant | داده مشتق‌شده از Chunk |
| Lexical Index | OpenSearch | داده مشتق‌شده برای BM25 |

### 6.1 اصل Rebuildability

Indexهای Qdrant و OpenSearch داده‌های مشتق‌شده هستند.

در صورت خرابی یا حذف Indexها، امکان بازسازی آن‌ها باید با استفاده از Canonical Chunks و Metadata معتبر وجود داشته باشد.

بنابراین:

- MinIO مرجع محتوای اصلی و پردازش‌شده است.
- PostgreSQL مرجع Metadata، مجوز و وضعیت عملیاتی است.
- Qdrant و OpenSearch مرجع داده‌های Index‌شده نیستند.

---

## 7. رابطه Data Lake با PostgreSQL

PostgreSQL نقش Operational Metadata Store و Catalog را بر عهده دارد.

مسئولیت‌های آن شامل:

- مدیریت Sources
- مدیریت Documents و Document Versions
- ثبت Artifactها و محل نگهداری آن‌ها
- ثبت Processing Runs و Processing Steps
- ثبت Chunk Manifests و Metadata
- مدیریت ACL و Access Policies
- مدیریت Pipeline State
- ثبت Index Registry
- مدیریت Transactional Outbox
- ثبت Audit Events

### 7.1 موجودیت‌های منطقی پیشنهادی

| Entity | مسئولیت |
|---|---|
| `sources` | تعریف منابع داده |
| `documents` | هویت منطقی سند |
| `document_versions` | نسخه‌های سند |
| `artifacts` | ثبت مصنوعات ذخیره‌شده در MinIO |
| `pipeline_runs` | اجرای Pipeline |
| `pipeline_steps` | مراحل پردازش |
| `chunk_manifests` | مشخصات و ارتباط Chunkها |
| `access_policies` | سیاست‌های دسترسی |
| `groups` | گروه‌های دسترسی |
| `document_acl` | مجوزهای سند |
| `index_registry` | نسخه‌های Index |
| `outbox_events` | رویدادهای قابل انتشار |
| `audit_events` | رخدادهای ممیزی |

این فهرست Logical Data Model پیشنهادی است و هنوز به‌معنای تصویب Schema نهایی PostgreSQL یا Migration Script نیست.

### 7.2 اصل نگهداری Chunk

PostgreSQL فقط Metadata و Lineage مربوط به Chunk را نگهداری می‌کند.

متن canonical Chunk در MinIO قرار می‌گیرد.

نمونه Metadata منطقی:

```json
{
  "chunk_id": "uuid",
  "document_id": "uuid",
  "document_version_id": "uuid",
  "artifact_id": "uuid",
  "chunk_index": 12,
  "content_hash": "sha256",
  "token_count": 420,
  "language": "fa",
  "page_start": 4,
  "page_end": 5
}
```

متن واقعی Chunk در Artifact مربوط به Processed Zone نگهداری می‌شود.

---

## 8. Data Lake و Hybrid Search

معماری جست‌وجوی ترکیبی پروژه بر دو موتور مستقل استوار است.

### 8.1 Qdrant

مسئولیت:

- ذخیره Embeddingها
- Vector Similarity Search
- نگهداری شناسه Chunk
- نگهداری Metadata لازم برای Filtering
- پشتیبانی از فیلترهای ACL در حد قابلیت‌های موتور

Qdrant محل نگهداری مرجع محتوای Chunk نیست.

### 8.2 OpenSearch

مسئولیت:

- نگهداری متن Index‌شده
- BM25 Retrieval
- Keyword Search
- فیلتر Metadata
- پشتیبانی از تحلیل و پردازش متنی متناسب با زبان

انتخاب Analyzer و تنظیمات اختصاصی زبان فارسی هنوز نیازمند Benchmark و تصمیم مستقل است.

### 8.3 ادغام نتایج

نتایج Vector Search و BM25 با روش Reciprocal Rank Fusion (RRF) ادغام می‌شوند.

پس از ادغام، اعتبار مجوزها و وضعیت سند باید مجدداً با مرجع عملیاتی PostgreSQL بررسی شود.

### 8.4 Rebuild

برای بازسازی Indexها:

1. شناسایی نسخه فعال سند
2. بازیابی Canonical Chunkها از MinIO
3. بررسی Metadata و Lineage در PostgreSQL
4. تولید Embedding با مدل و نسخه مشخص
5. ساخت Vector Index در Qdrant
6. ساخت Lexical Index در OpenSearch
7. اعتبارسنجی Indexها
8. ثبت نسخه Index و وضعیت آن

Index ناقص نباید برای Retrieval فعال شود.

---

## 9. پردازش داده‌های ساختاریافته

علاوه بر اسناد، ممکن است در آینده داده‌های دیتابیس‌های سازمانی مانند ERP، CRM و HR نیز وارد سامانه شوند.

نمونه فرمت‌های مناسب برای نگهداری داده‌های ساختاریافته:

- Parquet
- JSON یا JSONL
- CSV برای تبادل ساده
- فرمت‌های جدولی نسخه‌دار در صورت نیاز

### 9.1 Parquet

Parquet گزینه مناسبی برای ذخیره داده‌های جدولی و پردازش تحلیلی است.

مزایا:

- Columnar Storage
- فشرده‌سازی مناسب
- سازگاری با ابزارهای تحلیلی
- امکان خواندن انتخابی ستون‌ها
- مناسب برای پردازش Batch

با این حال، استفاده از Parquet برای تمام داده‌های سازمانی الزام معماری نیست.

انتخاب فرمت باید بر اساس نوع منبع، حجم، نیاز تحلیلی و الگوی مصرف انجام شود.

### 9.2 Partitioning

در صورت نیاز به نگهداری Datasetهای حجیم، Partitioning می‌تواند بر اساس مواردی مانند این‌ها انجام شود:

- منبع داده
- تاریخ استخراج
- نوع موجودیت
- نسخه Dataset

نمونه:

```text
curated/
└── erp/
    └── customers/
        └── extract_date=2026-11-10/
            └── part-0001.parquet
```

این Convention پیشنهادی است و باید با الگوی واقعی Query و حجم داده اعتبارسنجی شود.

### 9.3 Entity-Centric Documents

برای بعضی کاربردهای RAG، ایجاد اسناد غنی حول موجودیت‌های سازمانی مانند Customer، Order و Product می‌تواند کیفیت بازیابی را افزایش دهد.

اما این رویکرد باید از مسیر مستقل زیر ارزیابی شود:

1. شناسایی موجودیت‌های کسب‌وکاری
2. تعریف Schema
3. تعیین روابط و کلیدهای طبیعی
4. ساخت نمونه Curated Document
5. ارزیابی Retrieval و پاسخ‌گویی
6. تصمیم درباره ورود به معماری عملیاتی

این قابلیت در حال حاضر یک توسعه احتمالی است، نه الزام نسخه پایه.

---

## 10. ابزارهای توسعه‌ای Data Lake

| قابلیت | گزینه | وضعیت |
|---|---|---|
| Object Storage | MinIO | مصوب |
| Operational Catalog | PostgreSQL | مصوب |
| Structured File Format | Parquet | گزینه مناسب برای Datasetهای جدولی |
| SQL Query Engine | DuckDB | گزینه بررسی برای PoC |
| Distributed Query Engine | Trino | توسعه احتمالی |
| Table Format | Apache Iceberg | توسعه احتمالی |
| Advanced Catalog | Hive Metastore / Nessie | نیازمند تصمیم آینده |
| Data Processing | Python-based Workers | متناسب با معماری پردازش |

### 10.1 DuckDB

DuckDB می‌تواند برای تحلیل محلی فایل‌های Parquet و اجرای Queryهای تحلیلی مفید باشد.

استفاده از آن در فاز اولیه باید بر اساس نیاز واقعی و PoC ارزیابی شود.

### 10.2 Apache Iceberg

Iceberg قابلیت‌هایی مانند Table Versioning، Schema Evolution و مدیریت Snapshot را فراهم می‌کند.

با این حال، استفاده از آن به‌معنای نیاز فوری پروژه نیست.

افزودن Iceberg باید پس از مشخص‌شدن حجم داده، الگوی Query، نیازهای Concurrent Access و پیچیدگی عملیاتی بررسی شود.

---

## 11. ملاحظات فارسی و چندزبانه

Data Lake باید امکان نگهداری محتوای فارسی، انگلیسی و اسناد Mixed-Language را فراهم کند.

### 11.1 Encoding

- فایل‌های متنی و Metadata باید با Encoding معتبر ذخیره شوند.
- UTF-8 باید در اجزای مرتبط با متن به‌صورت یکپارچه پشتیبانی شود.
- Original Binary نباید به دلیل ملاحظات زبانی تغییر کند.
- نام اصلی فایل در Metadata حفظ شود.
- نام Objectها بهتر است از شناسه‌های پایدار و ASCII-safe استفاده کند.

### 11.2 Normalization

Normalization باید در Pipeline پردازش و با نسخه مشخص انجام شود.

مواردی که نیازمند تعیین Policy هستند:

- NFC یا NFKC
- نگاشت حروف عربی به فارسی
- مدیریت ZWNJ
- مدیریت فاصله و نیم‌فاصله
- مدیریت اعداد فارسی و لاتین
- حذف کاراکترهای کنترلی
- حفظ اطلاعات ساختاری و معنایی متن

**هیچ Policy نهایی برای NFC/NFKC یا تبدیل اعداد در این سند تصویب نمی‌شود.**

این تصمیم باید در قرارداد مستقل Normalization و پس از ارزیابی اثر آن بر متن، جست‌وجو و Retrieval ثبت شود.

### 11.3 Language Metadata

در صورت نیاز، Metadata زبان می‌تواند شامل موارد زیر باشد:

- `language`
- `script_direction`
- `language_detection_method`

تشخیص زبان باید بتواند اسناد Mixed-Language را نیز مدیریت کند.

---

## 12. امنیت و کنترل دسترسی

Data Lake بخشی از محدوده امنیتی اصلی سامانه است.

اصول الزامی:

- Least Privilege
- جداسازی دسترسی سرویس‌ها
- عدم انتشار مستقیم پورت‌های داخلی MinIO به شبکه کاربران
- استفاده از TLS در ارتباطات لازم
- مدیریت امن Credentials و Secrets
- ثبت رخدادهای حساس
- رمزنگاری Backupها
- کنترل دسترسی به Raw، Processed و Curated Zone

### 12.1 MinIO Access Control

سیاست‌های دسترسی باید بر اساس نقش سرویس‌ها و نیاز واقعی آن‌ها طراحی شوند.

نمونه:

- Ingestion Service: نوشتن در Raw Zone
- Processing Service: خواندن Raw و نوشتن Processed
- Retrieval Service: خواندن کنترل‌شده از Processed
- Administration Service: دسترسی مدیریتی محدود

این‌ها الگوی منطقی دسترسی هستند؛ Policy نهایی IAM باید جداگانه طراحی و تست شود.

### 12.2 ACL در Retrieval

PostgreSQL مرجع عملیاتی ACL است.

فیلترکردن نتایج در Qdrant و OpenSearch به‌تنهایی تضمین امنیت محسوب نمی‌شود.

پیش از Fetch کردن Chunk از MinIO و ورود آن به Context مدل، مجوز باید در برابر مرجع معتبر بررسی شود.

در صورت نامشخص بودن مجوز، رفتار سامانه باید Fail Closed باشد.

---

## 13. Versioning و Data Lineage

نسخه‌بندی باید در چند سطح مستقل انجام شود.

### 13.1 انواع Version

1. Document Version
2. Processing Pipeline Version
3. Normalization Version
4. Chunking Version
5. Embedding Model Version
6. Index Version

تغییر در هر یک از این موارد ممکن است باعث تولید Artifact یا Index جدید شود.

### 13.2 Data Lineage

برای هر Artifact باید امکان ردیابی به منبع اولیه وجود داشته باشد.

نمونه رابطه:

```text
Source
  └── Document
       └── Document Version
            └── Raw Artifact
                 └── Processing Run
                      ├── Extracted Artifact
                      ├── Normalized Artifact
                      └── Chunk Artifact
                           ├── Embedding
                           ├── Qdrant Index
                           └── OpenSearch Index
```

Lineage باید حداقل موارد زیر را پوشش دهد:

- منبع اولیه
- نسخه سند
- شناسه اجرای Pipeline
- نسخه Parser
- نسخه Normalization
- نسخه Chunking
- نسخه مدل Embedding
- شناسه Artifactهای خروجی
- نسخه Indexهای تولیدشده

---

## 14. Lifecycle، Update و Delete

### 14.1 Document Lifecycle

چرخه عمر سند در PostgreSQL مدیریت می‌شود.

وضعیت‌های اصلی:

```text
RECEIVED
   ↓
PROCESSING
   ↓
PROCESSED
   ↓
INDEXING
   ↓
VALIDATING
   ↓
ACTIVE
```

وضعیت‌های استثنایی می‌توانند شامل موارد زیر باشند:

- FAILED
- REJECTED
- SUPERSEDED
- DELETED

### 14.2 فعال‌سازی نسخه جدید

نسخه جدید نباید صرفاً با پایان Parsing فعال شود.

شرایط Active شدن باید شامل موارد زیر باشد:

- Artifactهای Processed معتبر و قابل بازیابی باشند.
- Metadata و Lineage ثبت شده باشند.
- Vector Index آماده و معتبر باشد.
- Lexical Index آماده و معتبر باشد.
- نسخه‌ها با یکدیگر سازگار باشند.
- کنترل‌های ACL تأیید شده باشند.

نسخه قبلی تا آماده‌شدن کامل نسخه جدید می‌تواند فعال باقی بماند.

### 14.3 Update

در زمان به‌روزرسانی سند:

1. نسخه جدید ایجاد می‌شود.
2. Pipeline مستقل برای نسخه جدید اجرا می‌شود.
3. Artifactهای جدید ذخیره می‌شوند.
4. Indexهای جدید ساخته و اعتبارسنجی می‌شوند.
5. نسخه جدید Active می‌شود.
6. نسخه قبلی SUPERSEDED می‌شود.

### 14.4 Delete

حذف سند باید هماهنگ میان اجزای سامانه انجام شود:

1. ثبت Tombstone در PostgreSQL
2. خارج‌کردن سند از Retrieval
3. حذف یا غیرفعال‌سازی رکوردهای Qdrant
4. حذف یا غیرفعال‌سازی رکوردهای OpenSearch
5. اعمال سیاست Retention در MinIO
6. ثبت Audit Event
7. اجرای Retry و Reconciliation در صورت شکست

حذف از Index نباید جایگزین سیاست نگهداری و حذف Artifactهای اصلی شود.

---

## 15. Backup و Recovery

Backup باید تمام اجزای دارای داده ارزشمند را پوشش دهد.

| جزء | راهبرد |
|---|---|
| MinIO | Backup یا Replication به مقصد مستقل |
| PostgreSQL | Logical/Physical Backup و در صورت نیاز WAL |
| Qdrant | Snapshot یا Rebuild از Canonical Data |
| OpenSearch | Snapshot یا Rebuild از Canonical Data |
| Model Vault | Backup و Hash Verification |
| Configuration | نسخه‌گذاری و Backup |
| Secrets | Backup رمزنگاری‌شده با کنترل دسترسی محدود |

### 15.1 اصول

- Backup روی همان Host به‌تنهایی کافی نیست.
- مقصد Backup باید از خرابی Host اصلی مستقل باشد.
- صحت Backup باید به‌صورت دوره‌ای بررسی شود.
- Restore Test باید اجرا و مستند شود.
- RPO و RTO باید توسط مالک سامانه تعیین شوند.
- Retention Policy باید با الزامات سازمانی هماهنگ باشد.

مقادیر نهایی RPO/RTO و Retention در این سند تصویب نشده‌اند.

---

## 16. ملاحظات محیط Air-Gapped

تمام اجزای Data Lake باید بدون اتصال به اینترنت قابل بهره‌برداری باشند.

موارد ضروری:

- نگهداری Container Imageهای مورد نیاز در Registry داخلی
- نگهداری Model و Artifactها در Model Vault
- ثبت و اعتبارسنجی Hash فایل‌های واردشده
- مدیریت نسخه و منبع بسته‌های نرم‌افزاری
- مستندسازی فرآیند به‌روزرسانی آفلاین
- کنترل دسترسی به رسانه‌های انتقال داده
- ثبت رخدادهای Import و Export

MinIO نباید به سرویس خارجی برای عملکرد اصلی خود وابسته باشد.

---

## 17. تصمیمات مصوب و موضوعات باز

### 17.1 تصمیمات مصوب

| شناسه | تصمیم | وضعیت |
|---|---|---|
| ADR-002 | MinIO به‌عنوان Object Storage اصلی | مصوب |
| ADR-004 | Canonical Chunks در MinIO / Processed Zone | مصوب |
| — | PostgreSQL برای Metadata و Operational State | مصوب در بسته معماری |
| — | Qdrant برای Vector Index | مصوب در بسته معماری |
| — | OpenSearch برای BM25 Index | مصوب در بسته معماری |
| — | Indexها مشتق‌شده و قابل بازسازی هستند | اصل معماری مصوب |

### 17.2 موضوعات باز

| موضوع | وضعیت | خروجی مورد انتظار |
|---|---|---|
| NFC در برابر NFKC | باز | Normalization Policy |
| Chunk Size و Overlap | باز | Benchmark و تصمیم نهایی |
| Tokenizer | باز | انتخاب و ثبت نسخه |
| PostgreSQL Schema | باز | Logical/Physical Data Model |
| MinIO IAM Policy | باز | Access Matrix و Policy |
| Persian Analyzer | باز | OpenSearch Benchmark |
| Parquet Usage | نیازمند ارزیابی | تصمیم بر اساس Datasetهای واقعی |
| DuckDB/Trino | نیازمند PoC | ارزیابی کاربرد |
| Iceberg | توسعه احتمالی | تصمیم در مرحله بلوغ |
| Entity-Centric Documents | نیازمند Spike | PoC و ارزیابی Retrieval |
| RPO/RTO | باز | اهداف Recovery |
| Backup Retention | باز | سیاست نگهداری Backup |

---

## 18. گام‌های پیشنهادی بعدی

گام‌ها باید بر اساس وابستگی‌های معماری و اولویت پیاده‌سازی انجام شوند.

### مرحله 1 — تثبیت قراردادهای ذخیره‌سازی

- نهایی‌سازی Object Key Convention
- تعریف Artifact Metadata
- تعریف Chunk Manifest
- تثبیت قرارداد MinIO و PostgreSQL
- تعیین مرز مالکیت داده‌ها

### مرحله 2 — طراحی Operational Data Model

- طراحی جداول اصلی PostgreSQL
- تعریف Document Lifecycle
- طراحی Versioning
- طراحی Data Lineage
- تعریف Outbox و Reconciliation

### مرحله 3 — طراحی Data Lake Security

- طراحی Bucket Policy
- تعریف Service Identity
- تعریف IAM Policy
- طراحی Backup Security
- تست دسترسی و جداسازی سرویس‌ها

### مرحله 4 — ارزیابی داده‌های ساختاریافته

- بررسی منابع ERP/CRM/HR
- ارزیابی Parquet
- اجرای PoC با DuckDB در صورت نیاز
- بررسی Entity-Centric Documents

### مرحله 5 — توسعه قابلیت‌های پیشرفته

- ارزیابی Iceberg
- ارزیابی Trino
- توسعه Catalog در صورت نیاز
- توسعه Governance و Lineage پیشرفته

---

## 19. جمع‌بندی معماری

معماری Data Lake پروژه بر پایه MinIO طراحی می‌شود، اما Data Lake به‌تنهایی مالک تمام داده‌های سامانه نیست.

مرزبندی نهایی:

- **MinIO:** مرجع فایل‌های اصلی و مصنوعات محتوایی، از جمله Canonical Chunks
- **PostgreSQL:** مرجع Metadata، ACL، Lifecycle، State و Lineage
- **Qdrant:** Vector Index
- **OpenSearch:** Lexical/BM25 Index

این تفکیک امکان بازسازی Indexها، ردیابی نسخه‌ها، کنترل دسترسی و مدیریت مستقل چرخه عمر داده را فراهم می‌کند.

هر توسعه جدید در حوزه Data Lake باید با این مرزبندی و با دو سند معماری مرجع پروژه سازگار باشد.

---

*End of Document — Proposed Version 2.0*
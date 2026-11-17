# راهنمای پیاده‌سازی POC

**نسخهٔ سند**: ۱.۱
**تاریخ آخرین به‌روزرسانی**: ۱۴۰۵/۰۸/۲۶ (2026-11-17)
**مبنای طراحی**: مستند طراحی POC زیرساخت هوش مصنوعی سازمانی آفلاین (نسخهٔ ۰.۳) + معماری منطقی POC — شرح تفصیلی لایه‌ها (نسخهٔ ۱.۰) + SINA_Roadmap_v1.5
**تغییرات نسخهٔ ۱.۱**: ثبت تصمیمات نهایی بازبینی Phase 0 (C1–C9، همه گزینهٔ الف) در بخش ۱۱؛ هم‌راستاسازی نقشهٔ راه با Taskهای Phase 0 (بخش ۱)؛ درج ملاحظات Compose، اسکیما و Bundle مربوط به تصمیمات (بخش‌های ۳، ۶ و ۷)
**دامنه**: تبدیل تصمیمات معماری موجود به کد، اسکریپت، اسکیما و سناریوی آزمونِ قابل اجرا
**زبان**: فارسی، با حفظ نام‌های فنی انگلیسی

---

## شناسنامهٔ سند

### هدف

دو مستند فعلی به این پرسش‌ها پاسخ می‌دهند: «چه چیزی بسازیم؟» (نسخهٔ ۰.۳) و «لایه‌ها چگونه به هم متصل می‌شوند؟» (معماری منطقی). اما به این پرسش‌ها پاسخ نمی‌دهند: **از کجا شروع کنیم؟** (ترتیب واقعی پیاده‌سازی)، **کد را چگونه سازمان دهیم؟** (ساختار پروژه و نمونه‌های حداقلی)، **چگونه تست کنیم؟** (سناریوهای یکپارچگی و معیارهای عبور)، و **چگونه استقرار آفلاین را عملی کنیم؟** (اسکریپت‌های واقعی).

این سند **پل بین طراحی و اجرا** است و دقیقاً همان چیزی است که یک تیم ۲–۳ نفره در Phase 1 نیاز دارد.

### مخاطب هدف

- **توسعه‌دهندگان بک‌اند**: برای شروع کدنویسی از نمونه‌های این سند، نه از صفر
- **مهندس DevOps / استقرار**: برای اجرای بستهٔ آفلاین، Backup/Restore و عیب‌یابی
- **QA و ارزیابان فنی**: برای اجرای سناریوهای آزمون T1–T8 و تکمیل چک‌لیست Go/No-Go

### این سند «POC حداقلی» را نقض نمی‌کند

۱. **هیچ ابزار یا قابلیت جدیدی اضافه نمی‌کند** — فقط آنچه در دو مستند فعلی هست به «کد، اسکریپت و اسکیمای قابل اجرا» تبدیل می‌شود.
۲. **هیچ تصمیم معماری جدیدی نمی‌گیرد** — فقط تصمیمات موجود را پیاده‌سازی می‌کند.
۳. **در همان ۶ کانتینر و همان ۱۰ لایه باقی می‌ماند** — هیچ لایه یا سرویس جدیدی اضافه نمی‌شود.
۴. **همهٔ نام‌های فایل و توابع با قراردادهای معماری منطقی سازگارند** — (مثلاً `parse_docx()`، `rrf_fuse()`، `get_current_user()`، جدول `processing_jobs`).
۵. **سیاست نسخه‌ها مطابق دو مستند فعلی است**: هر نسخه یا «نیازمند تأیید» است یا به‌عنوان «پیشنهاد برای قفل‌سازی در Phase 1» علامت خورده — هیچ نسخه‌ای به‌عنوان تصمیم مصوب ثبت نشده است.

### قراردادهای این سند

- همهٔ فایل‌های فنی (schema، compose، اسکریپت‌ها) عیناً قابل کپی در مخزن پروژه هستند، اما قبل از commit باید بازبینی و تأیید شوند.
- هر جا یک مقدار (نسخه، پارامتر، برآورد) «پیشنهاد» است نه تصمیم، صریحاً برچسب **نیازمند تأیید** یا **پیشنهادی** دارد.
- برآوردهای زمانی تخمین‌های مهندسی‌اند (روز کاری برای تیم ۲–۳ نفره با آشنایی متوسط به Python و Docker)، نه تعهد زمان‌بندی.

---

# بخش ۱ — نقشهٔ راه پیاده‌سازی گام‌به‌گام

به‌جای Roadmap سطح بالا (بخش ۱۴ نسخهٔ ۰.۳)، این بخش **ترتیب واقعی اجرا** را با معیار اتمام قابل اندازه‌گیری برای هر گام تعریف می‌کند.

| گام | کار | خروجی | معیار اتمام (DoD) | تخمین |
|---|---|---|---|---|
| ۱ | راه‌اندازی Docker Compose با ۶ کانتینر خالی (بدون کد) | `docker compose up` موفق | هر ۶ سرویس `healthy`؛ فقط پورت ۸۵۰۱ منتشر شده | ۰.۵ روز |
| ۲ | پیاده‌سازی L7 (Persistence): جداول PG، bucketهای MinIO | `schema.sql` + bucketها | اجرای `psql -f schema.sql` بدون خطا؛ سه bucket ساخته شود | ۱ روز |
| ۳ | پیاده‌سازی L8 (Security): Auth + ACL پایه | `POST /login` کار کند | login با رمز درست 200 و با رمز غلط 401 برگرداند | ۱ روز |
| ۴ | پیاده‌سازی L4 (Ingestion): پارس DOCX/XLSX + Chunking | تابع `parse_document()` تست‌شده | تست واحد با فایل نمونه، Chunkها با سرستون و `location_meta` صحیح | ۲ روز |
| ۵ | پیاده‌سازی L3 (Orchestration): Worker + State Machine | Job از `pending` به `active` برسد | آپلود یک DOCX تا وضعیت `active` بدون دست‌کاری دستی | ۱.۵ روز |
| ۶ | پیاده‌سازی L5-Embedding + L6 (Indexing): Qdrant/OS | Chunkها ایندکس شوند | تعداد Point در Qdrant = تعداد Doc در OS = تعداد ردیف در `chunks` | ۱ روز |
| ۷ | پیاده‌سازی L6 (Retrieval): RRF + ACL + Context Assembly | Query برگردد | پاسخ `/query` منابع معتبر با `location_meta` برگرداند | ۱.۵ روز |
| ۸ | پیاده‌سازی L5-LLM: اتصال به Ollama | پاسخ تولید شود | `POST /query` پاسخ متنی فارسی غیرخالی با منبع برگرداند | ۰.۵ روز |
| ۹ | پیاده‌سازی L1/L2: Streamlit UI + FastAPI نهایی | UI کار کند | چرخهٔ کامل login → upload → query در مرورگر | ۱ روز |
| ۱۰ | پیاده‌سازی L9 (Logging) + L10 (Offline Bundle) | بستهٔ آفلاین آماده | `load_bundle.sh` در ماشین ایزوله موفق؛ T1–T8 در آن پاس شوند | ۱ روز |

**جمع تخمین**: حدود ۱۰.۵ روز کاری (پیشنهادی — نیازمند تأیید در جلسهٔ برآورد تیم).

**گام صفر — Spike اندازه‌گیری عملکرد (تصمیم C8)**:

- پیش از قطعی‌سازی اعداد Quality Gate 0، یک **Spike نیم‌روزه در هفتهٔ ۱** روی سخت‌افزار هدف اجرا شود.
- خروجی Spike: تأخیر واقعی End-to-End پرسش با `qwen2.5:7b-instruct-q4_K_M` (پایهٔ قطعی **QG0.2**) و زمان استخراج/ایندکس‌گذاری ۱۰ Chunk با `multilingual-e5-large` (پایهٔ قطعی **QG0.3**).
- اعداد فعلی این دو معیار در Roadmap v1.5 فاقد مبنا هستند؛ پس از Spike، نتایج اندازه‌گیری باید در همان Roadmap ثبت و اعداد قطعی شوند.

**نگاشت نقشهٔ راه این سند به Taskهای Phase 0 (تصمیم C9)**:

- **تقسیم حاکمیت**: SINA_Roadmap_v1.5 حاکم بر **زمان‌بندی، Sprintها و Story Pointها**؛ این سند حاکم بر **«چگونه»** (کد، اسکریپت، اسکیما و سناریوهای آزمون).
- نگاشت مرجع: **گام ۱ → Task 2** (راه‌اندازی Compose)، **گام ۲ → Task 4** (پیاده‌سازی Persistence)، **گام ۳ → Sprint 3** (ورود به Ingestion).
- بقیهٔ گام‌ها (۴ تا ۱۰) در Sprintهای بعدی Roadmap v1.5 جای می‌گیرند؛ هر تغییر در زمان‌بندی در Roadmap ثبت می‌شود، نه در این سند.

**وابستگی‌ها و ترتیب اجباری**:

- گام ۲ پیش‌نیاز همهٔ گام‌های ۳ تا ۹ است (همه به PG و MinIO وابسته‌اند).
- گام ۵ (Worker) قبل از گام ۶ لازم است تا ایندکس‌گذاری در چرخهٔ Job جاری شود.
- گام ۷ (Retrieval) به گام ۶ وابسته است، اما گام ۸ (LLM) مستقل از گام ۷ قابل توسعه است (تست مستقل `POST /api/generate` روی Ollama از روز اول گام ۱ ممکن است).
- منطقی است که از روز اول، باقی‌ماندهٔ کارها روی همان Compose بخش ۷ اجرا شود تا هیچ کد «محلی» خارج از کانتینر عادت نشود.

**ارزش**: این جدول، Roadmap سطح بالا را به «کارهای قابل انجام روزانه» تبدیل می‌کند و هر عصر برای دموی روزانه معیار دارد.

---

# بخش ۲ — ساختار پروژه و الگوهای کدنویسی

بر اساس بخش ۱۸ معماری منطقی (Project Structure)، اما با **نمونه‌کد حداقلی** برای هر ماژول. اصل حاکم: **Modular Monolith** — یک فرایند Python، مرز ماژول‌ها = مرز لایه‌های منطقی.

## ۲.۱. ساختار پروژه

```
app/
├── main.py                     # نقطهٔ ورود: uvicorn FastAPI + bootstrap Streamlit
├── config.py                   # خواندن .env و تنظیمات typed
├── api/
│   ├── routes_auth.py          # POST /login, POST /logout
│   ├── routes_upload.py        # POST /upload
│   ├── routes_query.py         # POST /query
│   └── routes_documents.py     # GET /documents
├── security/
│   ├── auth.py                 # get_current_user, require_admin (Dependency)
│   └── acl.py                  # acl_revalidate(user_id, chunk_ids)
├── domain/
│   ├── worker/runner.py        # Polling Loop (فرایند جدا داخل همان کانتینر)
│   ├── state_machine.py        # transitionهای Document Lifecycle
│   └── idempotency.py          # بررسی content_hash
├── ingestion/
│   ├── parsers/docx_parser.py  # parse_docx()
│   ├── parsers/xlsx_parser.py  # parse_xlsx()
│   ├── chunker/docx_chunker.py
│   ├── chunker/xlsx_chunker.py # chunking هوشمند Excel
│   └── normalizer.py           # normalize_fa() — یکسان‌سازی ی/ک، ZWNJ، ارقام
├── embedding/
│   └── embedder.py             # Singleton SentenceTransformer + encode()
├── retrieval/
│   ├── vector_search.py        # qdrant search با filter document_status=active
│   ├── bm25_search.py          # OpenSearch query_string
│   ├── fusion.py               # rrf_fuse()
│   ├── acl_revalidate.py       # مرحلهٔ ۱۰ Query Flow — اجباری
│   └── context_assembler.py    # چیدمان Context با محدودیت توکن
├── storage/
│   ├── pg.py                   # Connection pool (psycopg)
│   ├── minio_client.py         # put_object/get_object
│   ├── qdrant_client.py
│   └── os_client.py
├── ui/
│   └── streamlit_app.py        # login، آپلود، پرسش‌وپاسخ، منابع
├── scripts/
│   ├── reconcile.py            # بازگرداندن failed → pending
│   ├── rebuild_index.py        # بازسازی Index از Canonical Data
│   └── evaluate_ragas.py       # ارزیابی آفلاین روی Golden Set
└── requirements.txt            # نسخه‌های قفل‌شده — نیازمند تأیید
```

## ۲.۲. الگوهای کدنویسی حاکم

- **توابع Pure در L4 و L6**: هیچ State ماژول‌سطحی؛ State فقط در PG و ایندکس‌ها.
- **Dependency Injection برای L8**: هیچ فراخوانی مستقیم Auth داخل routeها؛ فقط `Depends(get_current_user)`.
- **خطاها با Exception مشخص، Catch در L3**: توابع L4/L5/L6 فقط Exception پرتاب می‌کنند؛ سیاست Retry و `failed` شدن فقط در Worker.
- **Fail Closed در L6/L8**: هر خطا یا timeout در ACL → بازگشت نتیجهٔ خالی، بدون فراخوانی LLM.
- **ULID برای همهٔ شناسه‌ها** (`users`، `documents`، `chunks`، `processing_jobs`، `request_id`).

## ۲.۳. نمونه‌کدهای حداقلی

> این نمونه‌ها نقطهٔ شروع‌اند، نه کد نهایی. همه با نسخه‌های کتابخانهٔ «نیازمند تأیید» نوشته شده‌اند و در Phase 1 باید بازبینی شوند.

### `ingestion/parsers/docx_parser.py` — پارس DOCX

```python
# parse_docx — استخراج پاراگراف و جدول از DOCX (حدود ۲۰ خط)
import io
from docx import Document  # python-docx==1.1.0 — نیازمند تأیید
from ingestion.normalizer import normalize_fa

def parse_docx(file_bytes: bytes, document_id: str) -> list[dict]:
    doc = Document(io.BytesIO(file_bytes))
    elements, heading = [], None
    for para in doc.paragraphs:                       # Header/Footer طبق تصمیم POC رد می‌شود
        text = normalize_fa(para.text.strip())
        if not text:
            continue
        if para.style.name.startswith("Heading"):
            heading = text
        elements.append({"text": text, "source_type": "word_paragraph",
                         "location_meta": {"heading": heading}})
    for t_idx, table in enumerate(doc.tables):        # جدول‌ها به‌صورت خطی سرستون + سطرها
        header = " | ".join(c.text.strip() for c in table.rows[0].cells)
        for r_idx, row in enumerate(table.rows[1:], start=1):
            line = header + "\n" + " | ".join(c.text.strip() for c in row.cells)
            elements.append({"text": normalize_fa(line), "source_type": "word_paragraph",
                             "location_meta": {"page": None, "heading": heading,
                                               "table_index": t_idx, "row": r_idx}})
    return elements
```

### `ingestion/chunker/xlsx_chunker.py` — Chunking هوشمند Excel

```python
# chunking Excel: جدول کوچک (<۵۰ سطر) = یک Chunk با سرستون؛
# جدول بزرگ = Row Group با تکرار سرستون در هر Chunk (ROW_GROUP_SIZE=50 — قابل تنظیم)
import io
from openpyxl import load_workbook  # openpyxl==3.1.2 — نیازمند تأیید

ROW_GROUP_SIZE, SMALL_SHEET_ROWS = 50, 50

def chunk_xlsx(file_bytes: bytes) -> list[dict]:
    wb = load_workbook(io.BytesIO(file_bytes), data_only=True, read_only=True)
    chunks = []
    for ws in wb.worksheets:
        rows = [[("" if c is None else str(c)) for c in r] for r in ws.iter_rows(values_only=True)]
        if not rows:
            continue
        header, body = rows[0], rows[1:]
        base_meta = {"sheet_name": ws.title, "column_range": f"A1:{ws.max_column}{ws.max_row}"}
        if len(body) < SMALL_SHEET_ROWS:
            text = "\t".join(header) + "\n" + "\n".join("\t".join(r) for r in body)
            chunks.append({"content": text, "source_type": "excel_sheet_row",
                           "location_meta": {**base_meta, "row_range": f"1:{len(rows)}"}})
        else:
            for start in range(0, len(body), ROW_GROUP_SIZE):
                group = body[start:start + ROW_GROUP_SIZE]
                text = "\t".join(header) + "\n" + "\n".join("\t".join(r) for r in group)
                chunks.append({"content": text, "source_type": "excel_sheet_row",
                               "location_meta": {**base_meta,
                                                 "row_range": f"{start+2}:{start+1+len(group)}"}})
    return chunks
```

**نکتهٔ معماری**: تکرار سرستون در ابتدای هر Chunk بزرگ، دقیقاً همان قاعدهٔ بخش ۵ نسخهٔ ۰.۳ است و معیار قابل اندازه‌گیری Acceptance Criteria («بررسی وجود سرستون در ابتدای هر chunk جدول بزرگ») را تضمین می‌کند.

### `retrieval/fusion.py` — ادغام RRF

```python
# Reciprocal Rank Fusion — تابع خالص Python، k=60 (طبق معماری منطقی بخش ۸)
def rrf_fuse(vector_results: list[str], bm25_results: list[str], k: int = 60) -> dict[str, float]:
    scores: dict[str, float] = {}
    for rank, chunk_id in enumerate(vector_results, start=1):
        scores[chunk_id] = scores.get(chunk_id, 0.0) + 1.0 / (k + rank)
    for rank, chunk_id in enumerate(bm25_results, start=1):
        scores[chunk_id] = scores.get(chunk_id, 0.0) + 1.0 / (k + rank)
    return dict(sorted(scores.items(), key=lambda x: x[1], reverse=True))
```

### `security/auth.py` — Dependency احراز هویت

```python
# get_current_user — FastAPI Dependency (حدود ۱۵ خط)
from fastapi import Depends, HTTPException, Request

def get_current_user(request: Request):
    token = request.cookies.get("session_token")
    if not token:                                     # Fail Closed
        raise HTTPException(status_code=401, detail="missing session")
    payload = decode_jwt(token, secret=config.JWT_SECRET)   # python-jose==3.3.0 — نیازمند تأیید
    if payload is None or payload.get("exp", 0) < now_ts():
        raise HTTPException(status_code=401, detail="invalid or expired session")
    user = pg_fetch_one("SELECT user_id, username, role FROM users "
                        "WHERE user_id = %s AND is_active", payload["user_id"])
    if user is None:                                  # PG error / کاربر ناشناس → deny
        raise HTTPException(status_code=401, detail="unknown user")
    return {"user_id": user["user_id"], "username": user["username"], "role": user["role"]}

def require_admin(user=Depends(get_current_user)):
    if user["role"] != "admin":
        raise HTTPException(status_code=403, detail="admin required")
    return user
```

### `domain/worker/runner.py` — Polling Loop

```python
# Worker چرخشی — یک Job pending را با SKIP LOCKED برمی‌دارد و ماشین حالت را اجرا می‌کند
import time

POLL_INTERVAL = config.WORKER_POLL_INTERVAL   # پیشنهاد: ۳ ثانیه — قابل تنظیم از .env

def run_worker():
    while True:
        job = claim_job()                     # UPDATE ... SET status='running' WHERE job_id =
                                              #   (SELECT ... FROM processing_jobs
                                              #    WHERE status='pending'
                                              #    ORDER BY created_at FOR UPDATE SKIP LOCKED)
        if job is None:
            time.sleep(POLL_INTERVAL)
            continue
        try:
            process_document_lifecycle(job)   # validated → parsed → chunked → indexed → active
            mark_completed(job["job_id"])
        except Exception as exc:              # فقط سیاست Retry در L3؛ توابع L4/L5/L6 pure هستند
            handle_failure(job, exc)          # attempt_count++؛ اگر ≥ 3 → status='failed'
```

**نکته**: `claim_job()` باید ردیف‌های `pending` را با `FOR UPDATE SKIP LOCKED` بگیرد تا اگر در آینده دوباره Worker اجرا شود، دو فرایند یک Job را برندارند (پاسخ به ابهام race condition در بخش ۱۱ معماری منطقی).

---

# بخش ۳ — اسکیمای کامل پایگاه داده

این بخش همان فایل `schema.sql` است که در دو مستند فعلی «نیازمند تأیید» مانده بود (بخش ۹ معماری منطقی، ابهام ۲). طراحی بر اساس فیلدهای ذکرشده در Chunk Schema (ابهام ۱ نسخهٔ ۰.۳)، Document Lifecycle (L3)، ACL (L8) و Audit Log (L9) انجام شده است. **وضعیت: پیشنهادی — در Phase 1 باید با کل تیم بازبینی و مصوب شود.**

> **تصمیم C6 (بازبینی Phase 0)**: Task 4 در Phase 0 **همین `schema.sql` کامل را اجرا می‌کند** — هر ۷ جدول شامل `document_acl` و `processing_jobs`، نه اسکیمای ۳ جدولی حداقلی. ملاحظهٔ اجرایی: نصب فقط در اولین راه‌اندازی Volume انجام می‌شود (mount به `docker-entrypoint-initdb.d` در بخش ۷)؛ اگر Volume قبلاً ساخته شده، اسکیما باید با `psql -f` یا اسکریپت Migration دستی اعمال شود.

**اصول طراحی**:

- همهٔ شناسه‌ها ULID با نوع `CHAR(26)`.
- `documents.content_hash` **یکتا** است — تضمین Idempotency در سطح دیتابیس، نه فقط کد.
- `chunks.embedding_model_version` **الزامی** است — طبق قاعدهٔ معماری مرجع.
- PostgreSQL **مرجع معتبر ACL** است؛ به همین دلیل `document_acl` قید یکتایی دارد و با `ON DELETE CASCADE` به `documents` متصل است.
- `audit_log` **تغییرناپذیر** فرض می‌شود: هیچ UPDATE/DELETE نباید اجرا شود؛ `query_hash` به‌جای متن خام ذخیره می‌شود (حریم خصوصی).
- ایندکس `processing_jobs` از نوع Partial است تا Polling Worker فقط ردیف‌های `pending` را اسکن کند.

```sql
-- ============================================================
-- schema.sql — اسکیمای کامل PostgreSQL برای POC
-- نسخه: ۱.۰ | وضعیت: پیشنهادی — نیازمند تأیید در Phase 1
-- هدف: PostgreSQL 16+ | شناسه‌ها: ULID (CHAR(26))
-- ============================================================

-- ---------- users (L8) ----------
CREATE TABLE users (
    user_id        CHAR(26)     PRIMARY KEY,                 -- ULID
    username       VARCHAR(64)  NOT NULL UNIQUE,
    password_hash  VARCHAR(255) NOT NULL,                    -- argon2id (argon2-cffi)
    role           VARCHAR(16)  NOT NULL DEFAULT 'user'
                   CHECK (role IN ('admin', 'user')),        -- دو سطح دسترسی طبق AC
    is_active      BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT now()
);

-- ---------- documents (L7 / L3) ----------
CREATE TABLE documents (
    document_id         CHAR(26)     PRIMARY KEY,
    document_version    INTEGER      NOT NULL DEFAULT 1,
    filename            VARCHAR(255) NOT NULL,
    doc_type            VARCHAR(8)   NOT NULL
                        CHECK (doc_type IN ('docx', 'xlsx')),
    content_hash        CHAR(64)     NOT NULL UNIQUE,         -- SHA-256 hex → Idempotency
    file_size_bytes     BIGINT       NOT NULL
                        CHECK (file_size_bytes > 0),
    original_object_key VARCHAR(512) NOT NULL,                -- original/{doc_id}/{filename}
    status              VARCHAR(16)  NOT NULL DEFAULT 'uploaded'
                        CHECK (status IN ('uploaded', 'validated', 'parsed',
                                          'chunked', 'indexed', 'active', 'failed')),
    created_by          CHAR(26)     NOT NULL REFERENCES users(user_id),
    created_at          TIMESTAMPTZ  NOT NULL DEFAULT now(),
    updated_at          TIMESTAMPTZ  NOT NULL DEFAULT now()
);
CREATE INDEX idx_documents_status ON documents (status);   -- فهرست اسناد + فیلتر active

-- ---------- chunks (L7 / L4) ----------
CREATE TABLE chunks (
    chunk_id                 CHAR(26)     PRIMARY KEY,
    document_id              CHAR(26)     NOT NULL
                             REFERENCES documents(document_id) ON DELETE CASCADE,
    document_version         INTEGER      NOT NULL,
    chunk_index              INTEGER      NOT NULL,
    content                  TEXT         NOT NULL,
    content_object_key       VARCHAR(512) NOT NULL,         -- chunks/{doc_id}/{chunk_id}.txt در MinIO
    source_type              VARCHAR(32)  NOT NULL
                             CHECK (source_type IN ('word_paragraph', 'excel_sheet_row')),
    location_meta            JSONB        NOT NULL,         -- {page, heading} یا {sheet_name, row_range, column_range}
    content_hash             CHAR(64)     NOT NULL,         -- SHA-256 محتوای Chunk
    embedding_model_version  VARCHAR(64)  NOT NULL,         -- الزامی طبق معماری مرجع
    created_at               TIMESTAMPTZ  NOT NULL DEFAULT now(),
    UNIQUE (document_id, chunk_index)
);
CREATE INDEX idx_chunks_document          ON chunks (document_id);
CREATE INDEX idx_chunks_embedding_version ON chunks (embedding_model_version);  -- Re-embed و Rebuild

-- ---------- processing_jobs (L3) ----------
CREATE TABLE processing_jobs (
    job_id         CHAR(26)     PRIMARY KEY,
    document_id    CHAR(26)     NOT NULL
                   REFERENCES documents(document_id) ON DELETE CASCADE,
    stage          VARCHAR(16)  NOT NULL DEFAULT 'uploaded',   -- parsed / chunked / indexed
    status         VARCHAR(16)  NOT NULL DEFAULT 'pending'
                   CHECK (status IN ('pending', 'running', 'completed', 'failed')),
    attempt_count  INTEGER      NOT NULL DEFAULT 0
                   CHECK (attempt_count >= 0),
    error_log      TEXT,
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT now(),
    updated_at     TIMESTAMPTZ  NOT NULL DEFAULT now()
);
-- ایندکس مصرفی Worker (Polling): فقط ردیف‌های pending
CREATE INDEX idx_jobs_pending ON processing_jobs (created_at)
    WHERE status = 'pending';

-- ---------- document_acl (L8) — PostgreSQL مرجع معتبر ACL ----------
CREATE TABLE document_acl (
    acl_id        CHAR(26)     PRIMARY KEY,
    document_id   CHAR(26)     NOT NULL
                  REFERENCES documents(document_id) ON DELETE CASCADE,
    subject_type  VARCHAR(8)   NOT NULL CHECK (subject_type IN ('user', 'group')),
    subject_id    CHAR(26)     NOT NULL,
    permission    VARCHAR(8)   NOT NULL CHECK (permission IN ('read', 'write')),
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT now(),
    UNIQUE (document_id, subject_type, subject_id, permission)
);
CREATE INDEX idx_acl_document ON document_acl (document_id);
CREATE INDEX idx_acl_subject  ON document_acl (subject_type, subject_id);
-- یادآوری تصمیم معماری: در Phase 1 فقط subject_type='user' استفاده می‌شود؛
-- 'group' در ستون‌ها پشتیبانی می‌شود اما در کد Query Flow فعال نیست.

-- ---------- audit_log (L9) — تغییرناپذیر ----------
CREATE TABLE audit_log (
    entry_id             BIGSERIAL     PRIMARY KEY,
    request_id           CHAR(26)      NOT NULL,             -- ULID هر درخواست
    user_id              CHAR(26),
    action               VARCHAR(32)   NOT NULL,             -- login/upload/query/deny/llm_error/...
    document_id          CHAR(26),
    query_hash           CHAR(64),                           -- SHA-256(query_text) — بدون متن خام
    chunks_returned      JSONB,                              -- chunk_ids_returned
    chunks_denied_count  INTEGER       NOT NULL DEFAULT 0,   -- برای سناریوی T5
    latency_ms           INTEGER,
    outcome              VARCHAR(16)   NOT NULL
                         CHECK (outcome IN ('success', 'denied', 'error')),
    created_at           TIMESTAMPTZ   NOT NULL DEFAULT now()
);
CREATE INDEX idx_audit_created ON audit_log (created_at);
CREATE INDEX idx_audit_user    ON audit_log (user_id);
CREATE INDEX idx_audit_action  ON audit_log (action);

-- ---------- index_registry (L7) — Lineage ایندکس‌های Derived ----------
CREATE TABLE index_registry (
    registry_id              CHAR(26)     PRIMARY KEY,
    index_name               VARCHAR(128) NOT NULL,          -- نام Collection/Index در Qdrant/OS
    index_kind               VARCHAR(8)   NOT NULL
                             CHECK (index_kind IN ('vector', 'lexical')),
    embedding_model_version  VARCHAR(64),
    chunk_count              INTEGER      NOT NULL DEFAULT 0,  -- برای سناریوی T8 (PG=Qdrant=OS)
    status                   VARCHAR(16)  NOT NULL DEFAULT 'building'
                             CHECK (status IN ('building', 'valid', 'invalid')),
    last_validated_at        TIMESTAMPTZ,
    updated_at               TIMESTAMPTZ  NOT NULL DEFAULT now(),
    UNIQUE (index_name, index_kind)
);

-- ---------- Seed: کاربر ادمین اولیه ----------
-- password_hash باید بیرون از این فایل و با argon2-cffi تولید شود؛
-- مقدار placeholder زیر صرفاً نشان‌دهندهٔ قالب است.
INSERT INTO users (user_id, username, password_hash, role, is_active)
VALUES ('01JBG0000000000000000000AD', 'admin', 'REPLACE_WITH_ARGON2ID_HASH', 'admin', TRUE);
```

**ملاحظات باقی‌مانده برای Phase 1** (ثبت‌شده، نه تصمیم):

| مورد | وضعیت |
|---|---|
| Trigger یا update خودکار `updated_at` | پیشنهاد: یک تابع ساده `set_updated_at()` روی `documents` و `processing_jobs` — در Phase 1 تأیید شود |
| مهاجرت‌های آیندهٔ اسکیما (Alembic یا اسکریپت دستی) | پیشنهاد: برای POC فقط `schema.sql` idempotent با `IF NOT EXISTS`؛ ابزار Migration در تولیدی |
| Storage Parameterها (WAL، autovacuum) | مقادیر پیش‌فرض برای POC کافی است |

---

# بخش ۴ — قراردادهای API (OpenAPI Spec)

بر اساس بخش ۱۴ معماری منطقی (قرارداد C1 بین L1 و L2). این فایل `openapi.yaml` حداقلی است: مبنای تست یکپارچگی (بخش ۵) و مرجع قراردادی UI. **وضعیت: پیشنهادی — نیازمند تأیید.**

```yaml
openapi: 3.0.3
info:
  title: Offline Enterprise RAG POC API
  version: 1.0.0
  description: >
    قرارداد L1 (Streamlit) و L2 (FastAPI). همهٔ مسیرها به‌جز /health و /login
    نیازمند Session معتبر هستند. Fail Closed: هر عدم قطعیت → 401/403/نتیجهٔ خالی.
paths:
  /health:
    get:
      summary: Health check (برای healthcheck کانتینر و nginx)
      responses:
        "200": { description: سرویس فعال }
  /login:
    post:
      summary: احراز هویت و صدور Session
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/LoginRequest" }
      responses:
        "200":
          description: موفق — Cookie با Set-Cookie تنظیم می‌شود
          content:
            application/json:
              schema: { $ref: "#/components/schemas/UserResponse" }
        "401": { description: Credentials نامعتبر }
  /logout:
    post:
      summary: ابطال Session
      responses:
        "204": { description: Session باطل شد }
  /upload:
    post:
      summary: آپلود DOCX/XLSX (فقط admin)
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              required: [file]
              properties:
                file:
                  type: string
                  format: binary
                  description: DOCX یا XLSX — حداکثر اندازه از .env (پیشنهاد ۵۰MB)
      responses:
        "201":
          description: ایجاد شد (یا همان document_id موجود در Idempotency)
          content:
            application/json:
              schema: { $ref: "#/components/schemas/UploadResponse" }
        "400": { description: پسوند/اندازه/MIME نامعتبر }
        "403": { description: کاربر admin نیست }
  /query:
    post:
      summary: پرسش‌وپاسخ RAG با ACL و Fail Closed
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/QueryRequest" }
      responses:
        "200":
          description: پاسخ (sources ممکن است خالی باشد — ACL یا نبود نتیجه)
          content:
            application/json:
              schema: { $ref: "#/components/schemas/QueryResponse" }
        "401": { description: Session نامعتبر }
  /documents:
    get:
      summary: فهرست اسناد مجاز کاربر جاری (pagination)
      parameters:
        - { in: query, name: limit,  schema: { type: integer, default: 20 } }
        - { in: query, name: offset, schema: { type: integer, default: 0 } }
      responses:
        "200":
          description: لیست اسناد بر اساس ACL
          content:
            application/json:
              schema:
                type: array
                items: { $ref: "#/components/schemas/DocumentSummary" }
components:
  schemas:
    LoginRequest:
      type: object
      required: [username, password]
      properties:
        username: { type: string }
        password: { type: string, format: password }
    UserResponse:
      type: object
      properties:
        user_id: { type: string }
        username: { type: string }
        role: { type: string, enum: [admin, user] }
    UploadResponse:
      type: object
      required: [document_id, status]
      properties:
        document_id: { type: string, description: ULID }
        status: { type: string, enum: [uploaded] }
        duplicate: { type: boolean, description: "true اگر content_hash تکراری بود (Idempotency)" }
    QueryRequest:
      type: object
      required: [query]
      properties:
        query: { type: string, maxLength: 2000 }
    QueryResponse:
      type: object
      required: [answer, sources]
      properties:
        answer: { type: string }
        sources:
          type: array
          items:
            type: object
            required: [document_id, location_meta]
            properties:
              document_id: { type: string }
              location_meta: { type: object }
              score: { type: number, description: امتیاز RRF }
    DocumentSummary:
      type: object
      properties:
        document_id: { type: string }
        filename: { type: string }
        doc_type: { type: string, enum: [docx, xlsx] }
        status: { type: string, enum: [uploaded, validated, parsed, chunked, indexed, active, failed] }
        created_at: { type: string, format: date-time }
```

**نکات پیاده‌سازی**:

- `POST /upload` با `python-multipart` پیاده می‌شود؛ اعتبارسنجی پسوند + اندازه در L1 و دوباره در L2 (دفاع در عمق).
- خطای 422 اعتبارسنجی Pydantic با جزئیات فیلد به L1 برمی‌گردد (طبق بخش ۱۱ معماری منطقی).
- `/health` برای healthcheck کانتینر `app` در Compose استفاده می‌شود و هیچ Auth نمی‌خواهد.

---

# بخش ۵ — سناریوهای آزمون یکپارچگی

بر اساس سناریوهای End-to-End (بخش ۲۰ معماری منطقی) با جزئیات اجرایی. این سناریوها مستقیماً به Acceptance Criteria (بخش ۱۵ نسخهٔ ۰.۳) گره می‌خورند و قابل تبدیل به تست خودکار (مثلاً با `pytest` + `httpx` روی Compose در حال اجرا) هستند.

## ۵.۱. جدول سناریوها

| سناریو | ورودی | خروجی مورد انتظار | معیار عبور |
|---|---|---|---|
| T1: Upload DOCX | `report.docx` توسط admin | `document_id`، status `uploaded` | HTTP 201، `document_id` معتبر (۲۶ کاراکتر)، ردیف در `documents` با status=`uploaded` |
| T2: Ingestion Complete | همان سند | status `active` | Worker در **< ۳۰ ثانیه** سند را فعال کند؛ `processing_jobs.status=completed` |
| T3: Query Simple | «خلاصه گزارش چیست؟» | پاسخ + منبع | پاسخ غیرخالی؛ حداقل یک source با `document_id` معتبر؛ ردیف `audit_log` ثبت شده |
| T4: Query Excel | «Revenue ژانویه چقدر بود؟» | پاسخ + `location_meta` | `sheet_name='Sales'` در منبع؛ سرستون در Chunk مرجع وجود دارد |
| T5: ACL Deny | Query توسط کاربر بدون ACL روی آن سند | پاسخ خالی | `sources=[]` و `chunks_denied_count > 0` در `audit_log`؛ LLM با Context غیرمجاز فراخوانی نشده |
| T6: Fail Closed | قطع PG در حین Query | پاسخ خالی / خطا | LLM فراخوانی نشود (verify در `audit_log`: بدون ردیف `llm_call` موفق) |
| T7: Idempotency | آپلود مجدد همان فایل | همان `document_id` | بدون ردیف جدید در `documents`؛ `duplicate=true` در پاسخ |
| T8: Rebuild Index | حذف Collection + Rebuild | تعداد Chunkها برابر | `count(chunks)` در PG = Points در Qdrant = Docs در OS؛ `index_registry.status='valid'` |

## ۵.۲. پیش‌نیازها و محیط آزمون

- Compose کامل بخش ۷ در حال اجرا؛ Golden Set بخش ۸ بارگذاری شده (۲ سند نمونه + کاربران).
- سه کاربر آزمایشی: `admin` (نقش admin)، `alice` (ACL=read روی هر دو سند)، `bob` (بدون ACL — برای T5).
- روش قطع PG برای T6: `docker compose pause postgres` و سپس فراخوانی `/query`؛ پس از تست: `docker compose unpause postgres`.
- روش Rebuild برای T8: حذف Collection در Qdrant و Index در OS با فراخوانی مستقیم API، سپس `python scripts/rebuild_index.py` (بخش ۶).

## ۵.۳. جزئیات اجرایی چند سناریوی حساس

**T5 — ACL Deny (مهم‌ترین سناریوی امنیتی)**:
۱. `bob` با `POST /query` پرسش می‌پرسد که پاسخش فقط از اسناد دارای ACL می‌آید.
۲. در `acl_revalidate()` (مرحلهٔ ۱۰ Query Flow)، بعد از RRF، همهٔ `chunk_id`های اسنادِ بدون ACL حذف می‌شوند.
۳. اگر بعد از حذف، هیچ Chunk باقی نماند → **Context خالی** → LLM اصلاً فراخوانی نمی‌شود؛ پاسخ `{answer: "", sources: []}`.
۴. تایید در `audit_log`: `chunks_denied_count > 0`، `outcome='denied'`.

**T6 — Fail Closed**:
۱. PG متوقف می‌شود؛ کاربر `/query` می‌زند.
۲. `get_current_user` → PG error → 401 یا 500 با پیام عمومی (بدون leak جزئیات).
۳. اگر به هر دلیل جریان Query تا بعد از ACL ادامه یافت، هر Exception در ACL → نتیجهٔ خالی → بدون فراخوانی Ollama.
۴. پس از restore، هیچ داده‌ای نباید خراب شده باشد (`pg_dump` قبل و بعد مقایسه شود).

**T2 — معیار زمانی < ۳۰ ثانیه**:
با یک DOCX کمتر از ۱ مگابایت و مدل Embedding از قبل load شده در حافظه، زمان پردازش باید < ۳۰ ثانیه باشد (شامل Polling interval ۳ ثانیه‌ای). اگر شکست خورد، اول `WORKER_POLL_INTERVAL` و سپس زمان بارگذاری اولیهٔ مدل Embedding بررسی شود.

## ۵.۴. اتوماسیون پیشنهادی

- هر سناریو یک تابع pytest با fixture «compose up + seed داده» باشد؛ اجرای کل مجموعه در محیط متصل به شبکه قبل از ساخت بستهٔ آفلاین، و دوباره در ماشین ایزوله پس از `load_bundle.sh`.
- نتایج هر اجرا در `audit_log` قابل بازخوانی است؛ گزارش نهایی (pass/fail هر T) در انتهای چک‌لیست بخش ۹ ثبت شود.

---

# بخش ۶ — اسکریپت‌های عملیاتی (Bash/Python)

بر اساس بخش ۱۱ نسخهٔ ۰.۳ (Offline Strategy) و بخش ۱۱ معماری منطقی، اما با اسکریپت‌های واقعی. همهٔ اسکریپت‌ها با `chmod +x` در `app/scripts/` یا ریشهٔ مخزن نگهداری می‌شوند. **وضعیت: قابل استفاده پس از بازبینی — نسخه‌های تصاویر «نیازمند تأیید».**

> **تصمیم C7 (بازبینی Phase 0)**: به‌روزرسانی HDD میزبان‌های ایزوله **از طریق حمل فیزیکی همین `offline-bundle`** انجام می‌شود (دیسک خارجی + `load_bundle.sh` + تطبیق `SHA256SUMS`). هیچ فرآیند موازی یا مسیر دوم برای HDD Update وجود ندارد؛ هر به‌روزرسانی دقیقاً با همین اسکریپت‌های بخش ۶ اجرا و در Audit ثبت می‌شود.

## ۶.۱. `create_bundle.sh` — ساخت بستهٔ آفلاین

```bash
#!/usr/bin/env bash
# اجرا فقط در ماشین متصل به شبکه. خروجی: offline-bundle/<date>/
set -euo pipefail

BUNDLE="offline-bundle/$(date +%Y%m%d-%H%M)"
IMAGES=(
  "ollama/ollama:0.3.12"                    # همه نسخه‌ها پیشنهادی — نیازمند تأیید
  "qdrant/qdrant:v1.10.0"
  "opensearchproject/opensearch:2.14.0"
  "minio/minio:latest-tagged-in-phase1"
  "postgres:16.4"
)
mkdir -p "$BUNDLE"/{images,wheels,models,app}

# ۱) تصاویر Docker
docker save "${IMAGES[@]}" | gzip > "$BUNDLE/images/images.tar.gz"

# ۲) وابستگی‌های Python (wheel برای نصب --no-index)
pip download -r app/requirements.txt -d "$BUNDLE/wheels/"

# ۳) فایل‌های مدل (GGUF LLM + وزن‌های Sentence Transformers)
#    مسیرها مطابق .env: LLM_MODEL_FILE و EMBEDDING_MODEL_DIR
cp -r "$LLM_MODEL_FILE" "$BUNDLE/models/"
cp -r "$EMBEDDING_MODEL_DIR" "$BUNDLE/models/"

# ۴) سورس کد + compose + schema (بدون .env واقعی)
rsync -a --exclude '.env' --exclude 'offline-bundle' ./ "$BUNDLE/app/"

# ۵) اثبات صحت انتقال
( cd "$BUNDLE" && find . -type f -print0 | sort -z \
    | xargs -0 sha256sum > SHA256SUMS )

echo "OK: $BUNDLE (حجم: $(du -sh "$BUNDLE" | cut -f1))"
```

## ۶.۲. `load_bundle.sh` — بارگذاری در ماشین ایزوله

```bash
#!/usr/bin/env bash
# اجرا در ماشین Air-Gapped. مسیر بسته به‌عنوان آرگومان اول.
set -euo pipefail
BUNDLE="${1:?usage: load_bundle.sh <bundle-dir>}"

# ۱) تأیید یکپارچگی انتقال
( cd "$BUNDLE" && sha256sum --quiet -c SHA256SUMS ) || { echo "checksum FAILED"; exit 1; }

# ۲) تصاویر
docker load < "$BUNDLE/images/images.tar.gz"

# ۳) وابستگی‌های Python بدون اینترنت (داخل build یا مستقیم در volume)
pip install --no-index --find-links "$BUNDLE/wheels/" -r "$BUNDLE/app/app/requirements.txt"

# ۴) مدل‌ها به محل mount شدهٔ کانتینر app و ollama
docker compose up -d --no-deps app ollama
docker cp "$BUNDLE/models/."  app:/app/models/
docker cp "$BUNDLE/models/"   ollama:/root/.ollama/models-import/

# ۵) راه‌اندازی و اعتبارسنجی
docker compose up -d
docker compose ps            # هر ۶ سرویس باید healthy باشد
curl -fsS http://localhost:8501/_stcore/health
echo "OK: bundle loaded"
```

## ۶.۳. `backup.sh` — پشتیبان‌گیری

```bash
#!/usr/bin/env bash
# pg_dump + mc mirror + Qdrant snapshot + OpenSearch snapshot
# (OpenSearch snapshot نیاز به snapshot repository دارد — در POC کپی دیتادیر کافی است)
set -euo pipefail
BACKUP_ROOT="backups/$(date +%Y%m%d-%H%M)"
mkdir -p "$BACKUP_ROOT"

# ۱) PostgreSQL (مرجع معتبر ACL و Metadata — مهم‌ترین)
docker exec postgres pg_dump -U "$POSTGRES_USER" -Fc "$POSTGRES_DB" > "$BACKUP_ROOT/pg_dump.sql"

# ۲) MinIO (Canonical Artifacts — قابل بازسازی، اما باید backup شود)
mc mirror "local/"*bucket* "$BACKUP_ROOT/minio/"

# ۳) Qdrant (Derived — snapshot سریع)
curl -fsS -X POST "http://qdrant:6333/snapshots?collection_name=${QDRANT_COLLECTION}"

# ۴) OpenSearch (Derived — کپی دیتادیر ساده برای POC)
docker compose stop opensearch
cp -r "$(docker volume inspect --format '{{.Mountpoint}}' opensearch_data)" \
      "$BACKUP_ROOT/opensearch_data/" || true
docker compose start opensearch

echo "backup OK: $BACKUP_ROOT"
```

## ۶.۴. `restore.sh` — بازیابی

```bash
#!/usr/bin/env bash
# بازیابی کامل از یک پوشهٔ backup
set -euo pipefail
BACKUP="${1:?usage: restore.sh <backup-dir>}"

# ۱) PG — Clean restore
docker compose stop app
dropdb_cmd="docker exec postgres dropdb -U $POSTGRES_USER --if-exists $POSTGRES_DB"
createdb_cmd="docker exec postgres createdb -U $POSTGRES_USER $POSTGRES_DB"
$dropdb_cmd && $createdb_cmd
cat "$BACKUP/pg_dump.sql" | docker exec -i postgres pg_restore -U "$POSTGRES_USER" -d "$POSTGRES_DB"

# ۲) MinIO و OpenSearch
mc mirror "$BACKUP/minio/" "local/"*bucket*
rsync -a "$BACKUP/opensearch_data/" \
      "$(docker volume inspect --format '{{.Mountpoint}}' opensearch_data)/"

# ۳) Qdrant و OpenSearch را از Canonical بازسازی کن (تضمین PG = Qdrant = OS)
docker compose start app
python app/scripts/rebuild_index.py

echo "restore OK — تعداد chunkها را با سناریوی T8 تطبیق بده"
```

## ۶.۵. `reconcile.py` — اصلاح دستی Jobها

```python
# اجرا: python reconcile.py --status failed --retry
# بازگرداندن Jobهای failed با attempt_count < 3 به pending (طبق تصمیم L3 معماری منطقی)
import argparse
import psycopg2, os

def main() -> None:
    ap = argparse.ArgumentParser()
    ap.add_argument("--status", default="failed", choices=["failed", "running", "pending"])
    ap.add_argument("--retry", action="store_true", help="attempt_count را صفر کن")
    ap.add_argument("--dry-run", action="store_true")
    args = ap.parse_args()

    sql = "UPDATE processing_jobs SET status = 'pending'"
    if args.retry:
        sql += ", attempt_count = 0"
    sql += " WHERE status = %s AND attempt_count < 3"

    with psycopg2.connect(os.environ["DATABASE_URL"]) as conn, conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM processing_jobs WHERE status = %s", (args.status,))
        print("candidate jobs:", cur.fetchone()[0])
        if not args.dry_run:
            cur.execute(sql, (args.status,))
            print("requeued:", cur.rowcount)

if __name__ == "__main__":
    main()
```

## ۶.۶. `rebuild_index.py` — بازسازی Index از Canonical Data

```python
# اجرا: python rebuild_index.py
# منبع حقیقت: PG (chunks) + MinIO (content). خروجی: Qdrant + OpenSearch + index_registry
import os, psycopg2
from qdrant_client import QdrantClient
from opensearchpy import OpenSearch
from embedding.embedder import get_embedder   # Singleton مدل بارگذاری‌شده
from config import QDRANT_COLLECTION, OS_INDEX, EMBEDDING_MODEL_VERSION

def fetch_chunks() -> list[dict]:
    with psycopg2.connect(os.environ["DATABASE_URL"]) as conn, conn.cursor() as cur:
        cur.execute("""SELECT chunk_id, document_id, chunk_index, content, source_type,
                              location_meta, embedding_model_version
                       FROM chunks ORDER BY document_id, chunk_index""")
        return [dict(zip([d[0] for d in cur.description], r)) for r in cur.fetchall()]

def main() -> None:
    chunks = fetch_chunks()
    print(f"canonical chunks from PG: {len(chunks)}")

    # ۱) Vector index — از صفر بازسازی (Derived: قابل بازسازی طبق معماری)
    qdrant = QdrantClient(host="qdrant", port=6333)
    qdrant.delete_collection(QDRANT_COLLECTION)
    qdrant.recreate_collection(QDRANT_COLLECTION, vector_size=1024, distance="Cosine")
    embedder = get_embedder()
    vectors = embedder.encode([c["content"] for c in chunks], show_progress_bar=True)
    qdrant.upsert(QDRANT_COLLECTION, points=[
        {"id": c["chunk_id"], "vector": v.tolist(),
         "payload": {"document_id": c["document_id"], "chunk_index": c["chunk_index"],
                     "source_type": c["source_type"], "location_meta": c["location_meta"],
                     "document_status": "active",  # فقط اسناد active ایندکس می‌شوند
                     "embedding_model_version": c["embedding_model_version"]}}
        for c, v in zip(chunks, vectors)])

    # ۲) Lexical index (BM25)
    os_client = OpenSearch(hosts=[{"host": "opensearch", "port": 9200}])
    os_client.indices.delete(index=OS_INDEX, ignore=[400, 404])
    os_client.indices.create(index=OS_INDEX,
                             body={"mappings": {"properties": {
                                 "content": {"type": "text"},
                                 "document_id": {"type": "keyword"},
                                 "source_type": {"type": "keyword"},
                                 "document_status": {"type": "keyword"},
                                 "embedding_model_version": {"type": "keyword"}}}})
    for c in chunks:
        os_client.index(index=OS_INDEX, id=c["chunk_id"],
                        body={"content": c["content"], "document_id": c["document_id"],
                              "source_type": c["source_type"], "document_status": "active",
                              "embedding_model_version": c["embedding_model_version"]})

    # ۳) ثبت Lineage در PG (index_registry) — مبنای سناریوی T8
    with psycopg2.connect(os.environ["DATABASE_URL"]) as conn, conn.cursor() as cur:
        cur.execute("""INSERT INTO index_registry (registry_id, index_name, index_kind,
                       embedding_model_version, chunk_count, status, last_validated_at)
                       VALUES (gen_ulid(), %s, 'vector', %s, %s, 'valid', now()),
                               (gen_ulid(), %s, 'lexical', %s, %s, 'valid', now())""",
                    (QDRANT_COLLECTION, EMBEDDING_MODEL_VERSION, len(chunks),
                     OS_INDEX, EMBEDDING_MODEL_VERSION, len(chunks)))
    print(f"rebuild OK: {len(chunks)} chunks → Qdrant + OpenSearch")

if __name__ == "__main__":
    main()
```

**نکات پیاده‌سازی اسکریپت‌ها**:

- `gen_ulid()` یک تابع PL/pgSQL کوچک یا تولید ULID سمت Python است — در Phase 1 یکی انتخاب شود.
- در `rebuild_index.py` فقط Chunkهای اسناد `active` باید ایندکس شوند؛ فیلتر `document_id IN (SELECT document_id FROM documents WHERE status='active')` در نسخهٔ نهایی اضافه شود (در نمونه بالا برای سادگی حذف شده و با کامنت مشخص است).
- همهٔ اسکریپت‌ها باید در ماشین ایزوله بدون اینترنت اجرا شوند — تست نهایی بخشی از گام ۱۰ نقشهٔ راه است.

---

# بخش ۷ — نمونهٔ `docker-compose.yml` کامل

بر اساس بخش ۹ نسخهٔ ۰.۳ (که فقط طرح کلی داشت)، یک فایل **کامل و قابل اجرا**. **وضعیت: پیشنهادی — همهٔ تگ‌های تصویر نیازمند تأیید در Phase 1.**

```yaml
# docker-compose.yml — POC آفلاین، ۶ کانتینر، تک‌گره
# نسخه‌های تصویر: پیشنهادی — نیازمند تأیید از Release رسمی
services:
  app:
    build: ./app                       # Dockerfile پروژه (python:3.11-slim پایه — نیازمند تأیید)
    image: rag-poc/app:0.1.0
    networks: [frontend, backend]
    ports:
      - "${APP_PORT:-8501}:8501"       # تنها پورت منتشرشده روی Host
    env_file: .env
    volumes:
      - app_models:/app/models         # cache مدل Sentence Transformer (اختیاری)
    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://localhost:8501/_stcore/health || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 60s                # بارگذاری مدل Embedding در startup زمان می‌برد
    depends_on:
      postgres:   { condition: service_healthy }
      qdrant:     { condition: service_healthy }
      opensearch: { condition: service_healthy }
      minio:      { condition: service_healthy }
      ollama:     { condition: service_healthy }

  ollama:
    image: ollama/ollama:0.3.12        # نیازمند تأیید از Release رسمی
    networks: [backend]
    volumes:
      - ollama_models:/root/.ollama
    # در صورت وجود GPU — اختیاری، سناریوی ۱ CPU-only هم کار می‌کند:
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - { driver: nvidia, count: all, capabilities: [gpu] }
    healthcheck:
      test: ["CMD-SHELL", "ollama list >/dev/null 2>&1 || exit 1"]
      interval: 15s
      timeout: 5s
      retries: 10
      start_period: 30s

  qdrant:
    image: qdrant/qdrant:v1.10.0       # نیازمند تأیید — ≥1.10 برای ColBERT اختیاری
    networks: [backend]
    volumes:
      - qdrant_storage:/qdrant/storage
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://localhost:6333/healthz || exit 1"]
      interval: 15s
      timeout: 5s
      retries: 10

  opensearch:
    image: opensearchproject/opensearch:2.14.0   # نیازمند تأیید
    networks: [backend]
    environment:
      discovery.type: single-node
      OPENSEARCH_JAVA_OPTS: "-Xms512m -Xmx512m"  # جلوگیری از OOM — بخش ۱۶ نسخهٔ ۰.۳
      DISABLE_SECURITY_PLUGIN: "true"            # فقط داخل شبکه internal — نیازمند تأیید امنیتی
      OPENSEARCH_INITIAL_ADMIN_PASSWORD: "${OPENSEARCH_INITIAL_ADMIN_PASSWORD}"
    volumes:
      - opensearch_data:/usr/share/opensearch/data
    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://localhost:9200/_cluster/health || exit 1"]
      interval: 20s
      timeout: 10s
      retries: 10
      start_period: 90s                # JVM کند بالا می‌آید

  minio:
    image: minio/minio:RELEASE-2024-XX # نیازمند تأیید — تگ ثابت از Release رسمی
    command: server /data --console-address ":9001"
    networks: [backend]
    environment:
      MINIO_ROOT_USER: "${MINIO_ROOT_USER}"
      MINIO_ROOT_PASSWORD: "${MINIO_ROOT_PASSWORD}"
    volumes:
      - minio_data:/data
    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://localhost:9000/minio/health/live || exit 1"]
      interval: 15s
      timeout: 5s
      retries: 10

  postgres:
    image: postgres:16.4               # نیازمند تأیید
    networks: [backend]
    environment:
      POSTGRES_USER: "${POSTGRES_USER}"
      POSTGRES_PASSWORD: "${POSTGRES_PASSWORD}"
      POSTGRES_DB: "${POSTGRES_DB}"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./app/schema.sql:/docker-entrypoint-initdb.d/01-schema.sql:ro   # اجرای خودکار در اولین راه‌اندازی
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 10

volumes:
  postgres_data:
  qdrant_storage:
  opensearch_data:
  minio_data:
  ollama_models:
  app_models:

networks:
  frontend:                            # فقط app — تنها نقطهٔ تماس با کاربر
  backend:
    internal: true                     # هیچ خروجی به خارج — ایده‌آل برای POC آفلاین
```

## ۷.۱. `.env.example`

```bash
# .env.example — کپی به .env و chmod 600 (Secret Management طبق L8/L10)
APP_PORT=8501
POSTGRES_USER=rag_poc
POSTGRES_PASSWORD=change-me-strong
POSTGRES_DB=rag_poc
MINIO_ROOT_USER=ragpoc
MINIO_ROOT_PASSWORD=change-me-strong
JWT_SECRET=change-me-64-chars-random
OPENSEARCH_INITIAL_ADMIN_PASSWORD=change-me-strong

# تنظیمات L3
WORKER_POLL_INTERVAL=3          # ثانیه — پیشنهاد معماری منطقی
ATTEMPT_THRESHOLD=3             # آستانهٔ Retry

# تنظیمات L4
MAX_UPLOAD_MB=50
CHUNK_TARGET_TOKENS=768         # بازهٔ مجاز ۵۱۲–۱۰۲۴
CHUNK_OVERLAP_PERCENT=12        # بازهٔ مجاز ۱۰–۱۵
ROW_GROUP_SIZE=50

# تنظیمات L5
LLM_MODEL_NAME=qwen2.5:7b-instruct-q4_K_M
EMBEDDING_MODEL_NAME=intfloat/multilingual-e5-large
EMBEDDING_MODEL_VERSION=e5-large-v1

# تنظیمات L6
TOP_K=20
RRF_K=60
MAX_CONTEXT_TOKENS=30000        # < ۳۲K Context Window مدل — با Margin
```

**نکات پیاده‌سازی Compose**:

- **تصمیم C1 (بازبینی Phase 0)**: در این Compose **هیچ Redis یا Message Broker وجود ندارد** و در Phase 0 هم اضافه نمی‌شود. صف کارها همان جدول `processing_jobs` در PostgreSQL با الگوی Polling و `FOR UPDATE SKIP LOCKED` است — تعارض با اصل «بدون Broker» و سقف ۶ کانتینر از این مسیر رفع شده است.
- **تصمیم C2 (بازبینی Phase 0)**: OpenSearch **از نخستین نسخهٔ Compose (هفتهٔ اول) همراه بقیهٔ سرویس‌ها راه‌اندازی می‌شود**، نه پس از Phase اولیه. با تنظیم `OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m` از همین روز اول، که هم ریسک OOM را کنترل می‌کند و هم سناریوهای T1–T8 از ابتدا روی Stack کامل قابل اجرا هستند.
- **تصمیم C3 (بازبینی Phase 0)**: مدل LLM فقط از طریق **کانتینر Ollama** سرو می‌شود؛ انتقال مستقیم فایل GGUF از `/opt/models` به برنامهٔ Python در Phase 0 مطرح نیست. بستهٔ آفلاین مدل را در Volume `ollama_models` منتقل می‌کند (بخش ۶).
- **تصمیم C4 (بازبینی Phase 0)**: استک Embedding همان **Sentence Transformers با `intfloat/multilingual-e5-large`** است (نه ONNX Runtime). از روز اول، مقدار `EMBEDDING_MODEL_VERSION` در `.env` قفل می‌شود و هر دو محیط توسعه و ایزوله **دقیقاً یک نسخهٔ واحد** را استفاده می‌کنند — سازگاری بردارها بین ایندکس‌گذاری و جست‌وجو تضمین می‌شود (چرا که تغییر این مقدار بدون Rebuild کامل Index مجاز نیست، طبق بخش ۶.۶).
- **تصمیم C5 (بازبینی Phase 0)**: Gitea، CI Runner و PyPI Mirror به **فایل `docker-compose.dev-infra.yml` جداگانه** منتقل می‌شوند — خارج از ۶ کانتینر POC و خارج از بستهٔ استقرار ایزوله. این فایل فقط برای ماشین توسعهٔ متصل است؛ مالکیت، مسئولیت نگهداری و چرخهٔ به‌روزرسانی آن در یک **ADR** مستند و تصویب می‌شود.
- فقط `app` روی Host پورت دارد؛ چهار سرویس داده هیچ `ports:`ای ندارند و از خود Host هم در دسترس نیستند (طبق نکتهٔ امنیتی بخش ۹ نسخهٔ ۰.۳).
- `DISABLE_SECURITY_PLUGIN=true` برای OpenSearch یک تصمیم امنیتی است که باید صریحاً تأیید شود؛ چون شبکهٔ `backend` با `internal: true` از Host جدا است، ریسک POC پذیرفتنی است اما ثبت شود.
- `start_period` برای `app` و `opensearch` بلندتر است (بارگذاری مدل / JVM).
- فایل `schema.sql` با mount به `docker-entrypoint-initdb.d` فقط در **اولین** راه‌اندازی volume اجرا می‌شود؛ پس از آن تغییرات اسکیما دستی یا با اسکریپت Migration اعمال شود.

---

# بخش ۸ — مجموعه‌دادهٔ آزمون (Golden Set) نمونه

برای ارزیابی Ragas (بخش ۱۵ نسخهٔ ۰.۳) یک نمونهٔ حداقلی. این داده‌ها **ساختگی و صرفاً آزمایشی** هستند و الگوی واقعی Golden Set را نشان می‌دهند؛ تیم در Phase 1 با اسناد واقعی سازمانی جایگزین/گسترش می‌دهد.

## ۸.۱. اسناد نمونهٔ آزمون

| فایل | محتوا | ساختار |
|---|---|---|
| `sample_report.docx` | «گزارش فروش ۱۴۰۵» — ۲ صفحه | دو Heading («خلاصهٔ اجرایی»، «تحلیل درآمد»)، هر کدام ۲–۳ پاراگراف، یک جدول کوچک ۴ سطری |
| `sample_sales.xlsx` | «دادهٔ فروش» | شیت `Sales`: ستون‌های `month, revenue, unit_count` + ۱۲ سطر داده (فروردین تا اسفند)؛ شیت `Notes`: چند سطر توضیح متنی |

## ۸.۲. JSON Schema فرمت Golden Set

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Golden Set — RAG POC",
  "type": "array",
  "items": {
    "type": "object",
    "required": ["question", "ground_truth", "relevant_chunk_ids", "source_type", "expected_location_meta"],
    "properties": {
      "question":              { "type": "string", "maxLength": 2000 },
      "ground_truth":          { "type": "string", "description": "پاسخ مرجع تأییدشده توسط متخصص حوزه" },
      "relevant_chunk_ids":    { "type": "array", "items": { "type": "string" },
                                 "description": "chunk_idهای مرتبط — پس از اولین Ingestion پر می‌شود" },
      "source_type":           { "type": "string", "enum": ["word_paragraph", "excel_sheet_row"] },
      "expected_location_meta":{ "type": "object",
                                 "description": "مثلاً {\"sheet_name\":\"Sales\"} یا {\"heading\":\"تحلیل درآمد\"}" }
    }
  },
  "minItems": 5
}
```

## ۸.۳. نمونهٔ ۶ جفت (پرسش، پاسخ مرجع، محل مرجع)

> اعداد این نمونه ساختگی‌اند؛ بعد از آپلود اسناد نمونه، `chunk_id`های واقعی در ستون مربوطه ثبت می‌شوند.

```json
[
  {
    "question": "خلاصهٔ گزارش فروش ۱۴۰۵ چیست؟",
    "ground_truth": "درآمد کل فروش سال ۱۴۰۵ ۴۸ میلیارد ریال بوده که نسبت به سال قبل ۱۲٪ رشد نشان می‌دهد.",
    "relevant_chunk_ids": [],
    "source_type": "word_paragraph",
    "expected_location_meta": { "heading": "خلاصهٔ اجرایی" }
  },
  {
    "question": "Revenue ژانویه چقدر بود؟",
    "ground_truth": "Revenue ژانویه ۳٬۲۰۰ میلیون ریال بود.",
    "relevant_chunk_ids": [],
    "source_type": "excel_sheet_row",
    "expected_location_meta": { "sheet_name": "Sales" }
  },
  {
    "question": "کدام ماه بیشترین درآمد را داشت؟",
    "ground_truth": "اسفند با ۵٬۱۰۰ میلیون ریال بیشترین درآمد را داشت.",
    "relevant_chunk_ids": [],
    "source_type": "excel_sheet_row",
    "expected_location_meta": { "sheet_name": "Sales" }
  },
  {
    "question": "دلایل رشد درآمد در گزارش چه ذکر شده است؟",
    "ground_truth": "افزایش مشتریان شرکتی و عرضهٔ دو محصول جدید به‌عنوان دلایل رشد ذکر شده است.",
    "relevant_chunk_ids": [],
    "source_type": "word_paragraph",
    "expected_location_meta": { "heading": "تحلیل درآمد" }
  },
  {
    "question": "تعداد واحدهای فروش‌رفته در خرداد چقدر بود؟",
    "ground_truth": "۴۲۰ واحد در خرداد فروش رفت.",
    "relevant_chunk_ids": [],
    "source_type": "excel_sheet_row",
    "expected_location_meta": { "sheet_name": "Sales" }
  },
  {
    "question": "ملاحظات ثبت‌شده در فایل دادهٔ فروش چیست؟",
    "ground_truth": "داده‌های فروردین بعد از اصلاح صورت‌های مالی به‌روزرسانی شده‌اند.",
    "relevant_chunk_ids": [],
    "source_type": "excel_sheet_row",
    "expected_location_meta": { "sheet_name": "Notes" }
  }
]
```

## ۸.۴. راهنمای گسترش Golden Set

- هدف نهایی طبق بخش ۱۵ نسخهٔ ۰.۳: **۲۰–۳۰ نمونهٔ متنوع** (ترکیب DOCX و XLSX، پرسش‌های مفهومی و عددی، موارد مرزی مثل «پرسش بدون پاسخ در اسناد» برای سنجش عدم توهم).
- پرسش‌های بدون پاسخ (`ground_truth: ""`) برای اندازه‌گیری Faithfulness اضافه شوند — سیستم باید «اطلاعی ندارم» بگوید، نه توهم تولید کند.
- هر ویرایش در Golden Set باید version شود (`golden_set_v1.json`، `v2`، ...) تا نتایج Ragas بین نسخه‌ها قابل مقایسه بمانند.

---

# بخش ۹ — چک‌لیست Phase 1 (Go/No-Go)

این چک‌لیست «دروازهٔ عبور» از Phase 1 به Phase 2 است و از «شروع زودهنگام Phase 2 بدون زیرساخت پایدار» جلوگیری می‌کند. هر آیتم باید با امضای تأییدکننده و تاریخ ثبت شود.

- [ ] `docker compose up -d` موفق (۶ سرویس healthy)
- [ ] `POST /upload` یک DOCX را دریافت و `document_id` برگرداند
- [ ] سند در < ۳۰ ثانیه `active` شود (سناریوی T2)
- [ ] `POST /query` پاسخ با منبع برگرداند (T3 و T4)
- [ ] ACL Deny تست شده — پاسخ خالی و `chunks_denied_count > 0` در audit_log (T5)
- [ ] Fail Closed تست شده — قطع PG بدون فراخوانی LLM (T6)
- [ ] Idempotency تست شده — آپلود مجدد، همان document_id (T7)
- [ ] Offline Bundle در ماشین ایزوله Load شود — همهٔ T1–T8 در آن پاس شوند (T8 شامل Rebuild)
- [ ] Backup/Restore تست شده — `backup.sh` و `restore.sh` با تطبیق تعداد Chunkها
- [ ] نسخه‌های قفل‌شده در `requirements.txt` و `docker-compose.yml` ثبت شده و از لیست «نیازمند تأیید» خارج شده‌اند

**قاعدهٔ Go/No-Go**: هر آیتم «خیر»، دلیلی مستندشده دارد یا یک «ریسک پذیرفته‌شدهٔ ثبت‌شده» یا «کار باقی‌ماندهٔ مشخص‌شده در Phase 2». عبور با آیتم‌های حل‌نشدهٔ بدون ثبت = No-Go.

---

# بخش ۱۰ — راهنمای عیب‌یابی (Troubleshooting)

بر اساس ریسک‌های بخش ۱۶ نسخهٔ ۰.۳، راهنمای حداقلی رایج‌ترین مشکلات Phase 1:

| مشکل | علت احتمالی | راه‌حل |
|---|---|---|
| Ollama پاسخ نمی‌دهد | مدل دانلود/انتقال نشده | فایل GGUF را با `docker cp` به volume منتقل کنید یا `docker exec ollama ollama pull qwen2.5:7b` (فقط در محیط متصل) |
| Qdrant Collection خالی | Ingestion کامل نشده | `docker compose logs app` را بررسی کنید؛ وضعیت `processing_jobs` و `documents.status` را در PG چک کنید |
| OpenSearch OOM یا restart مکرر | heap پیش‌فرض بزرگ | `OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m` (در Compose بخش ۷ تنظیم شده) |
| Embedding Model Load خطا | فایل مدل در volume نیست | `docker cp models/ app:/app/models/` و restart `app`؛ مسیر را در `.env` تطبیق بدهید |
| ACL Deny اشتباه (کاربر مجاز هم خالی می‌گیرد) | کاربر بدون ردیف ACL | `INSERT INTO document_acl (acl_id, document_id, subject_type, subject_id, permission) VALUES (...)`؛ subject_id باید user_id واقعی باشد |
| سند در وضعیت `failed` ماند | خطای پارس یا ایندکس | `error_log` در `processing_jobs` را بخوانید؛ پس از رفع، `python scripts/reconcile.py --status failed --retry` |
| `pip install --no-index` شکست خورد | wheel ناقص در bundle | در ماشین متصل دوباره `pip download -r requirements.txt` اجرا و SHA256SUMS بازسازی شود |
| Streamlit بالا نمی‌آید | پورت اشغال یا healthcheck fail | `docker compose logs app`؛ `_stcore/health` را داخل کانتینر تست کنید؛ `start_period` را افزایش دهید |
| Query خیلی کند است (>۱۰ ثانیه) | مدل LLM روی CPU یا Context بزرگ | TOP_K و MAX_CONTEXT_TOKENS را کم کنید؛ GPU (اختیاری در Compose) را فعال کنید |

**اصل کلی عیب‌یابی**: همیشه اول `docker compose ps` (وضعیت healthcheck) → سپس `docker compose logs <service>` → سپس وضعیت در PG (`documents`، `processing_jobs`، `audit_log`). ترتیب همین است؛ لاگ‌های JSON ساختاریافته L9 برای همین طراحی شده‌اند.

---

# بخش ۱۱ — دفتر ثبت تصمیمات بازبینی Phase 0 (C1–C9)

این بخش نتیجهٔ بازبینی تطبیق برنامهٔ **Phase 0** (هفتهٔ ۱–۴، Sprint 1–2، مبتنی بر SINA_Roadmap_v1.5) با مستندات POC است. ۱۰ تناقض شناسایی شد (C1–C10) و برای ۹ مورد آن (C1 تا C9) تصمیم نهایی گرفته شده است؛ **C10** (Branch Strategy + ADR) با سازگاری کامل تأیید و بدون تغییر پذیرفته شد. این تصمیمات از این پس **بخشی از مبنای طراحی این سند** هستند و در متن بخش‌های مربوطه علامت‌گذاری شده‌اند.

## ۱۱.۱. جدول تصمیمات نهایی

| کد | موضوع تناقض | تصمیم نهایی (همه گزینهٔ الف) | اثر بر این سند |
|---|---|---|---|
| C1 | Redis در Compose Phase 0 (تعارض با «بدون Broker» و سقف ۶ کانتینر) | **حذف Redis از Phase 0** — صف کارها همان `processing_jobs` در PG با Polling و `FOR UPDATE SKIP LOCKED` | نکات Compose بخش ۷ |
| C2 | حذف OpenSearch از Compose اولیهٔ Phase 0 | **OpenSearch از هفتهٔ اول در Compose** با `OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m` | Compose بخش ۷ — طبق همین نمونه |
| C3 | اجرای مدل با GGUF مستقیم از `/opt/models` به‌جای Ollama | **کانتینر Ollama** تنها مسیر سرو LLM است؛ بستهٔ آفلاین مدل را در `ollama_models` می‌گذارد | نکات Compose بخش ۷ + بخش ۶ (Bundle) |
| C4 | ONNX به‌جای Sentence Transformers برای Embedding | **Sentence Transformers با `multilingual-e5-large`**؛ `EMBEDDING_MODEL_VERSION` از روز اول در همهٔ محیط‌ها یکسان و قفل | `.env.example` بخش ۷ + بخش ۶.۶ (Rebuild) |
| C5 | Gitea/CI Runner/PyPI Mirror خارج از مستندات POC | **Compose جداگانهٔ `docker-compose.dev-infra.yml`** خارج از ۶ کانتینر POC؛ مالکیت و نگهداری در **ADR** | نکات Compose بخش ۷ |
| C6 | اسکیمای ۳ جدولی Task 4 به‌جای ۷ جدول | **Task 4 = اجرای `schema.sql` کامل بخش ۳** (۷ جدول با ACL و `processing_jobs`) | یادداشت ابتدای بخش ۳ |
| C7 | فرآیند HDD Update موازی با offline-bundle | **HDD Update = حمل فیزیکی offline-bundle** + `load_bundle.sh` + تطبیق `SHA256SUMS` — بدون مسیر دوم | یادداشت ابتدای بخش ۶ |
| C8 | اعداد QG0.2/QG0.3 بدون مبنا (پاسخ <۶۰s، ایندکس ۱۰ Chunk <۱۰s، CI <۵ دقیقه) | **Spike نیم‌روزه در هفتهٔ ۱** روی سخت‌افزار هدف؛ اندازه‌گیری واقعی Qwen2.5-7B Q4 و e5-large، سپس قطعی اعداد QG و ثبت در Roadmap v1.5 | «گام صفر» بخش ۱ |
| C9 | دو Roadmap موازی (مالکیت سند) | **Roadmap v1.5 حاکم بر زمان‌بندی/SP**؛ این سند حاکم بر «چگونه»؛ نگاشت: گام ۱→Task 2، گام ۲→Task 4، گام ۳→Sprint 3 | بخش ۱ (نگاشت) |
| C10 | Branch Strategy + ADR | **پذیرش بدون تغییر** — با سازگاری کامل؛ آیتم خلا سازگار | — |

## ۱۱.۲. اثر تجمیعی تصمیمات بر قیدهای معماری

- **تعداد کانتینرهای POC**: با حذف Redis (C1) و انتقال Gitea/CI/Mirror به dev-infra (C5)، سقف **۶ کانتینر** پابرجا می‌ماند: `app`، `ollama`، `qdrant`، `opensearch`، `minio`، `postgres`. خروجی `docker compose ps` در گام ۱ باید دقیقاً همین ۶ سرویس را نشان دهد.
- **اصل «بدون Message Broker»**: با C1 حفظ شد؛ تنها مکانیزم صف، Polling دیتابیسی است (الگوی بخش ۲.۳).
- **سناریوهای T1–T8**: با C2 از همان Sprint اول روی Stack کامل (شامل OpenSearch) قابل اجرا هستند — نیاز به «محیط آزمون موقت» حذف شد.
- **Quality Gate 0**: با C8، معیارهای QG0.2/QG0.3 تا پایان Spike در هفتهٔ ۱ وضعیت «موقت/غیرقطعی» دارند و پس از اندازه‌گیری قطعی و ثبت در Roadmap v1.5، gate قابل ارزیابی است.
- **زنجیرهٔ آپدیت آفلاین**: با C7 فقط یک مسیر وجود دارد: ماشین متصل (`create_bundle.sh`) → دیسک فیزیکی → ماشین ایزوله (`load_bundle.sh` + `SHA256SUMS`) → Audit Log.

## ۱۱.۳. اقدامات باقی‌مانده بر عهدهٔ تیم

- [ ] تنظیم فایل `docker-compose.dev-infra.yml` و ثبت ADR مالکیت آن (C5) — مالک: تعیین‌شده در Phase 0
- [ ] اجرای Spike نیم‌روزه و ثبت نتایج در Roadmap v1.5 برای قطعی‌سازی QG0.2/QG0.3 (C8)
- [ ] به‌روزرسانی SINA_Roadmap_v1.5 با رفع تناقض‌های C1، C2، C3، C4 و C6 از متن Taskهای Sprint 1–2
- [ ] بازبینی و تأیید نهایی `schema.sql` (بخش ۳) پیش از اجرای Task 4
- [ ] تأیید تگ‌های تصاویر Docker در Compose بخش ۷ (همه «نیازمند تأیید»)

**وضعیت کل**: تصمیمات C1–C9 گرفته شده‌اند اما مطابق سیاست نسخه‌های این سند، تأیید نهایی و ثبت رسمی در نهاد معماری (ثبت در Roadmap/ADR) همچنان بر عهدهٔ مالکان مستندات است.

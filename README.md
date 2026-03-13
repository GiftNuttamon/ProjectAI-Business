# ChatBuddy AI — ระบบผู้ช่วยแม่ค้าออนไลน์อัจฉริยะ

> **AI-Powered LINE Chatbot สำหรับธุรกิจ SME ไทย**  
> รับออเดอร์ · วิเคราะห์ลูกค้า · บันทึกข้อมูล · แสดง Dashboard อัตโนมัติ

---

## สารบัญ

- [ภาพรวมระบบ](#-ภาพรวมระบบ)
- [Tech Stack](#-tech-stack)
- [สถาปัตยกรรม](#-สถาปัตยกรรม)
- [Workflow ที่มีในโปรเจกต์](#-workflow-ที่มีในโปรเจกต์)
- [โครงสร้างโปรเจกต์](#-โครงสร้างโปรเจกต์)
- [การติดตั้งและใช้งาน](#-การติดตั้งและใช้งาน)
- [Google Sheets Database](#-google-sheets-database)
- [Dashboard](#-dashboard)
- [การตั้งค่า](#-การตั้งค่า)

---

## ภาพรวมระบบ

ChatBuddy คือระบบ **AI Chatbot สำหรับ LINE** ที่ออกแบบมาเพื่อช่วยแม่ค้าออนไลน์ไทย (SME) จัดการการสนทนากับลูกค้าแบบอัตโนมัติ ตั้งแต่รับออเดอร์ ตอบคำถามสินค้า ไปจนถึงแจ้งสถานะการจัดส่ง

### ฟีเจอร์หลัก

| ฟีเจอร์ | รายละเอียด |
|---|---|
| **รับออเดอร์อัตโนมัติ** | AI เข้าใจและบันทึกออเดอร์จากการสนทนาลงชีทได้เลย |
| **ตอบคำถามสินค้า** | ดึงข้อมูลสินค้า ราคา สต็อก จาก Database แบบ Real-time |
| **เช็คสถานะออเดอร์** | ลูกค้าถามสถานะพัสดุได้ทันทีผ่านชื่อหรือเบอร์โทร |
| **เช็คสลิปโอนเงิน** | ตรวจสอบสลิปธนาคารจากลูกค้า |
| **จัดการ Complaint** | ตรวจจับลูกค้าไม่พอใจ แจ้งเตือน Admin ทันที |
| **Dashboard วิเคราะห์** | สรุป Intent, Sentiment, ยอดขาย แบบ Real-time |
| **จำบริบทการสนทนา** | Memory Buffer เก็บบริบทแยกตาม userId |

---

## Tech Stack

```
LINE Messaging API    →  รับส่งข้อความกับลูกค้า
n8n (Self-hosted)     →  Workflow Automation Engine
OpenRouter API        →  LLM (arcee-ai/trinity-large-preview)
Google Sheets API     →  Database (Orders, Products, Complaints)
Docker + Nginx        →  Deploy n8n + Dashboard
ngrok                 →  Expose localhost เป็น Public URL
EasySlip,ThunderSlip  →  เช็ค Slip จากธนาคารประเทศไทย
```

---

## สถาปัตยกรรม

```
ลูกค้า (LINE)
     │
     ▼
LINE Webhook ──────────► n8n Workflow
                              │
                    ┌─────────┼──────────┐
                    │         │          │
             Pre-process   AI Agent   Parse Output
             Payload       (LLM)          │
                    │                     │
                    │          ┌──────────┼──────────┬──────────┐
                    │          │          │          │          │
                    │       Purchase  Complaint  Inquiry  Status_Check
                    │          │          │          │          │
                    │     Save Order  Save       (ตอบจาก    Fetch Order
                    │     to Sheets  Complaint   Tool)     Build Reply
                    │          │          │
                    └──────────┴──────────┘
                              │
                    Reply to LINE + Log to Sheets
```

---

### WorkFlow

```
Webhook → Pre-process Payload → [Is Image?]
                                    │ No
                                    ▼
                               AI Agent (LLM + Tools)
                                    │
                            Parse Agent Response
                                    │
                    ┌───────────────┼───────────────┬───────────────┐
                    │               │               │               │
              Is Negative?    Is Purchase?    Is Inquiry?   Is Status Check?
                    │               │               │               │
              Save Complaint  Save Order      (AI ตอบ)      Fetch Orders
              + Notify Admin  to Sheets                    Build Status Reply
                    │               │               │               │
                    └───────────────┴───────────────┴───────────────┘
                                    │
                            Reply to LINE
```

---

## โครงสร้างโปรเจกต์

```
ProjectAI-Business/
├── workflows/                        # n8n Workflow Files (JSON)
│   ├── ChatBuddy v3 — True Agentic.json   # Workflow หลัก
│
├── docker/                           # Docker Configuration
│   ├── docker-compose.yml            # n8n + Nginx services
│   ├── dashboard/
│   │   └── public/
│   │       └── dashboard.html        # ChatBuddy Insights Dashboard
│   ├── n8n_data/                     # n8n persistent data
│   └── files/                        # Shared files
│
├── dashboard/                        # Dashboard source (dev)
│   └── public/
│       └── dashboard.html
│
├── ChatBuddy_DevGuide.docx           # คู่มือนักพัฒนา
├── ChatBuddy_UserManual.docx         # คู่มือผู้ใช้งาน
└── ขั้นตอน setting N8N+Docker เบื้องต้น.pdf
```

---

## การติดตั้งและใช้งาน

### ข้อกำหนดเบื้องต้น

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [ngrok](https://ngrok.com/) (สำหรับ Webhook URL)
- LINE Messaging API Channel
- OpenRouter API Key
- Google Cloud Project (สำหรับ Sheets API)

### 1. Clone และ Setup

```bash
git clone https://github.com/GiftNuttamon/ProjectAI-Business/
cd ProjectAI-Business/docker
```

### 2. รัน Docker Compose

```bash
docker compose up -d
```

Services ที่จะ start:
- **n8n** → `http://localhost:5678`
- **Dashboard (Nginx)** → `http://localhost:8080`

### 3. Setup ngrok

```bash
ngrok http 5678
```

คัดลอก URL ที่ได้ (เช่น `https://xxxx.ngrok-free.dev`) ไปใส่ใน `docker-compose.yml`:

```yaml
- N8N_HOST=https://your-ngrok-url.ngrok-free.dev
- WEBHOOK_URL=https://your-ngrok-url.ngrok-free.dev
```

### 4. Import Workflow เข้า n8n

1. เปิด n8n ที่ `http://localhost:5678`
2. ไปที่ **Workflows → Import from file**
3. เลือกไฟล์จากโฟลเดอร์ `workflows/`

### 5. ตั้งค่า Credentials ใน n8n

| Credential | วิธีการ |
|---|---|
| **OpenRouter API** | ใส่ API Key จาก [openrouter.ai](https://openrouter.ai) |
| **Google Sheets OAuth2** | เชื่อมต่อผ่าน Google OAuth |
| **LINE Messaging API** | ใส่ Channel Access Token จาก LINE Developers |

### 6. ตั้งค่า LINE Webhook

ไปที่ [LINE Developers Console](https://developers.line.biz/) → เลือก Channel → **Basic Settings**

```
Webhook URL: https://your-ngrok-url.ngrok-free.dev/webhook/tnibot
```

---

## Google Sheets Database

Spreadsheet หนึ่งไฟล์ประกอบด้วย 4 Sheets:

### Sheet: `Order`
เก็บข้อมูลออเดอร์ลูกค้า 

### Sheet: `Products`
เก็บข้อมูลสินค้า ราคา สต็อก และโปรโมชันส

### Sheet: `Complaints`
บันทึกข้อร้องเรียนและ Negative Sentiment จากลูกค้า

### Sheet: `Log_Others`
บันทึก Intent ทั่วไป เช่น ทักทาย ขอบคุณ

---

## 📈 Dashboard

**ChatBuddy Insights Dashboard** — เข้าถึงได้ที่ `http://localhost:8080`

### KPI ที่แสดง

| KPI | คำอธิบาย |
|---|---|
| ยอดขายวันนี้ | รวมจาก Purchase + Payment |
| Conversion Rate | Purchase / ยอดแชททั้งหมด |
| แชทตกหล่น | รายการที่ยังรอตอบ |
| ลูกค้าโกรธ | จำนวน Negative Sentiment (ด่วน) |

### กราฟและข้อมูล

- **Line Chart** — สินค้า: ยอดซื้อ
- **Doughnut Chart** — อารมณ์ลูกค้า (Positive / Neutral / Negative)
- **Intent Summary** — สรุปสาเหตุที่ลูกค้าทักมา
- **Real-time Feed** — ตารางรายการล่าสุดพร้อม Tag และ Sentiment

### การเชื่อมต่อกับ Google Sheets จริง

แก้ไขใน `dashboard.html`:

```javascript
const GOOGLE_SHEET_ID = 'YOUR_SPREADSHEET_ID_HERE';
const SHEET_TAB_NAME = 'Order';
```

> ต้อง Publish Google Sheet เป็น Web (File → Share → Publish to web) เพื่อให้ Dashboard อ่านข้อมูลได้

---

## การตั้งค่า

### Environment Variables (docker-compose.yml)

```yaml
N8N_BASIC_AUTH_ACTIVE: true
N8N_BASIC_AUTH_USER: admin
N8N_BASIC_AUTH_PASSWORD: <your-password>
N8N_PORT: 5678
N8N_HOST: https://<ngrok-url>
WEBHOOK_URL: https://<ngrok-url>
TZ: Asia/Bangkok
```

### Intent Classification

| Intent | ความหมาย | การจัดการ |
|---|---|---|
| `Purchase` | ลูกค้าต้องการสั่งซื้อ | บันทึกออเดอร์ลง Sheets |
| `Inquiry` | ถามข้อมูลสินค้า/ราคา | ดึงข้อมูลจาก Tool แล้วตอบ |
| `Status_Check` | เช็คสถานะพัสดุ | ค้นหาออเดอร์และแจ้งสถานะ |
| `Complaint` | ร้องเรียน/ไม่พอใจ | บันทึก + แจ้ง Admin ทันที |
| `Other` | ทักทาย ขอบคุณ ทั่วไป | ตอบกลับและ Log |

### สถานะออเดอร์

| Status | ความหมาย |
|---|---|
| `Pending_Payment`  | รอชำระเงิน |
| `Paid`  | จ่ายเงินแล้ว |
| `Shipped`  | จัดส่งแล้ว |
| `Delivered`  | ส่งถึงแล้ว |

---

## เอกสารเพิ่มเติม

- [คู่มือนักพัฒนา](./ChatBuddy_DevGuide.docx)
- [คู่มือผู้ใช้งาน](./ChatBuddy_UserManual.docx)
- [ขั้นตอน Setting N8N+Docker เบื้องต้น](./ขั้นตอน%20setting%20N8N+Docker%20เบื้องต้น.pdf)

---

## หมายเหตุสำหรับนักพัฒนา

- **LLM ที่ใช้:** `arcee-ai/trinity-large-preview:free` ผ่าน OpenRouter (สามารถเปลี่ยนได้ใน node `OpenRouter Chat Model`)
- **Timezone:** ตั้งค่าเป็น `Asia/Bangkok` (UTC+7)
---


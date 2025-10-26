เอกสารโครงการ: The n8n AI Architect (รหัส: Archon)
เวอร์ชัน: 0.1.0
วันที่: 26 ตุลาคม 2025
ผู้จัดทำ: Manus & [TJSierra]
1. ภาพรวมและเป้าหมาย (Overview & Vision)
The n8n AI Architect (Archon) คือแพลตฟอร์มผู้ช่วยพัฒนา AI อัจฉริยะสำหรับ n8n ที่ถูกออกแบบมาเพื่อปฏิวัติวิธีการสร้าง, ทดสอบ, จัดการ, และทำเอกสาร Workflow โดยเปลี่ยนจากการทำงานด้วยมือ (Manual) ไปสู่การทำงานด้วยการสั่งการผ่านภาษาธรรมชาติ (Natural Language Commands)
เป้าหมายหลัก:
เร่งความเร็วในการพัฒนา (Accelerate Development): ลดเวลาในการสร้าง Workflow ที่ซับซ้อนจากหลายชั่วโมงเหลือเพียงไม่กี่นาที
ทำงานอัตโนมัติ (Automate Tedious Tasks): จัดการงานที่น่าเบื่อและเกิดข้อผิดพลาดได้ง่ายโดยอัตโนมัติ เช่น การทำเอกสาร, การทดสอบ, และการดีบัก
เพิ่มคุณภาพและความน่าเชื่อถือ (Enhance Quality & Reliability): สร้าง Workflow ที่ผ่านการทดสอบ, มีเอกสารประกอบครบถ้วน, และสามารถแก้ไขข้อผิดพลาดของตัวเองได้
สร้างศูนย์กลางความรู้ (Centralize Knowledge): ใช้ AI ในการวิเคราะห์และสร้างเอกสารภาพรวมสถาปัตยกรรมของระบบทั้งหมด ทำให้ทุกคนในทีมเข้าใจการทำงานของ Workflow ได้ง่าย
2. สถาปัตยกรรมระบบ (System Architecture)
ระบบทั้งหมดทำงานบน Docker Compose และประกอบด้วย Service หลักดังนี้:
n8n Service:
หน้าที่: เป็น User Interface หลักที่ผู้ใช้โต้ตอบด้วย และเป็นที่สำหรับรัน "Orchestrator Workflow"
Database: เชื่อมต่อกับ Postgres เพื่อเก็บข้อมูล Workflows และ Executions อย่างถาวร
Version: ถูกล็อคเวอร์ชันไว้อย่างชัดเจน (N8N_VERSION) เพื่อความเสถียร
claude-archon Service:
หน้าที่: เป็น "สมอง" ของระบบ ประกอบด้วย Claude CLI และเครื่องมือ (MCPs) ที่จำเป็นทั้งหมด
การทำงาน: รับคำสั่งจาก n8n ผ่าน Execute Command Node และประมวลผลโดยอ้างอิงจาก Config และ Context ใน /config Volume
Memory:
Long-term Memory: เชื่อมต่อกับ Qdrant (Vector Database) เพื่อเก็บและค้นหาข้อมูล Context, เอกสารเก่า, หรือตัวอย่าง Workflow ที่ซับซ้อน (Semantic Search)
Short-term/Cache Memory: เชื่อมต่อกับ Redis เพื่อใช้เป็น Cache สำหรับ Prompt, ผลลัพธ์ที่เคยประมวลผล, หรือการจัดการสถานะ (State) ชั่วคราว
postgres Service:
หน้าที่: เป็นฐานข้อมูลหลักของ n8n สำหรับเก็บข้อมูลทั้งหมด
qdrant Service:
หน้าที่: เป็น "หน่วยความจำระยะยาว" ของ Claude ช่วยให้ AI สามารถค้นหาข้อมูลที่เกี่ยวข้องจากอดีตมาใช้ตอบคำถามปัจจุบันได้
redis Service:
หน้าที่: เป็น "หน่วยความจำระยะสั้น" และ Cache ช่วยเพิ่มความเร็วในการตอบสนองและจัดการสถานะชั่วคราว
Volumes:
n8n_data: เก็บข้อมูลของ n8n
archon_config: (สำคัญที่สุด) เป็นโฟลเดอร์ที่เก็บการตั้งค่าและกฎเกณฑ์ทั้งหมดของ AI ซึ่งจะถูก Mount เข้าไปใน claude-archon Service
3. โครงสร้างโฟลเดอร์ควบคุม AI (archon_config)
นี่คือหัวใจของการควบคุมพฤติกรรมของ AI โครงสร้างจะถูกจัดระเบียบดังนี้:
Plain Text
archon_config/
├── settings.json               # ไฟล์ตั้งค่าหลักของ AI
├── system_prompt.md            # Prompt หลัก กำหนดตัวตนและกฎเหล็กของ AI
│
├── modes/                      # โฟลเดอร์สำหรับกำหนด "โหมด" การทำงาน
│   ├── 01_code.md              # Prompt สำหรับ Code Mode
│   ├── 02_test.md              # Prompt สำหรับ Test Mode
│   ├── 03_document.md          # Prompt สำหรับ Document Mode
│   ├── 04_architecture.md      # Prompt สำหรับ Architecture Mode
│   └── 05_debug.md             # Prompt สำหรับ Debug Mode
│
└── context/                    # โฟลเดอร์สำหรับเก็บข้อมูลบริบทถาวร
    ├── company_guidelines.md   # แนวทางการพัฒนาขององค์กร
    ├── security_rules.md       # กฎด้านความปลอดภัยที่ต้องยึดถือ
    └── examples/               # ตัวอย่าง Workflow ที่ดี
        └── webhook_to_line.json
3.1. settings.json (ตัวอย่าง)
ไฟล์นี้ใช้กำหนดค่าการทำงานต่างๆ ของ Claude CLI
JSON
{
  "model": "claude-3-opus-20240229",
  "temperature": 0.1,
  "max_tokens": 4096,
  "allowed_shell_commands": [
    "n8n",
    "git",
    "jq",
    "cat"
  ],
  "vector_db_config": {
    "host": "qdrant",
    "port": 6333
  },
  "cache_config": {
    "host": "redis",
    "port": 6379
  }
}
4. แผนการดำเนินงาน (Roadmap)
Phase 1: The Creator (สร้าง PoC)
[TJSierra] ตั้งค่า Docker Compose และ Environment ทั้งหมด
[Manus] ออกแบบโครงสร้าง Orchestrator Workflow ใน n8n
[Manus] พัฒนาสคริปต์ claude-cli เบื้องต้นที่สามารถอ่าน settings.json และ Prompt จากไฟล์ได้
เป้าหมาย: ทำให้ Code Mode พื้นฐานทำงานได้: พิมพ์ใน Note -> Workflow ใหม่ถูก Import เข้ามาได้สำเร็จ
Phase 2: The Documenter
พัฒนาระบบอ่านข้อมูล Workflow ที่มีอยู่
สร้าง Prompt สำหรับ Document Mode และ Architecture Mode
เป้าหมาย: สามารถสั่ง "@claude mode: document" แล้ว AI สร้างไฟล์ README.md ใน Sticky Note ได้
Phase 3: The Debugger & Tester
พัฒนากลไก Feedback Loop (ส่งผลลัพธ์การรันกลับไปให้ AI)
สร้าง Prompt สำหรับ Debug Mode และ Test Mode
เป้าหมาย: เมื่อ Workflow มี Error, AI สามารถวิเคราะห์และเสนอวิธีแก้ใน DEBUG.md ได้
# Archon

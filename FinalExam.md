# ข้อสอบปลายภาค

**ENG23 3074 — Serverless and Cloud Architectures**

| รายการ | รายละเอียด |
|---|---|
| ภาคการศึกษา | 3/2568 |
| คะแนนเต็ม | **100 คะแนน** |
| รูปแบบ | Take-home Exam |
| ระยะเวลา | 24–48 ชั่วโมง |
| การส่ง | ไฟล์ **`.md`** หนึ่งไฟล์ ตั้งชื่อ `<รหัสนักศึกษา>_FinalExam.md` |

---

## คำชี้แจง

1. ข้อสอบเป็นแบบ **ถาม–ตอบ** มี 5 ส่วนตามหัวข้อที่เรียน รวม 15 ข้อ
2. ตอบใต้แต่ละข้อในไฟล์นี้ (ลบข้อความใน _[...]_ แล้วเขียนคำตอบแทน)
3. **Open book / Open internet**
4. **ทำคนเดียว** ห้ามปรึกษาเพื่อน
5. **ตอบให้กระชับ ตรงประเด็น** — ปริมาณไม่ใช่คะแนน ความเข้าใจคือคะแนน

---

## สถานการณ์อ้างอิง (ใช้ตอบข้อที่ระบุว่า "อ้างอิงสถานการณ์")

> ทีมของคุณกำลังเปลี่ยนเว็บแอป **Python Flask + MySQL** ที่เดิม deploy ด้วยการ SSH เข้าไป `git pull` แล้ว restart (ระบบล่มทุกครั้ง 15–20 นาที) ให้เป็นระบบ cloud-native รันบน **Kubernetes on-premise** มีทีม dev 4 คน ทีม ops 2 คน ใช้ Jenkins, Docker, Terraform, Ansible, Prometheus, Grafana

---

## ข้อมูลผู้ทำข้อสอบ

| | |
|---|---|
| ชื่อ–นามสกุล | นาย ณัฐทักษ์ดนัย แหยงกระโทก |
| รหัสนักศึกษา | B6611613 |

---

## ส่วนที่ 1: Git & CI/CD (25 คะแนน)

### ข้อ 1.1 (8 คะแนน) — *อ้างอิงสถานการณ์*

ทีม dev 4 คนเดิม push เข้า `main` พร้อมกันทำให้ conflict บ่อย ออกแบบ **Git branching strategy** ให้ทีมนี้ อธิบาย: branch ที่มี, วัตถุประสงค์ของแต่ละ branch, และ flow เมื่อจะ deploy ขึ้น production

> **ตอบ:** ใช้แนวทางแบบ short-lived feature branch + protected main เพราะทีมมี dev 4 คน ถ้าให้ทุกคน push เข้า main ตรง ๆ จะชนกันง่ายและทำให้ production พังได้

### ข้อ 1.2 (10 คะแนน)

CI/CD pipeline ของระบบนี้ควรมี stage อะไรบ้างเรียงตามลำดับ (ตั้งแต่ developer push code จนถึง deploy)? อธิบายว่า **แต่ละ stage ทำอะไร** และ **ทำไมต้องเรียงลำดับนี้**

>**ตอบ:**
>
>1. **Developer push code หรือ Pull Request**  
>   จุดเริ่มต้นคือ dev แก้ code แล้ว push ขึ้น Git เพื่อให้ระบบ CI/CD เริ่มทำงานอัตโนมัติ
>
>2. **Jenkins รับ webhook จาก Git**  
>   เมื่อมีการ push หรือ merge Git จะส่งสัญญาณไปหา Jenkins เพื่อให้เริ่ม pipeline ทันที ไม่ต้องให้คนกดเอง
>
>3. **Checkout source code**  
>   Jenkins ดึง code version ล่าสุดจาก Git มาใช้ build เพื่อให้มั่นใจว่าใช้ code ตัวเดียวกับที่ dev push จริง
>
>4. **Lint / Format / Static check**  
>   ตรวจ style และ syntax ของ Python Flask ก่อน เช่น import ผิด ตัวแปรผิด หรือ code ที่เสี่ยงพังง่าย ต้องทำก่อน test เพราะ code ยังเขียนผิดพื้นฐานก็ไม่ควรเสียเวลา build ต่อ
>
>5. **Run unit test / integration test**  
>   ทดสอบ logic ของระบบ เช่น API, database connection และ function สำคัญ เพื่อเช็กว่าแก้ code แล้วไม่ทำให้ของเดิมพัง
>
>6. **Build Docker image**  
>   ถ้า test ผ่าน Jenkins จะ build application เป็น Docker image เพื่อให้รันเหมือนกันทุก environment ไม่ว่าจะ dev, staging หรือ production
>
>7. **Push Docker image ไปที่ registry**  
>   เอา image ที่ build แล้วไปเก็บใน Docker Registry เช่น Docker Hub หรือ private registry เพื่อให้ Kubernetes ดึงไปใช้งานได้
>
>8. **Deploy ไป staging บน Kubernetes**  
>   Deploy ไป environment ทดสอบก่อน production เพื่อดูว่าระบบทำงานจริงบน Kubernetes ได้ไหม
>
>9. **Smoke test หลัง deploy**  
>   ทดสอบแบบเร็ว ๆ เช่น health check, login API, connect MySQL ได้ไหม เพื่อเช็กว่า container ไม่ได้แค่ start ได้ แต่ระบบใช้งานได้จริง
>
>10. **Manual approval ก่อน production**  
>    ให้คนในทีม เช่น dev หรือ ops ตรวจและยืนยันก่อน deploy จริง เพื่อลดโอกาสที่ code ผิดจะหลุดขึ้น production
>
>11. **Deploy ไป production**  
>    Jenkins สั่ง Kubernetes update deployment เช่นเปลี่ยน image version ใหม่ ระบบจะค่อย ๆ rollout โดยไม่ต้อง SSH เข้าเครื่องเอง
>
>12. **Monitoring ด้วย Prometheus + Grafana**  
>    หลัง deploy ต้องดู metric เช่น CPU, memory, error rate และ response time ถ้ามีปัญหาจะได้รู้เร็วและ rollback ได้
>
>**เหตุผลที่ต้องเรียงลำดับแบบนี้:**  
>เพราะต้องตรวจ code ก่อน build, ต้อง build ก่อน deploy และต้องทดสอบบน staging ก่อนขึ้น production เพื่อให้ลดความเสี่ยงกับระบบจริง

### ข้อ 1.3 (7 คะแนน)

การเปลี่ยนจาก deploy แบบ manual (SSH + git pull) มาเป็น CI/CD อัตโนมัติด้วย Jenkins **แก้ปัญหาอะไรได้บ้าง** และ **webhook** มีบทบาทอย่างไรในกระบวนการนี้?

> **ตอบ:** 
>1. ลด human error เพราะเดิมต้องให้คนจำคำสั่งเอง เช่น SSH เข้าเครื่อง, git pull, restart service ถ้าลืม restart หรือดึงผิด branch ระบบก็พังได้ แต่ Jenkins จะทำตามขั้นตอนเดิมทุกครั้ง
>2. ลด downtime เดิม restart ระบบทีใช้เวลา 15–20 นาที และต้องทำมือเอง แต่ถ้าใช้ **Kubernetes** ร่วมกับ **Jenkins** สามารถ deploy แบบ rolling update ได้
>3. ตรวจเจอ error ก่อนขึ้น production เพราะ **Jenkins** สามารถรัน test, build Docker image และ deploy staging ก่อน ถ้า test ไม่ผ่าน pipeline จะหยุดทันที
>4. rollback ง่ายกว่าเดิม เพราะแต่ละรอบ deploy จะมี Docker image tag ชัดเจน เช่น commit hash ถ้า version ใหม่มีปัญหาก็ย้อนกลับไป image เก่าได้ ไม่ต้องเดาว่า server ตอนนั้นใช้ code ชุดไหน
>ส่วน **webhook** มีหน้าที่เป็นตัวแจ้ง **Jenkins** ว่ามีการเปลี่ยนแปลงใน Git แล้ว เช่น dev push code หรือ merge เข้า main จากนั้น Jenkins จะเริ่ม pipeline อัตโนมัติทันที
---

## ส่วนที่ 2: Docker & Containerization (20 คะแนน)

### ข้อ 2.1 (7 คะแนน)

อธิบายว่า **container ต่างจาก virtual machine (VM) อย่างไร** และการ containerize แอป Flask + MySQL ช่วยแก้ปัญหา "บนเครื่องผมรันได้แต่บน server รันไม่ได้" ได้อย่างไร

> **ตอบ:** _[เขียนที่นี่]_

### ข้อ 2.2 (7 คะแนน)

ทำไมในระบบ production **ไม่ควรใช้ Docker image tag `latest`**? ถ้าไม่ใช้ `latest` ควรใช้รูปแบบ tag แบบไหนแทน และมีผลต่อการ rollback อย่างไร?

> **ตอบ:** _[เขียนที่นี่]_

### ข้อ 2.3 (6 คะแนน)

Docker image ประกอบด้วย layer หลายชั้น การเรียงคำสั่งใน Dockerfile มีผลต่อ **ความเร็วในการ build** และ **ขนาด image** อย่างไร? ยกตัวอย่างการเรียงที่ดี 1 ตัวอย่าง

> **ตอบ:** _[เขียนที่นี่]_

---

## ส่วนที่ 3: Infrastructure as Code (20 คะแนน)

### ข้อ 3.1 (8 คะแนน)

**Terraform** กับ **Ansible** ต่างกันอย่างไร? (ตอบในแง่ declarative vs imperative และ provisioning vs configuration management)

> **ตอบ:** _[เขียนที่นี่]_

### ข้อ 3.2 (6 คะแนน) — *อ้างอิงสถานการณ์*

ในระบบนี้ งานต่อไปนี้ควรใช้ Terraform หรือ Ansible? ตอบพร้อมเหตุผลสั้น ๆ แต่ละข้อ:
- (ก) สร้าง Kubernetes namespace และ persistent volume
- (ข) ติดตั้ง Docker และ kubelet บน server ทั้ง 5 เครื่อง
- (ค) ตั้งค่า cron job สำหรับ backup ฐานข้อมูล

> **ตอบ:** _[เขียนที่นี่]_

### ข้อ 3.3 (6 คะแนน)

**Idempotency** คืออะไร และทำไมจึงสำคัญสำหรับ Infrastructure as Code? (อธิบายว่าถ้ารัน script เดิมซ้ำ 2 ครั้งควรเกิดอะไรขึ้น)

> **ตอบ:** _[เขียนที่นี่]_

---

## ส่วนที่ 4: Kubernetes (20 คะแนน)

### ข้อ 4.1 (6 คะแนน)

อธิบายบทบาทของ **Pod**, **Deployment**, และ **Service** ใน Kubernetes และความสัมพันธ์ระหว่างทั้งสาม

> **ตอบ:** _[เขียนที่นี่]_

### ข้อ 4.2 (7 คะแนน)

**Readiness probe** กับ **Liveness probe** ต่างกันอย่างไร? ถ้าแอปตั้ง liveness probe ผิด (เช่น ตรวจถี่เกินไป หรือ timeout สั้นเกิน) จะเกิดปัญหาอะไร?

> **ตอบ:** _[เขียนที่นี่]_

### ข้อ 4.3 (7 คะแนน) — *อ้างอิงสถานการณ์*

ระบบเดิม deploy แล้วล่ม 15–20 นาที Kubernetes ทำ **zero-downtime deployment** ได้อย่างไร (อธิบายกลไก rolling update)? และถ้า deploy เวอร์ชันใหม่แล้วพบ bug จะ **rollback** อย่างไร?

> **ตอบ:** _[เขียนที่นี่]_

---

## ส่วนที่ 5: Monitoring (15 คะแนน)

### ข้อ 5.1 (8 คะแนน)

**Prometheus** และ **Grafana** ทำหน้าที่ต่างกันอย่างไร? และสำหรับเว็บแอปนี้ ควรเก็บ metric อะไรบ้างอย่างน้อย 3 ตัว พร้อมเหตุผล

> **ตอบ:** _[เขียนที่นี่]_

### ข้อ 5.2 (7 คะแนน)

ทีม ops อยากรู้ปัญหา **ก่อน** ผู้ใช้ร้องเรียน ควรตั้ง alert อะไรบ้าง? ยกตัวอย่าง alert 2 ข้อ พร้อม condition และเหตุผลว่าทำไม alert นี้สำคัญ

> **ตอบ:** _[เขียนที่นี่]_

---

# เกณฑ์การให้คะแนน (สำหรับอาจารย์)

| ส่วน | ข้อ | คะแนน | เกณฑ์เต็ม | เกณฑ์หักคะแนน |
|---|---|---|---|---|
| **1. Git & CI/CD** | 1.1 | 8 | branching เหมาะกับทีม + flow deploy ชัด | ขาด flow / ไม่เหมาะกับ 4 คน |
| | 1.2 | 10 | stage ครบ เรียงถูก + อธิบายเหตุผลลำดับ | ขาด stage / ไม่อธิบายลำดับ |
| | 1.3 | 7 | ระบุปัญหาที่แก้ + บทบาท webhook ถูก | ตอบลอย ไม่เชื่อมปัญหาเดิม |
| **2. Docker** | 2.1 | 7 | แยก container/VM ถูก + อธิบาย portability | สับสน container กับ VM |
| | 2.2 | 7 | เหตุผลไม่ใช้ latest + ผลต่อ rollback | ตอบแค่ "ไม่ดี" ไม่มีเหตุผล |
| | 2.3 | 6 | เข้าใจ layer cache + ตัวอย่างถูก | ไม่เข้าใจ layer |
| **3. IaC** | 3.1 | 8 | แยก declarative/imperative + provision/config ถูก | สลับบทบาท 2 tool |
| | 3.2 | 6 | ตอบถูก 3 ข้อ พร้อมเหตุผล (ก,ข Terraform/Ansible; ค Ansible) | ตอบผิด/ไม่มีเหตุผล |
| | 3.3 | 6 | อธิบาย idempotency + รันซ้ำได้ผลเดิม | ไม่เข้าใจแนวคิด |
| **4. Kubernetes** | 4.1 | 6 | บทบาท 3 ตัว + ความสัมพันธ์ถูก | สับสนบทบาท |
| | 4.2 | 7 | แยก readiness/liveness + ผลของการตั้งผิด | แยกไม่ออก |
| | 4.3 | 7 | อธิบาย rolling update + rollback ถูก | ไม่เข้าใจ zero-downtime |
| **5. Monitoring** | 5.1 | 8 | แยกบทบาท Prometheus/Grafana + 3 metric มีเหตุผล | สับสนบทบาท / metric ลอย |
| | 5.2 | 7 | 2 alert มี condition + เหตุผล proactive | alert reactive / ไม่มี condition |

## เกณฑ์เกรด

| คะแนน | เกรด | ลักษณะ |
|---|---|---|
| 90–100 | **A** | เข้าใจลึกทุกหัวข้อ ตอบเชื่อมโยงกับสถานการณ์จริง เห็นเหตุผลและ trade-off |
| 75–89 | **B** | เข้าใจดี ตอบถูกเป็นส่วนใหญ่ แต่บางข้อยังไม่ลึก |
| 60–74 | **C** | เข้าใจพื้นฐาน แต่มีจุดสับสนหรือตอบผิวเผินหลายข้อ |
| 50–59 | **D** | ตอบผิด/ไม่เข้าใจหลายหัวข้อ |
| < 50 | **F** | ไม่เข้าใจแนวคิดหลักของวิชา |

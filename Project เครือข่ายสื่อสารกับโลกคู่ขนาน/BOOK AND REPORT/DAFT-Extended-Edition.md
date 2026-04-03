# DAFT: Dynamic Adaptive Framing Topology [Extended Edition]
**เอกสารอธิบายโครงสร้างแพ็กเก็ต (Packet Structure) และกลไก FEC**

## 1. Introduction to DAFT
DAFT คือโครงสร้างการจัดรูปแบบแพ็กเก็ต (Packet Framing) ที่ถูกคิดค้นมาเพื่อใช้งานร่วมกับ UKB Protocol โดยเฉพาะ ในรุ่น **Extended Edition** นี้ โครงสร้างได้ถูกปรับปรุงให้รองรับความเป็นไดนามิก (Dynamic Allocation) และสถาปัตยกรรมชดเชยความผิดพลาด (FEC Redundancy)

## 2. The Knowledge Pod Structure (โครงสร้างหน่วยข้อมูล)
ในสถาปัตยกรรม TIA เราไม่ใช้คำว่า "Packet" แต่ใช้คำว่า **"Knowledge Pod"** เพราะแต่ละชิ้นส่วนมีสถานะเป็นอิสระต่อกัน (Self-contained) หาก Pod ชิ้นหนึ่งสูญหาย ชิ้นอื่นๆ ยังคงมีความหมายในตัวมันเอง โครงสร้างของ 1 Knowledge Pod ประกอบด้วย:

| Field Name | Type | Size | Description |
| :--- | :--- | :--- | :--- |
| `Universal_Logic_ID` | Hash | 16-bit | รหัสอ้างอิงชุดข้อมูล (Session ID) เพื่อให้ AI รู้ว่า Pods เหล่านี้มาจากคำศัพท์เดียวกัน |
| `Pod_Type` | Enum | 2-bit | ระบุชนิดของ Pod: `01` = Base Data, `10` = FEC Parity Pod |
| `Semantic_Hash` | Hex | 32-bit | ค่าแฮชที่สกัดจากรากศัพท์ความหมาย (Domain Mapping) |
| `Vector_Space` | Float Array| 64-bit | พิกัด [x, y] ของความหมายใน Universal Vector Space |
| `Cognitive_Weight` | Float | 16-bit | ค่าน้ำหนักความสำคัญของคำ (0.00 - 1.00) |

## 3. FEC Redundancy Engine (กลไกการชดเชยข้อมูลสำรอง)
การอัปเกรดที่สำคัญที่สุดใน Extended Edition คือการผนวก **FEC (Forward Error Correction)** เข้ามาใน Layer B
* **กลไกการทำงาน:** ก่อนที่ระบบจะส่งข้อมูลออกไป (Burst Firing) อัลกอริทึมจะคำนวณฐานข้อมูล (Base Burst Size) และสร้าง **Parity Pods (ข้อมูลสำรอง)** เพิ่มเข้าไปตามเปอร์เซ็นต์ที่ผู้ใช้ (HITL) กำหนด
* **ตัวอย่างเชิงปฏิบัติ:** หากผู้ใช้กำหนด Base Burst ที่ 50 Pods และปรับค่า FEC Redundancy ที่ 50% ระบบจำลองจะสร้าง Pod พิเศษเพิ่มขึ้นมา 25 Pods (รวมส่ง 75 Pods) 
* **ผลลัพธ์:** แม้ The Void จะทำลาย Pod ชุดหลัก (Base Data) ไปเกือบหมด แต่ถ้า Parity Pods ฝ่าฟันไปถึงปลายทางได้ AI จะสามารถนำข้อมูลสำรองนี้มาทำสมการย้อนกลับ (XOR/Matrix Reconstruction) เพื่อกู้คืนคำศัพท์ต้นฉบับได้อย่างแม่นยำ
# เอกสารเปรียบเทียบ (Diff): SNP-63820 vs SNP-65454

**หัวข้อ:** Hardline Hub Checkout — เปรียบเทียบ requirement 2 story
**บันทึกเมื่อ:** 2026-07-09
**ที่มา:** Jira bigc-dgt.atlassian.net

> **สรุปสั้น:** ทั้งสองเป็นเรื่องเดียวกันในเชิงโดเมน (Hardline Hub checkout) แต่เป็นคนละ requirement / คนละ scope / คนละ Epic — **ต่อยอดกัน ไม่ใช่ซ้ำกัน** SNP-65454 เป็นเวอร์ชันล่าสุดที่ควรยึดเป็นหลัก

---

## 1. ภาพรวม

| หัวข้อ | SNP-63820 | SNP-65454 |
| :--- | :--- | :--- |
| **ชื่อ** | [SPO][APP/WEB/M-WEB][REVAMP] เพิ่ม Hardline Hub Polygon | [SPO][Revamp][Web/M-Web/App][Hardline][Checkout] Customers can purchase Hardline products, stock ตัดจากสาขารับผิดชอบ |
| **ประเภท** | Story | Story |
| **Epic** | SNP-52506 (Servicegate Revamp 2026) | SNP-63876 (Hardline Hub — Change logic Remove "Is Non Hub") |
| **Priority** | Highest | High |
| **Component** | Order, Platform | Order |
| **Status** | To Do | To Do |
| **แก่นของงาน** | **เพิ่ม** Polygon Hardline เป็น fallback ของ Nextday เพื่อให้ลูกค้าต่างจังหวัดสั่ง Hardline ได้ | **เลิกใช้** is_non_hub + รวม logic BKK/UPC + นิยาม store/stock/price ต่อ shipping |
| **มุมเน้น** | availability & การแสดงผล (Home/Cart/Checkout แสดง option ไหน) | routing เบื้องหลัง (order ลงสาขาไหน, ตัด stock ที่ไหน, ราคาสาขาไหน) |

---

## 2. จุดที่เหมือนกัน (Overlap)

- ทั้งคู่เป็น flow ซื้อสินค้า **Hardline** ผ่าน Web / M-Web / App
- ใช้แนวคิด **Polygon Nextday + Polygon Hardline** เหมือนกัน
- **Priority การหาสาขา: เช็ค Nextday ก่อน แล้ว fallback** เป็นแนวเดียวกัน
- ตาราง "polygon coverage → shipping ที่แสดง" ใช้ **แกนเดียวกัน** (มี Hardline polygon → มี Bulky; ไม่มี Hardline → ไม่มี Bulky)
- Parcel ใช้ Config Store เหมือนกัน

---

## 3. จุดที่แตกต่าง (Diff)

| ประเด็น | SNP-63820 | SNP-65454 | หมายเหตุ |
| :--- | :--- | :--- | :--- |
| **เงื่อนไขสั่งซื้อได้** | อยู่ใน Nextday **หรือ** Hardline polygon | ต้องมี Hardline polygon **และ** (Nextday **หรือ Express หรือ Pickup**) | ⚠️ ต่างกัน — 65454 เพิ่ม Express/Pickup เข้าเงื่อนไข |
| **is_non_hub** | ไม่พูดถึง | **เลิกเช็ค is_non_hub** (แกนหลัก) | งานใหม่ของ 65454 |
| **BKK vs UPC** | ไม่แยกชัด | **รวม logic ให้เหมือนกัน แต่ใช้ API คนละตัว** | งานใหม่ของ 65454 |
| **Bulky fallback** | Nextday → Hardline Hub (Nearest) | Nextday → **Express → Pickup**, ตัด stock ที่ **DC** | 65454 ละเอียดขึ้น |
| **stock/price ต่อ shipping** | แนวคิด "Stock 3 Stores" (Nearest / Hardline / Parcel) | นิยามชัดต่อ shipping: ตัด stock สาขาไหน + ราคาสาขาไหน | 65454 ลงรายละเอียด |
| **override สาขาเอง** | ไม่มี | Express / Pickup ลูกค้าเลือกสาขาเองได้ | ฟีเจอร์ใหม่ 65454 |
| **Default / no address** | ไม่มี address → Default พระราม 4, รัศมี mapping 500 ม. | ไม่ระบุ (โฟกัส routing) | 63820 ครอบคลุมกว่า |
| **เคสตัวอย่างเด่น** | ฉะเชิงเทรา, อมก๋อย, สิชล นครศรีธรรมราช, no-address | ซอยบาร์โบส 2, ขอนแก่น, เคสวัดหล่อ (ลูกค้า B) | คนละชุดตัวอย่าง |

---

## 4. ตาราง Availability Matrix เทียบกัน

### SNP-63820 (4 combos)

| Polygon coverage | express | nextday | pickup | parcel | bulky |
| :--- | :---: | :---: | :---: | :---: | :---: |
| มี Nextday + มี Hardline | ✅ | ✅ | ✅ | ✅ | ✅ |
| มี Nextday + ไม่มี Hardline | ✅ | ✅ | ✅ | ✅ | — |
| ไม่มี Nextday + ไม่มี Hardline | ✅ | — | ✅ | ✅ | — |
| ไม่มี Nextday + มี Hardline | ✅ | — | ✅ | ✅ | ✅ |

### SNP-65454 (6 AC)

| AC | Polygon coverage | shipping ที่ได้ |
| :--- | :--- | :--- |
| AC1 | Nextday + Hardline | express, nextday, pickup, parcel, bulky |
| AC2 | Express + Hardline | express, pickup, parcel, bulky |
| AC3 | Pickup + Hardline | pickup, parcel, bulky |
| AC4 | Nextday, ไม่มี Hardline | express, nextday, pickup, parcel (ไม่มี bulky) |
| AC5 | Express, ไม่มี Hardline | express, pickup, parcel (ไม่มี bulky) |
| AC6 | Pickup, ไม่มี Hardline | pickup, parcel (ไม่มี bulky) |

> **ข้อสังเกต:** 65454 แตก matrix ละเอียดกว่า โดยแยกกรณีที่พื้นที่รองรับแค่ Express หรือแค่ Pickup (AC2/AC3/AC5/AC6) ซึ่ง 63820 รวมไว้กว้างๆ

---

## 5. สรุป & ข้อควรระวัง

- **SNP-63820** = งาน "รากฐาน" → เปิดให้ลูกค้าต่างจังหวัดในเขต Polygon Hardline สั่ง Hardline ได้ + ควบคุมว่าแต่ละพื้นที่โชว์ shipping อะไร (เน้น availability/display)
- **SNP-65454** = งาน "รอบถัดไป" → ถอด is_non_hub, รวม BKK/UPC, ลงรายละเอียด order ลงสาขาไหน / ตัด stock ที่ไหน / ราคาไหน ต่อ shipping (เน้น routing/stock/price)

⚠️ **ความไม่ตรงกันที่ต้องเคลียร์:**
1. **เงื่อนไขสั่งซื้อได้ไม่ตรงกัน** — 63820: Nextday *หรือ* Hardline · 65454: Hardline *และ* (Nextday/Express/Pickup) → ควรยึด **65454 เป็นเวอร์ชันล่าสุด** และตรวจว่า logic เก่าจาก 63820 ไม่ขัดกัน
2. **Bulky fallback** — 63820 ไป Hardline Hub Nearest, 65454 ไป Express→Pickup + ตัด stock DC → ตรวจว่า implementation ตรงตาม 65454
3. เวลาทำ dev/test ให้อ้าง **SNP-65454** เป็นหลัก และใช้ 63820 เป็น context ของ availability/edge case (default พระราม 4, รัศมี 500 ม., เคสสิชล) ที่ 65454 ไม่ได้พูดถึง

## 6. ไฟล์ที่เกี่ยวข้องในโปรเจค

- `docs/SNP-65454-requirement.md` — requirement เต็มของ SNP-65454
- `test-cases/SNP-65454-hardline-checkout.md` — 40 test cases (5-layer pyramid)
- `test-cases/SNP-63820.md` — test cases ของ SNP-63820 (15 เคส)

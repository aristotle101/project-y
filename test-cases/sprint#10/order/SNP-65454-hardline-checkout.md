# Test Cases — SNP-65454

**เรื่อง:** [SPO][Revamp][Web/M-Web/App][Hardline][Checkout] Customers can purchase Hardline products, stock deducted from responsible store
**Epic:** SNP-63876 · **Sprint:** ORD Y2026 Sprint 10
**สร้างเมื่อ:** 2026-07-09

## สรุป Requirement

- **การเปลี่ยนแปลงหลัก:** เลิกใช้ flag `Is Non-Hub` — Code ต้องไม่เช็ค `is_non_hub` (Opal ยังคง toggle "Hide is-non-hub" ไว้ แต่ไม่มีผล)
- **รวม logic BKK/Metropolitan และ UPC ให้เหมือนกัน** แต่ใช้ API คนละตัว
- **เงื่อนไขการขายได้:** ที่อยู่ต้องครอบคลุม Polygon Hardline **และ** (Polygon Nextday หรือ Express หรือ Pickup) และต้องมี stock ที่สาขา/DC

### Logic หาสาขา + ตัด stock/ราคา ตามประเภทจัดส่ง

| Shipping | Logic หาสาขา | ตัด stock | ราคาแสดง |
| :--- | :--- | :--- | :--- |
| express | Nearest; ลูกค้าเลือกเองได้ (override) | สาขา Express | สาขา Express |
| nextday | สาขา Polygon Nextday | สาขา Nextday | สาขา Nextday |
| pickup | Nearest; ลูกค้าเลือกเองได้ (override) | สาขา Pickup | สาขา Pickup |
| parcel | สาขาตาม Config (บางนา — สาขาเดียวที่ส่ง parcel ได้) | สาขา Config | สาขา Config |
| bulky | Priority 1 Nextday → 2 Express → 3 Pickup | **DC** | สาขาที่สร้าง order |

### Acceptance Criteria (ตาม polygon ที่ที่อยู่ครอบคลุม)

- **AC1** Nextday + Hardline
- **AC2** Express + Hardline
- **AC3** Pickup + Hardline
- **AC4** Nextday, ไม่มี Hardline (ไม่มี Bulky)
- **AC5** Express, ไม่มี Hardline (ไม่มี Nextday/Bulky)
- **AC6** Pickup, ไม่มี Hardline (เหลือ Pickup + Parcel)

---

## Test Cases

| Test ID | ชื่อเทสต์ | เงื่อนไขก่อนทดสอบ | ขั้นตอนทดสอบ | ผลลัพธ์ที่คาดหวัง | ความสำคัญ | ชั้นพีระมิด |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TC-F-001 | เลิกเช็ค `is_non_hub` ในการคำนวณ availability | สาขามี `is_non_hub=true` + มี stock Hardline + ที่อยู่เข้าเงื่อนไข polygon | 1. เรียก logic คำนวณ availability | สาขาถูกนับว่าขายได้ตามปกติ ไม่มี logic ใดอ่านค่า `is_non_hub` | สูง | Unit |
| TC-F-002 | ส่ง `is_non_hub=true` มาใน API ก็ยังสั่งซื้อได้ | payload/ข้อมูลสาขามี `is_non_hub=true` | 1. เรียก API availability | ตอบ 200 สั่งซื้อได้ ค่า flag ไม่มีผลต่อผลลัพธ์ | สูง | Component (BE) |
| TC-E-001 | toggle "Hide is-non-hub" ใน Opal ไม่มีผลต่อระบบ | เข้า Opal ได้ มี toggle | 1. สลับ toggle เปิด/ปิด 2. รัน availability ใหม่ | พฤติกรรมเหมือนกันทั้งสองสถานะ toggle เป็นแค่ UI | กลาง | Exploratory |
| TC-F-003 | Express → เลือกสาขาที่ใกล้ที่สุด (Nearest) | ที่อยู่ครอบคลุม Express มีหลายสาขามี stock | 1. หาสาขารับผิดชอบแบบ express | ได้สาขา Nearest | สูง | Unit |
| TC-F-004 | Express → ลูกค้าเลือกสาขาเอง (override Nearest) | ลูกค้าเลือกสาขาที่ไม่ใช่ Nearest | 1. หาสาขา express พร้อม `selectedStore` | ได้สาขาตามที่ลูกค้าเลือก | สูง | Unit |
| TC-F-005 | Nextday → เลือกสาขาเจ้าของ Polygon Nextday | ที่อยู่อยู่ในโซน Polygon Nextday | 1. หาสาขารับผิดชอบแบบ nextday | ได้สาขาเจ้าของ Polygon Nextday | สูง | Unit |
| TC-F-006 | Pickup → ค่าเริ่มต้นเป็นสาขา Nearest | ที่อยู่ครอบคลุม Pickup ไม่เลือกสาขาเอง | 1. หาสาขา pickup | ได้สาขา Nearest | สูง | Unit |
| TC-F-007 | Pickup → ลูกค้าเลือกสาขาเอง (override) | ลูกค้าเลือก pickup + สาขาที่ระบุ (เช่น พระราม 4) | 1. หาสาขา pickup พร้อม `selectedStore` | ได้สาขาตามที่ลูกค้าเลือก | สูง | Unit |
| TC-F-008 | Parcel → เลือกสาขาตาม config (บางนา) | config parcel = บางนา | 1. หาสาขา parcel | ได้สาขาบางนาตาม config | กลาง | Unit |
| TC-E-002 | Parcel → ที่อยู่จังหวัดใดก็ได้บางนาเสมอ | บางนาเป็นสาขาเดียวที่จัดส่ง parcel ได้ | 1. หาสาขา parcel ด้วยที่อยู่หลายจังหวัด (กทม./ขอนแก่น/เลย) | คืนบางนาทุกครั้ง ที่อยู่ไม่กระทบผลลัพธ์ | สูง | Unit |
| TC-F-009 | Bulky → Priority 1 = Nextday | ที่อยู่ครอบคลุม Nextday+Express+Pickup+Hardline | 1. หาสาขา bulky | ได้สาขา Nextday (Priority 1) | สูง | Unit |
| TC-E-003 | Bulky → fallback เป็น Express เมื่อไม่มี Nextday | ที่อยู่ครอบคลุม Express+Pickup+Hardline (ไม่มี Nextday) | 1. หาสาขา bulky | ได้สาขา Express (Priority 2) | สูง | Unit |
| TC-E-004 | Bulky → fallback เป็น Pickup เมื่อไม่มี Nextday/Express | ที่อยู่ครอบคลุม Pickup+Hardline เท่านั้น | 1. หาสาขา bulky | ได้สาขา Pickup (Priority 3) | สูง | Unit |
| TC-F-010 | ต้องอยู่ใน Polygon Hardline ถึงจะซื้อ Hardline ได้ | ที่อยู่ครอบคลุม Nextday แต่ไม่มี Polygon Hardline | 1. ตรวจสถานะการสั่งซื้อ SKU Hardline | สั่งซื้อได้ผ่าน shipping อื่น แต่ไม่มี Bulky (ตาม AC4) | สูง | Unit |
| TC-E-005 | Bulky แสดงเฉพาะเมื่อที่อยู่มี Polygon Hardline | ที่อยู่แบบ AC4/AC5/AC6 (ไม่มี Hardline polygon) | 1. สร้างรายการตัวเลือก shipping | ไม่มี Bulky ในตัวเลือก | สูง | Unit |
| TC-ERR-001 | ไม่มี polygon ใดเข้าเงื่อนไข → สั่งซื้อไม่ได้ | ที่อยู่ไม่ครอบคลุมทั้ง Nextday/Express/Pickup | 1. ตรวจสถานะการสั่งซื้อ | SKU สั่งซื้อไม่ได้ ไม่มีตัวเลือก shipping | สูง | Unit |
| TC-E-006 | [ขอบเขต] จุดในเส้น Hardline แต่นอกเส้น Nextday | พิกัดอยู่ใน Polygon Hardline แต่นอก Polygon Nextday พอดี | 1. คำนวณ availability ด้วยพิกัดนี้ | Nextday หายไป แต่ Hardline ยังซื้อได้ ตัวเลือก = Pickup + Bulky (+ Parcel) | สูง | Unit |
| TC-E-007 | [เคสวัดหล่อ] ลูกค้า B: order ลงสาขา C ตัด stock ที่ DC | ที่อยู่ครอบคลุมแค่ Pickup + Polygon Hardline; สาขา C = Nearest; DC มี stock | 1. Checkout แบบ Bulky 2. Checkout แบบ Pickup | Bulky: order ลง C ราคา C ตัด stock ที่ DC · Pickup: order+stock ที่ C · Nextday/Express เลือกไม่ได้ | สูง | Integration |
| TC-ERR-002 | [ขอบเขต] หลุดนอกเส้น Hardline นิดเดียว → Bulky หาย | พิกัดอยู่นอก Polygon Hardline เล็กน้อย เหลือแค่ Pickup (=AC6) | 1. คำนวณ availability ด้วยพิกัดนี้ | ไม่มี Bulky เหลือแค่ Pickup + Parcel (เส้นขอบตัดสินถูกฝั่ง) | สูง | Integration |
| TC-F-011 | [API-BKK] availability คืนตัวเลือกครบ (AC1) | ที่อยู่ซอยบาร์โบส 2; เอกมัย/สำโรง2/บางนา + DC มี stock | 1. เรียก API availability ของ BKK | 200; ตัวเลือก = express/nextday/pickup/parcel/bulky; โครงสร้างตรง contract | สูง | Component (BE) |
| TC-F-012 | [API-UPC] availability (คนละ API) คืนตัวเลือกครบ (AC1) | ที่อยู่ขอนแก่น 2; ขอนแก่น1/ขอนแก่น2 มี stock | 1. เรียก API availability ของ UPC | 200; มี bulky; response ใช้ contract ของ UPC | สูง | Component (BE) |
| TC-F-013 | [API] details-by-shipping v2 คืนราคาตามสาขาของแต่ละ shipping | SKU มี stock กระจายหลายสาขา | 1. เรียก details-by-shipping v2 | แต่ละ shipping คืนสาขาที่ตัด stock + ราคาของสาขานั้น | สูง | Component (BE) |
| TC-ERR-003 | [API] ไม่มี stock ทุกที่ → คืนสถานะไม่พร้อมขาย | SKU Hardline stock = 0 ทุกสาขาและ DC | 1. เรียก API availability | คืนสถานะว่าง/ไม่พร้อมขาย (ไม่มี option) — ไม่ใช่ error 500 | สูง | Component (BE) |
| TC-ERR-004 | [API] เลือกสาขา pickup ไม่ถูกต้อง → ปฏิเสธ | ส่ง pickup พร้อมสาขาที่ไม่มี SKU/ไม่ถูกต้อง | 1. เรียก API ด้วย `selectedStore` ที่ผิด | คืน error 4xx พร้อมเหตุผลชัดเจน ไม่มีการสร้าง order | กลาง | Component (BE) |
| TC-F-014 | [UI] แสดงตัวเลือก shipping เฉพาะที่รองรับตามที่อยู่ | mock availability ตาม AC1 เทียบ AC6 | 1. render widget ด้วย payload แต่ละ AC | AC1 แสดงครบ 5 แบบ · AC6 แสดงแค่ pickup+parcel | กลาง | Component (FE) |
| TC-E-008 | [UI] ซ่อนตัวเลือก Bulky เมื่อไม่เข้าเงื่อนไข | payload availability ไม่มี bulky (AC4) | 1. render widget | ไม่แสดงปุ่ม Bulky | กลาง | Component (FE) |
| TC-ERR-005 | [UI] SKU สั่งซื้อไม่ได้ → แสดงสถานะ + ปิดปุ่ม checkout | payload availability = ไม่พร้อมขาย | 1. render widget | แสดง "ส่งไปที่อยู่นี้ไม่ได้" + ปิดปุ่ม Add-to-cart/Checkout | กลาง | Component (FE) |
| TC-F-015 | [BKK·AC1] Nextday → สำโรง 2 (order/stock/ราคา) | ซอยบาร์โบส 2; สำโรง 2 = Polygon Nextday มี stock | 1. Checkout แบบ Nextday | order ลงสำโรง 2 · ตัด stock สำโรง 2 · ราคาสำโรง 2 | สูง | Integration |
| TC-F-016 | [BKK·AC1] Express → เอกมัย (Nearest) | ซอยบาร์โบส 2; เอกมัย Nearest มี stock | 1. Checkout แบบ Express (ไม่เลือกสาขา) | order+stock+ราคา ที่เอกมัยทั้งหมด | สูง | Integration |
| TC-F-017 | [BKK·AC1] Pickup เลือกสาขาเอง → พระราม 4 | ลูกค้าเลือก Pickup + พระราม 4 | 1. Checkout แบบ Pickup เลือกพระราม 4 | order+stock ที่พระราม 4 (ไม่ใช่เอกมัยที่ Nearest) | สูง | Integration |
| TC-F-018 | [BKK·AC1] Bulky → สำโรง 2, ราคาสำโรง 2, ตัด stock DC วังน้อย | DC วังน้อยมี stock | 1. Checkout แบบ Bulky | order ลงสำโรง 2 · ราคาสำโรง 2 · **ตัด stock ที่ DC วังน้อย** | สูง | Integration |
| TC-F-019 | [Parcel] route ไปบางนาเสมอ ไม่ว่าลูกค้าอยู่จังหวัดใด | ลูกค้าต่างจังหวัด (เช่น ขอนแก่น) | 1. Checkout Hardline แบบ Parcel | order+stock+ราคา ที่บางนา (สาขาเดียวที่ส่ง parcel ได้) | สูง | Integration |
| TC-F-020 | [UPC·AC1] Bulky → ขอนแก่น 1, ตัด stock DC 91301 | ขอนแก่น 2; ขอนแก่น 1 อยู่ใน Polygon Hardline | 1. Checkout แบบ Bulky (UPC API) | order ลงขอนแก่น 1 · ราคาขอนแก่น 1 · ตัด stock DC 91301 (ตาม mapping ใหม่ Data Sci) | สูง | Integration |
| TC-E-009 | [AC4] ไม่มี Polygon Hardline → ไม่มี Bulky แต่แบบอื่นได้ | ที่อยู่นครพนม (Nextday ไม่มี Hardline) | 1. ลองทุก shipping | Nextday/Express/Pickup/Parcel สำเร็จ · ไม่มี Bulky | สูง | Integration |
| TC-E-010 | [AC6] ที่อยู่ Pickup อย่างเดียว → เหลือ Pickup + Parcel | ที่อยู่ยโสธร (Pickup ไม่มี Hardline) | 1. list + ลอง shipping | มีแค่ Pickup (ยโสธร) + Parcel (บางนา) · ไม่มี Nextday/Express/Bulky | สูง | Integration |
| TC-ERR-006 | สาขาที่เลือกหมด stock ตอน checkout | สาขา express Nearest stock เหลือ 0 ระหว่าง browse→checkout | 1. สั่งซื้อแบบ express | order ถูกปฏิเสธ/re-route ตาม logic · stock ไม่ติดลบ · error ชัดเจน | สูง | Integration |
| TC-ERR-007 | สั่งซื้อพร้อมกันชิ้นสุดท้าย (race condition) | ลูกค้า 2 คน stock 1 ชิ้นที่สาขาเดียวกัน | 1. ส่ง checkout พร้อมกันทั้งคู่ | สำเร็จเพียง 1 ราย · อีกรายได้ out-of-stock · stock ไม่ติดลบ | สูง | Integration |
| TC-ERR-008 | บางนาหมด stock → parcel ใช้ไม่ได้ทั้งประเทศ | บางนา stock = 0 สำหรับ SKU นั้น | 1. เรียก availability / ลอง checkout แบบ parcel | ไม่มีตัวเลือก parcel (หรือถูกปฏิเสธ) · shipping อื่นยังใช้ได้ปกติ | สูง | Integration |
| TC-F-021 | [E2E] Checkout Hardline Bulky ครบวงจร (แพลตฟอร์มเดียว) | ระบบจริง; ซอยบาร์โบส 2; DC มี stock | 1. Home → PDP → Cart → เลือก Bulky → Checkout → ยืนยัน | ยืนยัน order สำเร็จ · สาขา = สำโรง 2 · ราคาสำโรง 2 · stock DC ลดลง | สูง | E2E |
| TC-E-011 | [Exploratory] journey เหมือนกันข้าม Web / M-Web / App | build ครบ 3 device; journey เดียวกัน | 1. ไล่ flow เดียวกันบน M-Web และ App เทียบ Web 2. ดู UI/ตัวเลือก shipping | flow + ผลลัพธ์เหมือนกันทุก device · ต่างได้แค่ layout ตามอุปกรณ์ · ไม่มี step หาย | กลาง | Exploratory |

---

## สรุปพีระมิด (รวม 40 เคส)

| ชั้น | จำนวน |
| :--- | :--- |
| Unit | 15 |
| Component (BE) | 6 |
| Component (FE) | 3 |
| Integration | 13 |
| E2E | 1 |
| Exploratory | 2 |
| **รวม** | **40** |

- **E2E = 1/40 (~2.5%)** — ต่ำกว่าเพดาน ~10% ✅
- **Unit + Component = 24/40 (60%)** — ฐานแน่น ไม่เป็นทรงกรวยไอศกรีม ✅
- **เคส negative/edge (TC-E + TC-ERR) = 19/40 (~48%)** — ผ่านเกณฑ์ ≥40% ✅

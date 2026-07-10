# Test Case: SNP-63820 — Hardline Hub Polygon Fallback

### 🟢 Functional (Happy Path & Core Logic)

**TC-F-001 | พื้นที่ครอบคลุมทั้ง Nextday + Hardline Polygon** — *Priority: High · Layer: Integration*
- **Precondition:** ที่อยู่ลูกค้าอยู่ในทั้ง Nextday polygon และ Hardline polygon
- **ขั้นตอน:**
  1. ตั้งที่อยู่ลูกค้าเป็นที่อยู่ที่ครอบคลุมทั้งสอง polygon
  2. เพิ่มสินค้า Hardline (เครื่องใช้ไฟฟ้า) ลงตะกร้า
  3. ไปหน้า Checkout
- **Expected:** หน้า Checkout แสดงตัวเลือกจัดส่งครบ 5 แบบ — Express, Nextday, Pickup, Parcel, Bulky

**TC-F-002 | มี Nextday Polygon แต่ไม่มี Hardline Polygon** — *Priority: High · Layer: Integration*
- **Precondition:** ที่อยู่อยู่ใน Nextday polygon แต่ไม่อยู่ใน Hardline polygon
- **ขั้นตอน:**
  1. ตั้งที่อยู่เป็นที่อยู่ที่มีแค่ Nextday coverage
  2. เพิ่มสินค้า Hardline ลงตะกร้า
  3. ไปหน้า Checkout
- **Expected:** Checkout แสดง Express, Nextday, Pickup, Parcel **ไม่แสดง Bulky**

**TC-F-003 | หน้า Home เลือกสาขา Hardline Pickup เป็น Default เมื่อไม่มี Nextday** — *Priority: Medium · Layer: Integration*
- **Precondition:** ที่อยู่ไม่อยู่ใน Nextday polygon แต่อยู่ใน Hardline polygon
- **ขั้นตอน:**
  1. ตั้งที่อยู่เป็นที่อยู่ที่มีแค่ Hardline coverage
  2. เปิดหน้า Home
- **Expected:** หน้า Home แสดงสาขา Hardline (Pickup) ที่ใกล้ที่สุดเป็นสาขาหลัก (เช่น "ศรีมหาโพธิ์")

**TC-F-004 | หน้า Cart แสดง Stock จาก Hardline(Pickup)/Parcel เท่านั้น** — *Priority: Medium · Layer: Integration*
- **Precondition:** ที่อยู่ไม่อยู่ใน Nextday polygon แต่อยู่ใน Hardline polygon, มีสินค้าในตะกร้า
- **ขั้นตอน:**
  1. ตั้งที่อยู่เป็นที่อยู่ที่มีแค่ Hardline coverage
  2. เพิ่มสินค้าลงตะกร้า
  3. เปิดหน้า Cart
- **Expected:** Cart แสดง stock เฉพาะจากสาขา Hardline (Pickup) และ Parcel (config store) เท่านั้น ต้องไม่มี stock ของ Nextday ปนมา

**TC-F-005 | ระบบเช็ค Priority Nextday ก่อนเมื่อตรงทั้งสอง Polygon** — *Priority: High · Layer: Unit*
- **Precondition:** ที่อยู่ตรงทั้ง Nextday polygon และ Hardline polygon
- **ขั้นตอน:**
  1. ทำให้ระบบคำนวณ store/warehouse resolution (โหลดหน้า Home หรือ Add to Cart)
- **Expected:** ระบบต้องเลือก Nextday store เป็นแหล่งหลักก่อน แล้วจึงค่อยพิจารณา Hardline

**TC-F-006 | ระบบ Fallback ไปที่ Hardline เมื่อไม่ตรง Nextday** — *Priority: High · Layer: Unit*
- **Precondition:** ที่อยู่ไม่ตรง Nextday polygon แต่ตรง Hardline polygon
- **ขั้นตอน:**
  1. ทำให้ระบบคำนวณ store/warehouse resolution
- **Expected:** ระบบข้าม Nextday ไปเลย แล้วเลือก Hardline Hub ที่ใกล้ที่สุดเป็นตัวสำรอง

**TC-F-007 | [Regression] เคสสิชล นครศรีธรรมราช — Home/Cart แสดงสินค้า Nextday + Hardline ของสุราษฎร์ธานี** — *Priority: High · Layer: E2E*
- **Precondition:** ที่อยู่ = อ.สิชล จ.นครศรีธรรมราช (Lat 8.973887, Long 99.783398) — ไม่มี Nextday polygon ในพื้นที่ แต่ครอบคลุมด้วย Hardline polygon ของสุราษฎร์ธานี
- **ขั้นตอน:**
  1. ตั้งที่อยู่ตามพิกัดสิชล
  2. เปิดหน้า Home
  3. เปิดหน้า Cart
- **Expected:** Home/Cart แสดงสินค้า Nextday ของบิ๊กซีสุราษฎร์ธานี พร้อมสินค้า Hardline ด้วย

---

### 🟡 Boundary (Edge Cases)

**TC-E-001 | ที่อยู่อยู่ที่ระยะพอดี 500 เมตร ถือว่าตรงกับที่อยู่ที่บันทึกไว้** — *Priority: Medium · Layer: Unit*
- **Precondition:** ลูกค้ามีที่อยู่บันทึกไว้; จุดทดสอบอยู่ห่างพอดี 500 เมตร
- **ขั้นตอน:**
  1. ตั้ง location ปัจจุบันให้ห่างจากที่อยู่ที่บันทึกไว้พอดี 500 เมตร
  2. โหลดหน้า Home/Checkout
- **Expected:** ระบบจับคู่กับที่อยู่ที่บันทึกไว้ (ไม่ fallback ไปที่ Default พระราม 4)

**TC-E-002 | ไม่มีที่อยู่บันทึกไว้เลย → Default พระราม 4 + บังคับ Pickup อย่างเดียว** — *Priority: Medium · Layer: Integration*
- **Precondition:** บัญชีลูกค้าไม่มีที่อยู่บันทึกไว้เลย
- **ขั้นตอน:**
  1. Login ด้วยบัญชีที่ไม่มีที่อยู่
  2. เปิดหน้า Home
  3. ลอง Add to Cart
- **Expected:** ระบบ default location เป็นพระราม 4; Add to Cart บังคับ Delivery Type = Pickup เท่านั้น เลือกอย่างอื่นไม่ได้

---

### 🔴 Validation / Negative (Error & Unauthorized Cases)

**TC-ERR-001 | บล็อกการเลือก Nextday เมื่อที่อยู่มีแค่ Hardline coverage** — *Priority: High · Layer: Component (FE)*
- **Precondition:** ที่อยู่มีแค่ Hardline polygon coverage (ไม่มี Nextday)
- **ขั้นตอน:**
  1. ตั้งที่อยู่เป็นที่อยู่ที่มีแค่ Hardline coverage
  2. เพิ่มสินค้า Hardline ลงตะกร้า
  3. ที่หน้า Checkout ลองเลือก "Nextday"
- **Expected:** ระบบบล็อกการเลือก; ตัวเลือก Nextday ต้อง disable หรือเลือกไม่ได้

**TC-ERR-002 | ไม่มี Polygon coverage เลย → Checkout เหลือแค่ Pickup/Parcel** — *Priority: High · Layer: Integration*
- **Precondition:** ที่อยู่อยู่นอกทั้ง Nextday และ Hardline polygon (เช่น อ.อมก๋อย เชียงใหม่)
- **ขั้นตอน:**
  1. ตั้งที่อยู่เป็นพื้นที่ที่ไม่มี coverage
  2. เพิ่มสินค้าลงตะกร้า
  3. ไปหน้า Checkout
- **Expected:** มีแค่ Pickup และ Parcel เท่านั้น; Express/Nextday/Bulky เลือกไม่ได้

**TC-ERR-003 | แสดงข้อความ Error ที่ถูกต้องเมื่อไม่พบ Polygon ใดเลย** — *Priority: High · Layer: Component (FE)*
- **Precondition:** ที่อยู่/พิกัด resolve ไม่พบสาขาใดเลยในทั้งสอง polygon
- **ขั้นตอน:**
  1. ตั้งที่อยู่เป็นพิกัดที่ไม่มีสาขาใด resolve ได้
  2. ลอง Add to Cart / ไป Checkout
- **Expected:** แสดงข้อความตรงตามนี้: "พื้นที่นี้ไม่รองรับการจัดส่ง ลองเลือกที่อยู่ใหม่หรือรับที่สาขาแทน"

**TC-ERR-004 | สินค้า Nextday ที่โชว์ใน Cart ต้องถูกบล็อกที่ Checkout สำหรับที่อยู่ mismatch region** — *Priority: High · Layer: Integration*
- **Precondition:** ที่อยู่แบบเคสสิชล — Cart แสดงสินค้า Nextday จากสาขาที่ไม่ใช่พื้นที่ตัวเอง (อ้างอิง TC-F-007)
- **ขั้นตอน:**
  1. ตั้งที่อยู่ตามพิกัดสิชล
  2. เพิ่มสินค้า Nextday (จากสุราษฎร์ธานี) ลงตะกร้า
  3. ไปหน้า Checkout
- **Expected:** Checkout ต้องบล็อก/ซ่อน Nextday สำหรับสินค้านั้น; เหลือแค่ Hardline (Bulky จัดส่งจาก Hardline Hub สุราษฎร์ธานี) ที่ดำเนินการต่อได้

**TC-ERR-005 | ที่อยู่เกินระยะ 500 เมตรเล็กน้อย ต้องไม่จับคู่กับสาขาผิด** — *Priority: Medium · Layer: Unit*
- **Precondition:** จุดทดสอบห่างจากที่อยู่ที่บันทึกไว้ 501 เมตร (เกินขอบเขตเล็กน้อย)
- **ขั้นตอน:**
  1. ตั้ง location ปัจจุบันห่างจากที่อยู่ที่บันทึกไว้ 501 เมตร
  2. โหลดหน้า Home/Checkout
- **Expected:** ระบบต้อง**ไม่**จับคู่กับที่อยู่ที่บันทึกไว้ และ fallback ไป Default พระราม 4 แทนการจับคู่สาขาใกล้เคียงผิดพลาด

**TC-ERR-006 | บล็อก Add to Cart จนกว่าจะเลือก Delivery Type ในพื้นที่ไม่มี coverage** — *Priority: Medium · Layer: Component (FE)*
- **Precondition:** ที่อยู่อยู่นอกทั้งสอง polygon (เช่น อ.อมก๋อย), ลูกค้ายังไม่ได้เลือก Delivery Type
- **ขั้นตอน:**
  1. ตั้งที่อยู่เป็นพื้นที่ที่ไม่มี coverage
  2. กด "Add to Cart" โดยไม่เลือก Delivery Type
- **Expected:** ระบบบล็อก action และบังคับให้เลือก Delivery Type (tooltip สีแดง); สุดท้ายเลือกได้แค่ Pickup ถ้าไม่เปลี่ยนที่อยู่

---

**สรุปความครอบคลุม:** Functional 7 เคส, Boundary 2 เคส, Negative 6 เคส (Negative คิดเป็น 40% ของทั้งหมด)

ครอบคลุมครบทั้ง 4 ช่องของ polygon matrix, ลำดับ fallback (Nextday ก่อน แล้วค่อย Hardline), ช่องว่างระหว่าง "สิ่งที่แสดงบนหน้าจอ" กับ "สิ่งที่ระบบบังคับใช้จริง" (บั๊กเคสสิชล), เกณฑ์ระยะ 500 เมตร และข้อความ error ภาษาไทย

**สรุปตาม Testing Pyramid:** Unit 4 · Component (BE) 0 · Component (FE) 3 · Integration 7 · E2E 1 · Exploratory 0 (รวม 15 เคส)

- E2E = 1/15 (≈7%) อยู่ในเกณฑ์ ≤10% ✅
- ⚠️ ฐานพีระมิดยังบางกว่าที่ควร — Unit/Component รวมกัน 7 เคส ขณะที่ Integration หนักถึง 7 เคส ซึ่งเป็นธรรมชาติของฟีเจอร์ geo/coverage ที่ข้ามหลาย service แต่ถ้ามี endpoint เดียวที่ resolve delivery options จากพิกัดได้ (เช่น `GET /delivery-options?lat=..&long=..`) ควรดึงเคสกลุ่ม TC-F-001/002/003 ลงมาเป็น Component (BE) เพื่อลดต้นทุน/ความเปราะของชั้น Integration

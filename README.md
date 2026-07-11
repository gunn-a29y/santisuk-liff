# santisuk-liff — LIFF UI (static · GitHub Pages)

หน้าตา (UI) ของบอทรับบิลร้านยาสันติสุข โฮสต์เป็น static บน GitHub Pages
แยกออกจาก backend (`santisuk-rx`, private) เพื่อ **ฆ่าแบนเนอร์ Google Apps Script + แก้จอขาว iOS** (TQ-143).

> repo นี้ **public โดยตั้งใจ** — มีแค่ HTML/JS หน้าตา **ไม่มีความลับ** (ไม่มี key / ไม่มีข้อมูลบิล).
> ความลับทั้งหมดอยู่ Script Properties ฝั่ง `santisuk-rx` · ข้อมูลบิลกันด้วย LINE ID token ที่ endpoint.

## ไฟล์
| ไฟล์ | หน้าที่ |
|---|---|
| `index.html` | หน้าแรก (home) 2 แท็บ: จัดการร้าน (ถ่าย/เลือกรูปบิล) + บิลทั้งหมด · เป็น LIFF endpoint |
| `edit.html` | หน้าตรวจ/แก้บิล (flow B) |
| `.nojekyll` | บอก Pages ไม่ต้องรัน Jekyll (เสิร์ฟไฟล์ดิบ) |

## สถาปัตยกรรม (หลังย้าย · TQ-143 ทาง A)
```
LINE (richmenu / Flex card) → liff.line.me/<LIFF_ID> → endpoint = Pages index.html
  index.html: liff.init() → getIDToken() → api('listMonths', …)
  ทุก api() = fetch(EXEC_URL, text/plain, redirect:follow, body={action, idToken, …})
       ↓
  santisuk-rx Code.gs doPost → handleApi_ → verifyIdToken_ (LINE verify + allow-list) → api* → JSON
```
- **auth ทาง A:** ทุก call แนบ LINE ID token → backend verify กับ LINE เอง + เช็ค userId ว่าเป็นสตาฟที่อนุมัติ (fail-closed)
- **navigation ภายใน = same-origin จริง** (`index.html` ↔ `edit.html`) → ไม่มี nested-iframe → **จอขาว finding-3 หาย** + **ไม่มีแบนเนอร์**
- **CORS:** Apps Script /exec ตอบ 302 → googleusercontent (ACAO:*) → `text/plain` เลี่ยง preflight, `redirect:'follow'` ตาม 302 (verified · vault tooling-facts)

## config ที่ฝังในหน้า (ไม่ลับ)
- `LIFF_ID = 2010669250-NflL6kvi`
- `EXEC_URL = https://script.google.com/macros/s/AKfycbz…TXYSz0Z1/exec` (deployment เดิม · ถ้า re-deploy ใหม่ที่ /exec เดิม = ไม่ต้องแก้)
- version marker บนหน้า = `v1.0.0` (bump patch เวลาแก้บั๊ก / minor เวลาเพิ่มฟีเจอร์)

## deploy (ครั้งแรก — checklist)
1. สร้าง repo **public** `gunn-a29y/santisuk-liff` → push ไฟล์ทั้งหมด
2. GitHub → Settings → Pages → Source = `main` / root → รอ URL `https://gunn-a29y.github.io/santisuk-liff/`
3. **santisuk-rx backend:** เพิ่ม Script Properties แล้ว `clasp push --force` + `clasp update-deployment <id>` (ดู `santisuk-rx/PLAN.md`)
   - ถ้ายังไม่มี: `LOGIN_CHANNEL_ID` (optional · ปกติ derive จาก LIFF_ID prefix ได้) · `STAFF_IDS` (ว่างได้ · seed จาก OWNER_USER_ID)
4. **LINE Developers → LIFF app `2010669250-NflL6kvi`:**
   - **⚠️ Scope ต้องมี `openid`** (ไม่งั้น `getIDToken()` = null → auth พังทั้งแอป) · เช็ค/เปิดที่ LIFF settings → Scopes
   - **Endpoint URL = `https://gunn-a29y.github.io/santisuk-liff/`**
   - richmenu / Flex ยังชี้ `liff.line.me/<id>` เหมือนเดิม (LIFF redirect ไป endpoint ใหม่เอง)
5. **seed สตาฟ:** พิมพ์ `id` ใส่บอท "สันติสุข หลังร้าน" 1 ครั้ง (ตั้ง OWNER_USER_ID) → คนนั้นผ่าน auth ทันที
6. เทสบนมือถือผ่าน LINE: เปิดริชเมนู → ไม่มีแบนเนอร์ · บิลโหลดขึ้น (= auth ผ่าน) · ถ่าย→ตรวจ→เซฟ ครบ

## rollback (ถ้า auth/liff.init บน Pages ไม่ทำงาน)
เปลี่ยน LIFF Endpoint URL กลับเป็น `EXEC_URL` (Apps Script) — UI เก่ายังเสิร์ฟจาก doGet อยู่ (ยังไม่ลบ) = กลับไปสภาพเดิมทันที.

## update
แก้ไฟล์ → commit → push `main` → Pages รีเฟรชเอง (~1 นาที) · ไม่ต้องแตะ backend ถ้าไม่ได้แก้ API
> spec เต็ม = vault `system-gpd` การ์ด **TQ-143** (parent TQ-79)

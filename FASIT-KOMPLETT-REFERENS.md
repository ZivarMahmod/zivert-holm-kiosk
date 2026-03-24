# FASIT: Zivert Holms hörna — Komplett teknisk referens

> Denna fil beskriver exakt hur hela kiosk-applikationen är uppbyggd.
> Syftet är att en ADMIN-sida ska kunna replika alla inställningar och
> stå i standby tills kiosk-repot kopplas upp.

---

## 1. ARKITEKTUR — EN FIL, TVÅ LÄGEN

Hela applikationen finns i **en enda HTML-fil** (`index.html`, ~6 600 rader).
All CSS och JavaScript är inline — inga separata filer.

### Två lägen:

| Läge | Aktiveras av | Storlek |
|------|-------------|---------|
| **Desktop-kiosk** | Standard (ingen parameter) | 1920×1080px, skalas till skärm |
| **Mobil-kiosk** | `?mobile` i URL eller `window.innerWidth < 768` | Responsiv, 100dvh |

### Fil-struktur:
```
index.html              ← Allt (HTML + CSS + JS, ~284 KB)
manifest.json           ← PWA-manifest
icon-192.png            ← PWA-ikon 192×192
icon-512.png            ← PWA-ikon 512×512
images/                 ← Produktbilder (26 st PNG)
```

---

## 2. EXTERNA BEROENDEN

Tre externa script laddas från CDN:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-database-compat.js"></script>
```

---

## 3. FIREBASE-KONFIGURATION

```javascript
const _firebaseApp = firebase.initializeApp({
  apiKey: "AIzaSyCjLMN3qVOyn0M5Eu7P0EtA9rnqzAto5Zw",
  authDomain: "zivert-holm.firebaseapp.com",
  databaseURL: "https://zivert-holm-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "zivert-holm",
  messagingSenderId: "1078871076184",
  appId: "1:1078871076184:web:573fc9d2d7ddb621fa892d"
});
const _fbDb = firebase.database();
window._fbDb = _fbDb;
```

### Firebase-databas-struktur (Realtime Database):

| Sökväg i Firebase | localStorage-nyckel | Innehåll |
|---|---|---|
| `/produkter` | `zivert-holm-catalog-v3` | Produktkatalog (kategorier + artiklar) |
| `/lager` | `zivert-holm-inventory-v1` | Lagersaldo per produkt |
| `/erbjudanden` | `zivert-holm-offers-v1` | Erbjudanden/kampanjer |
| `/rapport` | `zivert-holm-sales-v1` | Försäljningshistorik |
| `/kvitton` | `zivert-holm-receipts-v1` | Kvittoregister |
| `/onskningar` | `zivert-holm-wishes-v1` | Kundönskningar |
| `/taggar` | `zivert-holm-tags-v1` | Tagg-definitioner |
| `/receiptCounter` | `zivert-holm-receipt-counter-v1` | Kvitto-räknare (atomisk) |
| `/bubblainställningar` | `zivert-holm-bubbla-v1` | Bubbla-inställningar |
| `/valjKnappInstallningar` | `zivert-holm-vajl-knapp-v1` | Välj-knapp synlighet |
| `/installningar` | `zivert-holm-settings-v1` | Allmänna inställningar (aldrig lösenord) |

### Synk-princip:
- **Admin är källa** för kvitton — Firebase uppdateras efter 10 sekunders fördröjning
- **Lösenord lagras ALDRIG i Firebase** — bara i localStorage
- Vid start: ladda från Firebase (3 sek timeout), fallback till localStorage
- Alla skrivningar: localStorage först → Firebase synk
- Real-time listener: Firebase-ändringar uppdaterar alla anslutna enheter live

---

## 4. ALLA KONSTANTER

```javascript
const DESIGN_W = 1920;
const DESIGN_H = 1080;
const SWISH_NUMBER = "0704798009";
const SWISH_MESSAGE_PREFIX = "Zivert Holm";
const SWISH_LOCK_MASK = 0;
const REPORT_PASSWORD = "123456";                        // Standard admin-lösenord
const DEFAULT_RECEIPT_DELETE_PASSWORD = "072940";         // Standard kvitto-lösenord
const DEFAULT_MOBILE_ADMIN_PW = "123456";                // Standard mobil admin-lösenord
const DEFAULT_RENSA_PW = "123456";                       // Standard rensa-lösenord

// localStorage-nycklar
const SALES_STORAGE_KEY = "zivert-holm-sales-v1";
const CATALOG_STORAGE_KEY = "zivert-holm-catalog-v3";
const OFFERS_STORAGE_KEY = "zivert-holm-offers-v1";
const SETTINGS_STORAGE_KEY = "zivert-holm-settings-v1";
const WISHES_STORAGE_KEY = "zivert-holm-wishes-v1";
const RECEIPT_STORAGE_KEY = "zivert-holm-receipts-v1";
const RECEIPT_COUNTER_KEY = "zivert-holm-receipt-counter-v1";
const INVENTORY_STORAGE_KEY = "zivert-holm-inventory-v1";
const RECEIPT_DELETE_PASSWORD_KEY = "zivert-holm-receipt-delete-pw-v1";
const TAGS_STORAGE_KEY = "zivert-holm-tags-v1";
const MOBILE_ADMIN_PW_KEY = "zivert-holm-mobile-admin-pw-v1";
const NOLLA_ALLT_KEY = "nolladAllt";
const RENSA_PW_KEY = "zivert-holm-rensa-pw-v1";
const BUBBLA_STORAGE_KEY = "zivert-holm-bubbla-v1";
const VAJL_KNAPP_STORAGE_KEY = "zivert-holm-vajl-knapp-v1";
```

Mobilvy aktiveras av:
```javascript
const IS_MOBILE_VIEW = window.location.search.includes("mobile") || window.innerWidth < 768;
```

---

## 5. CSS — EXAKT FÄRGPALETT OCH VARIABLER

```css
:root {
  --bg:            linear-gradient(160deg, #f0f7f4 0%, #e8f4f8 50%, #f5f0eb 100%);
  --surface:       rgba(0,0,0,.03);
  --surface-hover: rgba(0,0,0,.06);
  --border:        rgba(0,0,0,.06);
  --text:          #2c3e35;
  --text-muted:    #6b7c74;
  --accent:        #5b8fa8;
  --accent2:       #c47a3a;
  --highlight:     rgba(91,143,168,.12);
  --shadow:        0 2px 12px rgba(0,0,0,.08);
}
```

### Typografi:
- **Font-family:** `Georgia, "Times New Roman", serif` (överallt)
- **Titlar:** font-style: italic
- **Textfärg (primär):** `#2c3e35` (mörk grön)
- **Textfärg (sekundär):** `#6b7c74` (grå-grön)
- **Prisfärg:** `#c47a3a` (orange/brun)
- **Accentfärg (knappar):** `#5b8fa8` (blå)
- **Felfärg:** `#ff7a6a` / `#f87171` (röd)
- **Gröna stats:** `#34d399`

---

## 6. CSS — ALLA ANIMATIONER

```css
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

@keyframes shimmer {
  0% { background-position: -400px 0; }
  100% { background-position: 400px 0; }
}

@keyframes fadeSlideIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: .6; }
}

@keyframes pilHopp {
  0%, 100% { transform: translateY(0) rotate(-1deg); }
  50% { transform: translateY(-6px) rotate(1deg); }
}

@keyframes pilPeka {
  0% { transform: translateY(0); }
  100% { transform: translateY(-4px); }
}

@keyframes bubblaHopp {
  0%, 100% { transform: translateY(0) scale(1); }
  30% { transform: translateY(-6px) scale(1.03); }
  60% { transform: translateY(-3px) scale(1.01); }
}

@keyframes ssRotera {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes ssPulsera {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.03); }
}

@keyframes ssHintBlink {
  0%, 100% { opacity: 0.4; }
  50% { opacity: 1; }
}

@keyframes ssFadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes ssFadeOut {
  from { opacity: 1; }
  to { opacity: 0; }
}
```

### Var animationerna används:
| Animation | Används på | Effekt |
|---|---|---|
| `float` | Produktbilder, kategori-emojis | Svävande upp/ned (3s loop) |
| `shimmer` | Erbjudande-kort (::after) | Guldskimmer-effekt (3s loop) |
| `fadeSlideIn` | Produktkort | Fade-in med glidning (0.3s) |
| `pulse` | Diverse | Opacity-pulsering |
| `pilHopp` | Tips-bubbla under erbjudanden | Gungande rörelse (3.2s loop) |
| `pilPeka` | Pil-emoji i bubbla | Upp/ned-pekande (1s loop) |
| `bubblaHopp` | Påminnelse-bubbla i Swish-modal | Studsande (1.5s loop) |
| `ssRotera` | Guldring i screensaver | 360° rotation (25s loop) |
| `ssPulsera` | Logo-wrapper i screensaver | Skalningspuls (8s loop) |
| `ssHintBlink` | "Tryck var som helst" i screensaver | Blinkande opacity (2.5s loop) |
| `ssFadeIn` | Screensaver vid visning | Fade-in (1.5s) |
| `ssFadeOut` | Screensaver vid stängning | Fade-out |

---

## 7. DESKTOP-KIOSK — LAYOUT & EXAKTA MÅTT

### Skalningssystem:
```css
.viewport {
  width: 100vw; height: 100vh;
  display: flex; align-items: center; justify-content: center;
  background-image:
    linear-gradient(rgba(255,255,255,0.82), rgba(255,255,255,0.82)),
    url('images/backgrund.png');
  background-size: cover;
}

.stage {
  width: 1920px; height: 1080px;
  transform-origin: center center;
  /* Skalas dynamiskt via JS: scale(Math.min(vw/1920, vh/1080)) */
}

.app {
  width: 1920px; height: 1080px;
  overflow: hidden;
  display: flex; flex-direction: column;
  background: linear-gradient(160deg, #f0f7f4 0%, #e8f4f8 50%, #f5f0eb 100%);
}
```

### Dekorativa orbs:
```css
.orb { position: absolute; border-radius: 50%; filter: blur(80px); pointer-events: none; z-index: 0; }
.orb1 { width: 500px; height: 500px; background: rgba(91,143,168,.08); top: -100px; right: 100px; }
.orb2 { width: 350px; height: 350px; background: rgba(180,200,180,.1); bottom: -80px; left: 150px; }
.orb3 { width: 250px; height: 250px; background: rgba(196,122,58,.06); top: 200px; left: 300px; }
```

### Top bar:
```css
.top-bar {
  height: 52px;
  background: rgba(255,255,255,.7);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(0,0,0,.06);
  padding: 0 28px;
}
.top-bar-title { font-size: 38px; font-style: italic; color: #2c3e35; }
.top-bar-hint { font-size: 14px; color: #6b7c74; letter-spacing: 2px; text-transform: uppercase; }
```

### Huvudlayout (body):
```css
.body {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 380px;  /* Innehåll | Varukorg */
  overflow: hidden;
}
```

---

## 8. ALLA KNAPPAR — EXAKTA MÅTT, FÄRGER OCH FUNKTIONER

### 8.1 BETALA-KNAPP (Swish)
```css
.pay-btn {
  background: linear-gradient(135deg, #72383D, #8B4449);
  color: #EFE9E1;
  border: none;
  border-radius: 12px;
  font-size: 26px;
  font-weight: 700;
  padding: 14px;
  width: 100%;
  box-shadow: 0 4px 15px rgba(114,56,61,0.4);
  letter-spacing: 1px;
}
.pay-btn:disabled { opacity: .35; cursor: not-allowed; box-shadow: none; }
```
**Funktion:** Öppnar Swish QR-modal. Disabled om varukorgen är tom.

### 8.2 RENSA VAL-KNAPP
```css
.clear-btn {
  background: transparent;
  border: 1px solid #D1C7BD;
  border-radius: 12px;
  font-size: 20px;
  color: #AC9C8D;
  padding: 10px;
  width: 100%;
}
.clear-btn:not(:disabled):hover { background: rgba(0,0,0,.03); color: #72383D; }
```
**Funktion:** Tömmer varukorgen (nollställer state.qty).

### 8.3 KVANTITETS-KNAPPAR (+/-)
```css
.qty-btn {
  width: 44px; height: 44px;
  min-width: 44px; min-height: 44px;
  background: rgba(91,143,168,.12);
  border: 2px solid rgba(91,143,168,.3);
  color: #5b8fa8;
  border-radius: 10px;
  font-size: 28px;
}
```
**Funktion:** `+` kör `updateQty(catKey, itemId, +1)`, `-` kör `updateQty(catKey, itemId, -1)`.

### 8.4 TILLBAKA-KNAPP (Kategorivy)
```css
.back-btn {
  background: #ffffff;
  border: 1px solid rgba(0,0,0,.08);
  border-radius: 12px;
  font-size: 26px;
  color: #2c3e35;
  height: 72px;
  width: 200px;
  box-shadow: 0 2px 8px rgba(0,0,0,.06);
}
```
**Funktion:** Navigerar tillbaka till översiktsvyn (`openOverview()`).

### 8.5 ERBJUDANDE VÄLJ-KNAPP ("Ja, tack!")
```css
.erbjudande-välj-knapp {
  background: linear-gradient(135deg, #72383D, #8B4449);
  color: #EFE9E1;
  border: none;
  border-radius: 14px;
  padding: 14px 28px;
  font-size: 18px;
  font-weight: bold;
  box-shadow: 0 4px 15px rgba(114,56,61,0.4);
}
```
**Funktion:** Lägger till erbjudandets produkter i varukorgen (`läggTillErbjudande(offer)`).

### 8.6 ÖNSKEKNAPP
```css
.wish-trigger-btn {
  background: #ffffff;
  border: 2px solid #C9A84C;
  border-radius: 14px;
  color: #322D29;
  font-size: 22px;
  font-weight: bold;
  padding: 12px 32px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.12);
}
```
**Funktion:** Öppnar önsknings-modalen (`openWishModal()`).

### 8.7 ADMIN-KNAPPAR (generell)
```css
.admin-btn {
  background: #5b8fa8;
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 18px;
  font-weight: 700;
  padding: 10px 20px;
}
.admin-btn.danger { background: #e74c3c; }
.admin-btn.secondary {
  background: rgba(0,0,0,.04);
  color: #6b7c74;
  border: 1px solid rgba(0,0,0,.08);
}
```

### 8.8 FILTER-KNAPPAR (Rapport)
```css
.filter-btn {
  background: rgba(0,0,0,.03);
  border: 1px solid rgba(0,0,0,.08);
  border-radius: 8px;
  font-size: 18px;
  color: #6b7c74;
  padding: 8px 18px;
}
.filter-btn.danger { color: rgba(255,100,80,.8); border-color: rgba(255,100,80,.2); }
```
**Funktioner:**
- "Senaste 7 dagar" → `renderReport("7days")`
- "Denna vecka" → `renderReport("thisweek")`
- "All data" → `renderReport("all")`
- "Kopiera" → `copyReportToClipboard()`
- "Rensa" → Raderar försäljningsdata (kräver lösenord)

### 8.9 MODAL-KNAPPAR
```css
.modal-btn {
  border: none;
  border-radius: 10px;
  background: #5b8fa8;
  color: #fff;
  font-size: 22px;
  font-weight: 700;
  padding: 12px;
}
.modal-btn.cancel {
  background: #f0f4f2;
  color: #6b7c74;
  border: 1px solid rgba(0,0,0,.08);
}
```

### 8.10 REGISTRERA KÖP / VISA KVITTO (Swish-modal)
```css
.pay-register-btn {
  min-height: 64px;
  border: 1px solid rgba(0,0,0,.1);
  border-radius: 14px;
  background: #f0f4f2;
  color: #2c3e35;
  font-size: 24px;
}
.pay-receipt-btn {
  min-height: 64px;
  border: none;
  border-radius: 14px;
  background: #5b8fa8;
  color: #fff;
  font-size: 24px;
  font-weight: 700;
  box-shadow: 0 4px 14px rgba(91,143,168,.3);
}
```
**Funktion:** "Registrera köp" sparar försäljningen. "Visa kvitto" visar kvitto-modalen.

### 8.11 STÄNG KVITTO-KNAPP
```css
.receipt-close-btn {
  min-height: 60px;
  border: none;
  border-radius: 14px;
  background: #5b8fa8;
  color: #fff;
  font-size: 24px;
  font-weight: 700;
  box-shadow: 0 4px 14px rgba(91,143,168,.3);
}
```

### 8.12 DOLD ADMIN-TRIGGER (Desktop)
```css
.hidden-report-trigger {
  position: absolute;
  top: 0; right: 0;
  width: 140px; height: 140px;
  background: rgba(255,255,255,0.001);
  z-index: 200;
}
```
**Funktion:** Osynlig knapp. 5 klick inom kort tid → öppnar lösenords-modal för admin.

### 8.13 DOLD ADMIN-TRIGGER (Mobil)
```html
<div id="mobileLogoTrigger" style="
  position: fixed; top: 60px; right: 0;
  width: 80px; height: 80px;
  z-index: 9999; opacity: 0; cursor: default;
"></div>
```
**Funktion:** 5 klick → öppnar mobil admin-lösenord.

### 8.14 CATALOG SMALL BUTTON (Ta bort etc.)
```css
.catalog-small-btn {
  border: none;
  border-radius: 8px;
  background: rgba(231,76,60,.15);
  color: #e74c3c;
  font-size: 16px;
  padding: 8px 12px;
}
```

### 8.15 ADMIN NAV ITEMS
```css
.anav-item {
  padding: 14px 20px;
  font-size: 18px;
  color: #6b7c74;
  border-left: 3px solid transparent;
}
.anav-item.active {
  color: #2c3e35;
  border-left-color: #5b8fa8;
  background: rgba(91,143,168,.08);
}
```

### 8.16 ADMIN LOGOUT
```css
.admin-logout-btn {
  background: rgba(0,0,0,.04);
  border: 1px solid rgba(0,0,0,.08);
  border-radius: 8px;
  color: #6b7c74;
  font-size: 18px;
  padding: 6px 16px;
}
```
**Funktion:** Stänger admin → tillbaka till kiosk-vy (`openOverview()`).

---

## 9. KATEGORIKORT — DESIGN OCH GRADIENTER

### Desktop-kort:
```css
.category-card {
  border-radius: 20px;
  min-height: 180px;
  padding: 20px;
  border: 1px solid rgba(0,0,0,.06);
  box-shadow: 0 2px 12px rgba(0,0,0,.08);
}
.category-card:hover { transform: scale(1.03); box-shadow: 0 12px 32px rgba(0,0,0,.12); }
.category-card:active { transform: scale(.98); }
.category-card-emoji { font-size: 52px; animation: float 3s ease-in-out infinite; }
.category-card-name { font-size: 36px; font-style: italic; }
.category-card-sub { font-size: 18px; color: #6b7c74; }
.category-card-count { font-size: 14px; color: #8a9b93; letter-spacing: 1px; }
```

### Grid:
```css
.category-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}
```

### Kategori-gradienter (JS):
Varje kategori har en unik bakgrundsgradient. Bestäms av `getCategoryGradient(key)` i JS:
- **Dryck:** Blågrön gradient
- **Snacks:** Orange/gul gradient
- **Mellanmål:** Grön gradient
- **Mat:** Röd/rosa gradient

### Kategori-emojis (JS):
Bestäms av `getCategoryEmoji(key)`:
- **Dryck:** 🥤
- **Snacks:** 🍿
- **Mellanmål:** 🍫
- **Mat:** 🍕

---

## 10. PRODUKTKORT — EXAKTA MÅTT

### Desktop:
```css
.product-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
  padding: 0 28px 24px;
}
.product-card {
  background: #ffffff;
  border: 1px solid rgba(0,0,0,.06);
  box-shadow: 0 2px 12px rgba(0,0,0,.08);
  border-radius: 16px;
  animation: fadeSlideIn 0.3s ease both;
}
.product-img-box { height: 100px; }
.product-img-box img {
  height: 80px;
  animation: float 3s ease-in-out infinite;
  filter: drop-shadow(0 4px 12px rgba(0,0,0,.15));
}
.product-name { font-size: 20px; font-weight: 700; color: #2c3e35; }
.product-price { font-size: 17px; font-style: italic; color: #c47a3a; }
.product-note { font-size: 14px; font-style: italic; color: #8a9b93; }
.qty-val { font-size: 24px; font-weight: 700; color: #2c3e35; min-width: 30px; }
```

### Mobil:
```css
.mobile-product-card {
  background: #ffffff;
  border: 1px solid rgba(0,0,0,.06);
  border-radius: 16px;
}
.mobile-product-img { height: 120px; }
.mobile-product-img img { height: 90px; }
.mobile-product-name { font-size: 15px; font-style: italic; }
.mobile-product-price { font-size: 20px; font-weight: 700; color: #c47a3a; }
```

---

## 11. VARUKORGS-PANEL (Höger sida, desktop)

```css
.cart-panel {
  background: rgba(255,255,255,.75);
  backdrop-filter: blur(8px);
  border-left: 1px solid rgba(0,0,0,.06);
  /* Bredd: 380px (satt av grid-template-columns i .body) */
}
.cart-top { padding: 20px 16px; }
.cart-top h3 { font-size: 28px; font-style: italic; }
.cart-row {
  font-size: 20px;
  padding: 8px 0;
  border-bottom: 1px dashed rgba(0,0,0,.08);
  color: #6b7c74;
}
.cart-count-row { font-size: 20px; }
.cart-total-row {
  font-size: 32px;
  font-style: italic;
  font-weight: 700;
  border-top: 1px solid rgba(0,0,0,.08);
  padding-top: 8px;
}
```

---

## 12. ERBJUDANDE-KORT

```css
.offers-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  padding-bottom: 60px;
}
.offer-card {
  background: #fdf8f0;
  border: 1px solid #e8c87a;
  border-radius: 14px;
  padding: 16px 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,.05);
}
/* Shimmer-effekt (::after): */
background: linear-gradient(90deg, transparent, rgba(232,200,122,.06), transparent);
animation: shimmer 3s infinite;

.offer-pill {
  background: rgba(196,122,58,.15);
  color: #c47a3a;
  font-size: 13px;
  letter-spacing: 1.5px;
  border-radius: 8px;
  padding: 6px 12px;
}
.offer-title { font-size: 24px; font-style: italic; }
.offer-desc { font-size: 15px; color: #6b7c74; }
.offer-price { font-size: 18px; font-weight: 700; color: #c47a3a; }
```

### Tips-bubbla under erbjudanden:
```css
.erbjudande-pil-bubbla {
  background: linear-gradient(135deg, #72383D, #8B4449);
  color: #EFE9E1;
  border-radius: 16px;
  padding: 14px 24px;
  box-shadow: 0 4px 15px rgba(114,56,61,0.4);
  animation: pilHopp 3.2s ease-in-out infinite;
  max-width: 220px;
}
/* Triangel-pil uppåt: */
.erbjudande-pil-bubbla::before {
  border-width: 0 10px 12px;
  border-color: transparent transparent #72383D;
}
.pil-emoji { font-size: 28px; animation: pilPeka 1s ease-in-out infinite alternate; }
.pil-text { font-size: 15px; font-weight: bold; color: #EFE9E1; }
.pil-subtext { font-size: 14px; color: #D1C7BD; }
```

---

## 13. SWISH BETALNINGS-MODAL

```css
.pay-overlay {
  position: absolute; inset: 0;
  background: rgba(0,0,0,.3);
  z-index: 40;
}
.pay-modal {
  background: #ffffff;
  border-radius: 22px;
  max-width: 680px;
  padding: 28px;
}
.pay-modal-head h3 { font-size: 40px; font-style: italic; }
.pay-qr-box {
  width: 320px; height: 320px;
  border-radius: 16px;
  padding: 14px;
  box-shadow: 0 8px 32px rgba(0,0,0,.3);
}
.pay-amount { font-size: 44px; font-weight: 700; color: #c47a3a; }
.pay-message, .pay-number { font-size: 20px; color: #6b7c74; }
```

### Påminnelse-bubbla:
```css
.paminnelse-bubbla {
  background: #fffbe6;
  border: 2px solid #f0c040;
  border-radius: 20px;
  padding: 12px 20px;
  font-size: 15px;
  font-weight: bold;
  animation: bubblaHopp 1.5s ease-in-out infinite;
}
```

### Swish QR-payload format:
```
C{nummer};{belopp med komma};{meddelande};{lås}
Exempel: C0704798009;125,50;Zivert Holm 3 varor KV-0042;0
```

### Nedräkning:
- Startar på 15 sekunder
- Visas i `.receipt-countdown-wrap` med stora siffror (28px, color `#c47a3a`)
- Vid 0: stänger modalen automatiskt

---

## 14. KVITTO-MODAL

```css
.receipt-overlay { z-index: 50; }
.receipt-modal {
  max-width: 600px;
  border-radius: 22px;
  padding: 28px;
}
.receipt-logo { font-size: 28px; font-style: italic; }
.receipt-id-value { font-size: 32px; font-weight: 700; color: #c47a3a; }
.receipt-item-row { font-size: 22px; }
.receipt-total-row { font-size: 28px; font-weight: 700; }
```

### Kvitto-format:
```
Zivert Holms hörna
KVITTO #0001
2026-03-19 14:32

Coca-Cola Zero × 2     40 kr
Nocco × 1              25 kr
────────────────────────
TOTALT                  65 kr
Betald via Swish till 0704798009
```

---

## 15. ADMIN-PANEL — LAYOUT OCH SEKTIONER

### Struktur:
```css
.admin-shell { width: 100%; height: 100%; display: flex; flex-direction: column; }
.admin-top-bar {
  height: 52px;
  background: rgba(91,143,168,.12);
  backdrop-filter: blur(10px);
}
.admin-body {
  flex: 1;
  display: grid;
  grid-template-columns: 180px 1fr;
}
.admin-nav { background: #f0f4f2; padding: 16px 0; }
.admin-content { background: #ffffff; padding: 24px; }
```

### Admin-sektioner (nav-items):

| Sektion | ID | Ikon | Beskrivning |
|---|---|---|---|
| Rapport | `adminSectionReport` | 📊 | Försäljningsrapport med filter |
| Produkter | `adminSectionProducts` | 📦 | Katalog-editor |
| Lager | `adminSectionInventory` | 📦 | Lagersaldo-hantering |
| Erbjudanden | `adminSectionOffers` | 🏷 | Kampanj-editor |
| Önskningar | `adminSectionWishes` | 💌 | Kundönskningar |
| Kvitton | `adminSectionReceipts` | 🧾 | Kvitto-hantering med taggar |
| Inställningar | `adminSectionSettings` | ⚙ | Swish, lösenord, nolla allt |

### Rapport — Statistik-kort:
```css
.stat-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.stat-card {
  background: rgba(0,0,0,.02);
  border-radius: 12px;
  border: 1px solid rgba(0,0,0,.06);
  padding: 16px 18px;
}
/* Gröna stats (inline-styles i HTML): */
border: 1px solid rgba(52,211,153,.3);
background: rgba(52,211,153,.05);
color: #34d399;  /* Både label och value */
```

### Tre stats som visas:
1. **Registrerade köp** (antal)
2. **Sålda artiklar** (antal)
3. **Summerat värde** (kr)

### Rapport-output:
```css
.report-group {
  background: rgba(0,0,0,.02);
  border-radius: 12px;
  border: 1px solid rgba(0,0,0,.06);
  padding: 14px 18px;
}
.report-group h4 { font-size: 24px; font-style: italic; }
.report-row {
  font-size: 20px;
  padding: 6px 0;
  border-bottom: 1px dashed rgba(0,0,0,.06);
}
```

---

## 16. INSTÄLLNINGAR-SEKTIONEN

### Layout:
```css
/* 2-kolumns grid: Swish-inställningar | Lösenord */
display: grid;
grid-template-columns: 1fr 1fr;
gap: 20px;
```

### Swish-inställningar (vänster kolumn):
- **Fält:** Swish-nummer (text input)
- **Fält:** Anteckningar (textarea, 100px hög)
- **Knapp:** "Spara Swish-inställningar"

### Lösenord (höger kolumn):
Fyra lösenords-kort med identisk struktur:
```css
/* Varje lösenords-kort: */
background: rgba(0,0,0,.02);
border: 1px solid rgba(0,0,0,.06);
border-radius: 12px;
padding: 14px;
```

1. **ADMINLÖSENORD** — Skyddar rapport/admin-åtkomst
2. **KVITTO-LÖSENORD** — Skyddar tagga/avtagga kvitton
3. **MOBIL ADMIN-LÖSENORD** — Mobil admin-åtkomst (5 tryck)
4. **RENSA-KNAPPENS LÖSENORD** — Skyddar rensa-knappen i rapport

### "Nolla allt"-sektion:
```css
/* Rött varningskort: */
padding: 20px;
background: rgba(248,113,113,.06);
border: 2px solid rgba(248,113,113,.2);
border-radius: 16px;
```
**Funktion:** Raderar ALL kvitto- och rapportdata permanent.

### Skärmsläckare-sektion:
```css
/* Guldigt kort: */
padding: 20px;
background: rgba(201,168,76,.06);
border: 2px solid rgba(201,168,76,.2);
border-radius: 16px;
```
**Funktion:** Förhandsgranskning av screensavern.

---

## 17. KATALOG-EDITOR (Produkter)

### Layout:
```css
.catalog-editor { display: grid; gap: 14px; }
.catalog-group {
  background: rgba(0,0,0,.02);
  border-radius: 12px;
  border: 1px solid rgba(0,0,0,.06);
  padding: 16px;
}
.catalog-item-row {
  display: grid;
  grid-template-columns: 1fr 120px 100px;
  gap: 10px;
}
```

### Input-fält:
```css
.catalog-input {
  border: 1px solid rgba(0,0,0,.1);
  border-radius: 8px;
  background: #f8faf9;
  padding: 8px 12px;
  font-size: 18px;
}
.catalog-input:focus {
  border-color: rgba(91,143,168,.4);
  box-shadow: 0 0 0 2px rgba(91,143,168,.1);
}
```

### Knappar:
- "Lägg till kategori" → `addCategory()`
- "Spara ändringar" → `saveCatalogChanges()`
- "↩ Återgå" → Återställ till senast sparade
- Per kategori: "Lägg till produkt", "Ta bort kategori"
- Per produkt: "Ta bort" (röd knapp)

---

## 18. ERBJUDANDE-EDITOR

```css
.offers-edit-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
.offer-edit-card {
  background: rgba(0,0,0,.02);
  border-radius: 16px;
  border: 1px solid rgba(0,0,0,.06);
  padding: 20px;
}
.offer-type-pill {
  padding: 6px 14px;
  border-radius: 8px;
  font-size: 14px;
  letter-spacing: 1px;
}
.offer-type-pill.active {
  background: rgba(91,143,168,.15);
  color: #5b8fa8;
  border-color: rgba(91,143,168,.3);
}
```

### Erbjudande-typer (pills):
- VECKANS DEAL
- KOMMER SNART
- NYHET
- (Anpassade)

### Extra admin-kontroller under Erbjudanden:
1. **Flytande tips-bubbla:** Checkbox + 2 textfält (rad 1 & rad 2)
2. **"Ja, tack!"-knapp:** Checkbox för att visa/dölja på erbjudanden

---

## 19. LAGER-EDITOR

### Kolumnstruktur:
```css
/* Rubrikrad: */
display: grid;
grid-template-columns: 1fr 90px 90px 100px 80px 40px;
gap: 8px;
```

| Kolumn | Bredd | Beskrivning |
|---|---|---|
| Produkt | 1fr | Produktnamn |
| Lager | 90px | Totalt inköpt (orange) |
| Stock | 90px | Framme vid kiosken (blå) |
| Fyll på | 100px | Flytta från Lager → Stock |
| Justera | 80px | Ändra lager direkt |
| (Ta bort) | 40px | Röd knapp |

---

## 20. KVITTO-HANTERING & TAGGAR

### Standard-taggar:
```javascript
[
  { id: "TEST",       label: "TEST",       color: "#ffd966", bg: "rgba(255,217,102,.15)", border: "rgba(255,217,102,.35)", emoji: "🧪" },
  { id: "EGET_BRUK",  label: "EGET BRUK",  color: "#a0c4ff", bg: "rgba(160,196,255,.15)", border: "rgba(160,196,255,.35)", emoji: "🏠" },
  { id: "GRATIS",     label: "GRATIS",     color: "#b4f8c8", bg: "rgba(180,248,200,.15)", border: "rgba(180,248,200,.35)", emoji: "🎁" },
  { id: "TRASIG",     label: "TRASIG",     color: "#ff9b9b", bg: "rgba(255,155,155,.15)", border: "rgba(255,155,155,.35)", emoji: "💔" }
]
```

### Kvitto-datastruktur:
```javascript
{
  id: "0001",                    // Sekvensiellt nummer (0001-9999)
  status: "registrerad",
  time: "2026-03-19T14:32:00Z",  // ISO timestamp
  datum: "2026-03-19",           // sv-SE format
  tid: "14:32",                  // sv-SE format
  items: [
    { name: "Coca-Cola Zero", qty: 2, price: 20, sum: 40 }
  ],
  total: 40,
  tagged: false,
  tagType: null                  // eller "TEST", "EGET_BRUK", etc.
}
```

---

## 21. ÖNSKNINGS-MODAL

```css
.wish-modal {
  max-width: 700px;
  border-radius: 22px;
  padding: 28px;
}
.wish-cat-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}
.wish-cat-btn {
  background: #f0f4f2;
  border: 1px solid rgba(0,0,0,.08);
  border-radius: 12px;
  font-size: 18px;
  padding: 14px 10px;
}
.wish-cat-btn.selected {
  background: rgba(91,143,168,.2);
  border-color: rgba(91,143,168,.5);
}
.wish-textarea {
  height: 100px;
  border-radius: 12px;
  font-size: 20px;
  padding: 14px 16px;
}
```

### Flöde:
1. Välj kategori (4-kolumns grid med knappar)
2. Skriv önskan i textarea (max 200 tecken)
3. "Skicka önskan" → sparar till localStorage + Firebase
4. Bekräftelsetext: "✅ Tack! Din önskan är skickad."

---

## 22. SCREENSAVER

### Aktiveras: efter 120 000 ms (2 min) inaktivitet
### Avaktiveras: touch/click var som helst

```css
#screensaver {
  position: fixed; inset: 0;
  z-index: 99999;
  animation: ssFadeIn 1.5s ease forwards;
}
.ss-bakgrund {
  background: url('images/backgrund.png') center/cover no-repeat,
    linear-gradient(135deg, #322D29 0%, #72383D 40%, #AC9C8D 100%);
}
.ss-logga-wrapper { width: 360px; height: 360px; animation: ssPulsera 8s infinite; }
.ss-ring {
  border: 14px solid transparent;
  background: conic-gradient(from 0deg,
    #8B6914, #C9A84C, #E8D48B, #C9A84C, #8B6914, #6B4F0E, #8B6914) border-box;
  animation: ssRotera 25s linear infinite;
}
.ss-logga { font-size: 128px; font-weight: bold; color: #ac9c8d; letter-spacing: 6px; }
.ss-titel { font-size: 44px; color: #3a2d28; }
.ss-text { font-size: 48px; color: #3a2d28; }
.ss-hint { font-size: 32px; letter-spacing: 4px; animation: ssHintBlink 2.5s infinite; }
```

### Screensaver-innehåll:
```
[Bakgrundsbild + gradient]
   ┌─────────────────┐
   │   [Guldring]    │ ← roterande conic-gradient ring
   │      ZH         │ ← stor logga (128px, #ac9c8d)
   │                 │
   └─────────────────┘
   Välkommen!           ← 44px
   Hungrig? Vi har snacks! 👋  ← 48px
   TRYCK VAR SOM HELST FÖR ATT BÖRJA  ← 32px, blinkande
```

---

## 23. MOBIL ADMIN-VY

### Admin-trigger: 5 klick på osynlig 80×80 box (top-right)

### Admin-grid (knappar):
```css
/* 2-kolumns grid */
grid-template-columns: 1fr 1fr;
gap: 12px;
```

| Knapp | data-section | Ikon |
|---|---|---|
| Rapport | `report` | 📊 |
| Produkter | `products` | 📦 |
| Lager | `inventory` | 📦 |
| Erbjudanden | `offers` | 🏷 |
| Önskningar | `wishes` | 💌 |
| Kvitton | `receipts` | 🧾 |
| Inställningar | `settings` | ⚙ (full bredd, `grid-column: 1/-1`) |

### Alla mobil-admin-knappar har samma stil:
```css
background: #ffffff;
border: 1px solid rgba(0,0,0,.06);
border-radius: 14px;
padding: 20px 12px;
color: #2c3e35;
font-family: Georgia, serif;
font-size: 16px;
box-shadow: 0 2px 8px rgba(0,0,0,.04);
display: flex; flex-direction: column; align-items: center; gap: 8px;
```

### Mobil topbar (admin):
```css
.mav-topbar {
  position: fixed; top: 0; left: 0; right: 0; z-index: 91;
  background: rgba(255,255,255,.85);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(0,0,0,.06);
}
```

---

## 24. MOBIL KATEGORI-KORT

```css
.mobile-cat-card {
  border-radius: 16px;
  padding: 24px 16px;
  border: 1px solid rgba(0,0,0,.06);
}
.mobile-cat-emoji { font-size: 36px; }
.mobile-cat-name { font-size: 17px; font-style: italic; }
.mobile-cat-count { font-size: 12px; color: #6b7c74; letter-spacing: 1px; }

/* Grid: */
#mobileCatGrid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
```

---

## 25. RESPONSIVE ADMIN (under 768px)

```css
@media (max-width: 768px) {
  .admin-hamburger-btn { display: block; }
  .admin-body { flex-direction: column !important; }
  .admin-nav {
    position: fixed !important;
    width: 260px !important; height: 100dvh !important;
    transform: translateX(-100%);
    transition: transform .25s ease;
    z-index: 200;
  }
  .admin-nav.mobile-nav-open { transform: translateX(0); }
  .admin-nav-overlay { /* Mörk overlay bakom sidonav */ }
  .admin-nav-overlay.active { display: block; }
}
```

---

## 26. SCROLLBARS

```css
/* Alla scrollbara element: 8px bredd, rundade */
::-webkit-scrollbar { width: 8px; }
::-webkit-scrollbar-thumb {
  background: rgba(91,143,168,.25);
  border-radius: 8px;
}
::-webkit-scrollbar-track { background: rgba(0,0,0,.02); }
```

Gäller för: `.product-grid`, `.view`, `.cart-top`, `.admin-content`, `.report-output`, `.catalog-editor`

---

## 27. PWA-MANIFEST

```json
{
  "name": "Zivert Holms hörna",
  "short_name": "ZH Kiosk",
  "start_url": "/zivert-holm-kiosk/?mobile",
  "display": "standalone",
  "background_color": "#0f0c29",
  "theme_color": "#f5a623",
  "icons": [
    { "src": "icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

---

## 28. PRODUKTBILDER (images/)

Alla bilder mappas via `PRODUCT_IMAGES` i JS till produkt-ID:

| Filnamn | Produkt |
|---|---|
| Arla-pudding.png | Arla Pudding |
| billys-pan-pizza*.png | Billy's Pan Pizza (3 varianter) |
| bonaqua.png | Bonaqua |
| celsius.png | Celsius |
| cola-zero.png | Coca-Cola Zero |
| cornybigg.png | Corny Big |
| delicatoboll.png | Delicatoboll |
| fanta.png | Fanta |
| flapjack.png | Flapjack |
| kexchoklad.png | Kexchoklad |
| lantchips.png | Lantchips |
| löfbergs.png | Löfbergs |
| monster.png | Monster |
| nocco.png | Nocco |
| nudlar.png | Nudlar |
| pepsi-max.png | Pepsi Max |
| pringles.png | Pringles |
| proteinbar.png | Proteinbar |
| punchrulle.png | Punchrulle |
| redbull.png | Red Bull |
| risifrutti.png | Risifrutti |
| sportlunch.png | Sportlunch |
| trocadero.png | Trocadero |
| backgrund.png | Bakgrundsbild (screensaver + viewport) |

---

## 29. STATE-OBJEKT (JS)

```javascript
const state = {
  qty: {},                       // { "kategori__produktId": antal }
  offerCart: [],                  // Erbjudande-artiklar i korgen
  currentCategory: null,         // Aktiv kategori-nyckel
  qrInstance: null,              // QRCode-objekt
  currentReportMode: "7days",    // "7days" | "thisweek" | "all"
  lastRenderedReportText: "",    // Cache för kopiera-till-clipboard
  currentAdminSection: "rapport" // Aktiv admin-sektion
};
```

---

## 30. DATASTRUKTURER

### Katalog (kategorier + produkter):
```javascript
[
  {
    key: "dryck",
    title: "Dryck",
    subtitle: "Kylda drycker & energi",
    items: [
      { id: "coca-cola-zero", name: "Coca-Cola Zero", price: 20, note: "" },
      { id: "nocco", name: "Nocco", price: 25, note: "Välj smak vid kylen" }
    ]
  }
]
```

### Erbjudande:
```javascript
{
  type: "VECKANS DEAL",
  title: "Billigare lunch",
  desc: "Nudlar + Nocco för 40 kr",
  price: "40 kr",
  produkter: [
    { produktId: "nudlar", antal: 1 },
    { produktId: "nocco", antal: 1 }
  ]
}
```

### Försäljning:
```javascript
{
  receiptId: "0042",
  time: "2026-03-19T14:32:00Z",
  items: [
    { name: "Coca-Cola Zero", qty: 2, price: 20, sum: 40 }
  ],
  total: 40
}
```

### Lager:
```javascript
{
  "coca-cola-zero": { lager: 50, stock: 12 },
  "nocco": { lager: 30, stock: 8 }
}
```

### Inställningar:
```javascript
{
  swishNumber: "0704798009",
  swishNotes: "Backup: 073XXXXXXX"
  // Lösenord lagras ALDRIG här — bara i localStorage direkt
}
```

---

## 31. ADMIN-ÅTKOMST SAMMANFATTNING

### Desktop:
1. Klicka 5 gånger på osynlig 140×140px yta (top-right av `<div class="app">`)
2. Lösenordsmodal visas
3. Korrekt lösenord → `reportView` med admin-nav

### Mobil:
1. Klicka 5 gånger på osynlig 80×80px yta (`position:fixed; top:60px; right:0`)
2. Lösenordsmodal visas
3. Korrekt lösenord → `mobileAdminPanel` med knapp-grid

### Lösenord (standard):
| Typ | Standard | localStorage-nyckel |
|---|---|---|
| Admin | `123456` | Via `REPORT_PASSWORD` (konstant) eller settings |
| Kvitto | `072940` | `zivert-holm-receipt-delete-pw-v1` |
| Mobil admin | `123456` | `zivert-holm-mobile-admin-pw-v1` |
| Rensa | `123456` | `zivert-holm-rensa-pw-v1` |

---

## 32. KVITTO-RÄKNARE

- Atomisk Firebase-transaction på `/receiptCounter`
- Format: 0001-9999 (wraps vid 1000 i originalkoden)
- Fallback till localStorage om Firebase ej tillgänglig
- Garanterar unika kvitto-ID oavsett antal enheter

---

## 33. STARTUP-ORDNING (bootstrap)

```javascript
async function bootstrap() {
  1. await loadFromFirebase();      // Hämta data (3s timeout)
  2. migrateToFirebase();           // Migrera saknade nycklar
  3. loadCatalog();                 // Ladda kategorier
  4. cleanupQtyState();             // Rensa gamla kvantiteter
  5. renderOverview();              // Rita hemskärm
  6. renderSummary();               // Rita varukorgs-sidebar
  7. openOverview();                // Visa hemvy
  8. startRealtimeSync();           // Starta Firebase-listener
  9. // Initiera desktop/mobil UI baserat på IS_MOBILE_VIEW
}
```

---

## 34. HUR ADMIN-SIDAN SKA ANVÄNDA DENNA DATA

En separat ADMIN-sida behöver:

1. **Samma Firebase-config** — anslut till samma databas
2. **Läsa alla nycklar** i Firebase-databasen (se sektion 3)
3. **Rendera admin-sektionerna** med samma styling (se sektionerna 15-19)
4. **Skriva tillbaka** ändringar till Firebase → kiosken plockar upp det via real-time sync
5. **Stå i standby** och lyssna på Firebase-ändringar tills kiosken kopplas upp
6. **Aldrig hantera lösenord via Firebase** — de finns bara lokalt

### Admin-sidan behöver INTE:
- Kiosk-vyn (kategori-grid, produktkort, varukorg, Swish-modal)
- Screensaver
- Mobilvyn
- QRCode.js

### Admin-sidan behöver:
- Firebase SDK (samma version)
- Alla admin-sektioner: Rapport, Produkter, Lager, Erbjudanden, Önskningar, Kvitton, Inställningar
- Samma CSS-klasser och mått för att se identisk ut
- Tagg-systemet
- Kvitto-hantering

---

*Denna fil genererades som en komplett teknisk referens för replikering.*
*Alla värden, mått och färgkoder är exakta från `index.html`.*

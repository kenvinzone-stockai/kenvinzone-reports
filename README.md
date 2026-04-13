# KenvinZone Report Dashboard

Dashboard web tĩnh hiển thị báo cáo phân tích XAUUSD/NQ tự động từ pipeline KenvinZone.

## Cấu trúc file

```
kenvinzone-dashboard/
├── index.html        ← Web app chính (không cần sửa)
├── reports.json      ← Database báo cáo (Make.com tự cập nhật)
├── reports/          ← Thư mục chứa file DOCX (tùy chọn)
│   └── *.docx
└── README.md
```

## Deploy lên GitHub Pages (một lần duy nhất)

### Bước 1 – Tạo GitHub repo
1. Vào github.com → New repository
2. Tên repo: `kenvinzone-reports` (hoặc bất kỳ tên nào)
3. Chọn **Public** (bắt buộc để GitHub Pages miễn phí hoạt động)
4. Create repository

### Bước 2 – Upload files
```bash
# Hoặc kéo thả trực tiếp trên giao diện web GitHub
git clone https://github.com/YOUR_USERNAME/kenvinzone-reports.git
cd kenvinzone-reports
# Copy toàn bộ files vào đây
git add .
git commit -m "Initial deploy"
git push
```

### Bước 3 – Bật GitHub Pages
1. Repo → Settings → Pages
2. Source: **Deploy from a branch**
3. Branch: **main** / folder: **/ (root)**
4. Save

Sau 2-3 phút, web app live tại:
`https://YOUR_USERNAME.github.io/kenvinzone-reports/`

---

## Tích hợp Make.com – Cập nhật reports.json tự động

### Cấu trúc 1 report trong reports.json

```json
{
  "id": "RPT-20260413-01",
  "datetime": "2026-04-13T06:30:00+07:00",
  "date_display": "13/04/2026",
  "time_display": "06:30",
  "instrument": "XAUUSD",
  "archetype": "Bản nối tiếp / Cập nhật 4 giờ",
  "bias": "SELL THE RALLY",
  "bias_type": "bearish",
  "price_current": 4736.09,
  "catalyst": "PPI tháng 3 ra 20:30 – rủi ro USD tăng",
  "summary": "Giá đã vượt 4.730–4.740...",
  "key_levels": {
    "resistance": [4754, 4779],
    "support": [4700, 4672],
    "current": 4736.09
  },
  "timeframes": {
    "D1": "Giảm có hồi kỹ thuật",
    "H4": "Bearish nhẹ – hồi kỹ thuật",
    "H1": "Hồi rồi chững lại",
    "M15": "Tăng ngắn hạn suy yếu"
  },
  "scenarios": [
    {"id": "S1", "name": "Sell the rally nâng", "prob": "50%", "zone": "4.754–4.779"},
    {"id": "S2", "name": "Ngắn hạn continuation", "prob": "20%", "zone": "Trên 4.754"},
    {"id": "S3", "name": "Range 4.720–4.754", "prob": "20%", "zone": "4.720–4.754"},
    {"id": "S4", "name": "Phản ứng ngược", "prob": "10%", "zone": "Dưới 4.720"}
  ],
  "session_quote": "Câu chốt dùng trong phiên...",
  "docx_url": "reports/XAUUSD_20260413_0630_final.docx"
}
```

### Make.com Scenario – Flow tổng quát

```
[TradingView Webhook]
        ↓
[HTTP: Gọi Claude API] → nhận JSON phân tích
        ↓
[JSON Parse + Format] → tạo object report mới
        ↓
[GitHub: Get file] → đọc reports.json hiện tại
        ↓
[JSON: Prepend report mới] → giữ tối đa 60 reports
        ↓
[GitHub: Update file] → commit reports.json lên repo
        ↓
[Telegram: sendMessage] → thông báo "Báo cáo mới"
```

### Module GitHub trong Make.com
- **Module**: GitHub → Update a File
- **Connection**: Dùng GitHub Personal Access Token
- **Repository**: kenvinzone-reports
- **File Path**: reports.json
- **Content**: Base64 của JSON mới
- **Commit Message**: `Auto: Report ${instrument} ${datetime}`

### Prompt gửi Claude API (HTTP module)

```
System: [Dán toàn bộ transfer pack skill của bạn]

User: Phân tích XAUUSD, cập nhật 4 giờ.
Thời điểm: {{datetime}}
Giá hiện tại: {{price}}
Bối cảnh: {{context_from_tradingview}}

Trả về JSON với cấu trúc sau (KHÔNG có text khác):
{
  "bias": "...",
  "bias_type": "bearish|bullish|neutral",
  "summary": "...",
  "catalyst": "...",
  "session_quote": "...",
  "key_levels": { "resistance": [n1,n2], "support": [n1,n2], "current": n },
  "timeframes": { "D1": "...", "H4": "...", "H1": "...", "M15": "..." },
  "scenarios": [
    {"id":"S1","name":"...","prob":"50%","zone":"..."},
    {"id":"S2","name":"...","prob":"20%","zone":"..."},
    {"id":"S3","name":"...","prob":"20%","zone":"..."},
    {"id":"S4","name":"...","prob":"10%","zone":"..."}
  ]
}
```

---

## Bảo mật (tùy chọn)

Dashboard hiện là public. Nếu muốn private:
1. **Cloudflare Pages + Access**: Deploy lên Cloudflare Pages, bật Cloudflare Access
   → Đăng nhập bằng email, hoàn toàn miễn phí cho 1 người dùng
2. **Password đơn giản**: Thêm localStorage check vào index.html
   → Không an toàn cao nhưng đủ cho dùng cá nhân

---

## Auto-refresh

Web app tự động fetch lại `reports.json` mỗi **5 phút**.
Khi Make.com push báo cáo mới, sau tối đa 5 phút sẽ xuất hiện trong dashboard.

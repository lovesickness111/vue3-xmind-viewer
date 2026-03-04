# XMind Web Preview

Ứng dụng Vue 3 để **xem và chỉnh sửa file `.xmind`** trực tiếp trên trình duyệt, không cần cài đặt phần mềm. Hỗ trợ 2 chế độ viewer để so sánh chất lượng.

## ✨ Tính năng

### Viewer ① — simple-mind-map
- **Xem & chỉnh sửa** sơ đồ tư duy (click đúp để sửa node, kéo thả)
- **Zoom / Pan** tự do
- **Xuất PNG** chất lượng cao
- **Tải lại file `.xmind`** gốc
- Chạy hoàn toàn **offline**, không cần internet

### Viewer ② — xmind-embed-viewer (chính thức từ XMind Ltd)
- Render trung thực 100% theo định dạng XMind gốc
- Hỗ trợ đầy đủ **theme, font, style** của XMind
- Chuyển qua lại nhiều **sheet**
- **Chỉ xem** (read-only) — yêu cầu kết nối internet

### Giao diện chung
- Drag & drop hoặc browse để mở file `.xmind`
- File mẫu có sẵn ("Preview Sample Map")
- Modal popup với animation mượt mà
- Selector để chọn viewer trước khi mở

---

## 🚀 Cài đặt & Chạy

```bash
# Cài dependencies
npm install

# Chạy dev server
npm run dev
```

Mở trình duyệt tại: **http://localhost:5173/**

---

## 🏗️ Cấu trúc dự án

```
Preview.Xmind/
├── public/
│   └── co_cau_to_chuc.xmind   # File mẫu
├── src/
│   ├── components/
│   │   ├── XMindViewer.vue     # Viewer #1: simple-mind-map (có edit)
│   │   ├── EmbedViewer.vue     # Viewer #2: xmind-embed-viewer (chính thức)
│   │   └── XMindModal.vue      # Modal bao bọc viewer
│   ├── App.vue                 # Trang chính + tab chọn viewer
│   ├── main.ts
│   └── style.css
├── vite.config.ts
└── package.json
```

---

## 📦 Dependencies

| Package | Mục đích |
|---------|----------|
| `simple-mind-map` | Core viewer + editor cho Viewer #1 |
| `xmind-embed-viewer` | Official embed viewer từ XMind Ltd |
| `vite-plugin-node-polyfills` | Polyfill `Buffer`, `stream` cho browser |
| `stream-browserify` | Alias `stream` module |

---

## ⚖️ So sánh 2 Viewer

| Tính năng | simple-mind-map | xmind-embed-viewer |
|-----------|:---:|:---:|
| Xem file `.xmind` | ✅ | ✅ |
| Chỉnh sửa node | ✅ | ❌ |
| Export PNG | ✅ | ❌ |
| Chuyển sheet | ❌ | ✅ |
| Render chính xác 100% | Tương đối | ✅ |
| Offline | ✅ | ❌ |

---

## 📝 Ghi chú

- `xmind-embed-viewer` yêu cầu kết nối internet vì nó load renderer từ `xmind.app`
- Dev server yêu cầu **Node.js >= 20.19** (hiện tại đang dùng 20.17 — warn nhưng vẫn chạy được)

# XMind Web Preview

Ứng dụng Vue 3 cho phép **xem, chỉnh sửa và re-export file `.xmind`** trực tiếp trên trình duyệt — không cần cài phần mềm. Tích hợp 3 thư viện viewer để so sánh.

# Link sử dụng Online

https://vue3-xmind-viewer.vercel.app/

## 🚀 Cài đặt & Chạy

```bash
npm install
npm run dev
# Mở http://localhost:5173/
```

---

## ✨ Viewer ① — mind-elixir ⭐ Recommended

> Editor hiện đại, hỗ trợ **đọc direction từ file XMind gốc** và **giữ nguyên direction khi export**.

### Tính năng
- Xem & chỉnh sửa đầy đủ (click đúp, kéo thả, thêm/xóa node)
- **Direction-aware import**: tự động detect layout (LEFT / RIGHT / SIDE) từ file XMind gốc
- **Direction-aware export**: export `.xmind` giữ đúng hướng layout hiện tại
- Export SVG chất lượng cao
- Offline hoàn toàn

### Cơ chế import direction

File `.xmind` là một **ZIP archive** chứa `content.json`. Layout direction được lưu tại:

```
content.json[0].rootTopic.structureClass
```

| XMind `structureClass` | Mind Elixir direction |
|---|---|
| `org.xmind.ui.logic.right` | `MindElixir.RIGHT` (1) |
| `org.xmind.ui.logic.left` | `MindElixir.LEFT` (0) |
| `org.xmind.ui.map.unbalanced` | `MindElixir.SIDE` (2) |
| `org.xmind.ui.map.clockwise` | `MindElixir.SIDE` (2) |

> **Lưu ý kỹ thuật**: XMind Desktop cũng ghi direction vào `extensions[].content.mainTopic`, nhưng đây là **style hint cho nhánh**, không phải direction tổng thể. Chỉ `rootTopic.structureClass` là authoritative.

### Cơ chế export direction

`@mind-elixir/export-xmind` hardcode `structureClass = "logic.right"`. Sau khi plugin tạo Blob, ta **unzip → patch 2 trường → re-zip**:

```
content[0].rootTopic.structureClass  →  ME direction → XMind structureClass
extensions[].content.mainTopic       →  ME direction → XMind structureClass
```

### Luồng xử lý đầy đủ

```
.xmind file (ZIP)
  ├─ [1] Đọc content.json → detect rootTopic.structureClass → ME direction
  ├─ [2] simple-mind-map parser → SMM node tree
  ├─ [3] smmNodeToME() converter → ME node format
  └─ [4] MindElixir.init({ nodeData, direction })

[User chỉnh sửa]

Export .xmind:
  ├─ [1] mind.exportXmind() → Blob (direction hardcoded sai)
  ├─ [2] JSZip unzip → patch structureClass theo direction thực tế
  └─ [3] Re-zip → download
```

---

## Viewer ② — simple-mind-map

Canvas-based viewer/editor.

- Xem & chỉnh sửa đầy đủ
- Export PNG chất lượng cao
- Export file `.xmind` gốc (download lại file đã upload)
- Offline hoàn toàn

---

## Viewer ③ — xmind-embed-viewer (Official)

Iframe viewer chính thức từ XMind Ltd.

- Render trung thực 100% — chính xác nhất về màu sắc, font, theme
- Hỗ trợ chuyển sheet
- **Read-only** — không cho phép chỉnh sửa
- ⚠️ Yêu cầu kết nối internet (load renderer từ `xmind.app`)

---

## ⚖️ So sánh 3 Viewer

| Tính năng | mind-elixir ⭐ | simple-mind-map | xmind-embed-viewer |
|---|:---:|:---:|:---:|
| Xem file `.xmind` | ✅ | ✅ | ✅ |
| Chỉnh sửa node | ✅ | ✅ | ❌ |
| Giữ direction khi import | ✅ | ❌ | ✅ |
| Export `.xmind` (có direction) | ✅ | ❌ | ❌ |
| Export PNG/SVG | ✅ SVG | ✅ PNG | ❌ |
| Chuyển sheet | ❌ | ❌ | ✅ |
| Render chính xác 100% | Tốt | Khá | ✅ Tuyệt đối |
| Offline | ✅ | ✅ | ❌ |

---

## 🏗️ Cấu trúc dự án

```
Preview.Xmind/
├── public/
│   └── co_cau_to_chuc.xmind   # File mẫu
├── src/
│   ├── components/
│   │   ├── MindElixirViewer.vue  # Viewer ① (recommended)
│   │   ├── XMindViewer.vue       # Viewer ②
│   │   ├── EmbedViewer.vue       # Viewer ③
│   │   └── XMindModal.vue        # Modal popup
│   └── App.vue                   # Trang chính + tab selector
└── vite.config.ts
```

## 📦 Dependencies

| Package | Mục đích |
|---|---|
| `mind-elixir` | Viewer/editor chính (Viewer ①) |
| `@mind-elixir/export-xmind` | Export `.xmind` cho mind-elixir |
| `simple-mind-map` | Viewer ② + XMind parser dùng chung |
| `xmind-embed-viewer` | Official embed viewer (Viewer ③) |
| `vite-plugin-node-polyfills` | Polyfill `Buffer`, `stream` cho browser |

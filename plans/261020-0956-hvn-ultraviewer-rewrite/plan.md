# Plan — HVN UltraViewer for Mac v1.0.0

- Trạng thái: **Phase 4 — chờ CI xanh** (Phase 0–3 xong)
- Phạm vi: [brainstorm-261020-0956-hvn-ultraviewer.md](../reports/brainstorm-261020-0956-hvn-ultraviewer.md) (Cổng ① đã chốt)
- Nhánh: `claude/clever-turing-ns95e6` → PR nháp vào `main`

| Phase | Việc | Trạng thái |
|:-:|---|---|
| 0–1 | Brief, chốt tên/bundle ID/phạm vi/ký số, tạo `main` | Xong |
| 2 | Khung: `Package.swift`, `version.conf`, scripts, workflow Build, docs, CHANGELOG, SECURITY, mẫu GitHub | Xong |
| 3 | Chuyển engine + app từ MacViewer; đổi định danh `vn.hvn.ultraviewer`; viết lại vỏ app (Launcher, menu, Giới thiệu, Trợ giúp, Cài đặt), tiếng Việt mặc định, brand (icon, logo, màu), kênh cập nhật HVN | Xong |
| 4 | **Cổng ② CI**: `swift test`, đóng gói universal, `verify-bundle.sh`, DMG/zip/SHA-256 | Đang chờ |
| 5 | **Cổng ③ Máy thật**: chủ sản phẩm chạy `docs/manual-test-checklist.md` | Chưa |
| 6 | **Cổng ④ Phát hành** v1.0.0: PR phát hành riêng, Release trên GitHub UI | Chưa |

## Tiêu chí nghiệm thu

Theo mục *Tiêu chí nghiệm thu* của báo cáo phạm vi (1–8). Mục 1–6 chỉ được đánh dấu sau Cổng ③.

## Rủi ro

- Lỗi biên dịch Swift 6 chỉ lộ trên CI (agent không có Xcode) → sửa theo log, rà mẫu tương tự.
- Pháp lý: tên "UltraViewer", `8bit.dat` — chờ ý kiến pháp chế trước khi phát hành ra ngoài.

## Hoàn tác

Mọi thay đổi nằm trong PR; `main` chỉ có báo cáo phạm vi. Đóng PR là hoàn tác.

---

HVN GROUP · hvn.vn

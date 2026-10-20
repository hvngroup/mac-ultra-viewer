# Plan — Giao thức riêng của HVN và HVN Remote Hub đa nền tảng

- Trạng thái: **Phase 0 — chờ chủ sản phẩm chọn hướng** (xem *Câu hỏi chặn quyết định*)
- Yêu cầu (2026-10-20): đổi tên thành HVN Remote Hub; hub làm ở 1.1.0; tham khảo RustDesk để có giao thức riêng cho HVN;
  sau đó làm bản Windows, iOS, Android.
- Liên quan: [research-261020-1221-universal-remote-feasibility.md](../reports/research-261020-1221-universal-remote-feasibility.md)

## Lộ trình đề xuất

| Bản | Nội dung | Nền tảng |
|---|---|---|
| **1.0.0** | HVN Remote Hub for Mac: engine UltraViewer gốc, brand HVN, tiếng Việt mặc định | macOS |
| **1.1.0** | Hub: danh bạ máy đa nền; mở TeamViewer, AnyDesk (client chính hãng), RDP, VNC; nhận biết app đã cài | macOS |
| **2.0.0** | **HVN Connect** — giao thức riêng: máy chủ ID/relay của HVN đặt trên HVN Cloud, kết nối mã hoá đầu-cuối; app Mac điều khiển và được điều khiển qua HVN Connect | macOS + máy chủ |
| **2.x** | Bản Windows, Android, iOS dùng chung lõi HVN Connect; hub trên từng nền tảng | Windows, Android, iOS |

## RustDesk — những gì tham khảo được (đã kiểm tra mã nguồn 2026-10-20)

- **Giấy phép:** `rustdesk/rustdesk` và `rustdesk/rustdesk-server` đều **AGPL-3.0**. Phân phối bản sửa đổi, hoặc chạy
  máy chủ đã sửa cho người khác dùng qua mạng, thì phải công bố toàn bộ mã nguồn tương ứng theo AGPL. Thương hiệu
  "RustDesk" không đi kèm giấy phép mã nguồn.
- **Kiến trúc:** lõi Rust; giao diện **Flutter** cho cả desktop lẫn mobile (Windows, macOS, Linux, Android, iOS).
  - `libs/hbb_common`: codec, cấu hình, TCP/UDP, protobuf dùng chung với máy chủ.
  - `libs/scrap`: chụp màn hình. `libs/enigo`: bàn phím/chuột. `libs/clipboard`: clipboard và file.
  - `src/rendezvous_mediator.rs`: làm việc với máy chủ.
- **Giao thức:** protobuf (`rendezvous.proto`: `RegisterPeer`, `PunchHoleRequest`, `RequestRelay`, `KeyExchange`, …;
  `message.proto` cho phiên). Mã hoá bằng libsodium (`sodiumoxide`). Kết nối thẳng bằng TCP hole punching, không được thì
  qua relay.
- **Máy chủ:** `hbbs` (ID/rendezvous) + `hbbr` (relay), tự host được (bản OSS). Bản Server Pro (trả phí) có web console và
  công cụ tạo client mang thương hiệu riêng.

## Các hướng làm "giao thức riêng của HVN"

| | **A. Fork RustDesk (AGPL)** | **B. Tự viết HVN Connect (tham khảo thiết kế)** | **C. Mua RustDesk Server Pro** |
|---|---|---|---|
| Cách làm | Fork client + server, đổi thương hiệu, máy chủ đặt trên HVN Cloud, sửa/bổ sung giao thức | Lõi Rust mới (protobuf, Noise/libsodium, rendezvous/relay, codec phần cứng), viết sạch, không chép mã RustDesk | License Pro + công cụ tạo client mang thương hiệu HVN, máy chủ tự host |
| Có Windows/Android/iOS | **Ngay** (RustDesk đã chạy trên cả 5 nền tảng) | Phải tự làm từng nền tảng | Ngay |
| "Giao thức riêng" | Có: HVN kiểm soát mã và máy chủ; tương thích ngược với RustDesk nếu muốn | Có, hoàn toàn của HVN | Không: vẫn là giao thức và client của RustDesk |
| Giấy phép mã HVN | **Phải mở mã** (AGPL) cho app và máy chủ đã sửa | Tự chọn (có thể đóng) | Theo điều khoản thương mại của RustDesk |
| Công sức (ước lượng thô) | 1–2 tháng cho bản đổi thương hiệu đầu tiên trên các nền tảng; cập nhật theo upstream liên tục | 6–12 tháng cho mức tương đương; rủi ro kỹ thuật cao (NAT, codec, quyền hệ điều hành trên từng nền tảng) | Vài tuần |
| Rủi ro chính | Nghĩa vụ AGPL; theo kịp upstream | Thời gian, chất lượng, bảo mật tự làm | Phụ thuộc nhà cung cấp; không phải giao thức của HVN |

**Khuyến nghị: A.** Đây là cách duy nhất có giao thức và máy chủ của HVN trên cả 4 nền tảng trong vài tháng. Repo này đã
công khai, nên việc mở mã theo AGPL ít thay đổi so với hiện trạng. Phần đóng góp của HVN nằm ở máy chủ trên HVN Cloud, tích
hợp WHMCS/HVN Works, dịch vụ hỗ trợ và thương hiệu. B chỉ đáng làm khi HVN bắt buộc giữ mã nguồn đóng. C hợp nếu cần ra
mắt gấp và chấp nhận không có giao thức riêng.

### Hệ quả của A với app Mac hiện tại

- App Mac (Swift) giữ vai trò hub + engine UltraViewer (1.0–1.1).
- Ở 2.0.0, có hai cách đưa HVN Connect vào Mac:
  - **A1 (khuyến nghị):** bản fork Flutter trở thành HVN Remote Hub trên **mọi** nền tảng. App Swift hiện tại chuyển
    thành mô-đun UltraViewer chỉ dành cho macOS (trên Windows đã có UltraViewer chính hãng để hub mở). Một codebase giao
    diện cho 5 nền tảng.
  - **A2:** giữ app Swift trên Mac, gọi lõi Rust qua FFI; trên Windows/Android/iOS dùng bản fork Flutter. Trải nghiệm Mac
    gốc hơn, nhưng phải duy trì hai giao diện.
- Phần mã MIT hiện có (MacViewer, phần HVN viết) gộp được vào dự án AGPL (MIT tương thích một chiều với AGPL).

## Câu hỏi chặn quyết định (Cổng ① của 2.0.0)

1. **Giấy phép:** HVN chấp nhận mở mã nguồn app và máy chủ theo AGPL-3.0 (hướng A) không, hay bắt buộc mã đóng (hướng B)?
2. **Khách hàng mục tiêu:** đội hỗ trợ nội bộ của HVN, hay bán cho khách như một dịch vụ (gói HVN Cloud/HVN Security)?
   Câu trả lời quyết định quy mô máy chủ, cách tính phí và yêu cầu kiểm thử bảo mật (VAPT).
3. **Giao diện Mac từ 2.0.0:** A1 (một codebase Flutter cho mọi nền tảng) hay A2 (giữ Swift trên Mac)?
4. **Hạ tầng:** đặt máy chủ ID/relay ở đâu (HVN Cloud tại Việt Nam, GCP, Azure), có cần nhiều vùng không?
5. **Ký số:** Windows (chứng chỉ code signing EV, HVN Security đã phân phối Sectigo/DigiCert/GlobalSign), Apple Developer
   (bắt buộc cho iOS và notarize macOS), Google Play.

## Việc kế tiếp sau khi chốt

1. Brainstorm chi tiết 2.0.0 theo `ab-app-sop` (Outcome · Constraints · Non-goals · Acceptance).
2. Bản thử nghiệm: dựng `hbbs`/`hbbr` trên HVN Cloud và kết nối hai máy bằng client RustDesk gốc, để đo độ trễ và tỉ lệ
   kết nối thẳng/relay ở Việt Nam trước khi đầu tư vào fork.
3. Repo mới cho bản fork (ví dụ `hvngroup/remote-hub`) và repo máy chủ (`hvngroup/remote-hub-server`).

---

HVN GROUP · hvn.vn

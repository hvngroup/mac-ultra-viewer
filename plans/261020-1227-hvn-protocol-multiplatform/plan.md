# Plan — Giao thức riêng của HVN và HVN Remote Hub đa nền tảng

- Trạng thái: **Đã chọn hướng A** (2026-10-20); còn 2 câu hỏi mở trước khi bắt đầu fork (xem *Quyết định đã chốt*)
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

## Quyết định đã chốt (Cổng ① của 2.0.0)

| Câu hỏi | Trả lời của chủ sản phẩm (2026-10-20) |
|---|---|
| Giấy phép | **Chấp nhận AGPL-3.0**: công bố mã nguồn app và máy chủ đã sửa |
| Hướng kỹ thuật | **A — fork RustDesk** (client + server), đổi thương hiệu thành HVN Remote Hub |
| Hạ tầng | Máy chủ ID/relay trên **HVN Cloud tại Việt Nam** |
| Ký số Apple | HVN **đã có Apple Developer** → ký Developer ID + notarize cho macOS, phát hành iOS qua App Store/TestFlight |
| Khách hàng mục tiêu | *Còn mở* — nội bộ hay bán như dịch vụ |
| Giao diện Mac từ 2.0.0 | *Còn mở* — A1 (Flutter cho mọi nền tảng, khuyến nghị) hay A2 (giữ Swift) |

## Các phase của 2.0.0

| Phase | Việc | Ai làm | Đầu ra |
|:-:|---|---|---|
| P0 | Máy chủ thử nghiệm trên HVN Cloud: `hbbs` + `hbbr`, DNS, khoá ký máy chủ; đo độ trễ và tỉ lệ kết nối thẳng/relay bằng client RustDesk gốc | Hạ tầng HVN Cloud (runbook: [phase-01-server-pilot.md](phase-01-server-pilot.md)) | Máy chủ chạy, số đo |
| P1 | Tạo fork `hvngroup/remote-hub` (từ `rustdesk/rustdesk`), `hvngroup/remote-hub-common` (từ `rustdesk/hbb_common`), `hvngroup/remote-hub-server` (từ `rustdesk/rustdesk-server`); thêm vào phiên agent | Chủ sản phẩm (GitHub UI) | 3 repo fork |
| P2 | Đổi thương hiệu: `APP_NAME`, `ORG` (`vn.hvn`), `RENDEZVOUS_SERVERS`, `RS_PUB_KEY`, bundle/application ID `vn.hvn.remotehub` trên 5 nền tảng, icon + logo HVN, tiếng Việt mặc định, gỡ link cập nhật/tài trợ của RustDesk, ghi công + thông báo AGPL | Agent | Client nói chuyện với máy chủ HVN |
| P3 | CI cho từng nền tảng: macOS (ký Developer ID + notarize), Windows (ký code signing), Android (APK/AAB), iOS (TestFlight) | Agent + chủ sản phẩm (secrets) | Bản build ký số trên 5 nền tảng |
| P4 | Hub + UltraViewer: tích hợp theo A1 hoặc A2 | Agent | Một app HVN Remote Hub |
| P5 | Cổng ③ trên máy thật từng nền tảng; VAPT bởi HVN Security nếu bán ra ngoài | Chủ sản phẩm, HVN Security | Biên bản kiểm thử |
| P6 | Phát hành 2.0.0: GitHub Releases, App Store, Google Play, công bố mã nguồn theo AGPL | Chủ sản phẩm | Bản phát hành |

## Việc chủ sản phẩm cần chuẩn bị

1. **Ngay cho bản 1.0.0:** thêm secrets Developer ID vào repo này (`MACOS_CERTIFICATE_P12`, `MACOS_CERTIFICATE_PASSWORD`,
   `APPSTORE_API_KEY_P8`, `APPSTORE_API_KEY_ID`, `APPSTORE_API_ISSUER_ID`; xem `docs/release.md`). Ký Developer ID ngay từ
   bản đầu thì người dùng không phải "Open Anyway", app tự cài bản cập nhật, quyền macOS không bị hỏi lại sau mỗi lần cập
   nhật.
2. P0: một VPS trên HVN Cloud (gợi ý 2 vCPU, 4 GB RAM, băng thông không giới hạn hoặc lớn, IP tĩnh) và một tên miền con,
   ví dụ `connect.hvn.vn`.
3. P1: tạo 3 repo fork rồi báo agent để thêm vào phiên.
4. P3: chứng chỉ code signing cho Windows (EV hoặc OV, qua HVN Security/Sectigo), tài khoản Google Play Console.

## Câu hỏi còn mở

- Khách hàng mục tiêu: nội bộ hay bán như dịch vụ? Câu trả lời quyết định quy mô máy chủ, mô hình tính phí và VAPT.
- Giao diện Mac từ 2.0.0: A1 hay A2?
- Tên miền cho máy chủ (`connect.hvn.vn` hay tên khác)?

---

HVN GROUP · hvn.vn

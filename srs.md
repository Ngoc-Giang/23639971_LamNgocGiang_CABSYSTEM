# SRS — Nền tảng đặt xe CAB

## Bước 1: Đọc và phân tích yêu cầu khách hàng ở giai đoạn sơ khởi (Business Context & Business Problem)

### 1.1 Ngữ cảnh nghiệp vụ (Business Context)

Công ty ABC cung cấp dịch vụ đặt xe trực tuyến. Hiện trạng: khách đặt xe qua **tổng đài** hoặc một **ứng dụng đơn giản**, điều phối phía sau **chủ yếu thủ công**. Ban lãnh đạo không muốn “vá” ứng dụng cũ mà xây một **nền tảng CAB mới**, phục vụ được số lượng lớn khách hàng và tài xế, và **mở rộng thêm tính năng trong tương lai** (loại dịch vụ, phương thức thanh toán, kênh thông báo…) mà không phải xây lại toàn bộ.

Các yếu tố định hình ngữ cảnh:

- **Ba nhóm người dùng chính:** Khách hàng, Tài xế, Nhân viên vận hành — là các tác nhân trung tâm.
- **Phụ thuộc bên ngoài:** nhà cung cấp thanh toán và (tương lai) nhà cung cấp thông báo — hệ thống tích hợp chứ không tự xử lý mọi thứ.

### 1.2 Vấn đề nghiệp vụ (Business Problem)

Vấn đề: hệ thống hiện tại vừa thủ công vừa thiếu tính module/khả năng mở rộng. Đây là lý do tài liệu nhấn mạnh yêu cầu phi chức năng: các thành phần (thanh toán, thông báo…) phải mở rộng độc lập, và một lỗi ở thanh toán/thông báo không được làm sập luồng đặt xe. Bài toán không chỉ là “làm app đặt xe” mà là “làm một nền tảng chịu tải, tách rời và dễ tiến hóa”.

| #  | Vấn đề hiện tại                      | Hệ quả nghiệp vụ                                                   |
|-----|--------------------------------------|--------------------------------------------------------------------|
| 1   | Phân công tài xế thủ công            | Chậm, dễ sai, không phục vụ được số lượng lớn, phụ thuộc con người |
| 2   | Khách khó theo dõi trạng thái chuyến | Trải nghiệm kém, gọi tổng đài nhiều, mất niềm tin                  |
| 3   | Thông tin thanh toán không tập trung | Khó đối soát, khó báo cáo doanh thu, khó kiểm toán                 |
| 4   | Bộ phận vận hành khó mở rộng         | Không chịu tải lúc cao điểm, tăng trưởng bị nghẽn                  |
| 5   | Khó thêm tính năng mới               | Kiến trúc cứng, mỗi thay đổi ảnh hưởng toàn hệ thống               |

## Bước 2: Xác định stakeholder & vẽ ma trận stakeholder

| Tên Stakeholder                                    | Vai trò trong hệ thống                                                                                            |
|----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| **Khách hàng (Customer)**                          | Đăng ký/đăng nhập, nhập điểm đón–đến, chọn loại xe, gửi yêu cầu, theo dõi chuyến, thanh toán, đánh giá tài xế.    |
| **Tài xế (Driver)**                                | Cập nhật hồ sơ/phương tiện, chuyển trạng thái sẵn sàng, nhận/từ chối chuyến, cập nhật tiến trình và vị trí.       |
| **Nhân viên vận hành (Operations Staff)**          | Quản trị khách hàng, tài xế, phương tiện, chuyến; theo dõi chuyến đang chạy, xử lý chuyến lỗi, tra cứu giao dịch. |
| **Quản trị viên hệ thống (System Admin)**          | Phân quyền, cấu hình, thao tác nhạy cảm, quản lý lưu vết (audit) và an toàn hệ thống.                             |
| **Ban lãnh đạo (Executive)**                       | Đặt tầm nhìn, phê duyệt phạm vi & ngân sách, theo dõi báo cáo doanh thu, tỷ lệ hoàn thành/hủy, hiệu quả tài xế.   |
| **Tài chính – Kế toán (Finance)**                  | Đối soát thanh toán, quản lý doanh thu tập trung, lập báo cáo tài chính.                                          |
| **Chăm sóc khách hàng (Customer Support)**         | Hỗ trợ khách qua hotline, tiếp nhận & xử lý khiếu nại, hỗ trợ sự cố chuyến.                                       |
| **Business Analyst (BA)**                          | Thu thập & làm rõ yêu cầu, xác định phạm vi/quy tắc/ngoại lệ, cầu nối giữa khách hàng và nhóm phát triển.         |
| **Nhóm phát triển (Dev Team)**                     | Thiết kế kiến trúc, xây dựng, tích hợp các thành phần.                                                            |
| **QA / Kiểm thử (Tester)**                         | Kiểm thử chức năng & phi chức năng, đảm bảo chất lượng, độ ổn định lúc cao điểm.                                  |
| **Nhà cung cấp thanh toán (Payment Provider)**     | Đối tác ngoài xử lý giao dịch điện tử; CAB **không lưu** thông tin thẻ/tài khoản nhạy cảm.                        |
| **Nhà cung cấp thông báo (Notification Provider)** | Đối tác ngoài gửi SMS/Push/Email; kiến trúc phải cho phép mở rộng kênh sau này.                                   |

```mermaid
quadrantChart
    title Ma trận Stakeholder - CAB System
    x-axis Interest thap --> Interest cao
    y-axis Power thap --> Power cao
    quadrant-1 Manage Closely
    quadrant-2 Keep Satisfied
    quadrant-3 Monitor
    quadrant-4 Keep Informed
    Ban lanh dao: [0.78, 0.92]
    System Admin: [0.60, 0.78]
    Business Analyst: [0.82, 0.72]
    NV Van hanh: [0.70, 0.64]
    Nhom phat trien: [0.80, 0.56]
    NCC Thanh toan: [0.28, 0.72]
    Tai chinh - Ke toan: [0.34, 0.60]
    QA Tester: [0.58, 0.44]
    Tai xe: [0.72, 0.36]
    Khach hang: [0.85, 0.30]
    Cham soc KH: [0.50, 0.26]
    NCC Thong bao: [0.20, 0.20]
```

## Bước 3: Xác định business goal (mục đích)

| Mã       | Business Goal                                   | Mục đích                                                                                                                                                                                                        |
|----------|-------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **BG01** | Giảm thời gian tìm tài xế và tối ưu ghép tài xế | Tự động tìm & ghép tài xế phù hợp theo vị trí và trạng thái sẵn sàng (thay phân công thủ công); ưu tiên tài xế gần & phù hợp và tự chuyển tài xế khác khi bị từ chối/không phản hồi mà không bắt khách tạo lại. |
| **BG02** | Hỗ trợ thanh toán linh hoạt                     | Cho phép thanh toán tiền mặt hoặc trực tuyến, tích hợp cổng ngoài mà không lưu dữ liệu thẻ nhạy cảm.                                                                                                            |
| **BG03** | Tăng khả năng theo dõi chuyến đi                | Cho khách theo dõi trạng thái chuyến: đang tìm tài xế, tài xế đã nhận, ETA, tiến trình chuyến.                                                                                                                  |
| **BG04** | Quản lý thanh toán & doanh thu tập trung        | Tập trung dữ liệu giao dịch để đối soát, xác định cước theo loại dịch vụ, báo cáo doanh thu chính xác.                                                                                                          |
| **BG05** | Thông báo đa kênh, dễ mở rộng                   | Gửi thông báo cho khách & tài xế ở các mốc quan trọng; kiến trúc cho phép thêm kênh mới.                                                                                                                        |
| **BG06** | Cung cấp công cụ vận hành & quản trị            | Giao diện quản trị để quản lý khách/tài xế/phương tiện/chuyến và xử lý chuyến lỗi.                                                                                                                              |
| **BG07** | Hỗ trợ ra quyết định bằng báo cáo               | Báo cáo số chuyến, doanh thu, tỷ lệ hoàn thành/hủy, hiệu quả tài xế cho ban lãnh đạo.                                                                                                                           |
| **BG08** | Đảm bảo độ ổn định khi tải cao                  | Giữ hệ thống ổn định lúc cao điểm; lỗi thanh toán/thông báo không làm sập luồng đặt xe.                                                                                                                         |
| **BG09** | Bảo mật & phân quyền                            | Xác thực người dùng, kiểm soát quyền cho thao tác quản trị, bảo vệ dữ liệu cá nhân/vị trí/giao dịch.                                                                                                            |
| **BG10** | Lưu vết phục vụ kiểm tra                        | Ghi log thao tác quan trọng để truy vết và điều tra khi có sự cố.                                                                                                                                               |

## Bước 4: Xác định phạm vi (scope)

### A. Trong phạm vi – Module cho MVP

| Mã      | Module                  | Chức năng cốt lõi                                                                                    |
|---------|-------------------------|------------------------------------------------------------------------------------------------------|
| **M01** | Quản lý khách hàng      | Đăng ký, đăng nhập, cập nhật thông tin, xem lịch sử chuyến.                                          |
| **M02** | Quản lý tài xế          | Tạo/đăng ký tài khoản, cập nhật hồ sơ & phương tiện, chuyển trạng thái sẵn sàng.                     |
| **M03** | Đặt xe (Booking)        | Nhập điểm đón–đến, chọn loại xe, gửi yêu cầu, tạo chuyến.                                            |
| **M04** | Ghép & phân công tài xế | Tự tìm tài xế theo vị trí & trạng thái; chuyển tài xế khác khi bị từ chối/không phản hồi.            |
| **M05** | Quản lý chuyến đi       | Cập nhật trạng thái: đã đến điểm đón, đã đón khách, đang di chuyển, hoàn thành.                      |
| **M06** | Tính cước & Thanh toán  | Tính tiền theo loại dịch vụ; thanh toán tiền mặt hoặc điện tử qua cổng ngoài; xử lý thất bại.        |
| **M07** | Thông báo               | Gửi thông báo cho khách & tài xế tại các mốc chính.                                                  |
| **M08** | Quản trị & vận hành     | Giao diện nhân viên quản lý khách/tài xế/phương tiện/chuyến; xem chuyến đang chạy, xử lý chuyến lỗi. |
| **M09** | Xác thực & phân quyền   | Xác thực người dùng; kiểm soát quyền cho thao tác quản trị.                                          |
| **M10** | Đánh giá sau chuyến     | Cho khách đánh giá tài xế sau khi hoàn thành chuyến.                                                 |

**Mức ưu tiên (đã điều chỉnh để khớp luồng end-to-end):**

- **Must-have (bắt buộc cho demo):** M01, M02, M03, M04, M05, **M06 (nhánh tiền mặt)**, M07, M09.
- **Should-have:** M06 (nhánh điện tử), M08 (bản tối giản), M10.

> **Ghi chú:** M06 nhánh điện tử làm ở dạng **mock provider** (trả success/fail để test), M07 **ghi log/in ra** thay vì tích hợp SMS/Push thật, M08 chỉ làm màn hình **xem danh sách chuyến + trạng thái tài xế** tối thiểu. Chi tiết ở Bước 4c

### B. Ngoài phạm vi ở MVP

| Hạng mục ngoài phạm vi                             | Lý do chưa làm                                                                              |
|----------------------------------------------------|---------------------------------------------------------------------------------------------|
| Báo cáo & phân tích nâng cao (BG07)                | Kỳ vọng dài hạn, không thuộc luồng lõi; để phase sau.                                       |
| Bản đồ định vị realtime & ETA chính xác            | Cần dữ liệu vị trí liên tục + tính toán phức tạp; MVP chỉ lưu vị trí cơ bản để ghép tài xế. |
| Đa dạng loại dịch vụ / nhiều gói cước phức tạp     | Cách tính cước dùng công thức mặc định; chỉ làm 1–2 loại cơ bản.                            |
| Tích hợp nhiều nhà cung cấp thanh toán / thông báo | MVP: 1 cổng thanh toán (mock) + 1 kênh thông báo (log); kiến trúc chừa chỗ mở rộng.         |
| Khuyến mãi, mã giảm giá, ví nội bộ, điểm thưởng    | Không có trong yêu cầu → ngoài phạm vi.                                                     |
| App di động native hoàn chỉnh                      | MVP tập trung luồng nghiệp vụ & demo.                                                       |
| Auto-scaling / HA đầy đủ                           | MVP đảm bảo kiến trúc tách rời/module hóa; tối ưu tải thực làm ở giai đoạn scale-up.        |

### C. Chiến lược cắt phạm vi cho 7 tuần (Build vs. Mock)

| Thành phần         | Cách làm trong MVP                                                             | Vẫn nghiệm thu được gì                          |
|--------------------|--------------------------------------------------------------------------------|-------------------------------------------------|
| Thanh toán điện tử | **Mock provider** trả success/fail theo cấu hình                               | Cơ chế retry + fallback tiền mặt (EX03/AC10)    |
| Thông báo (M07)    | **Ghi log / in ra** ở các mốc, không gửi SMS/Push thật                         | Đúng thời điểm & nội dung thông báo (AC33–AC34) |
| Vị trí tài xế      | Nhập tay toạ độ / chọn từ danh sách, không GPS realtime                        | Ghi nhận vị trí phục vụ ghép tài xế (AC35)      |
| Admin (M08)        | Chỉ màn hình xem danh sách chuyến + trạng thái tài xế                          | Vận hành theo dõi & phát hiện chuyến lỗi (AC36) |
| Audit (BR20)       | Log cơ bản cho thao tác nhạy cảm                                               | Mỗi thao tác nhạy cảm sinh log (AC37)           |
| Hủy chuyến (BR21)  | Chỉ làm nhánh **hủy miễn phí khi đang tìm tài xế**; phí hủy để config, làm sau | Cơ chế hủy + áp chính sách (AC38–AC39)          |
| Offline (EX09)     | Grace 60s ở mức xử lý lại/đồng bộ đơn giản                                     | Không mất trạng thái khi rớt mạng ngắn (AC40)   |
| Retention (NFR13)  | Chỉ đặt tham số ở config; job dọn dữ liệu tự động làm sau                      | Giá trị lưu trữ có ở cấu hình (AC41)            |

## Bước 5: Chuyển thành business requirement (BR)

| Mã       | Tên Business Requirement                | Diễn giải                                                                                                     |
|----------|-----------------------------------------|---------------------------------------------------------------------------------------------------------------|
| **BR01** | Đăng ký & quản lý tài khoản khách hàng  | Khách đăng ký, đăng nhập, cập nhật thông tin, xem lịch sử chuyến.                                             |
| **BR02** | Đăng ký & quản lý tài khoản tài xế      | Tài xế tự đăng ký hoặc được nhân viên tạo, cập nhật hồ sơ & phương tiện.                                      |
| **BR03** | Quản lý trạng thái hoạt động tài xế     | Tài xế chuyển trạng thái sẵn sàng/không sẵn sàng khi đang làm việc.                                           |
| **BR04** | Đặt chuyến xe                           | Khách nhập điểm đón–đến, chọn loại xe, gửi yêu cầu để tạo chuyến.                                             |
| **BR05** | Tự động tìm & phân công tài xế          | Hệ thống tìm tài xế theo vị trí & trạng thái, ưu tiên tài xế gần khách.                                       |
| **BR06** | Xử lý khi tài xế từ chối/không phản hồi | Tự tìm tài xế khác không bắt khách tạo lại; hết tài xế thì thông báo rõ.                                      |
| **BR07** | Chấp nhận / từ chối chuyến              | Tài xế nhận thông báo chuyến mới, chấp nhận hoặc từ chối.                                                     |
| **BR08** | Theo dõi trạng thái chuyến đi           | Khách theo dõi: đang tìm tài xế, tài xế đã nhận, ETA, trạng thái hiện tại.                                    |
| **BR09** | Cập nhật tiến trình chuyến              | Tài xế cập nhật: đã đến điểm đón, đã đón khách, đang di chuyển, hoàn thành.                                   |
| **BR10** | Lưu vị trí tài xế                       | Lưu vị trí tài xế để tìm tài xế gần và ước lượng thời gian đến.                                               |
| **BR11** | Tính cước chuyến đi                     | Sau khi hoàn thành, xác định số tiền theo loại dịch vụ & thông tin chuyến.                                    |
| **BR12** | Thanh toán chuyến đi                    | Khách thanh toán tiền mặt hoặc điện tử; tích hợp provider ngoài, không lưu thẻ nhạy cảm.                      |
| **BR13** | Xử lý thanh toán thất bại               | Điện tử thất bại → thông báo khách & cho xử lý lại theo chính sách.                                           |
| **BR14** | Gửi thông báo cho khách hàng            | Thông báo tại các mốc: tiếp nhận, có tài xế, đến điểm đón, hoàn thành, kết quả thanh toán.                    |
| **BR15** | Gửi thông báo cho tài xế                | Thông báo chuyến mới & thay đổi của chuyến đang thực hiện.                                                    |
| **BR16** | Đánh giá tài xế sau chuyến              | Khách đánh giá tài xế sau khi chuyến hoàn thành.                                                              |
| **BR17** | Quản trị & vận hành                     | Nhân viên quản lý khách/tài xế/phương tiện/chuyến, xem chuyến đang chạy, xử lý chuyến lỗi, tra cứu giao dịch. |
| **BR18** | Phân quyền thao tác quản trị            | Chức năng nhạy cảm chỉ dành cho vai trò được cấp quyền.                                                       |
| **BR19** | Xác thực người dùng                     | Khách & tài xế phải xác thực trước khi dùng chức năng cần tài khoản.                                          |
| **BR20** | Lưu vết thao tác quan trọng (audit)     | Ghi log thao tác quan trọng phục vụ truy vết.                                                                 |
| **BR21** | Hủy chuyến                              | Khách (hoặc tài xế) hủy chuyến theo chính sách phí hủy & mốc cho phép hủy.                                    |

## Bước 6: Xây dựng business process

### BP01 – Đặt chuyến & tìm tài xế (quy trình lõi)

1.  Khách đăng nhập, nhập **điểm đón + điểm đến**, chọn **loại xe**.
2.  Gửi yêu cầu → hệ thống **tạo chuyến** (trạng thái: *đang tìm tài xế*).
3.  Hệ thống **xác nhận tiếp nhận** và thông báo cho khách.
4.  Hệ thống **tìm tài xế** theo vị trí, trạng thái sẵn sàng, ưu tiên gần khách (bán kính **1 km – default**).
5.  Gửi đề xuất tới tài xế → tài xế **chấp nhận hoặc từ chối** trong **20 giây (default)**.
6.  **Chấp nhận** → gán tài xế, thông báo khách (tài xế đã nhận + ETA).
7.  **Từ chối / hết giờ** → tự tìm tài xế kế tiếp (không bắt khách tạo lại).
8.  **Hết tài xế** → thông báo rõ “không tìm được tài xế”.

### BP02 – Thực hiện chuyến đi

1.  Tài xế đến điểm đón → cập nhật **“đã đến điểm đón”** → thông báo khách.
2.  Đón khách → **“đã đón khách”**.
3.  **“đang di chuyển”** suốt hành trình.
4.  Tới đích → **“hoàn thành chuyến”** → chuyển sang tính cước.

### BP03 – Tính cước & thanh toán

1.  Chuyến hoàn thành → **tính cước** = **12.000đ + 10.000đ/km (default)** theo loại dịch vụ.
2.  Khách chọn **tiền mặt** hoặc **điện tử**.
3.  Tiền mặt → xác nhận đã thanh toán.
4.  Điện tử → gọi **provider ngoài** (không lưu thẻ nhạy cảm).
    - Thành công → cập nhật kết quả, thông báo khách.
    - Thất bại → **thử lại tối đa 2 lần (default)**; vẫn lỗi → **chuyển tiền mặt** + thông báo khách.
5.  Sau thanh toán → khách **đánh giá tài xế**.

### BP04 – Quản trị & xử lý sự cố (vận hành)

1.  Nhân viên xem **danh sách chuyến đang diễn ra** và trạng thái tài xế.
2.  Khi có **chuyến lỗi**, kiểm tra và hỗ trợ xử lý.
3.  Tra cứu **lịch sử giao dịch** khi cần.
4.  Thao tác nhạy cảm phải qua **kiểm tra phân quyền**; mọi thao tác quan trọng được **lưu vết (audit)**.

## Bước 7: Phân rã yêu cầu nghiệp vụ (FR)

| BR       | FR   | Chức năng (mức hệ thống)                                                                                                                |
|----------|------|-----------------------------------------------------------------------------------------------------------------------------------------|
| **BR01** | FR01 | Đăng ký tài khoản khách hàng                                                                                                            |
|          | FR02 | Đăng nhập / cấp phiên                                                                                                                   |
|          | FR03 | Cập nhật thông tin cá nhân                                                                                                              |
|          | FR04 | Xem lịch sử chuyến đi                                                                                                                   |
| **BR02** | FR05 | Tạo tài khoản tài xế (tự đăng ký / NV tạo)                                                                                              |
|          | FR06 | Cập nhật hồ sơ tài xế                                                                                                                   |
|          | FR07 | Cập nhật thông tin phương tiện                                                                                                          |
| **BR03** | FR08 | Đổi trạng thái sẵn sàng / không sẵn sàng                                                                                                |
| **BR04** | FR09 | Nhập & chuẩn hoá điểm đón, điểm đến                                                                                                     |
|          | FR10 | Chọn loại xe / loại dịch vụ                                                                                                             |
|          | FR11 | Gửi yêu cầu & tạo chuyến (trạng thái *đang tìm tài xế*)                                                                                 |
| **BR05** | FR12 | Xác định vị trí khách hàng (điểm đón)                                                                                                   |
|          | FR13 | Lọc tài xế online/sẵn sàng trong **bán kính 1 km (default)**                                                                            |
|          | FR14 | Lọc tài xế theo loại xe khách chọn                                                                                                      |
|          | FR15 | Xếp ưu tiên: **gần nhất; hòa → đánh giá cao hơn (default)**                                                                             |
|          | FR16 | Chọn tài xế ứng viên đầu danh sách                                                                                                      |
| **BR06** | FR17 | Gửi đề xuất chuyến tới tài xế                                                                                                           |
|          | FR18 | Chờ phản hồi trong **timeout 20 giây (default)**                                                                                        |
|          | FR19 | Từ chối/hết giờ → chọn tài xế kế tiếp (không bắt khách tạo lại)                                                                         |
|          | FR20 | Hết tài xế → thông báo khách “không tìm được tài xế”                                                                                    |
| **BR07** | FR21 | Tài xế nhận thông báo chuyến mới                                                                                                        |
|          | FR22 | Tài xế chấp nhận / từ chối                                                                                                              |
|          | FR23 | Gán tài xế vào chuyến khi chấp nhận                                                                                                     |
| **BR08** | FR24 | Truy vấn trạng thái chuyến hiện tại                                                                                                     |
|          | FR25 | Cung cấp thời gian dự kiến đến (ETA)                                                                                                    |
| **BR09** | FR26 | Cập nhật “đã đến điểm đón”                                                                                                              |
|          | FR27 | Cập nhật “đã đón khách”                                                                                                                 |
|          | FR28 | Cập nhật “đang di chuyển”                                                                                                               |
|          | FR29 | Cập nhật “hoàn thành chuyến”                                                                                                            |
| **BR10** | FR30 | Ghi nhận & cập nhật vị trí tài xế                                                                                                       |
| **BR11** | FR31 | Tính cước = **12.000đ + 10.000đ/km (default)** theo loại dịch vụ                                                                        |
| **BR12** | FR32 | Cho khách chọn tiền mặt / điện tử                                                                                                       |
|          | FR33 | Xác nhận thanh toán tiền mặt                                                                                                            |
|          | FR34 | Gọi cổng thanh toán ngoài (không lưu thông tin thẻ)                                                                                     |
| **BR13** | FR35 | Nhận kết quả giao dịch (thành công/thất bại)                                                                                            |
|          | FR36 | Thất bại → **thử lại 2 lần → chuyển tiền mặt (default)**                                                                                |
| **BR14** | FR37 | TB khách: tiếp nhận yêu cầu                                                                                                             |
|          | FR38 | TB khách: có tài xế nhận                                                                                                                |
|          | FR39 | TB khách: tài xế đến điểm đón                                                                                                           |
|          | FR40 | TB khách: hoàn thành chuyến                                                                                                             |
|          | FR41 | TB khách: kết quả thanh toán                                                                                                            |
| **BR15** | FR42 | TB tài xế: chuyến mới                                                                                                                   |
|          | FR43 | TB tài xế: thay đổi của chuyến đang chạy                                                                                                |
| **BR16** | FR44 | Cho khách đánh giá tài xế sau chuyến                                                                                                    |
| **BR17** | FR45 | Quản lý (CRUD) khách/tài xế/xe/chuyến                                                                                                   |
|          | FR46 | Xem chuyến đang diễn ra + trạng thái tài xế                                                                                             |
|          | FR47 | Hỗ trợ xử lý chuyến bị lỗi                                                                                                              |
|          | FR48 | Tra cứu lịch sử giao dịch                                                                                                               |
| **BR18** | FR49 | Kiểm tra phân quyền cho thao tác nhạy cảm                                                                                               |
| **BR19** | FR50 | Xác thực người dùng trước chức năng cần tài khoản                                                                                       |
| **BR20** | FR51 | Ghi log lưu vết thao tác quan trọng                                                                                                     |
| **BR21** | FR52 | Cho phép hủy chuyến (kiểm tra mốc cho phép hủy theo trạng thái)                                                                         |
|          | FR53 | Áp phí hủy theo chính sách: **miễn phí trước khi có tài xế; 10.000đ khi đã có tài xế chưa đón khách; chặn sau khi đón khách (default)** |
| — chung  | FR54 | Xử lý mất kết nối: cho reconnect trong **60 giây (default)**, đồng bộ trạng thái khi online                                             |

## Bước 8: Xây dựng business rule & exception

### A. Business Rules (RULE)

| Mã         | Quy tắc nghiệp vụ                                                                                                                                                                    | FR liên quan    |
|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| **RULE01** | Chỉ tài xế ở trạng thái sẵn sàng (available) mới được đề xuất bắt chuyến.                                                                                                            | FR08, FR13      |
| **RULE02** | Tài xế chỉ đổi sang sẵn sàng khi đang trong ca làm việc.                                                                                                                             | FR08            |
| **RULE03** | Một tài xế không được gán hai chuyến cùng lúc.                                                                                                                                       | FR13, FR23      |
| **RULE04** | Khi tìm tài xế, chỉ xét tài xế trong **bán kính 1 km (default)** và đúng loại xe.                                                                                                    | FR13, FR14      |
| **RULE05** | Ưu tiên **gần nhất**; hòa khoảng cách → **đánh giá cao hơn (default)**.                                                                                                              | FR15            |
| **RULE06** | Tài xế phải phản hồi trong **20 giây (default)**; quá hạn = từ chối.                                                                                                                 | FR17, FR18      |
| **RULE07** | Tài xế từ chối/hết giờ → tự chuyển tài xế kế tiếp, không bắt khách tạo lại.                                                                                                          | FR19            |
| **RULE08** | Trạng thái chuyến đi đúng thứ tự: đến điểm đón → đón khách → di chuyển → hoàn thành.                                                                                                 | FR26–FR29       |
| **RULE09** | Cước tính sau khi hoàn thành = **12.000đ + 10.000đ/km (default)**.                                                                                                                   | FR31            |
| **RULE10** | Không lưu thông tin thẻ/tài khoản nhạy cảm; điện tử xử lý qua provider ngoài.                                                                                                        | FR34            |
| **RULE11** | Khách chỉ đánh giá tài xế sau khi chuyến hoàn thành.                                                                                                                                 | FR44            |
| **RULE12** | Thao tác quản trị nhạy cảm chỉ dành cho vai trò được cấp quyền.                                                                                                                      | FR49            |
| **RULE13** | Khách & tài xế phải xác thực trước khi dùng chức năng cần tài khoản.                                                                                                                 | FR50            |
| **RULE14** | Mọi thao tác quan trọng phải ghi log (audit).                                                                                                                                        | FR51            |
| **RULE15** | Lỗi thanh toán/thông báo không được làm sập luồng đặt xe; các thành phần độc lập.                                                                                                    | FR34, FR37–FR43 |
| **RULE16** | Khách chỉ xem dữ liệu của chính mình (chuyến, lịch sử, trạng thái).                                                                                                                  | FR04, FR24      |
| **RULE17** | Hủy chuyến: **miễn phí khi còn *đang tìm tài xế*; đã có tài xế mà chưa đón khách → phí 10.000đ; đã đón khách → không cho hủy (default)**.                                            | FR52, FR53      |
| **RULE18** | Mất kết nối ≤ **60 giây (default)** → giữ nguyên trạng thái, đồng bộ lại khi online (last-write-wins theo timestamp); quá hạn khi đang chạy chuyến → đánh dấu *cần vận hành* (EX04). | FR54            |
| **RULE19** | Lưu trữ (default): chuyến & giao dịch **12 tháng**, vị trí tài xế **30 ngày**, audit **12 tháng**; sau đó archive/xoá theo chính sách.                                               | FR51, FR30      |

### B. Exceptions (EX)

| Mã       | Tình huống                                                       | Cách xử lý                                                                                                                                                                                                                         |
|----------|------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **EX01** | Không tìm được tài xế (hết ứng viên / đợi lâu).                  | Dừng vòng tìm khi **đã thử 5 tài xế HOẶC quá 120 giây (default)**, cái nào đến trước; thông báo rõ “không tìm được tài xế”.                                                                                                        |
| **EX02** | Tài xế được đề xuất từ chối hoặc không phản hồi.                 | Loại tài xế đó, tự chọn tài xế kế tiếp (RULE07).                                                                                                                                                                                   |
| **EX03** | Thanh toán điện tử thất bại.                                     | **Thử lại 2 lần → chuyển tiền mặt (default)** + thông báo khách.                                                                                                                                                                   |
| **EX04** | Nhân viên phát hiện chuyến bị lỗi (kẹt trạng thái, sai dữ liệu). | Nhân viên vận hành kiểm tra & hỗ trợ xử lý qua giao diện quản trị.                                                                                                                                                                 |
| **EX05** | Thành phần thông báo lỗi (không gửi được).                       | Không gián đoạn luồng đặt xe (RULE15); ghi nhận để xử lý lại/mở rộng kênh sau.                                                                                                                                                     |
| **EX06** | Người dùng chưa xác thực gọi chức năng cần tài khoản.            | Từ chối truy cập, yêu cầu đăng nhập (RULE13).                                                                                                                                                                                      |
| **EX07** | Nhân viên thường cố thực hiện thao tác nhạy cảm.                 | Kiểm tra phân quyền và từ chối (RULE12), ghi log.                                                                                                                                                                                  |
| **EX08** | Tài xế cập nhật trạng thái sai thứ tự vòng đời.                  | Từ chối cập nhật không hợp lệ, giữ đúng trình tự (RULE08).                                                                                                                                                                         |
| **EX09** | Mất kết nối mạng giữa chuyến (khách/tài xế offline).             | Cho reconnect trong **60 giây (default)**, giữ nguyên trạng thái & đồng bộ lại khi online; quá hạn khi đang chạy chuyến → đánh dấu *cần vận hành* (RULE18, EX04).                                                                  |
| **EX10** | Khách/tài xế hủy chuyến.                                         | Áp chính sách RULE17: miễn phí khi *đang tìm tài xế*; phí **10.000đ (default)** khi đã có tài xế chưa đón khách; chặn hủy sau khi *đã đón khách*. Tài xế hủy sau khi nhận → chuyến quay lại vòng tìm (EX02), không tính phí khách. |

## Bước 9: Xây dựng data modelling & vẽ sơ đồ ERD

### A. Thực thể (Entity)

| Thực thể             | Ý nghĩa                                               | Bám BR           |
|----------------------|-------------------------------------------------------|------------------|
| **CUSTOMER**         | Khách hàng đặt xe                                     | BR01             |
| **DRIVER**           | Tài xế (hồ sơ + trạng thái + điểm đánh giá)           | BR02, BR03       |
| **VEHICLE**          | Phương tiện gắn với tài xế                            | BR02             |
| **SERVICE_TYPE**     | Loại dịch vụ / loại xe (cơ sở tính cước)              | BR04, BR11       |
| **TRIP**             | Chuyến đi — thực thể trung tâm                        | BR04, BR08, BR09 |
| **TRIP_OFFER**       | Lượt đề xuất chuyến cho tài xế (nhận/từ chối/timeout) | BR05, BR06, BR07 |
| **FARE**             | Cước tính cho chuyến                                  | BR11             |
| **PAYMENT**          | Giao dịch thanh toán của chuyến                       | BR12, BR13       |
| **RATING**           | Đánh giá của khách cho tài xế sau chuyến              | BR16             |
| **DRIVER_LOCATION**  | Vị trí tài xế theo thời gian                          | BR10             |
| **NOTIFICATION**     | Thông báo gửi cho khách/tài xế                        | BR14, BR15       |
| **OPERATIONS_STAFF** | Nhân viên vận hành / quản trị                         | BR17             |
| **ROLE**             | Vai trò & quyền (phân quyền)                          | BR18             |
| **AUDIT_LOG**        | Lưu vết thao tác quan trọng                           | BR20             |

### B. Sơ đồ ERD

> **Lưu ý đọc sơ đồ:** `NOTIFICATION`, `DRIVER_LOCATION` và `AUDIT_LOG` được nối bằng quan hệ **đứt nét mang tính tham chiếu**. `NOTIFICATION` và `AUDIT_LOG` dùng cặp `(loai, id)` để trỏ **đa hình (polymorphic)** tới khách/tài xế/nhân viên nên **không** đặt FK cứng — đó là chủ ý thiết kế, không phải thiếu sót.

```mermaid
erDiagram
    CUSTOMER ||--o{ TRIP : "tạo"
    DRIVER ||--o{ TRIP : "thực hiện"
    DRIVER ||--o{ VEHICLE : "sở hữu"
    SERVICE_TYPE ||--o{ TRIP : "phân loại"
    TRIP ||--o{ TRIP_OFFER : "đề xuất tới"
    DRIVER ||--o{ TRIP_OFFER : "được đề xuất"
    TRIP ||--|| FARE : "sinh"
    TRIP ||--|| PAYMENT : "thanh toán"
    TRIP ||--o| RATING : "được đánh giá"
    DRIVER ||--o{ DRIVER_LOCATION : "cập nhật"
    OPERATIONS_STAFF }o--|| ROLE : "có vai trò"
    CUSTOMER ||..o{ NOTIFICATION : "nhận (đa hình)"
    DRIVER  ||..o{ NOTIFICATION : "nhận (đa hình)"
    CUSTOMER ||..o{ AUDIT_LOG : "bị ghi vết (đa hình)"

    CUSTOMER {
        string id PK
        string ho_ten
        string sdt
        string email
    }
    DRIVER {
        string id PK
        string ho_ten
        string trang_thai
        float  diem_danh_gia
    }
    VEHICLE {
        string id PK
        string driver_id FK
        string loai_xe
        string bien_so
    }
    SERVICE_TYPE {
        string id PK
        string ten
        money  don_gia_co_ban
        money  don_gia_moi_km
    }
    TRIP {
        string id PK
        string customer_id FK
        string driver_id FK
        string service_type_id FK
        string diem_don
        string diem_den
        string trang_thai
    }
    TRIP_OFFER {
        string id PK
        string trip_id FK
        string driver_id FK
        string trang_thai
        datetime thoi_diem
    }
    FARE {
        string id PK
        string trip_id FK
        money  tong_tien
        string chi_tiet_tinh
    }
    PAYMENT {
        string id PK
        string trip_id FK
        string phuong_thuc
        string trang_thai
        int    so_lan_thu
    }
    RATING {
        string id PK
        string trip_id FK
        int    so_sao
        string nhan_xet
    }
    DRIVER_LOCATION {
        string id PK
        string driver_id FK
        float  lat
        float  lng
        datetime thoi_diem
    }
    NOTIFICATION {
        string id PK
        string loai_doi_tuong
        string doi_tuong_id
        string noi_dung
    }
    OPERATIONS_STAFF {
        string id PK
        string ho_ten
        string role_id FK
    }
    ROLE {
        string id PK
        string ten
        string quyen
    }
    AUDIT_LOG {
        string id PK
        string loai_doi_tuong
        string doi_tuong_id
        string hanh_dong
        datetime thoi_diem
    }
```

### C. Giải thích quan hệ chính

Thực thể trung tâm là **TRIP**: một **CUSTOMER** tạo nhiều **TRIP**; một **DRIVER** thực hiện nhiều **TRIP**; mỗi **TRIP** thuộc một **SERVICE_TYPE** (cơ sở tính cước ở FR31).

Quá trình ghép tài xế mô hình hóa bằng **TRIP_OFFER**: một chuyến được đề xuất lần lượt tới nhiều tài xế, mỗi lượt có trạng thái offered/accepted/rejected/timeout — chỗ lưu vết cho quy tắc “từ chối thì chuyển tài xế kế tiếp” (RULE07/FR19) mà không cần khách tạo lại yêu cầu.

Mỗi **TRIP** hoàn thành sinh đúng một **FARE** và một **PAYMENT** (1–1), và 0 hoặc 1 **RATING**. **DRIVER_LOCATION** lưu vị trí tài xế theo thời gian phục vụ tìm tài xế gần (FR13) và ETA (FR25). **NOTIFICATION** và **AUDIT_LOG** dùng quan hệ đa hình như đã nêu ở phần B.

### D. Lưu ý

Trường `SERVICE_TYPE.don_gia_co_ban = 12.000đ` và `don_gia_moi_km = 10.000đ` là **giá trị mặc định (default)**; `FARE.chi_tiet_tinh` lưu vết công thức áp dụng cho từng chuyến (phục vụ đối soát BG04). `PAYMENT.so_lan_thu` phục vụ chính sách retry (default 2 lần). Tất cả để cấu hình được.

Nhóm thực thể lõi cho MVP: CUSTOMER, DRIVER, VEHICLE, SERVICE_TYPE, TRIP, TRIP_OFFER, FARE, PAYMENT, RATING. Các thực thể NOTIFICATION, DRIVER_LOCATION, AUDIT_LOG, ROLE, OPERATIONS_STAFF có thể rút gọn tùy quỹ thời gian 7 tuần.

## Bước 10: Xác định & thiết kế non-functional requirement (NFR)

| Mã        | Nhóm                            | Yêu cầu                                                                      | Định hướng MVP (7 tuần)                                                                                                                                    |
|-----------|---------------------------------|------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **NFR01** | Performance                     | Đáp ứng thao tác cơ bản trong thời gian hợp lý.                              | Không đặt mục tiêu độ trễ cực thấp; đủ mượt cho demo.                                                                                                      |
| **NFR02** | Scalability                     | Các thành phần mở rộng độc lập khi tải tăng.                                 | Thiết kế module/dịch vụ tách rời; chưa cần auto-scaling thật.                                                                                              |
| **NFR03** | Reliability / Fault Isolation   | Lỗi thanh toán/thông báo không làm sập luồng đặt xe.                         | **Bắt buộc**: tách thanh toán & thông báo khỏi luồng lõi, xử lý lỗi cục bộ.                                                                                |
| **NFR04** | Availability                    | Hoạt động ổn định giờ cao điểm.                                              | Ổn định cho demo; HA đầy đủ để giai đoạn scale-up.                                                                                                         |
| **NFR05** | Maintainability / Extensibility | Thêm dịch vụ/thanh toán/kênh thông báo mà không xây lại toàn bộ.             | **Bắt buộc**: abstraction/interface cho payment & notification.                                                                                            |
| **NFR06** | Deployability                   | Triển khai chức năng mới từng phần.                                          | Tách module rõ ràng; CI/CD đầy đủ làm sau.                                                                                                                 |
| **NFR07** | Authentication                  | Khách & tài xế xác thực trước khi dùng chức năng cần tài khoản.              | **Bắt buộc** (đăng nhập + phiên/token).                                                                                                                    |
| **NFR08** | Authorization                   | Thao tác quản trị nhạy cảm kiểm soát quyền theo vai trò.                     | **Bắt buộc**: phân quyền cơ bản.                                                                                                                           |
| **NFR09** | Data Protection                 | Bảo vệ dữ liệu cá nhân/phương tiện/vị trí/giao dịch; không lưu thẻ nhạy cảm. | **Bắt buộc**: hash mật khẩu, điện tử qua provider ngoài.                                                                                                   |
| **NFR10** | Auditability                    | Ghi vết thao tác quan trọng.                                                 | Log audit cơ bản cho thao tác quản trị & giao dịch.                                                                                                        |
| **NFR11** | Resilience under load           | Không sụp đổ dây chuyền khi một thành phần quá tải.                          | Tách dịch vụ + hàng đợi cho tác vụ nền (thông báo).                                                                                                        |
| **NFR12** | Configurability                 | Tham số nghiệp vụ để dạng cấu hình.                                          | **Bắt buộc**: bán kính, timeout, công thức cước, số lần retry, phí hủy, grace offline, retention ở config.                                                 |
| **NFR13** | Data Retention                  | Vòng đời & thời gian lưu trữ dữ liệu.                                        | Default: chuyến & giao dịch **12 tháng**, vị trí tài xế **30 ngày**, audit **12 tháng**. MVP chỉ đặt tham số ở config; job dọn dữ liệu tự động làm sau.    |
| **NFR14** | Usability                       | Giao diện rõ ràng, dễ thao tác; thông báo trạng thái & lỗi dễ hiểu.          | Bắt buộc ở mức tối thiểu cho demo: luồng đặt xe gọn (ít bước), hiển thị trạng thái chuyến & thông báo lỗi rõ ràng; chưa cần chuẩn UX/accessibility đầy đủ. |

## Bước 11: Vẽ Use Case (14 UC lõi – MVP)

### A. Danh sách Use Case (14 UC)

| Mã       | Use Case                    | Tác nhân chính   | FR liên quan     |
|----------|-----------------------------|------------------|------------------|
| **UC01** | Đăng ký / Đăng nhập         | Customer, Driver | FR01, FR02, FR05 |
| **UC02** | Quản lý hồ sơ cá nhân       | Customer         | FR03             |
| **UC03** | Đặt chuyến xe               | Customer         | FR09, FR10, FR11 |
| **UC04** | Theo dõi chuyến đi          | Customer         | FR24, FR25       |
| **UC05** | Thanh toán chuyến           | Customer         | FR32–FR36        |
| **UC06** | Đánh giá tài xế             | Customer         | FR44             |
| **UC07** | Xem lịch sử chuyến          | Customer         | FR04             |
| **UC08** | Quản lý hồ sơ & phương tiện | Driver           | FR06, FR07       |
| **UC09** | Đổi trạng thái sẵn sàng     | Driver           | FR08             |
| **UC10** | Nhận & phản hồi chuyến      | Driver           | FR21, FR22       |
| **UC11** | Cập nhật tiến trình chuyến  | Driver           | FR26–FR29        |
| **UC12** | Tìm & phân công tài xế      | System (từ UC03) | FR12–FR20, FR23  |
| **UC13** | Tính cước                   | System (từ UC11) | FR31             |
| **UC14** | Quản trị & vận hành         | Operations Staff | FR45–FR49, FR51  |

### B. Sơ đồ Use Case

```mermaid
graph LR
    KH["Khách hàng"]:::actor
    TX["Tài xế"]:::actor
    NV["NV Vận hành"]:::actor

    UC01(["UC01 · Đăng ký / Đăng nhập"])
    UC02(["UC02 · Quản lý hồ sơ cá nhân"])
    UC03(["UC03 · Đặt chuyến xe"])
    UC04(["UC04 · Theo dõi chuyến đi"])
    UC05(["UC05 · Thanh toán chuyến"])
    UC06(["UC06 · Đánh giá tài xế"])
    UC07(["UC07 · Xem lịch sử chuyến"])
    UC08(["UC08 · Quản lý hồ sơ & phương tiện"])
    UC09(["UC09 · Đổi trạng thái sẵn sàng"])
    UC10(["UC10 · Nhận & phản hồi chuyến"])
    UC11(["UC11 · Cập nhật tiến trình chuyến"])
    UC12(["UC12 · Tìm & phân công tài xế"])
    UC13(["UC13 · Tính cước"])
    UC14(["UC14 · Quản trị & vận hành"])

    KH --- UC01
    KH --- UC02
    KH --- UC03
    KH --- UC04
    KH --- UC05
    KH --- UC06
    KH --- UC07

    TX --- UC01
    TX --- UC08
    TX --- UC09
    TX --- UC10
    TX --- UC11

    NV --- UC14

    UC03 -.->|include| UC12
    UC12 -.->|include| UC10
    UC11 -.->|include| UC13
    UC05 -.->|extend| UC11
    UC06 -.->|extend| UC11

    classDef actor fill:#cfe3ff,stroke:#2b6cb0,stroke-width:1px,font-weight:bold;
```


### C. Quan hệ include/extend

Ba `include` cốt lõi: UC03 include UC12 (đặt xe kéo theo tìm tài xế), UC12 include UC10 (tìm tài xế phải qua bước tài xế phản hồi), UC11 include UC13 (hoàn thành chuyến thì tính cước). Hai `extend`: UC05 (thanh toán) và UC06 (đánh giá) mở rộng từ UC11 (sau khi chuyến hoàn thành). UC14 (quản trị & vận hành) độc lập, do nhân viên vận hành thực hiện. Hủy chuyến không tách thành use case riêng mà được xử lý như luồng ngoại lệ (EX10) theo quy tắc RULE17 gắn với FR52–FR53.

## Bước 12: Đặc tả Use Case (14 UC)

### UC01 — Đăng ký / Đăng nhập (Customer, Driver)

| Mục            | Nội dung                                                                                                                                       |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| Tác nhân       | Khách hàng, Tài xế (tự đăng ký hoặc do nhân viên vận hành tạo)                                                                                 |
| Tiền điều kiện | Chưa đăng nhập                                                                                                                                 |
| Luồng chính    | 1\) Nhập thông tin đăng ký (hoặc định danh + mật khẩu). 2) Hệ thống kiểm tra & tạo tài khoản / cấp phiên. 3) Vào được chức năng cần tài khoản. |
| Ngoại lệ       | Định danh đã tồn tại → báo lỗi; sai mật khẩu → từ chối, khóa sau N lần sai.                                                                    |
| Rule           | RULE13.                                                                                                                                        |

### UC02 — Quản lý hồ sơ cá nhân (Customer)

| Mục            | Nội dung                                                   |
|----------------|------------------------------------------------------------|
| Tác nhân       | Khách hàng                                                 |
| Tiền điều kiện | Đã đăng nhập                                               |
| Luồng chính    | 1\) Xem hồ sơ. 2) Sửa thông tin. 3) Kiểm tra hợp lệ & lưu. |
| Ngoại lệ       | Trường bắt buộc thiếu/không hợp lệ → từ chối lưu.          |
| Rule           | Chỉ chủ tài khoản mới sửa được hồ sơ của mình.             |

### UC03 — Đặt chuyến xe (Customer)

| Mục            | Nội dung                                                                                                                 |
|----------------|--------------------------------------------------------------------------------------------------------------------------|
| Tác nhân       | Khách hàng                                                                                                               |
| Tiền điều kiện | Đã đăng nhập (UC01)                                                                                                      |
| Hậu điều kiện  | Tạo chuyến *đang tìm tài xế*; kích hoạt UC12                                                                             |
| Luồng chính    | 1\) Nhập điểm đón/đến. 2) Chọn loại xe. 3) Gửi yêu cầu. 4) Kiểm tra & tạo chuyến. 5) Xác nhận tiếp nhận. 6) Chuyển UC12. |
| Ngoại lệ       | Thiếu điểm đón/đến → báo lỗi; loại xe không khả dụng → chọn lại.                                                         |
| Rule           | RULE13.                                                                                                                  |

### UC04 — Theo dõi chuyến đi (Customer)

| Mục            | Nội dung                                                                                |
|----------------|-----------------------------------------------------------------------------------------|
| Tác nhân       | Khách hàng                                                                              |
| Tiền điều kiện | Có chuyến đang xử lý                                                                    |
| Luồng chính    | 1\) Truy vấn trạng thái chuyến. 2) Xem: đang tìm / đã nhận / ETA / trạng thái hiện tại. |
| Ngoại lệ       | Chưa có tài xế → hiển thị “đang tìm tài xế”.                                            |
| Rule           | RULE16.                                                                                 |

### UC05 — Thanh toán chuyến (Customer)

| Mục             | Nội dung                                                                                                                                           |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| Tác nhân        | Khách hàng (phụ: Nhà cung cấp thanh toán khi điện tử)                                                                                              |
| Tiền điều kiện  | Chuyến hoàn thành & đã tính cước (UC13)                                                                                                            |
| Hậu điều kiện   | Chuyến ở trạng thái *đã thanh toán*                                                                                                                |
| Luồng chính     | 1\) Hiển thị số tiền. 2) Chọn tiền mặt / điện tử. 3a) Tiền mặt → xác nhận. 3b) Điện tử → gọi provider ngoài. 4) Cập nhật kết quả, thông báo khách. |
| Ngoại lệ        | Điện tử thất bại → **thử lại 2 lần → chuyển tiền mặt (default)** (EX03).                                                                           |
| Rule            | RULE10.                                                                                                                                            |
| Default áp dụng | Retry 2 lần → fallback tiền mặt. MVP ưu tiên demo nhánh tiền mặt trước.                                                                            |

### UC06 — Đánh giá tài xế (Customer)

| Mục            | Nội dung                                                             |
|----------------|----------------------------------------------------------------------|
| Tác nhân       | Khách hàng                                                           |
| Tiền điều kiện | Chuyến đã hoàn thành                                                 |
| Luồng chính    | 1\) Chọn số sao + nhận xét. 2) Lưu đánh giá gắn với chuyến & tài xế. |
| Ngoại lệ       | Chuyến chưa hoàn thành → không cho đánh giá.                         |
| Rule           | RULE11.                                                              |

### UC07 — Xem lịch sử chuyến (Customer)

| Mục            | Nội dung                                                 |
|----------------|----------------------------------------------------------|
| Tác nhân       | Khách hàng                                               |
| Tiền điều kiện | Đã đăng nhập                                             |
| Luồng chính    | 1\) Mở lịch sử. 2) Trả danh sách chuyến đã đi + số tiền. |
| Ngoại lệ       | Chưa có chuyến → danh sách rỗng.                         |
| Rule           | RULE16.                                                  |

### UC08 — Quản lý hồ sơ & phương tiện (Driver)

| Mục            | Nội dung                                                                       |
|----------------|--------------------------------------------------------------------------------|
| Tác nhân       | Tài xế                                                                         |
| Tiền điều kiện | Đã đăng nhập                                                                   |
| Luồng chính    | 1\) Cập nhật hồ sơ tài xế. 2) Cập nhật phương tiện (biển số, loại xe). 3) Lưu. |
| Ngoại lệ       | Thiếu trường bắt buộc → từ chối lưu.                                           |
| Rule           | Một tài xế gắn đúng phương tiện hợp lệ (dùng lọc loại xe ở UC12).              |

### UC09 — Đổi trạng thái sẵn sàng (Driver)

| Mục            | Nội dung                                                  |
|----------------|-----------------------------------------------------------|
| Tác nhân       | Tài xế                                                    |
| Tiền điều kiện | Đã đăng nhập, đang trong ca làm việc                      |
| Luồng chính    | 1\) Chọn sẵn sàng / không sẵn sàng. 2) Hệ thống cập nhật. |
| Ngoại lệ       | Không trong ca → không cho chuyển sẵn sàng.               |
| Rule           | RULE01, RULE02.                                           |

### UC10 — Nhận & phản hồi chuyến (Driver)

| Mục            | Nội dung                                                                                                       |
|----------------|----------------------------------------------------------------------------------------------------------------|
| Tác nhân       | Tài xế                                                                                                         |
| Tiền điều kiện | Tài xế *sẵn sàng*; nhận đề xuất từ UC12                                                                        |
| Hậu điều kiện  | Chuyến được gán (nếu chấp nhận) hoặc chuyển tài xế khác                                                        |
| Luồng chính    | 1\) Nhận thông báo chuyến mới. 2) Xem thông tin chuyến. 3) Chấp nhận → gán tài xế, thông báo khách.            |
| Ngoại lệ       | Từ chối → chuyển tài xế kế tiếp (EX02); không phản hồi trong **20 giây (default)** → coi như từ chối (RULE06). |
| Rule           | RULE01, RULE03.                                                                                                |

### UC11 — Cập nhật tiến trình chuyến (Driver)

| Mục            | Nội dung                                                                               |
|----------------|----------------------------------------------------------------------------------------|
| Tác nhân       | Tài xế                                                                                 |
| Tiền điều kiện | Đã được gán chuyến (UC10)                                                              |
| Hậu điều kiện  | Chuyến *hoàn thành* → kích hoạt UC13                                                   |
| Luồng chính    | 1\) “đã đến điểm đón”. 2) “đã đón khách”. 3) “đang di chuyển”. 4) “hoàn thành chuyến”. |
| Ngoại lệ       | Cập nhật sai thứ tự → từ chối (EX08).                                                  |
| Rule           | RULE08.                                                                                |

### UC12 — Tìm & phân công tài xế (System)

| Mục            | Nội dung                                                                                                                                                                                                                                                                                |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Tác nhân       | Hệ thống (kích hoạt từ UC03); phụ: Tài xế qua UC10                                                                                                                                                                                                                                      |
| Tiền điều kiện | Có chuyến *đang tìm tài xế*                                                                                                                                                                                                                                                             |
| Hậu điều kiện  | Gán được tài xế hoặc thông báo “không tìm được tài xế”                                                                                                                                                                                                                                  |
| Luồng chính    | 1\) Xác định vị trí khách. 2) Lọc tài xế sẵn sàng trong **bán kính 1 km (default)**. 3) Lọc theo loại xe. 4) Xếp ưu tiên **gần nhất → đánh giá cao hơn (default)**. 5) Chọn ứng viên đầu. 6) Gửi đề xuất (UC10), chờ **20 giây (default)**. 7) Chấp nhận → gán tài xế, thông báo khách. |
| Ngoại lệ       | Từ chối/hết giờ → chọn tài xế kế tiếp (EX02); dừng sau **5 ứng viên hoặc 120 giây (default)** → thông báo “không tìm được tài xế” (EX01).                                                                                                                                               |
| Rule           | RULE01, RULE03, RULE04, RULE05, RULE06.                                                                                                                                                                                                                                                 |

### UC13 — Tính cước (System)

| Mục            | Nội dung                                                                                                                     |
|----------------|------------------------------------------------------------------------------------------------------------------------------|
| Tác nhân       | Hệ thống (kích hoạt từ UC11)                                                                                                 |
| Tiền điều kiện | Chuyến *hoàn thành*                                                                                                          |
| Hậu điều kiện  | Có số tiền phải trả → chuyển UC05                                                                                            |
| Luồng chính    | 1\) Lấy loại dịch vụ + thông tin chuyến. 2) Áp công thức **12.000đ + 10.000đ/km (default)**. 3) Ghi nhận số tiền cho chuyến. |
| Ngoại lệ       | Thiếu dữ liệu chuyến → chưa tính được, chờ bổ sung.                                                                          |
| Rule           | RULE09.                                                                                                                      |

### UC14 — Quản trị & vận hành (Operations Staff)

| Mục            | Nội dung                                                                                                                                                                |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Tác nhân       | Nhân viên vận hành (phân quyền theo ROLE)                                                                                                                               |
| Tiền điều kiện | Đã đăng nhập & có quyền vận hành                                                                                                                                        |
| Luồng chính    | 1\) Xem danh sách chuyến đang diễn ra + trạng thái tài xế. 2) Quản lý (CRUD) khách/tài xế/phương tiện/chuyến. 3) Hỗ trợ xử lý chuyến lỗi. 4) Tra cứu lịch sử giao dịch. |
| Ngoại lệ       | Nhân viên thường cố thao tác nhạy cảm → từ chối (EX07).                                                                                                                 |
| Rule           | RULE12 (phân quyền), RULE14 (ghi log audit).                                                                                                                            |
| Phạm vi MVP    | Chỉ bắt buộc màn hình **xem danh sách chuyến + trạng thái tài xế**; CRUD đầy đủ & tra cứu giao dịch là Should-have.                                                     |

## Bước 13: Tiêu chí chấp nhận (Acceptance Criteria – AC)

### Nhóm Khách hàng

| Mã       | UC   | Tiêu chí chấp nhận (Given → When → Then)                                                                                    |
|----------|------|-----------------------------------------------------------------------------------------------------------------------------|
| **AC01** | UC01 | Given thông tin hợp lệ, When đăng ký, Then tạo tài khoản & đăng nhập được.                                                  |
| **AC02** | UC01 | Given định danh đã tồn tại, When đăng ký, Then báo lỗi, không tạo trùng.                                                    |
| **AC03** | UC01 | Given sai mật khẩu N lần, When đăng nhập tiếp, Then khóa tạm thời.                                                          |
| **AC04** | UC03 | Given đã đăng nhập, When nhập đủ điểm đón/đến + loại xe và gửi, Then tạo chuyến *đang tìm tài xế*.                          |
| **AC05** | UC03 | Given thiếu điểm đón/đến, When gửi, Then chặn và báo lỗi.                                                                   |
| **AC06** | UC03 | Given chưa đăng nhập, When cố đặt chuyến, Then từ chối, yêu cầu xác thực (RULE13).                                          |
| **AC07** | UC04 | Given chuyến đang xử lý, When mở theo dõi, Then thấy đúng trạng thái (đang tìm / đã nhận / ETA).                            |
| **AC08** | UC04 | Given khách A, When truy vấn chuyến khách B, Then từ chối (RULE16).                                                         |
| **AC09** | UC05 | Given chuyến hoàn thành & đã tính cước, When chọn tiền mặt và xác nhận, Then chuyển *đã thanh toán*.                        |
| **AC10** | UC05 | Given điện tử thất bại, When xử lý, Then **thử lại 2 lần → chuyển tiền mặt (default)** + thông báo; không lưu thẻ (RULE10). |
| **AC11** | UC06 | Given chuyến hoàn thành, When gửi sao + nhận xét, Then lưu đánh giá gắn chuyến/tài xế.                                      |
| **AC12** | UC06 | Given chuyến chưa hoàn thành, When cố đánh giá, Then từ chối (RULE11).                                                      |
| **AC13** | UC07 | Given đã đăng nhập, When mở lịch sử, Then thấy danh sách chuyến + số tiền của chính mình.                                   |

### Nhóm Tài xế

| Mã       | UC   | Tiêu chí chấp nhận                                                                                                           |
|----------|------|------------------------------------------------------------------------------------------------------------------------------|
| **AC14** | UC01 | Given tài khoản hợp lệ, When đăng nhập, Then vào được chức năng tài xế.                                                      |
| **AC15** | UC08 | Given đã đăng nhập, When cập nhật hồ sơ + phương tiện, Then lưu & dùng cho lọc ở UC12.                                       |
| **AC16** | UC09 | Given đang trong ca, When đổi *sẵn sàng*, Then cập nhật & đủ điều kiện được ghép (RULE01).                                   |
| **AC17** | UC09 | Given không trong ca, When cố chuyển *sẵn sàng*, Then từ chối (RULE02).                                                      |
| **AC18** | UC10 | Given tài xế *sẵn sàng* nhận đề xuất, When chấp nhận, Then chuyến được gán & khách được thông báo.                           |
| **AC19** | UC10 | Given không phản hồi trong **20 giây (default)**, When hết giờ, Then coi như từ chối & chuyển tài xế kế tiếp (RULE06, EX02). |
| **AC20** | UC10 | Given tài xế đang thực hiện chuyến, When có đề xuất mới, Then không gán trùng (RULE03).                                      |
| **AC21** | UC11 | Given đã được gán, When cập nhật đúng thứ tự, Then mỗi bước được ghi nhận.                                                   |
| **AC22** | UC11 | Given cập nhật sai thứ tự, When gửi, Then từ chối (RULE08, EX08).                                                            |

### Nhóm Hệ thống (ghép tài xế & tính cước)

| Mã       | UC   | Tiêu chí chấp nhận                                                                                                                                   |
|----------|------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| **AC23** | UC12 | Given có chuyến *đang tìm tài xế*, When chạy ghép, Then chỉ xét tài xế *sẵn sàng*, trong **bán kính 1 km (default)**, đúng loại xe (RULE01, RULE04). |
| **AC24** | UC12 | Given nhiều tài xế phù hợp, When xếp ưu tiên, Then **gần nhất trước; hòa → đánh giá cao hơn (default)** (RULE05).                                    |
| **AC25** | UC12 | Given tài xế đầu từ chối/không phản hồi, When xử lý, Then tự chọn tài xế kế tiếp không bắt khách tạo lại (EX02).                                     |
| **AC26** | UC12 | Given không còn tài xế phù hợp, When kết thúc vòng tìm, Then thông báo rõ “không tìm được tài xế” (EX01).                                            |
| **AC27** | UC13 | Given chuyến *hoàn thành*, When tính cước, Then số tiền = **12.000đ + 10.000đ/km (default)** & gắn vào chuyến (RULE09).                              |
| **AC28** | UC13 | Given chuyến chưa hoàn thành, When cố tính cước, Then không thực hiện.                                                                               |

### Nhóm Vận hành & phi chức năng

| Mã       | UC/NFR    | Tiêu chí chấp nhận                                                                                                                                                                       |
|----------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **AC29** | NFR03     | Given thành phần thanh toán/thông báo lỗi, When lỗi xảy ra, Then luồng đặt xe vẫn hoạt động, không sập toàn hệ thống.                                                                    |
| **AC30** | NFR07/08  | Given người dùng chưa xác thực hoặc không đủ quyền, When gọi chức năng cần tài khoản/nhạy cảm, Then từ chối (RULE12, RULE13).                                                            |
| **AC31** | NFR09     | Given giao dịch điện tử, When xử lý, Then không lưu thông tin thẻ/tài khoản nhạy cảm trong CAB (RULE10).                                                                                 |
| **AC32** | NFR12     | Given tham số chưa chốt (bán kính, timeout, công thức cước, retry), When triển khai, Then giá trị nằm ở cấu hình, không hard-code.                                                       |
| **AC33** | UC14/BR14 | Given chuyến đi qua các mốc, When mỗi mốc xảy ra, Then hệ thống phát thông báo khách đúng thời điểm (MVP: ghi log).                                                                      |
| **AC34** | BR15      | Given có chuyến mới/thay đổi, When xảy ra, Then hệ thống phát thông báo tài xế đúng thời điểm (MVP: ghi log).                                                                            |
| **AC35** | BR10      | Given tài xế cập nhật vị trí, When ghi nhận, Then vị trí dùng được cho ghép tài xế ở UC12.                                                                                               |
| **AC36** | UC14      | Given nhân viên vận hành đăng nhập, When mở màn hình vận hành, Then thấy danh sách chuyến đang chạy + trạng thái tài xế.                                                                 |
| **AC37** | BR20      | Given một thao tác nhạy cảm được thực hiện, When hoàn tất, Then sinh một bản ghi audit (RULE14).                                                                                         |
| **AC38** | —         | Given chuyến *đang tìm tài xế*, When khách hủy, Then hủy **miễn phí (default)** và thông báo; Given *đã có tài xế chưa đón khách*, When hủy, Then áp phí **10.000đ (default)** (RULE17). |
| **AC39** | —         | Given chuyến *đã đón khách*, When khách cố hủy, Then **bị chặn (default)** (RULE17, EX10).                                                                                               |
| **AC40** | NFR/EX09  | Given mất kết nối ≤ **60 giây (default)**, When online lại, Then trạng thái chuyến giữ nguyên & được đồng bộ (RULE18); quá hạn khi đang chạy → đánh dấu *cần vận hành*.                  |
| **AC41** | NFR13     | Given tham số retention (12 tháng / 30 ngày), When triển khai, Then giá trị nằm ở cấu hình, không hard-code (RULE19).                                                                    |

### Lưu ý khi dùng để nghiệm thu

Mỗi AC là một điều kiện kiểm chứng được — QA chuyển thẳng thành test case, khách hàng dùng làm checklist ký nghiệm thu. Sau khi gán default, các AC gắn tham số (AC10, AC19, AC23, AC24, AC27) **đã nghiệm thu được cả cơ chế lẫn ngưỡng số** ở mức giá trị mặc định; khi khách chốt giá trị chính thức chỉ cập nhật cấu hình (AC32/NFR12), không phải viết lại test. Riêng **AC26 (EX01)** chỉ nghiệm thu được phần “có thông báo khi hết tài xế”; phần “dừng vòng sau bao lâu / bao nhiêu ứng viên” chờ chốt.

## Bước 14: Truy xuất nguồn gốc yêu cầu – Ma trận RTM

### 

| BG         | BR                     | FR                     | UC           | AC   |
|------------|------------------------|------------------------|--------------|------|
| BG09       | BR01                   | FR01, FR02             | UC01         | AC01 |
| BG09       | BR01                   | FR01                   | UC01         | AC02 |
| BG09       | BR01, BR19             | FR02                   | UC01         | AC03 |
| BG01       | BR04                   | FR09, FR10, FR11       | UC03         | AC04 |
| BG01       | BR04                   | FR09                   | UC03         | AC05 |
| BG01, BG09 | BR04, BR19             | FR11, FR50             | UC03         | AC06 |
| BG03       | BR08                   | FR24, FR25             | UC04         | AC07 |
| BG03, BG09 | BR08                   | FR24                   | UC04         | AC08 |
| BG02       | BR12                   | FR32, FR33             | UC05         | AC09 |
| BG02       | BR12, BR13             | FR34, FR35, FR36       | UC05         | AC10 |
| BG01       | BR16                   | FR44                   | UC06         | AC11 |
| BG01       | BR16                   | FR44                   | UC06         | AC12 |
| BG03       | BR01                   | FR04                   | UC07         | AC13 |
| BG09       | BR02                   | FR05, FR02             | UC01         | AC14 |
| BG01, BG09 | BR02                   | FR06, FR07             | UC08         | AC15 |
| BG01       | BR03                   | FR08                   | UC09         | AC16 |
| BG01       | BR03                   | FR08                   | UC09         | AC17 |
| BG01       | BR07                   | FR21, FR22, FR23       | UC10         | AC18 |
| BG01       | BR06                   | FR18, FR22             | UC10         | AC19 |
| BG01       | BR07                   | FR22, FR23             | UC10         | AC20 |
| BG03       | BR09                   | FR26–FR29              | UC11         | AC21 |
| BG03       | BR09                   | FR26–FR29              | UC11         | AC22 |
| BG01       | BR05                   | FR12, FR13, FR14       | UC12         | AC23 |
| BG01       | BR05                   | FR15, FR16             | UC12         | AC24 |
| BG01       | BR06                   | FR17, FR18, FR19       | UC12         | AC25 |
| BG01       | BR06                   | FR20                   | UC12         | AC26 |
| BG04       | BR11                   | FR31                   | UC13         | AC27 |
| BG04       | BR11                   | FR31                   | UC13         | AC28 |
| BG08       | BR12, BR14             | FR34, FR37–FR43        | — (NFR03)    | AC29 |
| BG09       | BR18, BR19             | FR49, FR50             | — (NFR07/08) | AC30 |
| BG02, BG09 | BR12                   | FR34                   | — (NFR09)    | AC31 |
| BG08       | BR05, BR06, BR11, BR13 | FR13, FR18, FR31, FR36 | — (NFR12)    | AC32 |
| BG05       | BR14                   | FR37–FR41              | UC14         | AC33 |
| BG05       | BR15                   | FR42, FR43             | UC14         | AC34 |
| BG01       | BR10                   | FR30                   | UC12         | AC35 |
| BG06       | BR17                   | FR45, FR46, FR47, FR48 | UC14         | AC36 |
| BG10       | BR20                   | FR51                   | UC14         | AC37 |
| BG03       | BR21                   | FR52, FR53             | —            | AC38 |
| BG03       | BR21                   | FR53                   | —            | AC39 |
| BG08       | BR21 (chung)           | FR54                   | — (NFR/EX09) | AC40 |
| BG10       | BR20 (chung)           | FR51, FR30             | — (NFR13)    | AC41 |

## Bước 15 (bổ sung): Open Items – Câu hỏi cần làm rõ với khách hàng

| #  | Điểm chưa chốt (theo file gốc)        | Trạng thái                                                                                    | Câu hỏi cho khách hàng                                                   |
|-----|---------------------------------------|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| 1   | Cách tính cước                        | Có default (12k + 10k/km)                                                                     | Biểu giá chính thức? Có phụ phí giờ cao điểm/quãng đường tối thiểu?      |
| 2   | Tiêu chí ưu tiên tài xế               | Có default (gần nhất → đánh giá cao)                                                          | Có thêm tiêu chí (tỷ lệ nhận chuyến, thời gian chờ của tài xế)?          |
| 3   | Thời gian tài xế phản hồi             | Có default (20 giây)                                                                          | Ngưỡng chính thức? Khác nhau theo khung giờ?                             |
| 4   | Chính sách hủy chuyến & phí hủy       | Có default (miễn phí trước khi có tài xế; 10.000đ khi chưa đón khách; chặn sau khi đón khách) | Mức phí chính thức? Có phí hủy phía tài xế? Có giới hạn số lần hủy/ngày? |
| 5   | Xử lý mất kết nối mạng (EX09)         | Có default (grace 60 giây, đồng bộ lại khi online)                                            | Grace period chính thức? Quá hạn thì tự hủy hay giữ *cần vận hành*?      |
| 6   | Thời gian lưu trữ dữ liệu (retention) | Có default (chuyến/giao dịch 12 tháng, vị trí 30 ngày, audit 12 tháng)                        | Có ràng buộc pháp lý/kiểm toán buộc lưu lâu hơn?                         |
| 7   | Điều kiện dừng vòng ghép (EX01)       | Có default (tối đa 5 ứng viên **hoặc** 120 giây)                                              | Số ứng viên / tổng thời gian chính thức?                                 |
| 8   | Số lần retry thanh toán điện tử       | Có default (2 lần → tiền mặt)                                                                 | Số lần chính thức? Sau fallback có cho thử lại điện tử không?            |

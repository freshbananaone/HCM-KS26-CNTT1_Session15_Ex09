# Bài 9: Tự thiết kế hệ thống lưu trữ và phân quyền cho dự án CampusPay

## Phần 1: Phân tích bài toán (I/O)

### 1. Đầu vào và đầu ra

**Đầu vào**

- **Ban Kỹ thuật (3 người):** viết code, thiết kế hệ thống, kiểm thử API và quản lý tài liệu kỹ thuật.
- **Ban Thiết kế (2 người):** thiết kế UI/UX, wireframe và tài sản đồ họa.
- **Ban Kinh doanh (2 người):** viết Business Plan, nghiên cứu thị trường, dự báo tài chính và làm việc với đối tác.
- **Nhà đầu tư tiềm năng (1 người bên ngoài):** chỉ cần xem Business Plan và Slide Pitch đã được duyệt.
- Các cấp độ tài liệu cần quản lý:
  - Tài liệu làm việc riêng của từng ban.
  - Tài liệu phối hợp giữa nhiều ban.
  - Tài liệu nhạy cảm như API key, token, dữ liệu khách hàng và thông tin tài chính.
  - Tài liệu đã xuất bản để chia sẻ cho người bên ngoài.
- Yêu cầu hệ thống: phân quyền tối thiểu, không để tài liệu nhạy cảm ở chế độ công khai, có lịch sử phiên bản tối thiểu 30 ngày và tự động hết hạn quyền của nhà đầu tư sau 14 ngày.

**Đầu ra mong đợi**

- Một Shared Drive của dự án với cấu trúc thư mục rõ ràng.
- Bảng phân quyền cho 4 vai trò trên 5 nhóm tài liệu chính.
- Quy trình thiết lập, kiểm tra, cập nhật và thu hồi quyền.
- Cách xử lý cụ thể cho 3 bẫy dữ liệu trong đề bài.

### 2. Giải pháp và cấu trúc thư mục

Mình chọn dùng **Shared Drive tên CampusPay** thay vì để file rải rác trong My Drive của từng thành viên. Khi dùng Shared Drive, dữ liệu thuộc về dự án; nếu một thành viên nghỉ việc thì file vẫn ở lại và người quản lý có thể tiếp tục kiểm soát.

Cấu trúc chỉ gồm 2 cấp: cấp dự án và cấp ban/nhóm.

    CampusPay/                         (cấp dự án - Shared Drive)
    ├── 01_Tai_lieu_chung/             (dùng chung nội bộ)
    ├── 02_Ban_Ky_thuat/               (tài liệu kỹ thuật)
    ├── 03_Ban_Thiet_ke/               (tài liệu thiết kế)
    ├── 04_Ban_Kinh_doanh/             (tài liệu kinh doanh)
    └── 05_Chia_se_ben_ngoai/          (bản đã duyệt để gửi ngoài)

Ý nghĩa và cách sử dụng:

- **01_Tai_lieu_chung:** roadmap, biên bản họp, Business Summary và Slide Pitch bản đang phối hợp. Chỉ những người cần làm việc chung mới được truy cập.
- **02_Ban_Ky_thuat:** kiến trúc hệ thống, API specification, test case, hướng dẫn triển khai và tài liệu vận hành. API key, token, mật khẩu và dữ liệu khách hàng không được lưu trực tiếp trong thư mục này; nên dùng Secret Manager hoặc kho secret riêng.
- **03_Ban_Thiet_ke:** wireframe, thiết kế UI/UX, logo, icon và asset đồ họa. File nguồn chỉ cấp quyền sửa cho Ban Thiết kế.
- **04_Ban_Kinh_doanh:** Business Plan, nghiên cứu thị trường, thông tin đối tác và dự báo tài chính. Đây là khu vực có nhiều thông tin mật nên để Restricted.
- **05_Chia_se_ben_ngoai:** chỉ chứa bản đã duyệt, chẳng hạn PDF Slide Pitch. Không đặt Google Slides gốc, bản nháp, dữ liệu khách hàng hoặc thông tin tài chính chi tiết ở đây.

Tài liệu quan trọng phải nằm trên Drive để có Version history. Ngoài lịch sử phiên bản của Google Drive, nên đặt quy ước tên như CP_BusinessPlan_v1.3_2026-10-29 và CP_SlidePitch_PDF_2026-10-29 để dễ tra cứu, khôi phục và đối chiếu.

### 3. Các bước lập cấu hình hệ thống

1. Tạo Shared Drive CampusPay và chỉ định trưởng nhóm làm **Manager**.
2. Tạo 5 thư mục con theo đúng cấu trúc ở trên; thống nhất tiền tố số và quy tắc đặt tên file.
3. Tạo ba nhóm email nội bộ: campuspay-kythuat, campuspay-thietke, campuspay-kinhdoanh. Cập nhật thành viên qua nhóm thay vì chia sẻ thủ công từng người.
4. Thêm đúng thành viên vào từng nhóm và kiểm tra lại địa chỉ email nhà đầu tư vì đây là tài khoản bên ngoài.
5. Cấp quyền theo nguyên tắc tối thiểu:
   - **Editor:** chỉ cho người thực sự tạo hoặc sửa tài liệu.
   - **Commenter:** cho người cần đọc và góp ý nhưng không được sửa nội dung.
   - **Viewer:** cho người chỉ cần đọc hoặc tải bản đã duyệt.
   - Không thêm nhà đầu tư làm thành viên của Shared Drive; chỉ chia sẻ trực tiếp các file cần thiết.
6. Đặt các thư mục và file chứa API key, token, dữ liệu khách hàng, tài chính chi tiết ở chế độ **Restricted**. Tuyệt đối không dùng **Anyone with the link**.
7. Tách file nguồn và file phát hành. Ví dụ: giữ Slide Pitch gốc ở 01_Tai_lieu_chung, xuất một bản PDF đã duyệt sang 05_Chia_se_ben_ngoai.
8. Chia sẻ Business Plan và PDF Slide Pitch cho đúng email nhà đầu tư, đặt **Expiration date = 14 ngày** kể từ ngày cấp quyền.
9. Kiểm tra bằng tài khoản đại diện cho từng vai trò: mở file, sửa, bình luận, tải xuống, in và chia sẻ lại. Sửa ngay mọi quyền thừa.
10. Kiểm tra Version history của các tài liệu quan trọng và bảo đảm quy trình lưu trữ giữ được các phiên bản cần thiết trong ít nhất 30 ngày.
11. Khi có thay đổi nhân sự, rà soát quyền ở thư mục cha, thư mục con, file chia sẻ trực tiếp và Google Groups.
12. Khi một người nghỉ việc, thu hồi quyền ngay, đổi credential liên quan và kiểm tra lại bằng một tài khoản thử.

## Phần 2: Triển khai và chống lỗi

### 4. Bảng phân quyền hoàn chỉnh

Quy ước: **Sửa = Editor**, **Bình luận = Commenter**, **Xem = Viewer**, **Không truy cập = không cấp quyền**.

| Loại tài liệu | Ban KT | Ban TK | Ban KD | Nhà đầu tư | Ghi chú bảo mật |
|---|---|---|---|---|---|
| Tài liệu kỹ thuật: kiến trúc, API spec, test case | Sửa | Không truy cập | Xem khi cần | Không truy cập | Restricted; không ghi API key, token hoặc dữ liệu khách hàng vào tài liệu dùng chung |
| Tài liệu thiết kế: wireframe, UI, logo, asset | Xem khi cần | Sửa | Xem khi cần | Không truy cập | File nguồn chỉ cho Ban Thiết kế sửa; không chia sẻ công khai |
| Business Plan, nghiên cứu thị trường, dự báo tài chính | Bình luận | Xem khi cần | Sửa | Xem, hết hạn sau 14 ngày | Tài liệu mật; nhà đầu tư chỉ xem bản được chọn, không cấp quyền sửa |
| Business Summary, Slide Pitch bản làm việc | Bình luận | Bình luận | Sửa | Không truy cập file nguồn | Chỉ chia sẻ nội bộ; kiểm soát quyền sửa ở file gốc |
| PDF Slide Pitch đã duyệt | Xem khi cần | Xem khi cần | Quản lý bản phát hành | Xem và tải, hết hạn sau 14 ngày | PDF riêng để chia sẻ ngoài; không chứa dữ liệu mật không cần thiết và không dùng link công khai |

Ban Kinh doanh là nhóm sở hữu nội dung Business Plan nên có quyền **Editor**. Ban Kỹ thuật chỉ cần đọc và đặt câu hỏi nên dùng **Commenter**; quyền này đáp ứng nhu cầu hiểu context mà không cho phép sửa nội dung. Nhà đầu tư không được truy cập các thư mục nội bộ hoặc file nguồn.

### 5. Trả lời 3 bẫy dữ liệu

#### Bẫy 1: Ban Kinh doanh muốn Ban Kỹ thuật đọc Business Plan nhưng không được sửa

Cấp quyền **Commenter** cho nhóm Ban Kỹ thuật trên riêng file Business Plan. Ban Kỹ thuật có thể đọc và để lại nhận xét, nhưng không thể thay đổi nội dung gốc. Nếu họ chỉ cần đọc mà không cần trao đổi thì dùng **Viewer** sẽ chặt chẽ hơn.

File vẫn phải ở chế độ **Restricted**, chia sẻ đúng nhóm email; không chia sẻ bằng link công khai.

#### Bẫy 2: Nhà đầu tư cần tải PDF để thuyết trình offline

Xuất Slide Pitch thành **một file PDF riêng**, đặt trong 05_Chia_se_ben_ngoai, rồi chia sẻ trực tiếp cho email nhà đầu tư với quyền **Viewer** và ngày hết hạn sau 14 ngày. Không chia sẻ Google Slides gốc nên nhà đầu tư không có bản chỉnh sửa được.

Với file PDF này, **không tắt** tùy chọn **Viewers can download**, vì nhà đầu tư đã được yêu cầu tải file để dùng offline. Việc cho tải PDF không đồng nghĩa với cho sửa file nguồn. Nếu chia sẻ một file Google Slides cho người chỉ xem, có thể tắt tùy chọn tải xuống, in và sao chép để giảm nguy cơ phát tán; nhưng tùy chọn đó không phù hợp với bản PDF mà nhà đầu tư cần tải.

#### Bẫy 3: Một thành viên nghỉ việc

Thực hiện ngay các bước sau:

1. Xóa người đó khỏi Shared Drive và khỏi mọi Google Group của CampusPay.
2. Thu hồi các quyền chia sẻ trực tiếp theo địa chỉ email ở cả thư mục và file riêng lẻ.
3. Nếu dùng Google Workspace của công ty, **suspend tài khoản** ngay để chặn đăng nhập; sau đó bàn giao file cho trưởng nhóm trước khi xử lý xóa tài khoản.
4. Đổi hoặc thu hồi API key, token, mật khẩu và credential mà người đó từng được xem.
5. Kiểm tra nhật ký hoạt động và thử đăng nhập bằng tài khoản đã thu hồi để chắc chắn không còn mở được tài liệu.

Chỉ xóa người khỏi một thư mục là chưa đủ, vì quyền có thể được kế thừa từ thư mục cha, cấp qua nhóm email hoặc cấp trực tiếp trên từng file. Vì vậy phải rà soát tất cả nguồn cấp quyền.

### Kết luận

Thiết kế này tách rõ tài liệu nội bộ, tài liệu phối hợp và tài liệu gửi ra ngoài. Quyền được cấp theo đúng nhu cầu công việc; tài liệu nhạy cảm luôn Restricted; nhà đầu tư chỉ nhận bản đã duyệt với quyền xem, được tải PDF và tự động hết hạn sau 14 ngày. Nhờ đó CampusPay vừa cộng tác thuận tiện vừa giảm đáng kể nguy cơ sửa nhầm hoặc rò rỉ dữ liệu.

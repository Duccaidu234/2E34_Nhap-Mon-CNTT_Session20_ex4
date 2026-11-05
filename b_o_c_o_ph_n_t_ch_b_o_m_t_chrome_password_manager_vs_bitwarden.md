# BÁO CÁO PHÂN TÍCH BẢO MẬT: GOOGLE CHROME PASSWORD MANAGER VS BITWARDEN

**Người lập:** Senior Security Engineer / IT Administrator

**Đối tượng áp dụng:** Toàn bộ nhân viên và hạ tầng CNTT Doanh nghiệp

**Mục tiêu:** Đánh giá an toàn trước mã độc Infostealer và đề xuất giải pháp quản lý mật khẩu chuẩn doanh nghiệp

---

## 1. BÁO CÁO SO SÁNH GIẢI PHÁP CHUYÊN SÂU

| Tiêu chí Đánh giá | Google Chrome Password Manager | Bitwarden Password Manager |
| :--- | :--- | :--- |
| **Khả năng chống Infostealer** | **Rất Thấp:** Mã độc Infostealer (RedLine, Vidar, Raccoon...) dễ dàng khai thác file `Login Data` cục bộ thông qua các DPAPI (Data Protection API) của Windows khi phiên người dùng đang hoạt động. | **Rất Cao:** Dữ liệu trong Kho lưu trữ (Vault) được mã hóa bằng AES-CBC 256-bit kết hợp PBKDF2/Argon2. Ngay cả khi bị lấy cắp toàn bộ file dữ liệu cục bộ, kẻ tấn công không thể giải mã nếu thiếu Master Password. |
| **Kiến trúc Zero-Knowledge** | **Không tuyệt đối (Phụ thuộc cấu hình):** Mặc định Google có thể tiếp cận chìa khóa giải mã trên Cloud (trừ khi người dùng kích hoạt thủ công tính năng *On-device encryption*, vốn ít được quản trị tập trung dễ dàng). | **Có tuyệt đối (Pure Zero-Knowledge):** Tất cả quá trình mã hóa/giải mã diễn ra hoàn toàn tại phía thiết bị (Client-side). Server của Bitwarden chỉ lưu ciphertext và không bao giờ biết Master Password. |
| **Đồng bộ chéo nền tảng** | **Hạn chế trong hệ sinh thái:** Hoạt động tối ưu trên Chrome/Android. Rất bất tiện và thiếu đồng bộ nếu nhân viên dùng Safari, Firefox, Edge hoặc ứng dụng Native trên macOS/Linux. | **Độc lập nền tảng hoàn toàn:** Hỗ trợ ứng dụng Native trên Windows, macOS, Linux, iOS, Android, mọi trình duyệt web (Web Extension) và giao diện dòng lệnh (CLI). |
| **Quản trị chính sách doanh nghiệp (Enterprise Policy)** | **Hạn chế:** Phụ thuộc vào Google Workspace Admin Console hoặc Group Policy (GPO), khó kiểm soát chi tiết tính năng chia sẻ mật khẩu giữa các team. | **Chuyên sâu:** Hỗ trợ thiết lập Master Password Complexity, Bắt buộc 2FA/FIDO2, Auto-lock duration, phân quyền RBAC và chia sẻ mật khẩu an toàn theo nhóm (Organizations). |

---

## 2. PHÂN TÍCH CHUYÊN SÂU: TẠI SAO BITWARDEN THẮNG THẾ VỀ BẢO MẬT?

Dù Bitwarden yêu cầu nhân viên phải thực hiện thêm thao tác nhập **Master Password** (hoặc xác thực sinh trắc học PIN/FaceID), sự đánh đổi nhỏ về tính tiện lợi này mang lại giá trị an ninh tối thượng cho doanh nghiệp dựa trên các căn cứ kỹ thuật sau:

### 2.1. Triệt tiêu nguy cơ từ Cơ chế Hoạt động của Mã độc Infostealer

* **Điểm yếu cốt lõi của Trình duyệt (Chrome):**
  * Chrome được thiết kế tối ưu cho trải nghiệm người dùng ("Convenience First"). File `Login Data` của Chrome lưu trên đĩa cứng sử dụng cơ chế DPAPI của hệ điều hành. Khi người dùng đã đăng nhập vào Windows/macOS, bất kỳ tiến trình (Process) nào chạy dưới quyền người dùng đó — bao gồm cả mã độc Infostealer từ tệp đính kèm email/file crack — đều có thể gọi API hệ điều hành để trích xuất toàn bộ mật khẩu dạng rõ (Plaintext) chỉ trong vài giây.

* **Cơ chế Bảo vệ Tích cực của Bitwarden:**
  * Bitwarden không bao giờ lưu trữ khóa giải mã cố định trên ổ đĩa. Khi vault bị khóa, toàn bộ dữ liệu chỉ là các chuỗi vô nghĩa (Ciphertext).
  * Việc nhập Master Password chính là hành động **tạo ra Chìa khóa Giải mã (Encryption Key) tạm thời trong RAM**. Khi vault đóng lại hoặc hết thời gian chờ (Timeout), chìa khóa này lập tức bị hủy khỏi bộ nhớ RAM. Mã độc Infostealer dù có quét sạch ổ cứng cũng chỉ thu được dữ liệu đã mã hóa không thể phá giải.

### 2.2. Kiểm soát Phiên làm việc (Auto-Lock) và Nguy cơ Mất thiết bị vật lý

* Chrome giữ toàn bộ mật khẩu ở trạng thái sẵn sàng điền (Auto-fill) suốt toàn bộ thời gian trình duyệt mở. Nếu nhân viên rời khỏi bàn làm việc mà không khóa máy (`Win + L`), bất kỳ ai cũng có thể mở phần cài đặt Chrome để xem mật khẩu.
* Bitwarden thiết lập chính sách **Auto-Lock (Tự động khóa)** sau $N$ phút không hoạt động hoặc khi màn hình máy tính bị khóa. Điều này thiết lập ranh giới bảo mật chủ động (Active Security Boundary) ngăn chặn cả tấn công mạng lẫn truy cập vật lý trái phép.

### 2.3. Tách biệt Vai trò (Separation of Powers) giữa Trình duyệt và Trình quản lý Mật khẩu

* Việc dùng chung trình duyệt để vừa lướt web (nơi có nhiều nguy cơ bị khai thác lỗ hổng Zero-day, Malvertising, Phishing) vừa là nơi cất giữ "chìa khóa" toàn bộ tài khoản công ty là cấu trúc bảo mật cực kỳ rủi ro.
* Sử dụng Bitwarden tạo ra một **lớp cách ly (Isolation Layer)**: Trình duyệt chỉ đóng vai trò hiển thị UI, còn quyền xác thực và dữ liệu nhạy cảm được quản lý độc lập bởi một ứng dụng chuyên biệt đã qua kiểm toán an ninh nghiêm ngặt.

---

## 3. KẾT LUẬN & ĐỀ XUẤT HÀNH ĐỘNG

1. **Bắt buộc triển khai Bitwarden Enterprise** làm chuẩn quản lý mật khẩu duy nhất cho toàn bộ nhân viên.
2. **Ban hành chính sách GPO/Google Workspace:** Tắt hoàn toàn tính năng `Offer to save passwords` trên trình duyệt Chrome trong toàn bộ hệ thống máy tính công ty.
3. **Cấu hình chính sách Bitwarden:** 
   * Đặt Master Password tối thiểu 14 ký tự (kết hợp Passphrase).
   * Bắt buộc bật 2FA/TOTP khi mở khóa Vault trên thiết bị mới.
   * Thiết lập thời gian `Vault Timeout` tối đa là 15-30 phút.
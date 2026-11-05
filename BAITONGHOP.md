# Báo cáo Phân tích & Xử lý Sự cố An ninh Mạng: Dự án "CyberRealm"
*Đơn vị: PixelForge Studio*

## Phần 1: Phân tích Chuỗi Tấn Công & Lỗ Hổng Hệ Thống
### **Chuỗi tấn công (Kill Chain) của Hacker**
Quá trình hacker xâm nhập và phá hoại hệ thống diễn ra theo các bước sau:

Initial Access (Xâm nhập ban đầu): Dev trực ca tìm và tải phần mềm chỉnh sửa âm thanh crack, đồng thời tự ý tắt Windows Defender (bỏ qua lớp phòng thủ Endpoint).

Execution (Thực thi mã độc): File crack chứa mã độc ẩn (thường là Trojan/Infostealer) được kích hoạt khi dev chạy cài đặt phần mềm.

Credential Access (Đánh cắp danh tính): Mã độc quét toàn bộ máy tính của dev, trích xuất dữ liệu từ trình duyệt (Cookie phiên đăng nhập GitHub, mật khẩu lưu trên trình duyệt) và quét các file text cục bộ chứa Secret chưa được mã hóa.

Exfiltration & Abuse (Tuồn dữ liệu và Phá hoại):

GitHub: Hacker chiếm được Session Cookie hoặc Personal Access Token (PAT) của GitHub từ máy dev, dùng IP lạ đăng nhập và thực thi lệnh xóa nhánh main.

OpenAI: API Key bị lộ do dev vô tình push nhầm lên commit trước đó (hoặc bị mã độc đọc được từ máy cục bộ). Các bot quét tự động của hacker lập tức bắt được key này và sử dụng để spam API, gây thiệt hại 3.000 USD và phá hoại hội thoại NPC.

### **Giải mã tốc độ thu thập API Key và Quy trình khắc phục**
Nguyên nhân lộ nhanh: Các nhóm tin tặc thường triển khai các con bot tự động cào (scan) dữ liệu 24/7 trên các kho lưu trữ public của GitHub. Ngay khi một commit chứa các tiền tố như sk-... (chuỗi định dạng của OpenAI) được push lên, bot sẽ lập tức bắt được và tự động phân phối cho các kịch bản khai thác chỉ trong vòng vài giây đến vài phút.

Các bước kỹ thuật sửa chữa khẩn cấp:

Trên trang OpenAI Dashboard: Đăng nhập ngay lập tức, vào mục API Keys, tìm đến Key bị lộ và nhấn Revoke (Thu hồi) hoặc Delete. Tạo ngay một Secret Key mới.

Xử lý tệp .env: Tạo file .env ở thư mục gốc của dự án, chuyển giá trị OPENAI_API_KEY=sk-new_key_here... vào file này và lưu lại cục bộ.

Xử lý tệp .gitignore: Mở tệp .gitignore, thêm dòng .env vào danh sách để đảm bảo Git bỏ qua, tuyệt đối không track và không push file này lên GitHub nữa.

Làm sạch lịch sử Git: Chạy công cụ như git filter-repo hoặc BFG Repo-Cleaner để xóa sạch dấu vết của commit chứa key cũ khỏi lịch sử Git.

## Phần 2: Cấp Cứu Danh Tính & Dựng Lá Chắn Xác Thực
### **Quy trình Cấp cứu 4 Bước cho máy trạm nhiễm mã độc**
Dành cho máy của dev trực ca:

Bước 1: Cô lập mạng (Isolation): Rút ngay cáp mạng LAN hoặc ngắt kết nối Wi-Fi lập tức để chặn mã độc gửi thêm dữ liệu ra máy chủ của hacker.

Bước 2: Cấp cứu danh tính trên thiết bị sạch: Sử dụng một máy tính khác (hoặc điện thoại) an toàn, lập tức đổi toàn bộ mật khẩu các tài khoản trọng yếu (GitHub, Google Workspace, Steamworks...) và buộc đăng xuất (Force Sign-out) mọi phiên làm việc hiện tại.

Bước 3: Thu hồi đặc quyền: Admin tạm thời vô hiệu hóa quyền truy cập repo GitHub và AWS/Steam của tài khoản dev bị nhiễm.

Bước 4: Xóa sổ thiết bị (Wipe & Reinstall): Tuyệt đối không dùng phần mềm diệt virus quét và dùng tiếp. Bắt buộc format (định dạng) lại toàn bộ ổ cứng và cài mới lại hệ điều hành sạch 100%.

### **Chặn tấn công Credential Stuffing trên Steamworks**
Hướng dẫn thao tác cho Trưởng nhóm:

Tạo mật khẩu bằng Bitwarden: Mở extension Bitwarden, dùng công cụ Password Generator, thiết lập độ dài 30, tích chọn đủ chữ hoa, chữ thường, số, và ký tự đặc biệt để tạo ra một mật khẩu ngẫu nhiên (VD: X7#v9pL$2kM!qW8zR@4nY*6cT&3bF^).

Đổi mật khẩu Steamworks: Đăng nhập vào Steamworks, vào phần đổi mật khẩu, paste mật khẩu mới vừa tạo và lưu lại.

Lưu trữ vào Két sắt: Tạo mục mới trong Vault của Bitwarden, lưu tài khoản Steamworks cùng mật khẩu 30 ký tự này.

Kích hoạt 2FA TOTP: Trong cài đặt bảo mật Steamworks, bật Xác thực 2 bước. Mở app Google Authenticator trên điện thoại, quét mã QR hiển thị trên màn hình máy tính. Nhập mã 6 số sinh ra để hoàn tất. Lưu các Backup Codes vào phần Secure Note của Bitwarden.

### **Cơ chế bảo vệ của 2FA TOTP trước Credential Stuffing**
Credential Stuffing là kỹ thuật dùng tool tự động thử hàng loạt tổ hợp "Tên đăng nhập + Mật khẩu" bị rò rỉ. Nếu có 2FA TOTP (Time-Based One-Time Password), dù hacker có dò đúng mật khẩu (Yếu tố thứ nhất - Something you know), chúng vẫn sẽ bị chặn lại ở bước nhập mã OTP. Mã này chỉ sinh ra cứ mỗi 30 giây một lần trên thiết bị vật lý cầm tay của bạn (Yếu tố thứ hai - Something you have), khiến việc có mật khẩu trở nên vô dụng nếu không cầm điện thoại của nạn nhân.

## Phần 3: Bài Học Xương Máu & Tái Cấu Trúc An Ninh (Tổng Kết)
### **Bản Thông Cáo Nội Bộ PixelForge Studio**
THÔNG BÁO KHẨN: CHÍNH SÁCH BẢO MẬT KHÔNG GIAN SỐ BẮT BUỘC

Kính gửi toàn thể đội ngũ PixelForge Studio,
Sự cố nghiêm trọng sáng nay đã gây thiệt hại nặng nề, khởi nguồn từ một lỗ hổng cá nhân. Qua đây, công ty đúc kết 3 bài học sống còn: (1) Tuyệt đối không có ngoại lệ cho mã độc và crack, (2) Sự tiện lợi cục bộ không bao giờ được đánh đổi bằng rủi ro Secrets, và (3) 2FA là lá chắn cuối cùng cứu mạng hệ thống.

Từ hôm nay, công ty ban hành chính sách bắt buộc:

Cấm tuyệt đối 100%: Sử dụng phần mềm crack, tải lậu hoặc tự ý tắt Windows Defender trên máy làm việc.

Xác thực toàn diện: Bắt buộc sử dụng Bitwarden Password Manager sinh mật khẩu mạnh và bật 2FA (TOTP) cho toàn bộ tài khoản công việc.

Zero-Knowledge với Code: Mọi API Keys, Credentials phải nằm trong .env, quản lý cục bộ và bắt buộc được loại trừ bởi .gitignore. Tuyệt đối không commit Secret lên repo.

Bất kỳ vi phạm nào với các quy tắc trên sẽ dẫn đến hình thức kỷ luật cao nhất. Đề nghị các Trưởng nhóm rà soát ngay lập tức!Ban Giám Đốc PixelForge Studio.
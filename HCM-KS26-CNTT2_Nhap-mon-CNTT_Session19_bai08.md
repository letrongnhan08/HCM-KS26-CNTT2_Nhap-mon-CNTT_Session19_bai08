# Phần 1: Phân tích hậu quả & Sai lầm kỹ thuật

 **Chỉ rõ 4 Secrets nhạy cảm đã bị phơi bày công khai trong đoạn code trên:**
 
```
// HARDCODE TRỰC TIẾP CÁC CHÌA KHÓA BÍ MẬT VÀO MÃ NGUỒN:
const OPENAI_API_KEY = 'sk-proj-********';
const STEAM_WEB_API_KEY = 'STEAM_SECRET_KEY_********';
const MONGO_URI = 'mongodb+srv://admin_root:********@cluster0.pixelforge.net/cyberrealm_db';
const JWT_SECRET = '********';
```

**Phân tích hậu quả thảm họa nếu hacker nắm được từng chìa khóa đó:**
- OPENAI_API_KEY : hacker dùng key để gọi API làm tăng ngân sách của dự án
- STEAM_WEB_API_KEY : thu thập thông tin người dung, mã giao dịch ở ứng dụng Steam
- MONGO_URI: Truy cập, đọc, sửa, xóa toàn bộ dữ liệu cyberrealm_db; nguy cơ lộ dữ liệu người chơi
-  JWT_SECRET:Giả mạo token để xâm nhập, chiếm quyền bất kỳ tài khoản nào.

**Giải thích dù có xóa commit hoặc chuyển repo sang chế độ Private sau 10 phút thì chìa khóa đó coi như đã bị lộ và phải thu hồi (Revoke) ngay?**

- Vì khi commit hoặc chuyển repo sang chế độ Private vẫn bị Scraper Bot chỉ tầm 2 giây quét và chế độ Private thì không còn giá trị gì nữa khi đã bị bot của hacker lấy hết thông tin.
- Dù xóa commit thì vẫn còn dữ liệu khi Force-push không thể xóa phiên bản Lịch sử Git (Git History) trước đó, vẫn có thể nhìn thấy được file phiên bản cũ
- Việc thu hồi (Revoke) ngay giúp PixelForge Studio bảo mật và biến những dữ liệu đó không còn giá trị cho hacker
# Phần 2: Sửa lỗi & Chuẩn hóa an ninh

**Viết nội dung hoàn chỉnh cho tệp `.env` đặt trên máy cục bộ để lưu trữ 4 Secrets trên:**

Sửa tệp .env "" chỉ để trên máy cục bộ không commit lên GIT:

```
OPENAI_API_KEY=sk-proj-KEY_MOI_ABC
STEAM_WEB_API_KEY=KEY_MOI_ABC
MONGO_URI=mongodb+srv://USER_MOI_ABC:MAT_KHAU_MOI@cluster0.pixelforge.net/cyberrealm_db
JWT_SECRET=CHUOI_NGAU_NHIEN_DAI_32_KY_TU
```

**Viết lại đoạn mã server.js an toàn, sử dụng biến môi trường (ví dụ qua process.env.VARIABLE_NAME) thay vì hardcode trực tiếp:**

```
require('dotenv').config();
const express = require('express');
const app = express();

const OPENAI_API_KEY = process.env.OPENAI_API_KEY;
const STEAM_WEB_API_KEY = process.env.STEAM_WEB_API_KEY;
const MONGO_URI = process.env.MONGO_URI;
const JWT_SECRET = process.env.JWT_SECRET;

for (const k of ['OPENAI_API_KEY', 'STEAM_WEB_API_KEY', 'MONGO_URI', 'JWT_SECRET']) {
  if (!process.env[k]) {
    console.error(`Thiếu biến môi trường: ${k}`);
    process.exit(1);
  }
}

app.listen(3000, () => console.log('Game Server running on port 3000'));
```

**Viết nội dung cho tệp .gitignore để đảm bảo tệp .env không bao giờ bị Git theo dõi:**

```
.env
.env.*
.env.example
node_modules/
```

**Viết nội dung tệp .env.example chuẩn mẫu để commit và push lên GitHub cho các thành viên khác trong nhóm:**

```
OPENAI_API_KEY=your_openai_key_here
STEAM_WEB_API_KEY=your_steam_key_here
MONGO_URI=mongodb+srv://<user>:<password>@<host>/<database>
JWT_SECRET=your_random_secret_here
```


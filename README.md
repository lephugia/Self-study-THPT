# StudyMate THPT — tài khoản, dữ liệu riêng và triển khai

StudyMate là website học tập tiếng Việt dành cho học sinh THPT: lịch học, bài tập, tài liệu/tóm tắt, flashcards, trắc nghiệm, theo dõi điểm và gia sư AI tùy chọn.

## Chức năng tài khoản đã bổ sung

- Đăng ký bằng tên hiển thị, email và mật khẩu.
- Đăng nhập/đăng xuất bằng cookie phiên `HttpOnly`, `SameSite=Lax`; phiên hết hạn sau 14 ngày.
- Mật khẩu được băm bằng `scrypt` với salt riêng, không lưu mật khẩu dạng văn bản.
- Dữ liệu học tập được lưu vào PostgreSQL, tách theo tài khoản; lịch học, tài liệu, thẻ ôn tập và lịch sử điểm không dùng chung giữa học sinh.
- Tự tạo bảng dữ liệu khi máy chủ khởi động.
- Khi chưa cấu hình `DATABASE_URL`, website chạy ở chế độ demo; đăng ký, đăng nhập và đồng bộ đám mây sẽ không hoạt động.

## Chạy trên máy tính để thử

Yêu cầu Node.js 20 trở lên và PostgreSQL.

1. Tạo cơ sở dữ liệu PostgreSQL (xem hướng dẫn triển khai bên dưới) rồi lấy chuỗi kết nối.
2. Sao chép `.env.example` thành `.env`, điền `DATABASE_URL`. Chỉ đặt `OPENAI_API_KEY` nếu bạn muốn dùng AI trực tuyến.
3. Mở Terminal/PowerShell tại thư mục dự án và chạy:

   ```bash
   npm install
   npm start
   ```

4. Mở `http://localhost:3000` rồi tạo tài khoản.

Không tải `.env` lên GitHub và không gửi chuỗi kết nối database cho người khác.

## Cách đưa website lên Internet (đề xuất)

Dự án sử dụng Node.js/Express nên có thể chạy dưới dạng Web Service trên Render. Render cấp cho mỗi dịch vụ một địa chỉ `onrender.com`; sau này có thể gắn tên miền riêng. Hướng dẫn chính thức: https://render.com/docs/deploy-node-express-app và https://render.com/docs/custom-domains.

### A. Tạo cơ sở dữ liệu PostgreSQL trên Neon

1. Mở https://neon.tech/ và tạo tài khoản/dự án PostgreSQL mới.
2. Trong Neon Console, nhấn **Connect** và sao chép chuỗi kết nối PostgreSQL (ưu tiên pooled connection string). Tham khảo: https://neon.com/docs/get-started/connect-neon.
3. Giữ chuỗi này riêng tư. Bạn sẽ dán nó vào mục biến môi trường của dịch vụ web ở bước C, không cần gửi cho bất kỳ ai.

### B. Đưa bộ mã nguồn lên GitHub

1. Tạo một repository mới, ví dụ `studymate-thpt`. Chọn **Private** nếu bạn không muốn công khai mã nguồn.
2. Tải nội dung thư mục này lên repository, giữ nguyên `server.js`, `package.json`, `public/index.html` và các file hướng dẫn. Không tải `.env` hoặc `node_modules` lên.
3. Nếu sử dụng Git trên máy tính, chạy từ thư mục dự án:

   ```bash
   git init
   git add .
   git commit -m "Prepare StudyMate for deployment"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/studymate-thpt.git
   git push -u origin main
   ```

   Thay `YOUR-USERNAME` bằng tên tài khoản GitHub của bạn.

### C. Tạo địa chỉ website trên Render

1. Mở https://render.com/ và đăng ký/đăng nhập.
2. Chọn **New → Blueprint** (nếu có) và kết nối repository có file `render.yaml`; hoặc chọn **New → Web Service** rồi kết nối GitHub. Blueprint đã điền sẵn lệnh build/start và health check.
3. Nếu tự tạo Web Service, điền cấu hình:
   - **Runtime/Language:** Node
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Health Check Path:** `/api/health` (nếu giao diện có trường này)
4. Trong **Environment Variables**, thêm:
   - `DATABASE_URL` = chuỗi kết nối PostgreSQL từ Neon
   - `NODE_ENV` = `production`
   - `OPENAI_API_KEY` = khóa API (chỉ khi đã muốn bật AI; có thể bỏ qua)
   - `OPENAI_MODEL` = `gpt-5-mini` (tùy chọn, khi bật AI)
5. Chọn tạo dịch vụ. Render cần biến `DATABASE_URL` hợp lệ ngay lần khởi động đầu. Khi triển khai thành công, Render hiển thị địa chỉ dạng `https://TEN-DICH-VU.onrender.com`. Tên con cụ thể chỉ xuất hiện sau khi bạn tạo dịch vụ trong tài khoản Render của mình.
6. Mở địa chỉ đó, tạo tài khoản mới và thử đăng xuất/đăng nhập lại để xác nhận dữ liệu học tập được tải từ cơ sở dữ liệu.

**Lưu ý về gói miễn phí:** Render có thể cho dịch vụ miễn phí ngủ sau 15 phút không hoạt động và mất khoảng một phút để khởi động lại khi có lượt truy cập mới. Neon có hạn mức miễn phí và tự tạm ngừng compute khi nhàn rỗi; hãy kiểm tra giới hạn hiện hành của các nhà cung cấp trước khi sử dụng lâu dài. Không sử dụng cấu hình thử nghiệm miễn phí cho dữ liệu nhạy cảm hoặc dữ liệu trường/lớp mà bạn không có quyền lưu.

### D. Dùng tên miền riêng (không bắt buộc)

Bạn không cần mua tên miền để bắt đầu; địa chỉ `onrender.com` đủ để thử nghiệm. Nếu sau này mua tên miền, vào **Settings → Custom Domains** của Render, thêm tên miền rồi cấu hình DNS theo hướng dẫn mà Render cung cấp. TLS/HTTPS được Render tự cấu hình cho tên miền đã xác minh. Hướng dẫn: https://render.com/docs/custom-domains.

## Bảo mật và giới hạn cần biết

- Đây là bản MVP: chưa xác minh email, chưa có quên/đặt lại mật khẩu qua email, chưa có tài khoản giáo viên/quản trị, và chưa có xác thực hai lớp. Trước khi công khai rộng rãi cần bổ sung các quy trình này.
- Không thu thập dữ liệu nhạy cảm, mật khẩu trường học hoặc thông tin không cần thiết của học sinh. Dùng mật khẩu riêng, mạnh.
- Chấm bài bằng AI có thể không chính xác; đối chiếu với giáo viên và tài liệu chính thức.
- Khóa OpenAI API phải được lưu dưới biến môi trường máy chủ, tuyệt đối không đưa vào mã giao diện công khai.
- Dữ liệu có tài khoản nằm trong PostgreSQL và được phân tách theo user ID ở API. Cần cấu hình sao lưu cho cơ sở dữ liệu nếu website dùng cho học tập lâu dài.

## API

- `GET /api/health` — tình trạng máy chủ, AI và cơ sở dữ liệu tài khoản.
- `GET /api/auth/config` — cho giao diện biết hệ thống tài khoản đã được cấu hình hay chưa.
- `POST /api/auth/register` — tạo tài khoản.
- `POST /api/auth/login` — đăng nhập.
- `GET /api/auth/me` — thông tin tài khoản hiện tại.
- `POST /api/auth/logout` — kết thúc phiên.
- `GET /api/data` / `PUT /api/data` — tải/lưu dữ liệu học tập của tài khoản hiện tại.
- `POST /api/chat`, `POST /api/summarize`, `POST /api/grade` — tính năng AI; cần `OPENAI_API_KEY` để dùng trực tuyến.

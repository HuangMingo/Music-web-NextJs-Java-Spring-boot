# Music Player React

Ứng dụng nghe nhạc xây bằng React + Vite cho frontend và Express + PostgreSQL cho backend. Dự án có các chức năng chính như nghe nhạc, tìm kiếm, playlist cá nhân, yêu thích bài hát/album/playlist/nghệ sĩ, trang chi tiết nghệ sĩ và trang quản trị nội dung.

## Trang đã deploy

Truy cập bản online tại:

```txt
https://frontend-rmqd.onrender.com
```

Lưu ý: nếu server deploy đang ngủ, lần mở đầu tiên có thể mất vài chục giây để khởi động lại.

## Công nghệ sử dụng

- Frontend: React, Vite, React Router, Axios
- Backend: Node.js, Express
- Database: PostgreSQL
- Upload media: Cloudinary
- UI assets: Bootstrap Icons, Font Awesome, CSS thuần

## Yêu cầu trước khi chạy local

Cài sẵn:

- Node.js
- npm
- pnpm
- PostgreSQL database hoặc một database PostgreSQL online như Neon

## Cài đặt

Tại thư mục gốc dự án, chạy:

```bash
npm install
```

## Cấu hình môi trường

Frontend dùng file `.env` ở thư mục gốc:

```env
VITE_API_URL=http://localhost:3000
```

Backend dùng file `02-be/.env`:

```env
PORT=3000
DATABASE_URL=your_postgresql_connection_string
NODE_ENV=development
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
```

Nếu không upload file, thông tin Cloudinary có thể chưa cần chỉnh ngay. Database URL là phần bắt buộc để backend đọc/ghi dữ liệu.

## Chạy local

`pnpm` dùng workspace để chạy các script. Cài dependency bằng `npm install` theo hướng dẫn ở trên.

Chạy frontend và backend cùng lúc từ thư mục gốc:

```bash
pnpm dev
```

Backend chạy tại:

```txt
http://localhost:3000
```

Frontend thường chạy tại:

```txt
http://localhost:5173
```

Sau đó mở `http://localhost:5173` trên trình duyệt.

## Các script thường dùng

```bash
pnpm dev
```

Chạy frontend và backend ở chế độ development.

```bash
npm start
```

Chạy riêng backend Express. Có thể chạy lệnh này từ thư mục gốc hoặc trong `02-be`.

```bash
npm run build
```

Build frontend ra thư mục `dist`.

```bash
npm run preview
```

Xem thử bản build frontend.

```bash
npm run lint
```

Chạy ESLint cho toàn dự án.

Trên Windows PowerShell, nếu `npm run ...` bị chặn bởi execution policy, dùng:

```bash
npm.cmd run dev
npm.cmd run build
npm.cmd run lint
```

## Cấu trúc thư mục chính

```txt
02-be/              Backend Express
02-be/config/       Cấu hình database
02-be/controllers/  Controller xử lý request
02-be/routes/       Khai báo API routes
02-be/services/     Logic truy vấn và xử lý dữ liệu
public/             Static assets
src/                Frontend React
src/components/     Components giao diện
src/context/        AuthContext, MusicContext
src/api.js          Cấu hình API URL cho frontend
```

## Ghi chú

- Frontend lấy API base URL từ `VITE_API_URL`.
- Backend đọc biến môi trường từ `02-be/.env`.
- API local mặc định chạy ở port `3000`.
- Frontend local mặc định chạy ở port `5173`.

11/7/2026
- Xử lí việc chuyển đổi sang HTTP Range request
- Nghiên cứu liệu Nodejs có phù hợp không. có thể chuyển đổi sang kiểu khác không

### 23/11/2026 — Các điểm cần cải thiện

- **Phát nhạc.** Đo thời gian từ lúc bấm Play đến khi có tiếng và kiểm tra thao tác tua. Với bài hát công khai, thử phát trực tiếp từ CDN Cloudinary. Nếu giữ proxy Express, dùng `pipeline()` để chuyển tiếp audio ngay khi nhận được và xem xét cache metadata để giảm số lần gọi `HEAD`.
- **Trạng thái yêu thích.** Gộp các request kiểm tra từng bài trên trang Khám phá thành một request và một truy vấn PostgreSQL cho cả danh sách bài hát.
- **Top 10 và trang Khám phá.** Thử cache Top 10 trong 30–60 giây. Với một tiến trình Express, có thể dùng cache trong bộ nhớ. Cân nhắc Redis khi cần chia sẻ cache giữa nhiều instance. Đo chi phí của `ORDER BY RANDOM()` trước khi cache danh sách ngẫu nhiên, vì người dùng sẽ thấy cùng một danh sách trong thời gian cache còn hiệu lực.
- **Tìm kiếm.** Kiểm tra các truy vấn `unaccent(...) ILIKE` bằng `EXPLAIN (ANALYZE, BUFFERS)`. Nếu truy vấn chậm, cân nhắc cột tìm kiếm đã bỏ dấu với index GIN dùng `pg_trgm`. Hủy request gợi ý cũ khi người dùng nhập từ khóa mới.
- **Đo hiệu năng.** Ghi lại độ trễ API ở phân vị 95 (p95) và thời gian bắt đầu phát nhạc trước và sau mỗi thay đổi để so sánh.

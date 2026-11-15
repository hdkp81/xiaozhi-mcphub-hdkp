# xiaozhi-mcphub-hdkp: Bản fork cá nhân của xiaozhi-mcphub

[中文版](README.zh.md) | [English](README.md) | Tiếng Việt

> **Forked from** [huangjunsen0406/xiaozhi-mcphub](https://github.com/huangjunsen0406/xiaozhi-mcphub) của tác giả Huang Junsen.
> Tham khảo bản địa hóa tiếng Việt: [hdkp81/xiaozhi-mcphub-vn](https://github.com/hdkp81/xiaozhi-mcphub-vn).
> Bản fork này giữ nguyên `LICENSE` (Apache License 2.0) và `NOTICE` của dự án gốc.

**xiaozhi-mcphub-hdkp** là phiên bản cá nhân của xiaozhi-mcphub, một cầu nối MCP (Model Context Protocol) kèm bảng điều khiển quản lý, được tối ưu cho nền tảng Xiaozhi AI. Dự án hỗ trợ quản lý nhiều endpoint, tự động kết nối lại, định tuyến thông minh dựa trên vector và truy cập tương thích OpenAPI.

![Dashboard Preview](assets/dashboard.png)

## 🔄 Thay đổi trong bản fork này

So với dự án gốc, bản fork này:

- Build image Docker từ mã nguồn (`build: .`) thay vì tải image dựng sẵn.
- Bắt buộc đặt `JWT_SECRET` trong `.env` (Docker Compose sẽ dừng và báo rõ nếu thiếu).
- Hỗ trợ đăng nhập bằng Google OAuth (callback của Better Auth: `/api/auth/better/callback/google`).
- Dùng tên gói, repository và trang chủ cá nhân trong `package.json`.
- (Ghi thêm các thay đổi khác của bạn tại đây.)

Toàn bộ tính năng của bản gốc bên dưới vẫn được giữ nguyên.

## 🚀 Tính năng chính

- **Tích hợp Xiaozhi nâng cao**:
  - **Quản lý nhiều endpoint**: kết nối WebSocket hai chiều với nhiều endpoint Xiaozhi (bật/tắt, chỉnh sửa, kết nối lại, xem trạng thái).
  - **Kết nối lại thông minh**: chế độ kết nối lại nhanh, tăng thời gian chờ theo cấp số nhân, thử lại vô hạn kèm khoảng nghỉ, tất cả cấu hình được qua biến môi trường.
  - **Đồng bộ công cụ**: các endpoint Xiaozhi được thông báo khi công cụ của server thay đổi để danh sách luôn cập nhật.
  - **Nhóm và định tuyến thông minh**: endpoint có thể gắn với nhóm hoặc dùng định tuyến thông minh `$smart`.

- **Quản lý MCP (kế thừa và nâng cao)**:
  - Quản lý các MCP server chuẩn (chế độ stdio / SSE / HTTP).
  - Bật/tắt server, công cụ và prompt, kèm mô tả và phân nhóm.
  - Có cổng MCP thống nhất, cổng theo nhóm và endpoint riêng cho từng server.

- **Bảng điều khiển và xác thực**:
  - Giao diện quản lý (React + Vite + Tailwind) gồm Servers, Groups, Users, Logs, Settings, Xiaozhi Endpoints và Market.
  - Xác thực bằng JWT, có middleware ngữ cảnh người dùng và tài khoản admin tích hợp sẵn.

- **OpenAPI và gọi công cụ trực tiếp**:
  - Cung cấp tài liệu OpenAPI và các endpoint thống kê.
  - Gọi công cụ của từng server qua endpoint tương thích OpenAPI.

## 🔧 Bắt đầu nhanh

### Cách 1: Docker Compose (khuyên dùng)

Repo có sẵn `docker-compose.yml` gồm cả `pgvector` và ứng dụng. Image ứng dụng được build tại máy từ `Dockerfile`.

1. Tạo file `.env` cạnh `docker-compose.yml` (không bao giờ commit file này):

   ```env
   JWT_SECRET=thay-bang-chuoi-ngau-nhien-dai
   DB_PASSWORD=thay-bang-mat-khau-database
   # Google OAuth (chỉ cần nếu dùng đăng nhập Google)
   # Kiểm tra tên biến mà mã nguồn yêu cầu trước khi điền.
   GOOGLE_CLIENT_ID=client-id-cua-ban
   GOOGLE_CLIENT_SECRET=client-secret-cua-ban
   ```

2. Build và khởi động:

   ```bash
   docker compose up -d --build

   # Xem log (tùy chọn)
   docker compose logs -f mcphub
   ```

   Sau lần build đầu, chỉ cần `docker compose up -d`. Chỉ dùng lại `--build` khi sửa mã nguồn hoặc `Dockerfile`.

3. Mở bảng điều khiển tại <http://localhost:3000>.

Các biến chính (chỉnh trong compose hoặc `.env` nếu cần):

- `DB_URL`: `postgres://xiaozhi:<mật-khẩu>@db:5432/xiaozhi_mcphub` (biến cũ `DATABASE_URL` vẫn dùng được)
- `JWT_SECRET`: khóa bí mật JWT (bắt buộc trong bản fork này).
- `SMART_ROUTING_ENABLED`: bật/tắt định tuyến thông minh (mặc định "false").
- Tùy chọn: `BASE_PATH`, `OPENAI_API_KEY`, `OPENAI_API_BASE_URL`, `OPENAI_API_EMBEDDING_MODEL` (các biến OpenAI bắt buộc khi bật định tuyến thông minh).

Tài khoản admin mặc định: `admin` / `admin123` (hãy đổi mật khẩu sau lần đăng nhập đầu tiên).

### Cấu hình Google OAuth

1. Trong Google Cloud Console, mở **Google Auth Platform**, tạo **OAuth client** loại **Web application**.
2. Thêm **Authorized redirect URI** sau (phải khớp chính xác từng ký tự):

   ```
   http://localhost:3000/api/auth/better/callback/google
   ```

3. Khi ứng dụng còn ở trạng thái **Testing**, thêm tài khoản Google của bạn tại **Audience → Test users**.
4. Điền Client ID và Client Secret vào `.env`, rồi khởi động lại bằng `docker compose up -d`.

### Cách 2: Chạy phát triển trên máy

Yêu cầu: Node.js 24+, pnpm, PostgreSQL 16+ (nên dùng service `db` trong compose của repo).

```bash
git clone https://github.com/hdkp81/xiaozhi-mcphub-hdkp.git
cd xiaozhi-mcphub-hdkp
pnpm install

# Chạy database cục bộ (tùy chọn, dùng lại db của compose)
docker compose up -d db

# Đặt chuỗi kết nối database (hoặc ghi vào .env)
export DB_URL="postgres://xiaozhi:<mật-khẩu>@localhost:5432/xiaozhi_mcphub"

# Chạy đồng thời backend (:3000) và frontend (Vite :5173)
pnpm dev
```

Truy cập frontend dev tại `http://localhost:5173` (frontend chuyển tiếp yêu cầu tới backend `:3000`).

### Nâng cấp từ v1.0.3

Nếu bạn đã nhảy lên 1.1.x và các server biến mất, **đừng xóa volume Postgres**.
Từ 1.1.x trở đi, ứng dụng sẽ phát hiện bảng cũ `mcp_servers` (nếu còn) và sao chép các dòng còn thiếu sang bảng `servers` mới khi khởi động. Nên đổi tên biến môi trường:

```bash
# trước (v1.0.3)
DATABASE_URL=postgres://xiaozhi:…@db:5432/xiaozhi_mcphub

# sau (1.1.x), cùng tài khoản / cùng volume
DB_URL=postgres://xiaozhi:…@db:5432/xiaozhi_mcphub
```

Chi tiết: [documents/docs/configuration/upgrade-from-v103.md](documents/docs/configuration/upgrade-from-v103.md).

## 🔑 Đặt lại mật khẩu admin

Mật khẩu được lưu dạng băm nên không thể xem lại. Để đặt lại tài khoản admin tích hợp:

```bash
docker compose exec db psql -U xiaozhi -d xiaozhi_mcphub -c "DELETE FROM users WHERE username = 'admin';"
docker compose restart mcphub
```

Sau đó đăng nhập bằng `admin` / `admin123` và đổi mật khẩu ngay.

## 🗺️ Định tuyến thông minh (tùy chọn)

Đặt `SMART_ROUTING_ENABLED` thành `true` và cung cấp `OPENAI_API_KEY` để bật. Hệ thống dùng `pgvector` để lưu trữ và đánh chỉ mục vector. Nếu chưa có vector, bước dựng chỉ mục sẽ được bỏ qua và được điền sau bởi các dịch vụ vector.

## 🖥️ Tính năng bảng điều khiển (Frontend)

- Dashboard: tổng quan và trạng thái
- Servers: quản lý server và công cụ
- Groups: phân nhóm và gán quyền
- Users: người dùng và phân quyền (admin)
- Logs: nhật ký thời gian thực và lịch sử
- Settings: cấu hình hệ thống (gồm định tuyến thông minh)
- Xiaozhi Endpoints: quản lý endpoint Xiaozhi
- Market: tìm và cài đặt MCP server từ cộng đồng

## 📦 Mặc định và khởi tạo

- Admin mặc định: `admin` / `admin123`
- MCP server mặc định: amap / playwright / fetch / slack (có thể sửa trong bảng điều khiển và cấu hình qua ENV)

## 📄 Giấy phép (Apache License 2.0)

Dự án này là bản chỉnh sửa của [xiaozhi-mcphub](https://github.com/huangjunsen0406/xiaozhi-mcphub) và được cấp phép theo **Apache License 2.0**. Hãy giữ nguyên `LICENSE` và `NOTICE` khi phân phối lại, đồng thời ghi lại mọi chỉnh sửa (xem mục "Thay đổi trong bản fork này" ở trên). Xem [`LICENSE`](LICENSE) và [`NOTICE`](NOTICE).

## 🤝 Đóng góp

Hoan nghênh mọi đóng góp! Vui lòng mở issue/PR để cải thiện dự án.

## 🔗 Liên kết

- Bản fork này: <https://github.com/hdkp81/xiaozhi-mcphub-hdkp>
- Dự án gốc: <https://github.com/huangjunsen0406/xiaozhi-mcphub>
- Bản địa hóa tiếng Việt: <https://github.com/hdkp81/xiaozhi-mcphub-vn>
- Nền tảng Xiaozhi AI: <https://xiaozhi.me>
- Model Context Protocol: <https://modelcontextprotocol.io>

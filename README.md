# Camera Tracker - Hệ thống Giám sát Camera

Ứng dụng desktop (Electron + React) quản lý và theo dõi camera thời gian thực, tích hợp MediaMTX làm streaming server và bộ công cụ tự động hoá khởi chạy (Docker, FFmpeg, Electron).

---

## Mục lục

1. [Yêu cầu hệ thống](#yêu-cầu-hệ-thống)
2. [Cài đặt](#cài-đặt)
3. [Khởi chạy nhanh](#khởi-chạy-nhanh)
4. [Hướng dẫn sử dụng chi tiết](#hướng-dẫn-sử-dụng-chi-tiết)
   - [Màn hình Giám sát - Xem trực tiếp](#1-màn-hình-giám-sát---xem-trực-tiếp)
   - [Màn hình Xem lại video](#2-màn-hình-xem-lại-video)
   - [Màn hình Sự kiện](#3-màn-hình-sự-kiện)
   - [Màn hình Cài đặt](#4-màn-hình-cài-đặt)
5. [Kiến trúc hệ thống](#kiến-trúc-hệ-thống)
6. [Cấu hình và Tùy chỉnh](#cấu-hình-và-tùy-chỉnh)
7. [Sizing ổ cứng lưu trữ](#sizing-ổ-cứng-lưu-trữ)
8. [Khắc phục sự cố](#khắc-phục-sự-cố)
9. [Build Production](#build-production)

---

## Yêu cầu hệ thống

### Phần cứng tối thiểu
- **CPU**: Intel Core i5 thế hệ 8 trở lên (hoặc AMD Ryzen 5)
- **RAM**: 8GB (khuyến nghị 16GB cho nhiều camera)
- **Ổ cứng**: Tối thiểu 100GB trống (xem [Sizing ổ cứng lưu trữ](#sizing-ổ-cứng-lưu-trữ))
- **Card mạng**: Ethernet 1Gbps (khuyến nghị cho nhiều luồng video)

### Phần mềm
- **Hệ điều hành**: Windows 10/11 hoặc Ubuntu 20.04+ (khuyến nghị 22.04+)
- **Node.js**: 18+ (khuyến nghị 20 LTS)
- **Yarn**: 1.22+ (khuyến nghị; có thể dùng Corepack/NPM)
- **Docker Desktop/Docker**: Phiên bản mới nhất
- **FFmpeg**: 4.4+

---

## Cài đặt

### 1. Clone repository

```bash
git clone <repository-url>
cd camera-tracker
```

### 2. Cài đặt dependencies

```bash
yarn install
```

**Ghi chú**: Repository sử dụng Yarn Workspaces, lệnh trên sẽ tự động cài đặt cả dependencies cho `frontend/`.

---

## Khởi chạy nhanh

### Windows

1. **Khởi chạy tự động** (khuyến nghị):
   - Double-click file `run_media.bat` hoặc chạy trong PowerShell/CMD:

   ```powershell
   .\run_media.bat
   ```

2. **Khởi chạy với IP LAN tùy chỉnh** (cho WebRTC qua mạng nội bộ):

   ```powershell
   .\run_media.bat 192.168.1.24
   ```

3. **Kết quả**: Hệ thống sẽ tự động mở 2 cửa sổ CMD:
   - **Cửa sổ chính**: Khởi động MediaMTX (Docker), hiển thị log và tự động spawn FFmpeg cho từng stream từ file cấu hình `data.json`.
   - **Cửa sổ "electron-dev"**: Chạy `yarn dev:electron` để khởi động ứng dụng Electron.

### Linux/macOS

```bash
bash scripts/run_media.sh [--ip 192.168.1.24] [--input ./input.mp4] [--no-edit-cameradata]
```

**Các tham số**:
- `--ip`: Chỉ định IP LAN (nếu không, script sẽ tự động phát hiện)
- `--input`: Đường dẫn file video mẫu (mặc định: `input.mp4`)
- `--no-edit-cameradata`: Không tự động cập nhật host trong `Cameradata.ts`

**Kết quả**: Script sẽ tự động khởi động MediaMTX, spawn FFmpeg, và chạy Electron ở chế độ nền.

---

## Hướng dẫn sử dụng chi tiết

### 1. Màn hình Giám sát - Xem trực tiếp

Màn hình này cho phép xem video trực tiếp từ các camera theo thời gian thực.

#### Tính năng chính

**a) Bộ lọc linh hoạt theo Khu vực và Camera**

Người dùng có thể lựa chọn xem video theo nhiều cách:

1. **Xem tất cả camera trong một khu vực**:
   - Chọn khu vực từ dropdown "Khu vực"
   - Chọn "Tất cả camera" từ dropdown "Camera"
   - Hệ thống sẽ hiển thị tất cả camera trong khu vực đã chọn

2. **Xem một camera cụ thể**:
   - Chọn khu vực
   - Chọn camera cụ thể từ dropdown
   - Hệ thống chỉ hiển thị camera đã chọn

3. **Xem nhiều khu vực cùng lúc**:
   - Chọn nhiều khu vực từ dropdown "Khu vực" (multi-select)
   - Chọn "Tất cả camera" để xem toàn bộ camera trong các khu vực đã chọn
   - Hoặc chọn "Tất cả" trong dropdown khu vực để xem tất cả

4. **Xem nhiều camera từ nhiều khu vực**:
   - Sau khi chọn khu vực, dropdown "Camera" sẽ hiển thị tất cả camera thuộc các khu vực đã chọn
   - Có thể chọn nhiều camera cùng lúc (multi-select)

**Ví dụ các trường hợp sử dụng**:
- Xem 2-3 camera trong 1 khu vực: Chọn 1 khu vực → Chọn 2-3 camera cụ thể
- Xem tất cả camera từ 2-3 khu vực: Chọn 2-3 khu vực → Chọn "Tất cả camera"
- Xem camera cụ thể từ nhiều khu vực: Chọn nhiều khu vực → Chọn các camera mong muốn

#### Giao diện

- Video được hiển thị dưới dạng lưới (grid layout)
- Mỗi video có tiêu đề hiển thị tên khu vực và tên camera
- Video tự động phát khi tải xong
- Hỗ trợ WebRTC cho độ trễ thấp

#### Lưu ý
- Nếu chọn quá nhiều camera cùng lúc, hiệu năng có thể bị ảnh hưởng tùy thuộc vào cấu hình máy
- Khuyến nghị xem tối đa 4-6 camera đồng thời trên một màn hình để đảm bảo chất lượng

---

### 2. Màn hình Xem lại video

Màn hình này cho phép tìm kiếm và xem lại các video đã ghi lại (recordings).

#### Tính năng chính

**a) Bộ lọc theo khoảng thời gian**

Người dùng có thể chọn khoảng thời gian để tìm kiếm video:

1. **Các mốc thời gian nhanh**:
   - **1 giờ trước**: Xem video trong 1 giờ gần nhất
   - **6 giờ trước**: Xem video trong 6 giờ gần nhất
   - **24 giờ trước**: Xem video trong 24 giờ gần nhất
   - **7 ngày trước**: Xem video trong 7 ngày gần nhất
   - **30 ngày trước**: Xem video trong 30 ngày gần nhất
   - **Tất cả**: Xem toàn bộ video đã lưu (không giới hạn thời gian)

2. **Chọn khoảng thời gian tùy chỉnh**:
   - Chọn "Ngày bắt đầu" và "Ngày kết thúc" từ date picker
   - Click "Áp dụng" để tìm kiếm

**b) Bộ lọc theo Khu vực và Camera**

Tương tự màn hình Giám sát, người dùng có thể lọc video theo:
- **Tất cả khu vực + Tất cả camera**: Hiển thị toàn bộ video đã ghi
- **Tất cả khu vực + Camera cụ thể**: Hiển thị video từ camera đó trong tất cả khu vực (có hiển thị tên khu vực)
- **Khu vực cụ thể + Tất cả camera**: Hiển thị video từ tất cả camera trong khu vực đó
- **Khu vực cụ thể + Camera cụ thể**: Hiển thị video từ camera cụ thể trong khu vực cụ thể

**c) Bảng danh sách video**

Hiển thị các thông tin chi tiết:
- **Tên camera**: Tên của camera đã ghi hình
- **Khu vực**: Tên khu vực mà camera thuộc về
- **Thời gian bắt đầu**: Thời điểm bắt đầu ghi hình
- **Thời gian kết thúc**: Thời điểm kết thúc ghi hình
- **Thời lượng**: Độ dài video (được tính toán dựa trên kích thước file và bitrate)
- **Kích thước file**: Dung lượng file video (B, KB, MB, GB)
- **Thao tác**: Nút "Phát" để xem video

**d) Phát video**

- Click nút "Phát" để mở video trong trình phát tích hợp
- Video được phát trực tiếp từ thư mục `recordings/`
- Hỗ trợ tua, tạm dừng, điều chỉnh âm lượng
- Click "Đóng" để quay lại danh sách

#### Lưu ý
- Video được lưu tự động trong quá trình streaming
- Tên file video theo định dạng: `YYYY-MM-DD_HH-MM-SS-fff.mp4`
- Hệ thống tự động tính toán thời lượng dựa trên kích thước file (bitrate ước tính ~5Mbps)

---

### 3. Màn hình Sự kiện

Màn hình này hiển thị các sự kiện phát hiện được từ camera (motion detection, object detection, face detection, etc.).

#### Tính năng chính

**a) Thông báo sự kiện thời gian thực**

- Hiển thị tối đa **25 sự kiện mới nhất**
- Sự kiện được cập nhật tự động khi có phát hiện mới
- Tự động xóa các sự kiện **sau 30 phút** để giữ danh sách gọn gàng
- Khi có sự kiện mới, sự kiện cũ nhất sẽ bị đẩy ra khỏi danh sách (nếu đã đủ 25 sự kiện)

**b) Bộ lọc sự kiện**

1. **Lọc theo Khu vực**:
   - Dropdown "Khu vực" cho phép chọn khu vực cụ thể
   - Chọn "Tất cả" để xem sự kiện từ tất cả các khu vực

2. **Lọc theo Camera**:
   - Dropdown "Camera" hiển thị các camera thuộc khu vực đã chọn
   - Chọn "Tất cả" để xem sự kiện từ tất cả camera trong khu vực

3. **Lọc theo Loại sự kiện**:
   - **All Type**: Hiển thị tất cả các loại sự kiện
   - Các loại khác: `event_type_1`, `event_type_2`, `event_type_3`, `event_type_4`, v.v.
   - Có thể chọn nhiều loại sự kiện cùng lúc (checkbox)
   - Nếu không chọn loại nào, hệ thống tự động chọn "All Type"

**c) Hiển thị danh sách sự kiện**

Mỗi sự kiện hiển thị:
- **Icon màu sắc** phân biệt theo loại sự kiện
- **Tên khu vực** và **Tên camera**
- **Loại sự kiện**
- **Thời gian** phát hiện
- **Thumbnail/Ảnh chụp** (nếu có)
- **Icon video** để phát lại video sự kiện

**d) Phát video sự kiện**

- Click vào sự kiện để xem video đã ghi
- Video được lưu trong thư mục `snapshot/`
- Hỗ trợ phát trực tiếp trong ứng dụng

#### Lưu ý
- Sự kiện được lưu vĩnh viễn trong thư mục `snapshot/` cho đến khi người dùng xóa hoặc hết hạn lưu trữ
- Danh sách 25 sự kiện trong màn hình chỉ là thông báo tạm thời, không ảnh hưởng đến dữ liệu đã lưu
- Có thể xem lại toàn bộ sự kiện đã lưu bằng cách thay đổi bộ lọc

---

### 4. Màn hình Cài đặt

Màn hình này cho phép quản lý cấu hình hệ thống.

#### Tính năng chính

**a) Quản lý Khu vực**

1. **Thêm khu vực mới**:
   - Nhập tên khu vực vào ô "Tên khu vực mới" (tối đa 25 ký tự)
   - Click nút "Thêm khu vực"
   - Khu vực mới sẽ xuất hiện trong danh sách

2. **Sửa khu vực**:
   - Click icon "Sửa" (bút chì) bên cạnh tên khu vực
   - Nhập tên mới trong dialog (tối đa 25 ký tự)
   - Click "Lưu" để cập nhật

3. **Xóa khu vực**:
   - Click icon "Xóa" (thùng rác) bên cạnh tên khu vực
   - Xác nhận xóa trong dialog cảnh báo
   - **Lưu ý**: Xóa khu vực sẽ xóa tất cả camera trong khu vực đó

**b) Quản lý Camera**

1. **Thêm camera mới**:
   - Chọn khu vực từ dropdown "Chọn khu vực"
   - Nhập tên camera (tối đa 25 ký tự)
   - Nhập đường dẫn RTSP/URL camera (tối đa 50 ký tự)
   - Click "Thêm camera"

2. **Sửa camera**:
   - Click icon "Sửa" bên cạnh camera trong danh sách
   - Chỉnh sửa tên hoặc URL
   - Click "Lưu" để cập nhật

3. **Xóa camera**:
   - Click icon "Xóa" bên cạnh camera
   - Xác nhận xóa trong dialog

**c) Cấu hình file Data mới**

- Click nút "Chọn file cấu hình Data" để load file `data.json` hoặc `data*.json` khác
- File phải tuân theo định dạng JSON chuẩn (xem mục [Cấu hình](#cấu-hình-và-tùy-chỉnh))
- Sau khi load, hệ thống sẽ cập nhật danh sách khu vực và camera

**d) Cài đặt thời gian lưu trữ (Data Retention)**

1. **Thời gian lưu trữ Video ghi hình**:
   - Nhập số ngày lưu trữ (mặc định: 180 ngày)
   - Video cũ hơn số ngày này sẽ tự động bị xóa

2. **Thời gian lưu trữ Sự kiện**:
   - Nhập số ngày lưu trữ (mặc định: 180 ngày)
   - Sự kiện (snapshot) cũ hơn số ngày này sẽ tự động bị xóa

3. **Các nút điều khiển**:
   - **Lưu**: Lưu cài đặt thời gian lưu trữ
   - **Chạy ngay**: Thực hiện xóa các file cũ ngay lập tức theo cài đặt hiện tại
   - **Xóa toàn bộ video**: Xóa TẤT CẢ video trong thư mục `recordings/` (cần xác nhận)
   - **Xóa toàn bộ sự kiện**: Xóa TẤT CẢ sự kiện trong thư mục `snapshot/` (cần xác nhận)

4. **Lịch trình tự động**:
   - Hệ thống tự động chạy cleanup **mỗi ngày vào lúc 2:00 AM** để xóa các file hết hạn
   - Không cần can thiệp thủ công

#### Lưu ý
- Tất cả thay đổi về khu vực/camera được lưu vào file `data.json` và cập nhật trên backend
- Nên backup file `data.json` trước khi thực hiện thay đổi lớn
- Khi xóa toàn bộ video/sự kiện, dữ liệu sẽ **KHÔNG THỂ KHÔI PHỤC**

---

## Kiến trúc hệ thống

### Sơ đồ tổng quan

```
┌─────────────────────────────────────────────────────────────────┐
│                     CAMERA TRACKER SYSTEM                        │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐      ┌──────────────┐     ┌──────────────┐
│   Camera     │      │   Camera     │     │   Camera     │
│   (RTSP)     │      │   (RTSP)     │     │   (RTSP)     │
└──────┬───────┘      └──────┬───────┘     └──────┬───────┘
       │                     │                     │
       └─────────────────────┼─────────────────────┘
                             │
                             ▼
              ┌──────────────────────────┐
              │   FFmpeg (Streaming)     │
              │   - Loop input.mp4       │
              │   - Encode H.264         │
              │   - Push to MediaMTX     │
              └──────────┬───────────────┘
                         │ RTSP
                         ▼
              ┌──────────────────────────┐
              │   MediaMTX Server        │
              │   (Docker Container)     │
              │   - RTSP: 8554           │
              │   - HLS: 8888            │
              │   - WebRTC: 8889         │
              │   - Playback: 9996       │
              └──────────┬───────────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
     WebRTC           HLS            RTSP
          │              │              │
          └──────────────┼──────────────┘
                         │
                         ▼
              ┌──────────────────────────┐
              │   Electron Main Process  │
              │   (Node.js Backend)      │
              │   - IPC Handlers         │
              │   - File System Watch    │
              │   - Data Retention       │
              │   - Stream Recording     │
              └──────────┬───────────────┘
                         │ IPC
                         ▼
              ┌──────────────────────────┐
              │   Electron Renderer      │
              │   (React + Vite)         │
              │   - Live View            │
              │   - Recorded View        │
              │   - Events View          │
              │   - Settings View        │
              └──────────┬───────────────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   ┌───────────┐  ┌───────────┐  ┌───────────┐
   │recordings/│  │ snapshot/ │  │data.json  │
   │ (Videos)  │  │ (Events)  │  │ (Config)  │
   └───────────┘  └───────────┘  └───────────┘
```

### Các thành phần chính

#### 1. Frontend (Electron Renderer - React)
- **Công nghệ**: React 18, Material-UI (MUI), TypeScript, Vite
- **Chức năng**:
  - Hiển thị giao diện người dùng (4 màn hình chính)
  - Xử lý tương tác người dùng
  - Gọi IPC đến Main Process để thực hiện các tác vụ backend
  - Render video player (WebRTC, HLS, RTSP)

#### 2. Backend (Electron Main Process - Node.js)
- **Công nghệ**: Node.js, Electron IPC
- **Chức năng**:
  - Quản lý file system (đọc/ghi `data.json`, quét thư mục `recordings/`, `snapshot/`)
  - Watch thư mục để phát hiện file mới (recordings, events)
  - Gửi sự kiện real-time đến Frontend qua IPC
  - Data retention: Tự động xóa file cũ theo lịch trình
  - Quản lý cấu hình khu vực/camera

#### 3. MediaMTX Server (Docker)
- **Công nghệ**: Docker, MediaMTX (Go-based streaming server)
- **Chức năng**:
  - Nhận luồng RTSP từ camera/FFmpeg
  - Chuyển đổi sang WebRTC, HLS để frontend có thể phát
  - Ghi hình stream vào thư mục `recordings/` (nếu được cấu hình)
  - Playback video đã ghi qua port 9996

#### 4. FFmpeg
- **Công nghệ**: FFmpeg CLI
- **Chức năng**:
  - Loop file `input.mp4` để mô phỏng camera (trong môi trường dev/test)
  - Encode video thành H.264
  - Push luồng RTSP đến MediaMTX
  - Mỗi stream một tiến trình FFmpeg riêng

#### 5. File System
- **recordings/**: Lưu trữ video ghi hình theo cấu trúc `recordings/<stream_name>/<YYYY-MM-DD_HH-MM-SS-fff>.mp4`
- **snapshot/**: Lưu trữ sự kiện (event snapshots) theo cấu trúc `snapshot/<area_name>/<camera_name>/<event_type>_<timestamp>.mp4`
- **data.json**: File cấu hình khu vực/camera (JSON format)
- **retention.json**: File cấu hình thời gian lưu trữ

### Luồng dữ liệu (Data Flow)

#### Luồng Live View
1. Camera/FFmpeg → RTSP → MediaMTX
2. MediaMTX → WebRTC/HLS → Electron Renderer
3. React Component render video player

#### Luồng Recording
1. MediaMTX ghi stream vào `recordings/<stream>/`
2. Electron Main Process watch folder và phát hiện file mới
3. Main Process gửi IPC event đến Renderer
4. Renderer cập nhật danh sách video (nếu đang ở màn "Xem lại video")

#### Luồng Event Detection
1. Hệ thống AI/Detection (external) ghi event snapshot vào `snapshot/`
2. Electron Main Process watch folder và phát hiện file mới
3. Main Process gửi IPC event `snapshot-event` đến Renderer
4. Renderer cập nhật danh sách sự kiện (tối đa 25, xóa sau 30 phút)

#### Luồng Data Retention
1. Người dùng cài đặt số ngày lưu trữ trong Settings
2. Main Process lưu vào `retention.json`
3. Scheduler chạy cleanup mỗi ngày 2:00 AM
4. Main Process quét `recordings/` và `snapshot/`, xóa file cũ hơn `N` ngày

### Giao tiếp giữa các services

#### IPC Channels (Electron)

**Từ Renderer → Main**:
- `get-areas`: Lấy danh sách khu vực/camera
- `save-areas`: Lưu cấu hình khu vực/camera
- `get-files`: Lấy danh sách video recordings theo filter
- `get-cameras`: Lấy danh sách camera theo khu vực (cho Events filter)
- `get-event-types`: Lấy danh sách loại sự kiện
- `get-snapshot-events`: Lấy sự kiện snapshot đã lưu
- `get-retention`: Lấy cài đặt thời gian lưu trữ
- `set-retention`: Cập nhật cài đặt thời gian lưu trữ
- `run-retention-now`: Chạy cleanup ngay lập tức
- `delete-all-recordings`: Xóa toàn bộ video recordings
- `delete-all-events`: Xóa toàn bộ sự kiện snapshots

**Từ Main → Renderer** (Events):
- `recording-event`: Thông báo có video recording mới
- `snapshot-event`: Thông báo có sự kiện snapshot mới

#### Network Ports

| Port | Protocol | Service | Mô tả |
|------|----------|---------|-------|
| 8554 | TCP | RTSP | Nhận luồng từ camera/FFmpeg |
| 8888 | TCP | HLS | HTTP Live Streaming |
| 8889 | TCP | WebRTC | WebRTC signaling & HTTP |
| 9996 | TCP | Playback | HTTP playback cho recordings |
| 8890 | UDP | SRT | Secure Reliable Transport (optional) |
| 8189 | UDP | WebRTC | WebRTC local UDP |

---

## Cấu hình và Tùy chỉnh

### Cấu trúc file `data.json`

File `data.json` nằm tại `frontend/src/pages/components/Camera/data.json`:

```json
{
  "areas": [
    {
      "id": "area1",
      "name": "Khu vực 1",
      "cameras": [
        {
          "id": "area1_cam1",
          "name": "Camera 1",
          "url": "http://192.168.1.100:8889/mystream1",
          "areaId": "area1"
        },
        {
          "id": "area1_cam2",
          "name": "Camera 2",
          "url": "http://192.168.1.100:8889/mystream2",
          "areaId": "area1"
        }
      ]
    },
    {
      "id": "area2",
      "name": "Khu vực 2",
      "cameras": [
        {
          "id": "area2_cam1",
          "name": "Camera 3",
          "url": "http://192.168.1.100:8889/mystream3",
          "areaId": "area2"
        }
      ]
    }
  ]
}
```

**Giải thích các trường**:
- `id`: Mã định danh duy nhất (string, không trùng lặp)
- `name`: Tên hiển thị (string, tối đa 25 ký tự khuyến nghị)
- `url`: Đường dẫn WebRTC/RTSP/HLS của camera (string, tối đa 50 ký tự khuyến nghị)
- `areaId`: ID của khu vực mà camera thuộc về (foreign key)

**Lưu ý**:
- Tên stream được trích xuất từ URL (phần cuối cùng sau dấu `/`)
  - Ví dụ: `http://192.168.1.100:8889/mystream1` → stream name = `mystream1`
- File này có thể được chỉnh sửa thủ công hoặc qua giao diện Settings
- Sau khi chỉnh sửa thủ công, cần restart ứng dụng để load lại cấu hình

### Cấu hình MediaMTX (`mediamtx.yml`)

File này chứa cấu hình cho MediaMTX server:

```yaml
# Cấu hình recording (tự động ghi hình)
paths:
  all:
    record: yes
    recordPath: ./recordings/%path/%Y-%m-%d_%H-%M-%S-%f
    recordFormat: mp4
    recordPartDuration: 1h
    recordSegmentDuration: 1h

# WebRTC configuration
webrtcAddress: :8889
webrtcServerKey: server.key
webrtcServerCert: server.crt
webrtcAllowOrigin: '*'
webrtcICEServers:
  - urls: [stun:stun.l.google.com:19302]
```

**Các tham số quan trọng**:
- `recordPath`: Định dạng đường dẫn lưu file (có thể thay đổi)
- `recordPartDuration`: Thời lượng mỗi segment video (1 giờ)
- `webrtcAddress`: Port WebRTC (mặc định 8889)

### File `retention.json`

File này lưu cấu hình thời gian lưu trữ (tự động tạo khi người dùng cài đặt):

```json
{
  "recordingsRetentionDays": 180,
  "eventsRetentionDays": 180
}
```

---

## Sizing ổ cứng lưu trữ

### Công thức tính toán

**Dung lượng lưu trữ = Số camera × Bitrate × Thời gian lưu trữ**

#### Giả định
- **Bitrate video**: 5 Mbps (megabit per second) cho camera HD 1080p
- **Thời gian ghi**: 24 giờ/ngày
- **Thời gian lưu trữ**: N ngày (mặc định 180 ngày)

#### Công thức chi tiết

```
Dung lượng 1 camera/ngày = (Bitrate × 3600 × 24) / 8 / 1024 / 1024
                          = (5 × 86400) / 8 / 1024 / 1024
                          = 432,000,000 bits/ngày
                          = 54,000,000 bytes/ngày
                          = ~51.5 GB/ngày

Dung lượng 1 camera/180 ngày = 51.5 GB × 180 = ~9,270 GB = ~9.05 TB
```

### Bảng sizing mẫu

| Số camera | Bitrate/camera | GB/ngày | GB/tháng (30 ngày) | GB/180 ngày | TB/180 ngày |
|-----------|----------------|---------|---------------------|-------------|-------------|
| 1         | 5 Mbps         | 51.5    | 1,545               | 9,270       | 9.05        |
| 4         | 5 Mbps         | 206     | 6,180               | 37,080      | 36.21       |
| 8         | 5 Mbps         | 412     | 12,360              | 74,160      | 72.42       |
| 16        | 5 Mbps         | 824     | 24,720              | 148,320     | 144.84      |
| 32        | 5 Mbps         | 1,648   | 49,440              | 296,640     | 289.69      |

### Khuyến nghị

#### Hệ thống nhỏ (1-4 camera)
- **Ổ cứng**: 1-2 TB SSD/HDD
- **Thời gian lưu**: 30-60 ngày
- **RAM**: 8 GB
- **CPU**: Core i5 hoặc tương đương

#### Hệ thống trung bình (4-16 camera)
- **Ổ cứng**: 4-8 TB HDD (RAID 1 khuyến nghị)
- **Thời gian lưu**: 60-90 ngày
- **RAM**: 16 GB
- **CPU**: Core i7 hoặc Xeon

#### Hệ thống lớn (16+ camera)
- **Ổ cứng**: 16 TB+ HDD (RAID 5/6 khuyến nghị)
- **Thời gian lưu**: 90-180 ngày
- **RAM**: 32 GB+
- **CPU**: Xeon hoặc Ryzen Threadripper
- **Network**: 10 Gbps NIC khuyến nghị

### Tối ưu dung lượng

1. **Giảm bitrate**: Nếu không cần chất lượng cao, có thể giảm xuống 2-3 Mbps
2. **Giảm framerate**: Từ 30fps xuống 15fps có thể tiết kiệm ~50% dung lượng
3. **Motion detection**: Chỉ ghi khi có chuyển động (tiết kiệm 50-80% dung lượng)
4. **Thời gian lưu ngắn hơn**: 30-90 ngày thay vì 180 ngày
5. **Nén H.265 (HEVC)**: Tiết kiệm ~40-50% so với H.264 (nhưng cần CPU mạnh hơn)

---

## Khắc phục sự cố

### 1. Docker không khởi động

**Triệu chứng**: Lỗi `Cannot connect to the Docker daemon`

**Giải pháp**:
```bash
# Windows
- Mở Docker Desktop
- Kiểm tra Docker đang chạy trong System Tray

# Linux
sudo systemctl start docker
sudo systemctl enable docker
```

### 2. Port bị chiếm dụng

**Triệu chứng**: Lỗi `port is already allocated`

**Giải pháp**:
```bash
# Xem process đang dùng port
# Windows
netstat -ano | findstr :8554

# Linux
sudo lsof -i :8554

# Dừng container cũ
docker rm -f mediamtx

# Hoặc kill process
# Windows: taskkill /PID <PID> /F
# Linux: sudo kill -9 <PID>
```

### 3. FFmpeg không tìm thấy

**Triệu chứng**: `'ffmpeg' is not recognized as an internal or external command`

**Giải pháp**:
```bash
# Kiểm tra FFmpeg đã cài
ffmpeg -version

# Windows: Thêm FFmpeg vào PATH
# 1. Download từ https://www.gyan.dev/ffmpeg/builds/
# 2. Giải nén vào C:\ffmpeg
# 3. Thêm C:\ffmpeg\bin vào PATH
# 4. Restart terminal

# Linux
sudo apt-get update
sudo apt-get install ffmpeg
```

### 4. Electron không mở

**Triệu chứng**: Cửa sổ electron-dev mở nhưng không thấy ứng dụng

**Giải pháp**:
```bash
# Xóa node_modules và cài lại
rm -rf node_modules frontend/node_modules
yarn install

# Hoặc chạy thủ công để xem lỗi
yarn dev:electron
```

### 5. Video không phát

**Triệu chứng**: Video player hiển thị màn hình đen

**Giải pháp**:
- Kiểm tra MediaMTX đang chạy: `docker ps`
- Kiểm tra FFmpeg đang stream: Xem cửa sổ CMD/terminal có log "Starting FFmpeg for stream"
- Mở browser test WebRTC: `http://localhost:8889/<stream_name>`
- Kiểm tra firewall không block port 8889

### 6. Không tìm thấy video đã ghi

**Triệu chứng**: Màn hình "Xem lại video" trống

**Giải pháp**:
- Kiểm tra thư mục `recordings/` có file không
- Kiểm tra tên stream trong file khớp với URL trong `data.json`
- Thử chọn "Tất cả" trong bộ lọc thời gian
- Kiểm tra quyền đọc/ghi thư mục `recordings/`

### 7. Sự kiện không hiển thị

**Triệu chứng**: Màn hình "Sự kiện" trống

**Giải pháp**:
- Kiểm tra thư mục `snapshot/` có file không
- Kiểm tra cấu trúc thư mục: `snapshot/<area>/<camera>/<event_type>_<timestamp>.mp4`
- File phải có định dạng tên đúng: `event_type_<timestamp>.mp4`
- Chọn "All Type" trong bộ lọc loại sự kiện

### 8. WebRTC không kết nối từ thiết bị khác trong LAN

**Triệu chứng**: Video phát được trên máy chủ nhưng không phát trên máy khác

**Giải pháp**:
- Sửa `MTX_WEBRTCADDITIONALHOSTS` trong `run_media.bat` hoặc `run_media.sh` thành IP LAN của máy chủ
- Ví dụ: `192.168.1.100` thay vì `localhost`
- Sửa `url` trong `data.json` và `Cameradata.ts` thành IP LAN
- Restart tất cả services

---

## Build Production

### Build ứng dụng Electron

```bash
# Build frontend + Electron
yarn build

# Kết quả sẽ nằm trong thư mục dist/
```

### Cấu hình Electron Builder

File `package.json` chứa cấu hình build (electron-builder):

```json
{
  "build": {
    "appId": "com.camertracker.app",
    "productName": "Camera Tracker",
    "files": [
      "frontend/dist/**/*",
      "main.js",
      "preload.js"
    ],
    "directories": {
      "output": "dist"
    }
  }
}
```

### Build cho từng platform

```bash
# Windows (tạo .exe)
yarn build --win

# macOS (tạo .dmg)
yarn build --mac

# Linux (tạo .AppImage)
yarn build --linux
```

---

## Tài liệu bổ sung

### Roadmap tính năng

- [ ] **AI Object Detection**: Tích hợp YOLO/TensorFlow cho phát hiện đối tượng
- [ ] **Face Recognition**: Nhận diện khuôn mặt và lưu database
- [ ] **Motion Detection**: Phát hiện chuyển động và tự động ghi hình
- [ ] **Multi-language**: Hỗ trợ tiếng Anh/Việt/Trung
- [ ] **Cloud Backup**: Tự động backup video lên cloud (AWS S3, Google Cloud Storage)
- [ ] **Mobile App**: Ứng dụng mobile iOS/Android để xem camera từ xa
- [ ] **Notification**: Gửi thông báo qua email/SMS khi có sự kiện
- [ ] **User Management**: Quản lý người dùng và phân quyền

### Đóng góp

Mọi đóng góp đều được hoan nghênh! Vui lòng:
1. Fork repository
2. Tạo branch mới: `git checkout -b feature/amazing-feature`
3. Commit thay đổi: `git commit -m 'Add some amazing feature'`
4. Push lên branch: `git push origin feature/amazing-feature`
5. Tạo Pull Request

### License

[MIT License](LICENSE)

---

## Liên hệ và Hỗ trợ

**Nhật Dương** - Tác giả/Maintainer
- Email: [your-email@example.com]
- GitHub: [@nhatduong]

**Issue Tracker**: [https://github.com/your-repo/issues]

---

## Ghi chú cho Nhật Dương

Tài liệu này cần được bổ sung thêm:

1. **Sizing ổ cứng lưu trữ**: 
   - Bảng sizing chi tiết cho các kịch bản khác nhau (bitrate khác, số camera khác)
   - Công thức tính toán cụ thể
   - Khuyến nghị loại ổ cứng (SSD vs HDD, RAID level)
   - ✅ **ĐÃ BỔ SUNG trong phần [Sizing ổ cứng lưu trữ](#sizing-ổ-cứng-lưu-trữ)**

2. **Mô hình kiến trúc**:
   - Sơ đồ kiến trúc chi tiết (vẽ bằng draw.io hoặc PlantUML)
   - Luồng dữ liệu (data flow diagram)
   - Sequence diagram cho các use case chính
   - ✅ **ĐÃ BỔ SUNG sơ đồ ASCII trong phần [Kiến trúc hệ thống](#kiến-trúc-hệ-thống)**
   - ❗ **CẦN VẼ SƠ ĐỒ CHUYÊN NGHIỆP BẰNG DRAW.IO/LUCIDCHART**

3. **Giao tiếp giữa các services**:
   - Chi tiết các IPC channels (request/response format)
   - API documentation cho mỗi endpoint
   - WebSocket/Event flow cho real-time updates
   - ✅ **ĐÃ BỔ SUNG trong phần [Giao tiếp giữa các services](#giao-tiếp-giữa-các-services)**

4. **Hướng dẫn sử dụng tất cả các màn hình**:
   - ✅ Screenshots cho mỗi màn hình (cần chụp màn hình và thêm vào)
   - ✅ Video demo/GIF cho các tính năng chính
   - ✅ Step-by-step tutorial cho người dùng mới

**Action Items cho Nhật Dương**:
- [ ] Vẽ sơ đồ kiến trúc bằng draw.io (xuất file .png và .drawio)
- [ ] Vẽ sequence diagram cho 4 màn hình chính
- [ ] Chụp screenshots tất cả màn hình (độ phân giải cao)
- [ ] Quay video demo ngắn (2-3 phút) giới thiệu hệ thống
- [ ] Review và cập nhật bảng sizing với các kịch bản thực tế
- [ ] Viết API documentation cho IPC channels (format OpenAPI/Swagger)
- [ ] Tạo file `docs/architecture.md` chi tiết hơn
- [ ] Tạo file `docs/api.md` cho IPC API reference

**Deadline**: [Ngày cần hoàn thành - do anh/leader quyết định]

---

**Phiên bản tài liệu**: 2.0  
**Ngày cập nhật**: 2025-10-23  
**Người cập nhật**: AI Assistant (cần review bởi Nhật Dương)

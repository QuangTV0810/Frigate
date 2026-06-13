# 🚀 Frigate NVR trên Banana Pi BPI-CM5 Pro (Rockchip RK3576)

Repository này cung cấp bộ file cấu hình (`docker-compose.yml`, `config.yml`) và hướng dẫn chi tiết để triển khai hệ thống camera giám sát **Frigate NVR** trên bo mạch **Banana Pi BPI-CM5 Pro**.

Mục tiêu của dự án là thiết lập một môi trường Frigate tối ưu, tận dụng triệt để sức mạnh phần cứng của chip Rockchip nhằm giảm tải tối đa cho CPU.

## ✨ Tính năng Nổi bật

* **Tăng tốc phần cứng AI (Hardware Object Detection):** Sử dụng NPU (Neural Processing Unit) của Rockchip thông qua driver RKNN để nhận diện đối tượng với tốc độ cao, tiêu thụ ít điện năng.
* **Person Detection:** Cấu hình Frigate hỗ trợ nhận diện người (`person`) theo thời gian thực trên luồng camera, làm nền tảng cho cảnh báo, snapshot và lưu event.
* **Giải mã Video bằng phần cứng (Hardware Video Decoding):** Sử dụng VPU/MPP của Rockchip để xử lý luồng video (H.264/H.265), giúp CPU luôn ở mức tải thấp.
* **Tối ưu hóa Lưu trữ (RAM Disk):** Cấu hình `tmpfs` để đẩy các tác vụ ghi/xóa file tạm liên tục lên RAM, giúp bảo vệ tuổi thọ của thẻ nhớ MicroSD hoặc bộ nhớ eMMC.
* **Bảo mật & Ổn định:** Cấu hình Docker ánh xạ trực tiếp các thiết bị (`/dev/dri`, `/dev/rga`, `/dev/rknn`) an toàn và sử dụng image Docker phiên bản tối ưu riêng cho Rockchip (`-rk`).

## 📷 Khả năng Tương thích Camera

Cấu hình này đã add được camera ONVIF của các hãng sau:

- Dahua
- Hikvision
- Vogten

## 🖼 Ảnh Minh họa

### Giao diện và cấu hình Frigate

![Frigate UI 1](image/1.%20image.png)
![Frigate UI 2](image/2.%20image.png)
![Frigate UI 3](image/3.%20image.png)
![Frigate UI 4](image/4.%20image.png)

### Nhận diện người và event

![Person Detection 1](image/5.%20image.png)
![Person Detection 2](image/6%20image.png)
![Person Detection 3](image/7.%20image.png)

Các ảnh trên minh họa việc add camera, xem live view, theo dõi event và nhận diện `person` trong Frigate.

## 📁 Cấu trúc Repository

```text
├── docker-compose.yml    # File triển khai Docker Compose (đã map NPU/VPU)
├── config/
│   └── config.yml        # File cấu hình Frigate (Khai báo Camera, NPU, Video Decoder)
└── README.md             # File giới thiệu này

```

## ⚙️ Yêu cầu Hệ thống (Prerequisites)

Để chạy được cấu hình này, thiết bị của bạn cần đáp ứng:

1. **Hệ điều hành:** Linux (Khuyến nghị Armbian hoặc Ubuntu tùy chỉnh cho BPI-CM5).
2. **Kernel:** Phải có hậu tố `-rockchip` (ví dụ: `5.10.x-rockchip` hoặc `6.1.x-rockchip`).
3. **Driver NPU/VPU:** Hệ điều hành phải nhận diện được `/dev/dri` (VPU) và driver rknpu phiên bản `v0.9.2` trở lên.
4. **Docker & Docker Compose:** Đã được cài đặt sẵn trên Host OS.

## 🔍 Kiểm tra Phần cứng và Hệ điều hành

Frigate yêu cầu hệ điều hành Linux đi kèm Rockchip BSP kernel `5.10` hoặc `6.1` và các driver cần thiết. Khuyến nghị dùng Armbian cho bo mạch này.

Kiểm tra kernel:

```bash
uname -r
```

Yêu cầu: kết quả phải có hậu tố `-rockchip`.

Kiểm tra thiết bị VPU và NPU:

```bash
ls /dev/dri
```

Yêu cầu: có `renderD128` và `renderD129`.

Kiểm tra phiên bản driver NPU:

```bash
sudo cat /sys/kernel/debug/rknpu/version
```

Yêu cầu: phiên bản `v0.9.2` trở lên.

Kiểm tra số core NPU:

```bash
sudo cat /sys/kernel/debug/rknpu/load
```

Ghi lại số core hiển thị để chỉnh `num_cores` trong `config/config.yml`.

Chỉ nên chạy Docker sau khi 4 bước kiểm tra trên đều đạt yêu cầu. Nếu kernel không có hậu tố `-rockchip`, không thấy `/dev/dri`, hoặc driver `rknpu` không đúng version thì Frigate sẽ không dùng được tăng tốc phần cứng như cấu hình trong repo này.

## 🚀 Triển khai

### 1. Clone repository

```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

### 2. Tùy chỉnh cấu hình

- Mở `docker-compose.yml`: chỉnh đường dẫn lưu trữ video `./storage` nếu bạn muốn lưu ra ổ cứng ngoài hoặc SD card mount sẵn.
- Mở `config/config.yml`: đổi `path` RTSP thành luồng camera thực tế, cập nhật tài khoản/mật khẩu, và chỉnh `num_cores` nếu bạn muốn ép số core NPU cụ thể.

### 3. Khởi chạy Frigate

Lần chạy đầu giữ nguyên `privileged: true` trong compose để xác minh Frigate nhìn thấy đầy đủ NPU/VPU của Rockchip:

```bash
docker compose up -d
```

### 4. Kiểm tra log và truy cập UI

```bash
docker compose logs -f
```

Truy cập giao diện web tại:

```text
http://<IP_CUA_BANANA_PI>:5000
```

Sau khi xác nhận hệ thống ổn định và tăng tốc phần cứng hoạt động đúng, bạn có thể giảm quyền container nếu muốn siết bảo mật hơn.

*Nếu gặp `Bus error` hoặc crash khi start, kiểm tra lại `shm_size` trong compose theo số lượng camera và độ phân giải detect thực tế.*

## 🛠 Giải thích các Tham số Quan trọng

Phần này mô tả trực tiếp các tham số đang có trong [docker-compose.yml](/home/quangtv/Workspace/Gateway/Frigate/docker-compose.yml) và [config/config.yml](/home/quangtv/Workspace/Gateway/Frigate/config/config.yml), tham chiếu theo tài liệu Frigate chính thức.

### `docker-compose.yml`

```yaml
services:
  frigate:
    container_name: frigate
    restart: unless-stopped
    image: ghcr.io/blakeblackshear/frigate:stable-rk
    shm_size: "512mb"
    privileged: true
    security_opt:
      - apparmor=unconfined
      - systempaths=unconfined
    devices:
      - /dev/dri:/dev/dri
      - /dev/dma_heap:/dev/dma_heap
      - /dev/rga:/dev/rga
      - /dev/mpp_service:/dev/mpp_service
    volumes:
      - /sys/:/sys/:ro
      - /etc/localtime:/etc/localtime:ro
      - ./config:/config
      - ./storage:/media/frigate
      - type: tmpfs
        target: /tmp/cache
        tmpfs:
          size: 1000000000
    ports:
      - "8971:8971"
      - "5000:5000"
      - "8554:8554"
      - "8555:8555/tcp"
      - "8555:8555/udp"
```

- `container_name: frigate`: đặt tên container cố định để dễ xem log, restart và debug.
- `restart: unless-stopped`: tự khởi động lại sau reboot hoặc khi process trong container thoát ngoài ý muốn.
- `image: ghcr.io/blakeblackshear/frigate:stable-rk`: dùng image Rockchip do Frigate phát hành, phù hợp cho `preset-rkmpp` và NPU/VPU trên RK3576.
- `shm_size: "512mb"`: tăng shared memory cho `/dev/shm`; Frigate dùng vùng này để chứa raw decoded frames và một phần log runtime. Docker mặc định chỉ có `64MB`, dễ gây `Bus error` khi có nhiều camera hoặc detect resolution lớn.
- `privileged: true`: mở rộng quyền truy cập thiết bị cho lần bring-up đầu. Cách này tiện để xác minh NPU/VPU hoạt động; sau khi ổn định có thể giảm quyền nếu bạn muốn siết bảo mật hơn.
- `security_opt`: nới AppArmor và system path restrictions để container truy cập các interface phần cứng của host dễ hơn. Với Rockchip, mục này thường đi cùng mapping thiết bị.
- `devices`:
  - `/dev/dri`: giao tiếp phần cứng video decode/render.
  - `/dev/dma_heap`: cấp phát DMA buffer cho pipeline media.
  - `/dev/rga`: dùng Rockchip RGA cho scale/convert frame.
  - `/dev/mpp_service`: truy cập Media Process Platform của Rockchip.
- `volumes`:
  - `/sys/:/sys/:ro`: cho container đọc thông tin thiết bị và driver từ host.
  - `/etc/localtime:/etc/localtime:ro`: đồng bộ timezone của container với host.
  - `./config:/config`: nơi Frigate đọc `config.yml` và ghi SQLite database.
  - `./storage:/media/frigate`: nơi lưu clips, recordings, exports và dữ liệu media khác.
  - `tmpfs -> /tmp/cache`: Frigate ghi recording segments tạm vào đây trước khi xử lý và chuyển sang storage chính; dùng RAM để giảm ghi liên tục lên eMMC/SD card.
- `tmpfs.size: 1000000000`: giới hạn RAM disk `/tmp/cache` khoảng 1GB; tăng nếu bạn có nhiều camera bitrate cao và thấy cache đầy.
- `ports`:
  - `5000`: cổng truy cập web UI của Frigate trong cấu hình hiện tại.
  - `8971`: cổng service phụ đã được expose theo compose, chỉ cần dùng khi bạn có nhu cầu tích hợp hoặc routing riêng.
  - `8554`: RTSP restream do Frigate/go2rtc cung cấp.
  - `8555/tcp` và `8555/udp`: WebRTC transport cho live view hoặc camera có hai chiều âm thanh.

### `shm_size`

Frigate dùng `/dev/shm` để chứa raw frames. Nếu thiếu RAM vùng này, container dễ crash với lỗi `Bus error`.

Công thức tham khảo:

```text
((width * height * 1.5 * 20 + 270480) / 1048576) * so_luong_camera + 40
```

Ví dụ 2 camera `1920x1080` cần khoảng `160MB`. Mẫu trong repo dùng `512mb` để đủ cho vài camera.

### Đường dẫn lưu trữ

- `/config`: chứa file cấu hình và SQLite database.
- `/media/frigate`: chứa recordings, clips, snapshots, exports.

Nếu muốn lưu sang SD card hoặc USB, sửa bind mount `./storage:/media/frigate` thành đường dẫn thật trên host, ví dụ `/mnt/sdcard/frigate_media:/media/frigate`.

### `tmpfs`

`/tmp/cache` là nơi Frigate ghi liên tục các đoạn video tạm. Dùng `tmpfs` để đẩy ghi tạm lên RAM, giảm hao mòn eMMC hoặc SD card.

### Quyền truy cập phần cứng

Repo này giữ `privileged: true` cho lần chạy đầu, đồng thời đã map sẵn các thiết bị Rockchip và `security_opt`. Sau khi xác nhận hệ thống ổn định, có thể bỏ `privileged: true` và giữ cấu hình truy cập thiết bị tối thiểu.

### `config/config.yml`

```yaml
mqtt:
  enabled: false

detectors:
  rknn:
    type: rknn
    num_cores: 0

ffmpeg:
  hwaccel_args: preset-rkmpp

cameras:
  camera_san_truoc:
    ffmpeg:
      inputs:
        - path: rtsp://admin:password@192.168.1.10:554/stream1
          roles:
            - detect
            - record
    detect:
      width: 1280
      height: 720
      fps: 5
    record:
      enabled: true
      retain:
        days: 7
        mode: motion
    snapshots:
      enabled: true
```

- `mqtt.enabled: false`: tắt MQTT. Theo docs, MQTT là tùy chọn; chỉ bắt buộc khi bạn tích hợp Home Assistant qua Frigate integration hoặc muốn publish event qua broker.
- `detectors.rknn.type: rknn`: bật object detector dùng Rockchip NPU.
- `detectors.rknn.num_cores: 0`: để Frigate tự chọn số NPU core. Theo docs detector Rockchip, `0` nghĩa là auto; tăng giá trị này khi bạn biết rõ SoC của mình có nhiều core và muốn ép hiệu năng cao hơn.
- `ffmpeg.hwaccel_args: preset-rkmpp`: bật hardware video processing bằng preset MPP của Rockchip. Đây là preset Frigate docs khuyến nghị cho nền tảng Rockchip khi SoC hỗ trợ codec/độ phân giải của stream đầu vào.
- `cameras.camera_san_truoc`: tên logical của camera trong Frigate. Nên dùng tên không dấu, không khoảng trắng để dễ quản lý.
- `cameras.<name>.ffmpeg.inputs[].path`: URL RTSP thật của camera. Đây là tham số bạn bắt buộc phải đổi trước khi chạy.
- `cameras.<name>.ffmpeg.inputs[].roles`: xác định stream này phục vụ tác vụ nào.
  - `detect`: stream được decode để phục vụ object detection.
  - `record`: stream được dùng cho recording pipeline.
- `detect.width` và `detect.height`: độ phân giải Frigate dùng cho detect, không nhất thiết phải bằng độ phân giải gốc của stream. Để thấp hơn sẽ giảm tải CPU/NPU nhưng cũng có thể giảm chất lượng detect.
- `detect.fps: 5`: số frame mỗi giây dùng cho detect. Docs mẫu của Frigate cũng dùng `5fps` như điểm cân bằng tốt cho đa số camera, đủ cho các bài toán như `person detection` trong đa số tình huống giám sát.
- `record.enabled: true`: bật ghi hình cho camera này.
- `record.retain.days: 7`: giữ recording trong 7 ngày trước khi Frigate dọn dẹp theo policy.
- `record.retain.mode: motion`: chỉ giữ recording có motion. Nếu cần giữ toàn bộ 24/7, policy này phải đổi theo nhu cầu thực tế.
- `snapshots.enabled: true`: bật lưu snapshot cho event của camera.

### Khi nào cần chỉnh các tham số này

- Tăng `shm_size` nếu log có `Bus error`, camera detect ở độ phân giải cao, hoặc số camera tăng lên.
- Tăng `tmpfs.size` nếu `/tmp/cache` đầy khi record nhiều stream cùng lúc.
- Đổi `num_cores` khỏi `0` chỉ khi bạn muốn ép số core NPU cụ thể sau khi đã kiểm tra tải bằng `cat /sys/kernel/debug/rknpu/load`.
- Giảm `detect.width`, `detect.height`, hoặc `detect.fps` nếu hệ thống quá tải.
- Đổi `./storage` sang ổ USB/SSD nếu không muốn ghi media lên eMMC hoặc SD card.

## ✅ Dấu hiệu Hoạt động Đúng

Sau khi chạy `docker compose up -d`, kiểm tra `docker compose logs -f`.

Hệ thống được xem là hoạt động đúng khi:

- không có lỗi liên quan `rknn`, `scale_rkrga`, `Bus error`
- Frigate nhận được hardware acceleration
- truy cập được web UI tại `http://<IP_CUA_BANANA_PI>:5000`

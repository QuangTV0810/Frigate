# 🚀 Frigate NVR trên Banana Pi BPI-CM5 Pro (Rockchip RK3576)

Repository này cung cấp bộ file cấu hình `docker-compose.yml`, `config/config.yml` và hướng dẫn triển khai Frigate NVR trên Banana Pi BPI-CM5 Pro.

Mục tiêu là tận dụng NPU/VPU của Rockchip để chạy detect, decode video bằng phần cứng, và lưu recordings trực tiếp ra thẻ nhớ đã mount trên host.

## ✨ Tính năng Nổi bật

* **Tăng tốc AI bằng NPU Rockchip:** dùng `rknn` để detect object với tải CPU thấp.
* **Person Detection và Car Detection:** cấu hình hiện tại track `person` và `car`.
* **Giải mã video bằng phần cứng:** dùng `preset-rkmpp` để giảm tải CPU khi xử lý RTSP.
* **Tối ưu lưu trữ:** dùng `tmpfs` cho cache tạm và ghi recordings trực tiếp ra thẻ nhớ.
* **Phù hợp camera ONVIF phổ biến:** đã add/test được với Dahua, Hikvision, Vogten.

## 🖼 Ảnh Minh họa

### Add camera HIK

<p><img src="image/1.%20image.png" alt="Add HIK 1"></p>
<p><img src="image/2.%20image.png" alt="Add HIK 2"></p>
<p><img src="image/4.%20image.png" alt="Add HIK 3"></p>

### Add camera Dahua

<p><img src="image/5.%20image.png" alt="Add Dahua 1"></p>
<p><img src="image/6.%20image.png" alt="Add Dahua 2"></p>
<p><img src="image/8.%20image.png" alt="Add Dahua 3"></p>

### Live view sau khi add camera

<p><img src="image/9.%20image.png" alt="Live View"></p>

### System metrics

<p><img src="image/10.%20image.png" alt="System Metrics 1"></p>
<p><img src="image/11.%20image.png" alt="System Metrics 2"></p>

### Set zone cho detection

<p><img src="image/12.%20image.png" alt="Detection Zone 1"></p>
<p><img src="image/13.%20image.png" alt="Detection Zone 2"></p>

### Snapshot person detection

<p><img src="image/14.%20image.png" alt="Person Snapshot"></p>

### Record khi có sự kiện AI

<p><img src="image/15.%20image.png" alt="AI Record Event"></p>

Các ảnh trên minh họa quy trình add camera HIK/Dahua, live view, metrics, cấu hình zone, snapshot `person detection` và phần record khi có sự kiện AI.

## 📁 Cấu trúc Repository

```text
├── docker-compose.yml
├── config/
│   └── config.yml
├── image/
└── README.md
```

## ⚙️ Yêu cầu Hệ thống

Trước khi chạy Docker, host cần đáp ứng:

1. Linux có kernel hậu tố `-rockchip`.
2. Có đủ thiết bị phần cứng Rockchip cho media/NPU.
3. Docker và Docker Compose đã được cài.
4. Thẻ nhớ đã được mount tại `/media/armsom/69E5-F78C`.

## 🔍 Kiểm tra Phần cứng và OS

Kiểm tra kernel:

```bash
uname -r
```

Yêu cầu: kết quả phải có hậu tố `-rockchip`.

Kiểm tra VPU/NPU:

```bash
ls /dev/dri
```

Yêu cầu: có `renderD128` và `renderD129`.

Kiểm tra version driver NPU:

```bash
sudo cat /sys/kernel/debug/rknpu/version
```

Yêu cầu: từ `v0.9.2` trở lên.

Kiểm tra tải và số core NPU:

```bash
sudo cat /sys/kernel/debug/rknpu/load
```

Kiểm tra thẻ nhớ đã mount đúng:

```bash
ls /media/armsom/69E5-F78C
```

Yêu cầu: path này phải tồn tại và ghi được.

Chỉ nên chạy Docker sau khi các kiểm tra trên đều đạt. Nếu thiếu `-rockchip`, thiếu `/dev/dri`, sai driver `rknpu`, hoặc chưa mount thẻ nhớ đúng path thì cấu hình này sẽ không chạy đúng.

## 🚀 Triển khai

### 1. Clone repository

```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

### 2. Kiểm tra và chỉnh `docker-compose.yml`

Compose hiện tại lưu media trực tiếp ra thẻ nhớ:

```yaml
- /media/armsom/69E5-F78C:/media/frigate
```

Nếu host của bạn mount thẻ nhớ ở path khác thì phải sửa lại đúng path thật.

### 3. Thêm camera vào `config/config.yml`

`config/config.yml` hiện đang để:

```yaml
cameras: {}
```

Bạn phải tự thêm camera thật trước khi chạy. Ví dụ:

```yaml
cameras:
  camera_san_truoc:
    ffmpeg:
      inputs:
        - path: rtsp://admin:password@192.168.1.10:554/stream1
          roles:
            - detect
            - record
```

### 4. Khởi chạy

Lần chạy đầu giữ nguyên `privileged: true` để xác minh Frigate nhìn thấy đầy đủ NPU/VPU:

```bash
docker compose up -d
```

### 5. Kiểm tra log và truy cập UI

```bash
docker compose logs -f
```

Truy cập web UI tại:

```text
http://<IP_CUA_BANANA_PI>:5000
```

Nếu gặp `Bus error`, kiểm tra lại `shm_size`, số camera, và tải detect thực tế.

## 🛠 Giải thích `docker-compose.yml`

File hiện tại:

```yaml
version: "3.9"

services:
  frigate:
    container_name: frigate
    restart: unless-stopped
    image: ghcr.io/blakeblackshear/frigate:stable-rk
    shm_size: "512mb"
    privileged: true
    # security_opt:
    #   - apparmor=unconfined
    #   - systempaths=unconfined
    devices:
      - /dev/dri:/dev/dri
      - /dev/dma_heap:/dev/dma_heap
      - /dev/rga:/dev/rga
      - /dev/mpp_service:/dev/mpp_service
    volumes:
      - /sys/:/sys/:ro
      - /etc/localtime:/etc/localtime:ro
      - ./config:/config
      - /media/armsom/69E5-F78C:/media/frigate
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

- `image: ghcr.io/blakeblackshear/frigate:stable-rk`: image Frigate cho Rockchip.
- `shm_size: "512mb"`: tăng shared memory để tránh `Bus error`.
- `privileged: true`: dùng cho lần bring-up đầu để dễ xác minh phần cứng.
- `security_opt`: hiện đang comment; có thể bật lại nếu sau này muốn giảm phụ thuộc vào `privileged`.
- `devices`: map thiết bị media/NPU của Rockchip vào container.
- `/sys/:/sys/:ro`: cho container đọc thông tin driver và thiết bị host.
- `./config:/config`: nơi Frigate đọc config và ghi DB.
- `/media/armsom/69E5-F78C:/media/frigate`: nơi lưu recordings, clips, snapshots, exports trên thẻ nhớ.
- `tmpfs -> /tmp/cache`: cache segment tạm trên RAM để giảm ghi lặp lên thẻ nhớ.
- `5000`: cổng truy cập web UI.
- `8971`: cổng service phụ đang expose theo compose.
- `8554`: RTSP restream.
- `8555/tcp`, `8555/udp`: WebRTC transport.

### `shm_size`

Frigate dùng `/dev/shm` để giữ raw frames. Nếu thiếu vùng này, container dễ crash với `Bus error`.

Công thức tham khảo:

```text
((width * height * 1.5 * 20 + 270480) / 1048576) * so_luong_camera + 40
```

## 🛠 Giải thích `config/config.yml`

File hiện tại:

```yaml
mqtt:
  enabled: false

detectors:
  rknn:
    type: rknn
    num_cores: 0

ffmpeg:
  hwaccel_args: preset-rkmpp

go2rtc:
  streams: {}

detect:
  enabled: true
  fps: 5
  min_initialized: 2
  max_disappeared: 25
  stationary:
    classifier: true
    interval: 50
    threshold: 50

record:
  enabled: true
  expire_interval: 60
  sync_recordings: false
  continuous:
    days: 0
  motion:
    days: 0
  alerts:
    pre_capture: 5
    post_capture: 5
    retain:
      days: 7
      mode: active_objects
  detections:
    pre_capture: 5
    post_capture: 5
    retain:
      days: 3
      mode: active_objects

objects:
  track:
    - person
    - car

snapshots:
  enabled: true
  bounding_box: true

cameras: {}

version: 0.17-0
```

- `mqtt.enabled: false`: tắt MQTT.
- `detectors.rknn`: dùng NPU Rockchip; `num_cores: 0` là để tự chọn core.
- `ffmpeg.hwaccel_args: preset-rkmpp`: bật decode bằng phần cứng Rockchip.
- `go2rtc.streams: {}`: hiện chưa khai báo restream riêng.
- `detect.enabled: true`: bật detect toàn cục.
- `detect.fps: 5`: tốc độ detect phù hợp cho `person` và `car`.
- `detect.min_initialized: 2`: object cần ổn định một vài frame trước khi được chấp nhận.
- `detect.max_disappeared: 25`: giảm nhấp nháy tracking khi object tạm mất dấu.
- `detect.stationary.*`: cấu hình xử lý object đứng yên.
- `record.enabled: true`: bật record toàn cục.
- `record.expire_interval: 60`: chu kỳ dọn dữ liệu hết hạn.
- `record.sync_recordings: false`: giảm I/O đồng bộ không cần thiết.
- `record.continuous.days: 0`: không giữ record liên tục 24/7.
- `record.motion.days: 0`: không giữ policy record-motion riêng.
- `record.alerts`: lưu alert clips với `pre_capture`, `post_capture`, giữ 7 ngày.
- `record.detections`: lưu detection clips, giữ 3 ngày.
- `objects.track`: hiện track `person` và `car`.
- `snapshots.enabled: true`: bật snapshot.
- `snapshots.bounding_box: true`: vẽ bounding box lên snapshot.
- `cameras: {}`: chưa hardcode camera; bạn phải thêm camera thật.
- `version: 0.17-0`: version schema hiện tại của config.

## 🔧 Khi nào cần chỉnh cấu hình

- Tăng `shm_size` nếu log báo `Bus error` hoặc số camera tăng.
- Tăng `tmpfs.size` nếu `/tmp/cache` đầy khi record nhiều stream.
- Đổi `num_cores` khỏi `0` nếu bạn muốn ép số core NPU cụ thể.
- Giảm `detect.fps` nếu hệ thống quá tải.
- Sửa volume `/media/armsom/69E5-F78C:/media/frigate` nếu thẻ nhớ của host mount ở path khác.

## ✅ Dấu hiệu Hoạt động Đúng

Sau khi chạy `docker compose up -d`, kiểm tra:

```bash
docker compose logs -f
```

Hệ thống được xem là ổn khi:

- không có lỗi `rknn`, `scale_rkrga`, `Bus error`
- Frigate nhận được hardware acceleration
- truy cập được UI tại `http://<IP_CUA_BANANA_PI>:5000`

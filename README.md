# PTIT CTF – Matrix Scoreboard (CTFd Plugin)

> **Một plugin scoreboard dạng ma trận (matrix) cho CTFd**, tối ưu cho các cuộc thi kiểu Jeopardy.
> Hiển thị đội × hạng mục × challenge theo dạng lưới, hỗ trợ nhóm đội (PTIT‑HN/PTIT‑HCM), huy chương 🥇🥈🥉,
> cột/tiêu đề cố định (sticky), tiêu đề rotate, “bounty” tags, và API JSON phục vụ tích hợp.

---

## 📸 Demo / Screenshots

> Thay ảnh minh hoạ của bạn vào thư mục `docs/screenshots/` rồi cập nhật đường dẫn bên dưới.

- **Matrix view** – lưới đội × challenge: `docs/screenshots/matrix-overview.png`
- **Medals & badges** – huy chương/top: `docs/screenshots/medals.png`
- **Sticky headers/columns** – cuộn mượt: `docs/screenshots/sticky.png`
- **Group label** – PTIT‑HN/PTIT‑HCM: `docs/screenshots/groups.png`

---

## ✨ Tính năng nổi bật

- **Matrix Scoreboard**: mỗi ô thể hiện trạng thái *đã solve/điểm* của đội với từng challenge.
- **Ẩn điểm từng bài (optional)**: chỉ hiển thị trạng thái ✓/**🚩**/huy hiệu; tổng điểm vẫn chuẩn theo CTFd.
- **Nhóm đội (Groups)**: hỗ trợ label **PTIT‑HN** / **PTIT‑HCM**; lọc/so sánh theo nhóm.
- **Medals & Badges**: tự động gán 🥇🥈🥉, icon ⭐ “pro”, 💸 “bounty” cho challenge đặc biệt.
- **Sticky & Rotated Headers**: cột đội cố định bên trái, tiêu đề challenge xoay dọc gọn gàng.
- **Tuỳ biến chủ đề**: CSS/Tailwind class utility, dark mode friendly.
- **API JSON**: endpoint trả về dữ liệu bảng ở dạng JSON cho việc nhúng/visualize bên ngoài.
- **Hiệu năng**: truy vấn gọn, có cache ngắn hạn, update mềm khi scoreboard thay đổi.

---

## 🧱 Kiến trúc & Cách hoạt động (Overview)

Plugin đăng ký một **Flask Blueprint** cung cấp:
- **View HTML**: `/matrix/scoreboard` – trang ma trận (đọc dữ liệu từ CTFd models).
- **API JSON**: `/matrix/api/scoreboard` – trả về dữ liệu đã gom theo *Đội → Hạng mục → Challenge*.
- **Tài nguyên tĩnh**: `static/css/matrix.css`, `static/js/matrix.js`, `static/img/*`.

Dữ liệu đọc từ các bảng lõi của CTFd (Challenges, Solves, Teams, Submissions…). Việc gắn **nhóm đội** sử dụng:
- Trường có sẵn (ví dụ `Team.affiliation`) **hoặc**
- Bảng map tên đội → nhóm (file `config/groups.yml`) **hoặc**
- Biểu thức regex/prefix mapping trong `config.py`.

---

## ✅ Yêu cầu

- **CTFd** 3.x hoặc 4.x (đã thử trên 3.6+ / 4.0+).
- Python 3.8+.
- Quyền quản trị để đặt plugin vào `CTFd/plugins/` và khởi động lại dịch vụ.

---

## ⚙️ Cài đặt

### 1) Thư mục plugin
Sao chép thư mục **`matrix_scoreboard/`** vào `CTFd/plugins/`:

```
CTFd/
└─ plugins/
   └─ matrix_scoreboard/
      ├─ __init__.py
      ├─ blueprint.py
      ├─ config.py
      ├─ models.py           # nếu dùng thêm bảng phụ
      ├─ templates/
      │  ├─ matrix.html
      │  └─ _macros.html
      ├─ static/
      │  ├─ css/matrix.css
      │  ├─ js/matrix.js
      │  └─ img/
      ├─ config/
      │  └─ groups.yml       # tùy chọn map team → group
      └─ README.md
```

> *Lưu ý*: Nếu bạn dùng giao diện riêng, có thể bổ sung `templates/matrix_base.html` hoặc hook vào theme hiện có.

### 2) Khởi động lại CTFd
- Docker Compose:
  ```bash
  docker compose restart ctfd
  # hoặc nếu lần đầu cài:
  docker compose up -d --build
  ```
- Bare‑metal:
  ```bash
  pkill -f 'gunicorn.*CTFd' || true
  # ví dụ
  gunicorn 'CTFd:create_app()' -b 0.0.0.0:8000 --workers 4
  ```

### 3) Truy cập
Mở: `https://<domain>/matrix/scoreboard`

---

## 🛠️ Cấu hình

### Hạng mục (Categories)
Mặc định plugin nhận các category phổ biến: **CRYPTO, FORENSIC, PWNABLE, REVERSE ENGINEERING, WEB**.
Bạn có thể chỉnh thứ tự/tên hiển thị trong `config.py`:

```python
CATEGORIES = [
    "CRYPTO",
    "FORENSIC",
    "PWNABLE",
    "REVERSE ENGINEERING",
    "WEB",
]
CATEGORY_ALIASES = {
    "RE": "REVERSE ENGINEERING",
    "REV": "REVERSE ENGINEERING",
}
```

### Ẩn điểm từng bài
Ẩn cột điểm per‑challenge, chỉ hiển thị trạng thái **✓/🚩/badge** (tổng điểm vẫn giữ nguyên từ CTFd):

```python
HIDE_PER_CHALL_POINTS = True
```

### Gắn nhóm đội (PTIT‑HN / PTIT‑HCM)
**Tuỳ chọn A – Dùng trường có sẵn**: điền `Team.affiliation` là `PTIT-HN` hoặc `PTIT-HCM`.
  
**Tuỳ chọn B – Dùng file map** `config/groups.yml`:
```yaml
PTIT-HCM:
  - DWY_YK
  - 3xpl017Cr3w
  - Aw4k3n
  - n0_Fl6g_N0_L0ve
  - tml
PTIT-HN:
  - ".*"   # còn lại auto vào HN (nếu không match HCM)
```

**Tuỳ chọn C – Regex mapping** trong `config.py` (ưu tiên cao hơn):
```python
GROUP_MAPPING_REGEX = [
    (r"^(DWY_YK|3xpl017Cr3w|Aw4k3n|n0_Fl6g_N0_L0ve|tml)$", "PTIT-HCM"),
]
DEFAULT_GROUP = "PTIT-HN"
```

### Huy chương & badge
- **Medals**: top 3 theo tổng điểm (🥇🥈🥉).
- **Badge “pro”**: thêm cờ `TEAM_PRO_LIST` (tên đội) hoặc theo điều kiện tuỳ chỉnh.
- **Bounty**: nhận diện challenge có tag `bounty` (hoặc tuỳ chỉnh):

```python
TEAM_PRO_LIST = {"Anti ChatGPT Pro", "QuackQuack"}
CHALLENGE_BOUNTY_TAG = "bounty"
```

### Giao diện
- CSS chính: `static/css/matrix.css`
- JS tương tác: `static/js/matrix.js`
- Sticky/rotate có thể bật/tắt bằng class util trong `matrix.css`.

---

## 🚀 Sử dụng

- Truy cập **Matrix Scoreboard**: `/matrix/scoreboard`
- Bộ lọc nhanh:
  - Lọc theo **Group** (PTIT‑HN/PTIT‑HCM).
  - Tìm theo **Team name**.
  - Ẩn/hiện hạng mục.
- Tooltip mỗi ô:
  - Tên challenge, số submit, thời gian solve đầu tiên, tag (bounty)…
- Legend:
  - **✓**: đã giải, **–**: chưa giải, **💸**: bounty, **⭐**: pro team.

---

## 📡 API

- **GET** `/matrix/api/scoreboard`
  - **Query** (tuỳ chọn): `group=PTIT-HN|PTIT-HCM`, `min_score=0`, `category=WEB`…
  - **Response (ví dụ)**:
    ```json
    {
      "generated_at": "2025-09-23T02:15:00Z",
      "categories": ["CRYPTO","FORENSIC","PWNABLE","REVERSE ENGINEERING","WEB"],
      "teams": [
        {
          "name": "Lugia",
          "group": "PTIT-HN",
          "score": 2641,
          "solved": {
            "CRYPTO": ["QuackQuack 🦆", "HubLot"],
            "WEB": ["web_1","web_2","web_3"]
          }
        }
      ]
    }
    ```

> **Lưu ý bảo mật**: API ở chế độ **read‑only** và *không* lộ flag/submission content.

---

## 🧰 Mẹo hiệu năng & vận hành

- Bật **cache** (Flask‑Caching hoặc cache tạm trong plugin) với TTL 5–15s.
- Hạn chế N+1 query bằng cách **preload** quan hệ (eager load).
- Nếu dữ liệu lớn: cân nhắc *materialized table/view* cập nhật định kỳ (celery/cron).
- Dùng **CDN**/reverse proxy để cache tĩnh (`/static/`).

---

## 🧪 Kiểm thử nhanh

```bash
# 1) Thêm vài đội mẫu + solves trên CTFd admin
# 2) Mở endpoint
curl -s http://localhost:8000/matrix/api/scoreboard | jq . | head

# 3) Mở trang HTML
xdg-open http://localhost:8000/matrix/scoreboard
```

---

## 🐞 Troubleshooting

| Vấn đề | Nguyên nhân thường gặp | Cách khắc phục |
|---|---|---|
| Trang 404 | Blueprint chưa load | Kiểm tra plugin trong `CTFd/plugins/`, restart dịch vụ |
| Không thấy CSS/JS | Đường dẫn static sai / reverse proxy | Xem log console, đảm bảo `/plugins/matrix_scoreboard/static/...` được serve |
| Nhóm không hiển thị | `affiliation` trống, map không khớp | Bổ sung `groups.yml` hoặc regex mapping trong `config.py` |
| Medals sai | Cache cũ | Xoá cache, tăng TTL hợp lý |
| Bounty không hiện | Tag không đúng | Sửa tag challenge đúng `bounty` (hoặc đổi `CHALLENGE_BOUNTY_TAG`) |

---

## 🗺️ Roadmap

- Bảng xếp hạng theo **nhóm** (tổng điểm/giải theo group).
- Chế độ **freeze** scoreboard cuối trận.
- **Export** PDF/CSV ảnh chụp matrix.
- **ELO‑like** ranking cho giải *bounty*.
- Mini‑API per‑team: `/matrix/api/team/<id_or_name>`.

---

## 📄 License

MIT – xem nội dung ở `LICENSE` (tuỳ chọn).

---

## 🙌 Credits

- **Dev**: Nguyễn Đình Hải (PTIT CTF 2025)
- Cảm ơn cộng đồng CTFd và PTIT‑CTF team.

---

## 🪵 Changelog (tóm tắt)

- **v1.5**: Thêm tag **bounty**, cải thiện tooltip, tối ưu cache.
- **v1.4**: Bổ sung **API JSON** `/matrix/api/scoreboard`.
- **v1.3**: Sticky header/first column, rotated headers.
- **v1.2**: Hiển thị **PTIT‑HN/PTIT‑HCM**, lọc theo nhóm.
- **v1.1**: Huy chương 🥇🥈🥉, badge ⭐ pro team.
- **v1.0**: Bản matrix cơ bản (đội × hạng mục × challenge).

---

### Ghi chú tích hợp Docker Compose (ví dụ)

```yaml
services:
  ctfd:
    image: ctfd/ctfd:latest
    volumes:
      - ./CTFd:/opt/CTFd
    environment:
      - UPLOAD_PROVIDER=filesystem
      - LOG_FOLDER=/var/log/CTFd
      - WORKERS=4
      # bật plugin (chỉ cần đặt thư mục plugin là auto load)
    ports: ["8000:8000"]
```

> Nếu bạn đóng gói plugin riêng: build thêm layer copy `plugins/matrix_scoreboard` vào image CTFd.

---

### Tuỳ biến giao diện nhanh

- **Đổi màu** nhóm/PTIT badges trong `static/css/matrix.css`.
- **Bật/tắt** rotate header bằng class `.rotate-header`.
- **Giảm chiều rộng** mỗi ô: tinh chỉnh `--cell-size` CSS variable.
- **Icon**: thêm vào `static/img/` và gọi trong template `_macros.html`.

---

**Happy hacking & have fun!** 🎯

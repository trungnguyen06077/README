# Mô phỏng cấp phát bộ nhớ: First-Fit, Best-Fit, Worst-Fit

> **Bài tập lập trình giải thuật hệ điều hành** – Nhóm Memory Management / Dynamic Storage Allocation
> Ứng dụng desktop có GUI mô phỏng ba chiến lược cấp phát vùng nhớ liên tục: **First-Fit**, **Best-Fit**, **Worst-Fit**.

---

## 1. Thông tin nhóm

| Mục | Nội dung |
|---|---|
| Tên đề tài | Mô phỏng First-Fit, Best-Fit, Worst-Fit |
| Nhóm | `<Tên nhóm>` |
| Lớp / Học phần | `<Mã lớp>` – Hệ điều hành |
| Giảng viên | `<Tên giảng viên>` |
| Link repository | `<URL Git repo>` |
| Link video demo | `<URL video>` |

### Thành viên và phân công

| MSSV | Họ tên | Vai trò | Phụ trách chính |
|---|---|---|---|
| `<MSSV>` | `<Họ tên>` | Trưởng nhóm | Quản lý tiến độ, tích hợp, review PR |
| `<MSSV>` | `<Họ tên>` | Thuật toán | Cài đặt 3 thuật toán, pseudocode/flowchart |
| `<MSSV>` | `<Họ tên>` | GUI / Dữ liệu | Giao diện, đọc/ghi CSV, validate dữ liệu |
| `<MSSV>` | `<Họ tên>` | Kiểm thử | Unit/integration test, stress test, oracle |
| `<MSSV>` | `<Họ tên>` | Báo cáo / Demo | Báo cáo, slide, video demo |

---

## 2. Mục tiêu

- Xây dựng ứng dụng **desktop có GUI** mô phỏng cơ chế cấp phát bộ nhớ liên tục trong hệ điều hành.
- Giải thích được ba thuật toán, chứng minh tính đúng đắn bằng dữ liệu mẫu có thể lặp lại.
- So sánh các thuật toán về mức độ phân mảnh, tỷ lệ cấp phát thành công và hiệu năng.
- Trình bày kết quả, demo trước lớp.

## 3. Tóm tắt bài toán và thuật toán

Cho danh sách **khối nhớ trống (hole)** và danh sách **yêu cầu cấp phát (process)**. Với mỗi yêu cầu theo thứ tự đến, hệ thống chọn một hole đủ lớn theo chiến lược:

| Thuật toán | Quy tắc chọn hole | Ưu điểm | Nhược điểm |
|---|---|---|---|
| **First-Fit** | Hole đầu tiên (theo địa chỉ) có `size ≥ yêu cầu` | Nhanh, đơn giản | Dồn phân mảnh ở đầu bộ nhớ |
| **Best-Fit** | Hole nhỏ nhất thỏa `size ≥ yêu cầu` | Tiết kiệm hole lớn | Chậm hơn, sinh nhiều mảnh vụn nhỏ |
| **Worst-Fit** | Hole lớn nhất thỏa `size ≥ yêu cầu` | Phần dư còn đủ lớn để tái sử dụng | Phá hủy hole lớn, thường kém hiệu quả |

Sau khi cấp phát, hole bị cắt: phần dư `size − yêu cầu` vẫn là một hole trống (nếu `> 0`). Nếu không có hole phù hợp, yêu cầu được ghi nhận **FAILED**.

### Pseudocode

```text
FIRST_FIT(holes, request):
    for each hole in holes (theo thứ tự địa chỉ):
        if hole.size >= request.size:
            return hole
    return NONE

BEST_FIT(holes, request):
    best = NONE
    for each hole in holes:
        if hole.size >= request.size and (best == NONE or hole.size < best.size):
            best = hole
    return best

WORST_FIT(holes, request):
    worst = NONE
    for each hole in holes:
        if hole.size >= request.size and (worst == NONE or hole.size > worst.size):
            worst = hole
    return worst

ALLOCATE(holes, request, strategy):
    h = strategy(holes, request)
    if h == NONE: ghi trace FAILED; return
    h.size = h.size - request.size
    h.start = h.start + request.size
    nếu h.size == 0 thì xóa h khỏi holes
    ghi trace ALLOCATED
```

> Flowchart: xem `docs/flowchart_first_fit.png`, `docs/flowchart_best_fit.png`, `docs/flowchart_worst_fit.png`.

---

## 4. Công nghệ và yêu cầu kỹ thuật

| Hạng mục | Lựa chọn |
|---|---|
| Ngôn ngữ | Python 3.10+ *(có thể đổi thành C# / C++ / Java theo nhóm)* |
| GUI | Tkinter (chạy trên Windows, không phải ứng dụng console) |
| Kiểm thử | `pytest` |
| Định dạng dữ liệu | CSV (UTF-8) |
| Kiến trúc | Tách riêng **tầng logic** (`core/`) và **tầng GUI** (`gui/`) |

### Yêu cầu hệ thống

- Windows 10/11 (khuyến nghị); Linux/macOS có Tkinter cũng chạy được.
- Python ≥ 3.10.

---

## 5. Cài đặt và chạy

```bash
# 1. Clone repository
git clone <URL Git repo>
cd memory-allocation-sim

# 2. (Tùy chọn) Tạo môi trường ảo
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux/macOS

# 3. Cài thư viện
pip install -r requirements.txt

# 4. Chạy ứng dụng GUI
python -m gui.main
```

### Chạy kiểm thử

```bash
pytest -v                         # toàn bộ test
pytest tests/test_unit.py -v      # unit test
pytest tests/test_stress.py -v    # stress / performance test
```

---

## 6. Hướng dẫn sử dụng GUI

1. Bấm **Open Blocks CSV** để nạp danh sách khối nhớ (hoặc dùng `data/blocks_sample.csv`).
2. Bấm **Open Requests CSV** để nạp danh sách yêu cầu (hoặc dùng `data/requests_sample.csv`).
3. Chọn thuật toán: **First-Fit / Best-Fit / Worst-Fit / So sánh cả ba**.
4. Bấm **Run** (chạy toàn bộ) hoặc **Step** (từng bước) để quan sát trạng thái bộ nhớ.
5. Xem bảng kết quả, biểu đồ trực quan bộ nhớ và chỉ số phân mảnh.
6. Bấm **Export Trace CSV** để xuất trace/kết quả.

> Ảnh chụp màn hình: `docs/screenshots/`.

---

## 7. Dữ liệu vào / ra (CSV)

### 7.1. Input – khối nhớ: `blocks.csv`

| Cột | Kiểu | Ý nghĩa | Ràng buộc |
|---|---|---|---|
| `block_id` | string/int | Mã khối nhớ | Duy nhất, không rỗng |
| `size` | int | Kích thước (KB) | Số nguyên > 0 |

### 7.2. Input – yêu cầu cấp phát: `requests.csv`

| Cột | Kiểu | Ý nghĩa | Ràng buộc |
|---|---|---|---|
| `process_id` | string/int | Mã tiến trình | Duy nhất, không rỗng |
| `size` | int | Kích thước yêu cầu (KB) | Số nguyên > 0 |

### 7.3. Output – trace: `trace_<algorithm>.csv`

| Cột | Ý nghĩa |
|---|---|
| `step` | Số thứ tự bước |
| `algorithm` | `FIRST_FIT` / `BEST_FIT` / `WORST_FIT` |
| `process_id` | Tiến trình được xử lý |
| `request_size` | Kích thước yêu cầu |
| `block_id` | Khối được chọn (rỗng nếu thất bại) |
| `block_size_before` | Kích thước khối trước khi cấp phát |
| `block_size_after` | Kích thước phần dư sau khi cấp phát |
| `status` | `ALLOCATED` / `FAILED` |

### 7.4. Kiểm tra dữ liệu (validation)

Ứng dụng báo lỗi rõ ràng (hiển thị trên GUI, không crash) khi: thiếu cột, file rỗng, giá trị không phải số, kích thước ≤ 0, `id` trùng, sai encoding.

### 7.5. Dữ liệu mẫu

`data/blocks_sample.csv`
```csv
block_id,size
B1,100
B2,500
B3,200
B4,300
B5,600
```

`data/requests_sample.csv`
```csv
process_id,size
P1,212
P2,417
P3,112
P4,426
```

---

## 8. Tính đúng đắn

Tính đúng đắn được kiểm chứng bằng **đối chiếu với pseudocode** và **kết quả tính tay (oracle)**. Với dữ liệu mẫu ở mục 7.5:

| Tiến trình | First-Fit | Best-Fit | Worst-Fit |
|---|---|---|---|
| P1 (212) | B2 (500 → 288) | B4 (300 → 88) | B5 (600 → 388) |
| P2 (417) | B5 (600 → 183) | B2 (500 → 83) | B2 (500 → 83) |
| P3 (112) | B2 (288 → 176) | B3 (200 → 88) | B5 (388 → 276) |
| P4 (426) | **FAILED** | B5 (600 → 174) | **FAILED** |

Kết luận minh họa: trên bộ dữ liệu này **Best-Fit cấp phát được cả 4 tiến trình**, còn First-Fit và Worst-Fit thất bại ở P4.

### Các trường hợp biên đã kiểm tra

- Yêu cầu **bằng đúng** kích thước hole (phần dư = 0, hole bị xóa).
- Yêu cầu **lớn hơn mọi hole** → `FAILED`.
- Nhiều hole **cùng kích thước** (quy ước: chọn hole có địa chỉ nhỏ nhất).
- Danh sách hole rỗng / danh sách yêu cầu rỗng.
- Kích thước rất lớn (gần giới hạn kiểu số nguyên).

Bảng **Expected vs Actual** đầy đủ: `docs/test_expected_vs_actual.md`.

---

## 9. Kiểm thử và hiệu năng

| Loại | Mô tả | Vị trí |
|---|---|---|
| Unit test | Từng hàm chọn hole, cắt hole, validate CSV | `tests/test_unit.py` |
| Integration test | Nạp CSV → chạy thuật toán → xuất trace | `tests/test_integration.py` |
| Test oracle | So kết quả với bảng tính tay ở mục 8 | `tests/test_oracle.py` |
| Stress test | Hàng chục nghìn khối / yêu cầu sinh ngẫu nhiên | `tests/test_stress.py` |
| Performance | Đo thời gian chạy 3 thuật toán theo kích thước đầu vào | `docs/performance.md` |

Chỉ số so sánh: số tiến trình cấp phát thành công, **phân mảnh ngoài** (tổng hole còn lại, hole lớn nhất), thời gian thực thi.

---

## 10. Cấu trúc thư mục

```text
memory-allocation-sim/
├── README.md
├── requirements.txt
├── core/                    # Tầng logic (không phụ thuộc GUI)
│   ├── allocator.py         # First-Fit, Best-Fit, Worst-Fit
│   ├── models.py            # Block, Request, TraceRecord
│   └── csv_io.py            # Đọc/ghi/validate CSV
├── gui/                     # Tầng giao diện
│   ├── main.py
│   └── views.py
├── data/
│   ├── blocks_sample.csv
│   ├── requests_sample.csv
│   └── expected_outputs/    # Kết quả mong đợi để đối chiếu
├── tests/
├── docs/
│   ├── flowchart_*.png
│   ├── screenshots/
│   ├── report.pdf           # Báo cáo 8–12 trang
│   └── slides.pdf
└── .github/                 # Issue / PR template (nếu có)
```

---

## 11. Kế hoạch 8 tuần

| Tuần | Nội dung | Trạng thái |
|---|---|---|
| 1 | Chọn đề tài, phân vai, đặc tả yêu cầu, tạo repo | ☐ |
| 2 | Mô hình dữ liệu, pseudocode/flowchart, kế hoạch test | ☐ |
| 3–4 | Hiện thực lõi thuật toán (`core/`) | ☐ |
| 5 | GUI, đọc/ghi CSV, trực quan hóa bộ nhớ | ☐ |
| 6 | Kiểm thử đúng đắn, edge case, stress/performance | ☐ |
| 7 | Hoàn thiện, review chéo, sửa lỗi | ☐ |
| 8 | Báo cáo, video, demo, bảo vệ ngắn | ☐ |

---

## 12. Quản lý repository và tiến độ

- Commit nhỏ, có ý nghĩa, theo dạng: `feat: ...`, `fix: ...`, `test: ...`, `docs: ...`.
- Mỗi tính năng làm trên branch riêng, merge qua Pull Request có review.
- Dùng **Issues** / checklist để theo dõi công việc; giảng viên có thể kiểm tra lịch sử commit vào cuối mỗi tuần.
- Repo luôn **chạy được** theo hướng dẫn mục 5 trên máy sạch.

---

## 13. Báo cáo và demo

- **Báo cáo** (8–12 trang, tiếng Việt, thuật ngữ kỹ thuật có thể kèm tiếng Anh lần đầu): giới thiệu, thiết kế, thuật toán, kiểm thử, hiệu năng, hạn chế, phân công. File: `docs/report.pdf`.
- **Video demo**: trình bày luồng chạy, kết quả, so sánh ba thuật toán. Link: `<URL video>`.
- **Slide / demo trực tiếp**: `docs/slides.pdf`.

---

## 14. Nguồn tham khảo và sử dụng AI

### Tài liệu tham khảo (tối thiểu 4 nguồn)

1. Silberschatz, Galvin, Gagne – *Operating System Concepts* (chương Main Memory).
2. Tanenbaum, Bos – *Modern Operating Systems* (chương Memory Management).
3. `<Slide/bài giảng của giảng viên>`
4. `<Tài liệu/website học thuật khác>`

### Khai báo sử dụng công cụ AI

| Công cụ | Dùng cho việc | Phần được dùng | Cách nhóm kiểm chứng |
|---|---|---|---|
| `<Tên công cụ>` | `<Gợi ý ý tưởng / giải thích / sinh test...>` | `<File / mục>` | `<Chạy test, đối chiếu tay, review chéo>` |

Nhóm chịu trách nhiệm hoàn toàn về nội dung nộp và có thể giải thích mọi dòng mã khi được hỏi.

---

## 15. Đánh giá cá nhân (contribution statement)

| Thành viên | Đóng góp cụ thể | Tỷ lệ (%) |
|---|---|---|
| `<Họ tên>` | `<Mô tả>` | `<..>` |
| `<Họ tên>` | `<Mô tả>` | `<..>` |
| `<Họ tên>` | `<Mô tả>` | `<..>` |

Giảng viên có thể hỏi riêng từng thành viên hoặc yêu cầu chạy lại một phần để xác nhận đóng góp.

---

## 16. Checklist tiêu chí hoàn thành

- [ ] Ứng dụng chạy được theo README, có GUI (không phải console).
- [ ] Logic và GUI được tách riêng.
- [ ] Đọc CSV đầu vào, có validate; hiển thị kết quả và xuất trace/kết quả CSV.
- [ ] Có dữ liệu mẫu và kết quả mong đợi trong repo.
- [ ] Có pseudocode/flowchart; kết quả khớp bảng tính tay (oracle).
- [ ] Có unit/integration test, stress test và đo hiệu năng.
- [ ] Repo Git có lịch sử commit rõ ràng, có issue/checklist.
- [ ] Báo cáo 8–12 trang, video demo và slide đã nộp.
- [ ] Có ít nhất 4 nguồn tham khảo và khai báo sử dụng AI.
- [ ] Có bảng đóng góp cá nhân.

---

## 17. Kết quả mong đợi

Sau dự án, nhóm hiểu và hiện thực được cơ chế **Memory Management** (cấp phát vùng nhớ liên tục), có sản phẩm phần mềm hoàn chỉnh kèm kho Git, bộ kiểm thử, và đánh giá được sự đánh đổi (trade-off) giữa First-Fit, Best-Fit, Worst-Fit về phân mảnh, tốc độ và khả năng sử dụng bộ nhớ.

## 18. Hạn chế và hướng phát triển

- Chưa mô phỏng giải phóng bộ nhớ và gộp hole (coalescing) / nén bộ nhớ (compaction) *(có thể mở rộng)*.
- Có thể bổ sung Next-Fit, biểu đồ so sánh động và xuất báo cáo tự động.

## 19. Giấy phép

Dự án phục vụ mục đích học tập. `<Chọn license nếu cần>`

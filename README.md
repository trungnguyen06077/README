# Mô phỏng cấp phát bộ nhớ: First-Fit, Best-Fit, Worst-Fit

> **Bài tập lập trình giải thuật hệ điều hành** – Cơ chế Memory Management (cấp phát vùng nhớ liên tục)
> Ứng dụng **desktop GUI viết bằng C++** mô phỏng ba chiến lược: **First-Fit**, **Best-Fit**, **Worst-Fit**.

---

## 1. Mục tiêu

- Xây dựng ứng dụng desktop có GUI mô phỏng cơ chế cấp phát vùng nhớ của hệ điều hành.
- **Giải thích được mô hình**: bộ nhớ gồm các hole (khối trống) và các yêu cầu cấp phát; mỗi thuật toán có một quy tắc chọn hole riêng.
- **Chứng minh kết quả**: đối chiếu với pseudocode và kết quả tính tay trên dữ liệu có thể tái lập.
- **Đo hiệu năng** của ba thuật toán và **trình bày sản phẩm** có thể tái lập (cùng input thì luôn cùng output).

## 2. Quy mô và phân vai

Nhóm **5 sinh viên**, thực hiện trong khoảng **8 tuần**.

| MSSV | Họ tên | Vai trò | Phụ trách chính |
|---|---|---|---|
| `<MSSV>` | `<Họ tên>` | Trưởng nhóm / Quản lý tiến độ | Kế hoạch, theo dõi issue, review & merge PR, tích hợp |
| `<MSSV>` | `<Họ tên>` | Thuật toán | Cài đặt First/Best/Worst-Fit, pseudocode, flowchart |
| `<MSSV>` | `<Họ tên>` | Thuật toán | Mô hình dữ liệu, đo hiệu năng, tối ưu |
| `<MSSV>` | `<Họ tên>` | Giao diện / Dữ liệu | GUI, đọc/ghi CSV, kiểm tra dữ liệu, xử lý lỗi |
| `<MSSV>` | `<Họ tên>` | Kiểm thử / Tài liệu | Bảng kiểm thử, báo cáo, video demo |

| Thông tin | Nội dung |
|---|---|
| Nhóm / Lớp | `<Tên nhóm>` – `<Mã lớp>` |
| Giảng viên | `<Tên giảng viên>` |
| Git repository | `<URL repo>` |
| Video demo (công khai) | `<URL video>` |

### Kế hoạch 8 tuần

| Tuần | Nội dung | Xong |
|---|---|---|
| 1 | Phân tích yêu cầu, phân vai, tạo repo, chọn thư viện GUI | ☐ |
| 2 | Pseudocode/flowchart, định nghĩa định dạng CSV, kế hoạch kiểm thử | ☐ |
| 3–4 | Cài đặt lõi thuật toán (`src/core`) + unit test | ☐ |
| 5 | GUI, đọc/ghi CSV, kiểm tra dữ liệu, xử lý lỗi | ☐ |
| 6 | Kiểm thử tích hợp, edge case, đo hiệu năng | ☐ |
| 7 | Hoàn thiện, review chéo, sửa lỗi | ☐ |
| 8 | Báo cáo, quay video demo, trình bày | ☐ |

---

## 3. Nhiệm vụ

1. **Phân tích yêu cầu** – xem mục 4.
2. **Mô tả thuật toán** bằng pseudocode / flowchart – xem mục 5.
3. **Xây dựng ứng dụng** C++ có GUI – xem mục 6, 7, 8.
4. **Kiểm thử** với dữ liệu có thể tái lập – xem mục 10.
5. **Viết báo cáo** và **nộp mã nguồn đầy đủ** – xem mục 12.

---

## 4. Phân tích bài toán

Cho danh sách **hole** (khối nhớ trống, có địa chỉ bắt đầu và kích thước) và danh sách **yêu cầu cấp phát** theo thứ tự đến. Với mỗi yêu cầu, hệ thống chọn một hole đủ lớn theo chiến lược:

| Thuật toán | Quy tắc chọn hole | Ưu điểm | Nhược điểm |
|---|---|---|---|
| **First-Fit** | Hole **đầu tiên theo địa chỉ** có `size >= yêu cầu` | Nhanh, đơn giản | Phân mảnh dồn ở đầu bộ nhớ |
| **Best-Fit** | Hole **nhỏ nhất** thỏa `size >= yêu cầu` | Giữ lại hole lớn | Duyệt toàn bộ, sinh nhiều mảnh vụn |
| **Worst-Fit** | Hole **lớn nhất** thỏa `size >= yêu cầu` | Phần dư còn đủ lớn | Phá hole lớn, thường kém hiệu quả |

**Quy ước để kết quả tái lập được (deterministic):**

- Duyệt hole theo **địa chỉ bắt đầu tăng dần**.
- Khi có nhiều hole cùng kích thước (Best/Worst-Fit), chọn hole có **địa chỉ nhỏ nhất**.
- Cấp phát từ **đầu hole**: `alloc_start = hole.start`; phần dư `hole.start += size`, `hole.size -= size`. Nếu `hole.size == 0` thì xóa hole.
- Không có hole phù hợp thì ghi `FAILED`; không dừng chương trình, tiếp tục yêu cầu kế tiếp.
- Mỗi thuật toán chạy trên **bản sao độc lập** của danh sách hole ban đầu.

## 5. Mô tả thuật toán (pseudocode)

```text
FIRST_FIT(holes, size):
    for h in holes (địa chỉ tăng dần):
        if h.size >= size: return h
    return NONE

BEST_FIT(holes, size):
    best = NONE
    for h in holes:
        if h.size >= size and (best == NONE or h.size < best.size):
            best = h              # dấu < giữ hole có địa chỉ nhỏ nhất khi bằng nhau
    return best

WORST_FIT(holes, size):
    worst = NONE
    for h in holes:
        if h.size >= size and (worst == NONE or h.size > worst.size):
            worst = h
    return worst

SIMULATE(holes, requests, strategy):
    for each r in requests (theo thứ tự):
        h = strategy(holes, r.size)
        if h == NONE:
            ghi trace(r, FAILED)
        else:
            ghi trace(r, ALLOCATED, h.id, alloc_start = h.start)
            h.start += r.size;  h.size -= r.size
            if h.size == 0: xóa h khỏi holes
```

Flowchart: `docs/flowchart_first_fit.png`, `docs/flowchart_best_fit.png`, `docs/flowchart_worst_fit.png`.

---

## 6. Yêu cầu kỹ thuật

| Yêu cầu | Cách đáp ứng |
|---|---|
| Ngôn ngữ **C++** | C++17 trở lên |
| **GUI chạy trên Windows**, không phải console, không phải Web App | Ứng dụng desktop **Qt 6 (Widgets)** |
| Xử lý lỗi, **không crash** với input hợp lệ | Kiểm tra dữ liệu trước khi chạy, báo lỗi bằng hộp thoại (mục 8) |
| **Tách phần thuật toán khỏi giao diện** | `src/core` là thư viện C++ thuần (không phụ thuộc Qt); `src/gui` chỉ gọi API của core |

### Công cụ cần có

- Windows 10/11
- Visual Studio 2022 (MSVC) hoặc MinGW-w64
- CMake ≥ 3.21
- Qt 6.x (Qt Widgets)
- Git

### Cấu trúc thư mục

```text
memory-allocation-sim/
├── README.md
├── CMakeLists.txt
├── src/
│   ├── core/                  # Thuật toán + CSV (KHÔNG dùng Qt)
│   │   ├── models.hpp         # Hole, Request, TraceRecord, Metrics
│   │   ├── allocator.hpp/.cpp # First-Fit, Best-Fit, Worst-Fit
│   │   ├── csv_io.hpp/.cpp    # Đọc / ghi CSV
│   │   └── validator.hpp/.cpp # Kiểm tra dữ liệu đầu vào
│   └── gui/                   # Giao diện Qt
│       ├── main.cpp
│       └── MainWindow.hpp/.cpp
├── tests/                     # Unit / integration / stress test
├── data/
│   ├── input/                 # CSV mẫu
│   │   ├── blocks_sample.csv
│   │   └── requests_sample.csv
│   └── output/                # Trace/kết quả xuất ra (CSV)
│       └── expected/          # Kết quả mong đợi để đối chiếu
└── docs/
    ├── flowchart_*.png
    ├── screenshots/
    ├── test_table.md          # Bảng kiểm thử
    ├── performance.md
    └── report.pdf
```

---

## 7. Cài đặt, build và chạy

```bash
# 1. Lấy mã nguồn
git clone <URL repo>
cd memory-allocation-sim

# 2. Cấu hình (chỉ đường dẫn tới Qt của máy bạn)
cmake -S . -B build -DCMAKE_PREFIX_PATH="C:/Qt/6.x.x/msvc2019_64"

# 3. Build
cmake --build build --config Release

# 4. Gom DLL của Qt để chạy độc lập (Windows)
windeployqt build/Release/memsim.exe

# 5. Chạy ứng dụng
build\Release\memsim.exe
```

Chạy kiểm thử:

```bash
ctest --test-dir build -C Release --output-on-failure
```

### Hướng dẫn sử dụng GUI

1. Bấm **Open Blocks CSV** và chọn `data/input/blocks_sample.csv`.
2. Bấm **Open Requests CSV** và chọn `data/input/requests_sample.csv`.
3. Chọn thuật toán: **First-Fit / Best-Fit / Worst-Fit / So sánh cả ba**.
4. Bấm **Run** (chạy hết) hoặc **Step** (từng bước).
5. Xem **bảng kết quả** và **hình minh họa bộ nhớ**; xem chỉ số phân mảnh và thời gian chạy.
6. Bấm **Export CSV** để xuất trace/kết quả chi tiết vào `data/output/`.

> Ảnh chụp giao diện: `docs/screenshots/`.

---

## 8. Dữ liệu vào / ra (CSV)

Quy ước chung: file **UTF-8**, dấu phân cách **dấu phẩy `,`**, **dòng đầu là tiêu đề** (header), kích thước tính bằng **KB**.

### 8.1. Input: `blocks.csv` (các hole ban đầu)

| Cột | Kiểu | Ý nghĩa | Ràng buộc |
|---|---|---|---|
| `block_id` | chuỗi | Mã khối nhớ | Không rỗng, **duy nhất** |
| `start` | số nguyên ≥ 0 | Địa chỉ bắt đầu | Các hole **không chồng lấn** |
| `size` | số nguyên > 0 | Kích thước hole | Số nguyên dương |

### 8.2. Input: `requests.csv` (yêu cầu cấp phát)

| Cột | Kiểu | Ý nghĩa | Ràng buộc |
|---|---|---|---|
| `order` | số nguyên ≥ 1 | Thứ tự xử lý | **Duy nhất**; xử lý theo `order` tăng dần |
| `process_id` | chuỗi | Mã tiến trình | Không rỗng |
| `size` | số nguyên > 0 | Kích thước yêu cầu | Số nguyên dương |

Hai file trên cùng với các **quy ước ở mục 4** là đủ để tái lập chính xác kết quả.

### 8.3. Output: `trace_<algorithm>.csv` (chi tiết từng bước)

| Cột | Ý nghĩa |
|---|---|
| `step` | Số thứ tự bước |
| `algorithm` | `FIRST_FIT` / `BEST_FIT` / `WORST_FIT` |
| `process_id` | Tiến trình được xử lý |
| `request_size` | Kích thước yêu cầu |
| `block_id` | Khối được chọn (rỗng nếu `FAILED`) |
| `alloc_start` | Địa chỉ được cấp phát (rỗng nếu `FAILED`) |
| `block_size_before` | Kích thước hole trước khi cấp phát |
| `block_size_after` | Kích thước phần dư sau khi cấp phát |
| `status` | `ALLOCATED` / `FAILED` |

### 8.4. Output: `summary.csv` (kết quả tổng hợp)

| Cột | Ý nghĩa |
|---|---|
| `algorithm` | Tên thuật toán |
| `allocated` / `failed` | Số yêu cầu thành công / thất bại |
| `total_free` | Tổng dung lượng hole còn lại |
| `largest_free` | Hole lớn nhất còn lại |
| `time_us` | Thời gian chạy (micro giây) |

### 8.5. Kiểm tra dữ liệu và xử lý lỗi

Ứng dụng **không crash**; khi gặp lỗi sẽ hiện thông báo nêu rõ **file, dòng, nguyên nhân**:

- File không tồn tại, không mở được, file rỗng, thiếu cột hoặc sai tên cột.
- Giá trị không phải số, số âm hoặc bằng 0 ở cột `size`.
- `block_id` / `order` bị trùng; các hole chồng lấn địa chỉ.
- Dòng thiếu trường, dòng trống ở giữa file.

### 8.6. Dữ liệu mẫu

`data/input/blocks_sample.csv`
```csv
block_id,start,size
B1,0,100
B2,150,500
B3,700,200
B4,950,300
B5,1300,600
```

`data/input/requests_sample.csv`
```csv
order,process_id,size
1,P1,212
2,P2,417
3,P3,112
4,P4,426
```

---

## 9. Kết quả mong đợi và tính đúng đắn

Với dữ liệu mẫu ở mục 8.6 (tính tay, dùng làm oracle):

| Yêu cầu | First-Fit | Best-Fit | Worst-Fit |
|---|---|---|---|
| P1 (212) | B2 (500 → 288) | B4 (300 → 88) | B5 (600 → 388) |
| P2 (417) | B5 (600 → 183) | B2 (500 → 83) | B2 (500 → 83) |
| P3 (112) | B2 (288 → 176) | B3 (200 → 88) | B5 (388 → 276) |
| P4 (426) | **FAILED** | B5 (600 → 174) | **FAILED** |

Nhận xét: trên bộ dữ liệu này **Best-Fit cấp phát thành công cả 4 yêu cầu**, còn First-Fit và Worst-Fit thất bại ở P4.

Kết quả mong đợi dạng CSV nằm trong `data/output/expected/` để chương trình kiểm thử tự đối chiếu.

---

## 10. Kiểm thử

### Bảng kiểm thử

| Mã | Mô tả | Input | Kết quả mong đợi | Kết quả thực tế | Đạt |
|---|---|---|---|---|---|
| TC01 | Dữ liệu mẫu, First-Fit | sample | P4 FAILED, 3 cấp phát | `<điền>` | ☐ |
| TC02 | Dữ liệu mẫu, Best-Fit | sample | Cả 4 ALLOCATED | `<điền>` | ☐ |
| TC03 | Dữ liệu mẫu, Worst-Fit | sample | P4 FAILED, 3 cấp phát | `<điền>` | ☐ |
| TC04 | Yêu cầu bằng đúng kích thước hole | hole 100, yêu cầu 100 | Hole bị xóa, phần dư 0 | `<điền>` | ☐ |
| TC05 | Yêu cầu lớn hơn mọi hole | yêu cầu > max | FAILED, không crash | `<điền>` | ☐ |
| TC06 | Nhiều hole cùng kích thước | 2 hole bằng nhau | Chọn hole địa chỉ nhỏ hơn | `<điền>` | ☐ |
| TC07 | Danh sách yêu cầu rỗng | file chỉ có header | Kết quả rỗng, không crash | `<điền>` | ☐ |
| TC08 | CSV thiếu cột | bỏ cột `size` | Báo lỗi rõ ràng | `<điền>` | ☐ |
| TC09 | `size` không phải số / âm | `abc`, `-5` | Báo lỗi đúng dòng | `<điền>` | ☐ |
| TC10 | `block_id` trùng | 2 dòng cùng ID | Báo lỗi trùng ID | `<điền>` | ☐ |
| TC11 | Dữ liệu lớn (stress) | 100.000 hole, 100.000 yêu cầu | Chạy xong, kết quả nhất quán | `<điền>` | ☐ |
| TC12 | Tái lập kết quả | chạy 2 lần cùng input | Hai output giống hệt nhau | `<điền>` | ☐ |

Bảng đầy đủ: `docs/test_table.md`.

### Loại kiểm thử

- **Unit test** từng hàm chọn hole, cắt hole, kiểm tra dữ liệu (`tests/`).
- **Integration test**: nạp CSV → chạy → xuất CSV → so với `expected/`.
- **Stress / hiệu năng**: đo `time_us` của ba thuật toán với kích thước đầu vào tăng dần (1.000 → 100.000), ghi vào `docs/performance.md`.

---

## 11. Sản phẩm bàn giao

| Sản phẩm | Vị trí / trạng thái |
|---|---|
| Git repository | `<URL repo>` |
| Ứng dụng chạy được | Build theo mục 7 |
| Thư mục input/output kèm CSV mẫu | `data/input/`, `data/output/` |
| README hướng dẫn cài đặt và chạy | File này |
| Báo cáo | `docs/report.pdf` |
| Bảng kiểm thử | Mục 10 và `docs/test_table.md` |
| Video demo công khai | `<URL video>` |

## 12. Checklist hoàn thành

- [ ] Ứng dụng C++ có GUI, chạy được trên Windows, không phải console/Web App.
- [ ] Phần thuật toán (`src/core`) tách khỏi giao diện (`src/gui`).
- [ ] Đọc CSV do nhóm định nghĩa, có kiểm tra dữ liệu, không crash với input hợp lệ.
- [ ] Hiển thị kết quả chính trên màn hình và xuất trace/kết quả chi tiết ra CSV.
- [ ] Input đủ trường để tái lập; README mô tả định dạng CSV.
- [ ] Có pseudocode/flowchart; kết quả khớp bảng tính tay.
- [ ] Có bảng kiểm thử và số liệu đo hiệu năng.
- [ ] Git repository có lịch sử commit rõ ràng của cả 5 thành viên.
- [ ] Có báo cáo và video demo công khai, link truy cập hoạt động.

## 13. Hạn chế và hướng phát triển

- Chưa mô phỏng giải phóng bộ nhớ, gộp hole (coalescing) và nén bộ nhớ (compaction).
- Có thể bổ sung Next-Fit, biểu đồ so sánh động và xuất báo cáo tự động.

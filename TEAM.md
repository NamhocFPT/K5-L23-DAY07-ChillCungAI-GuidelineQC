# Thông tin nhóm

- Tên nhóm: ChillCungAI
- Repo bài nộp (URL): https://github.com/NamhocFPT/K5-L23-DAY07-ChillCungAI-GuidelineQC
- Commit chốt bài (hash): ddec8a5
- Số thành viên: 5  → số người gán mỗi ảnh (K): 3

| STT | MSHV | Họ và tên | Lớp (L2 / L3 / khác) | Tài khoản GitHub |
|---|---|---|---|---|
| 1 | 2A202603186 | Nguyễn Hoài Nam | L2 | https://github.com/NamhocFPT |
| 2 | 2A202603116 | Nguyễn Quang Thực | L2 | https://github.com/QuocThangNguyen-091102 |
| 3 | 2A202603327 | Đinh Đức Hiếu | L2 | https://github.com/hieudd04268 |
| 4 | 2A202603158 | Nguyễn Quốc Thắng | L2 | https://github.com/QuocThangNguyen-091102 |
| 5 | 2A202603255 | Doãn Thị Phương Thảo | L2 | https://github.com/dpthao1810 |

Danh sách trên phải khớp dòng tiêu đề của `README.md` (đã đổi theo [SUBMISSION.md](SUBMISSION.md)) và `--members` đã dùng ở `scripts/split_assign.py`.

## Phân công (mỗi người làm gì ngoài phần gán nhãn)

| MSHV | Phụ trách thêm |
|---|---|
| 2A202603116 | Leader: chạy `split_assign`, phụ trách guideline |
| 2A202603186 - Nguyễn Hoài Nam | Phụ trách QC/report |
| 2A202603327 - Đinh Đức Hiếu | Phụ trách QC/report |
| 2A202603255 - Doãn Thị Phương Thảo | Phụ trách QC/report |

## Kết quả tóm tắt (điền cuối bài)

- Số thành viên / số lần gán mỗi ảnh (K): 5 / 3
- IAA trước calibration: IoU trung bình **0.863**, đồng thuận lớp **88.8%** (mẫu số: **474 cặp box ghép được**; ngưỡng lần lượt 0.85 và 95%).
- Số ảnh phải calibration: **38 / 40**.
- Tỉ lệ lỗi sau calibration: **chưa thực hiện** theo ảnh và theo đối tượng (chưa có mẫu QC sau calibration).
- **Quyết định:** **Rework** — IAA đồng thuận lớp mới đạt 88.8%, thấp hơn ngưỡng 95%; cần calibration và QC lại trước khi Release.

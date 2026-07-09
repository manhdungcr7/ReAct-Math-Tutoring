# Báo cáo quy trình trích xuất dữ liệu từ file DAP AN.pdf

## 1. Mục tiêu

Từ file `DAP AN.pdf` (247 trang, tổng hợp nhiều đề thi thử TN THPT có lời giải chi tiết), trích ra 66 câu trắc nghiệm thuộc chủ đề **số phức**, chuyển đề bài và lời giải sang định dạng Markdown/LaTeX để làm ngữ liệu thử nghiệm các mô hình ngôn ngữ lớn.

## 2. Quy trình thực hiện

**Bước 1 — Xác định vị trí các câu về số phức.**
Dùng thư viện `pdfplumber` trích xuất toàn bộ văn bản thô của 247 trang, sau đó tìm kiếm các dòng bắt đầu bằng "Câu N:" có chứa cụm từ "số phức" để khoanh vùng câu hỏi. Kết quả: 68 câu ứng viên, trải từ trang 10 đến trang 245.

**Bước 2 — Loại các câu phụ thuộc hình vẽ.**
Kiểm tra ngữ cảnh từng câu (đoạn văn bản từ câu đó đến câu tiếp theo) để phát hiện các câu có nhắc đến "hình vẽ" — tức phải nhìn hình mới xác định được đáp án. Phát hiện 2 câu như vậy (trang 11 câu 31, trang 24 câu 29) và loại bỏ, theo đúng quy ước nhóm đã thống nhất với thầy khi làm 200 mẫu trước đó. Còn lại **66 câu**.

**Bước 3 — Render trang PDF thành ảnh để trích chính xác công thức toán.**
Văn bản trích xuất thuần túy (pdfplumber) làm vỡ cấu trúc công thức LaTeX gốc (căn thức, chỉ số dưới, phân số bị tách rời không theo thứ tự đọc tự nhiên do đặc thù cách PDF được dàn trang từ LaTeX). Do đó, dùng thư viện `PyMuPDF` render từng trang liên quan thành ảnh PNG độ phân giải cao (matrix 2.2x), sau đó đọc trực tiếp ảnh (như đọc bằng mắt) để gõ lại chính xác đề bài, 4 phương án, và lời giải.

**Bước 4 — Transcribe sang Markdown/LaTeX.**
Với mỗi câu, gõ lại:
- Đề bài + 4 phương án A/B/C/D, công thức toán đặt trong dấu `$...$`.
- Đáp án đúng, đối chiếu ký hiệu "⁄ Chọn đáp án X" được khoanh tròn trong lời giải.
- Lời giải chi tiết gốc theo tài liệu (transcribe nguyên văn, không diễn giải lại).

Với các câu có lời giải bị ngắt sang trang sau, render thêm trang tiếp theo để lấy trọn vẹn kết luận "Chọn đáp án".

**Bước 5 — Kiểm tra chéo và sửa lỗi.**
Rà soát lại toàn bộ 66 câu, đối chiếu đáp án được gán với nội dung lời giải để phát hiện sai lệch. Phát hiện và sửa 1 trường hợp gán nhầm đáp án (trang 40, câu 39 — lời giải kết luận "4 số phức thỏa mãn" nhưng bị gán nhầm đáp án B thay vì đúng phải là C).

Ngoài ra, trong quá trình cho các mô hình giải và đối chiếu, phát hiện thêm 1 câu (câu 55 trong bộ 66) mà **chính đáp án gốc trong tài liệu PDF bị sai** (lỗi tính sai dấu khi quy đổi từ ẩn phụ về biến $w$ gốc). Đã tính lại và sửa đáp án đúng cho câu này.

## 3. Kết quả

- 66 câu số phức, đầy đủ đề bài (Markdown/LaTeX), đáp án đúng, và lời giải gốc theo tài liệu, lưu trong file `So_phuc_60_mau.xlsx` / `.csv`.
- Đã kiểm tra và sửa 2 lỗi phát sinh trong quá trình biên soạn/số hóa gốc của tài liệu (1 lỗi gán đáp án, 1 lỗi tính toán trong đáp án).

## 4. Công cụ sử dụng

| Công cụ | Mục đích |
|---|---|
| `pdfplumber` | Trích xuất văn bản thô để định vị câu hỏi |
| `PyMuPDF (fitz)` | Render trang PDF thành ảnh để đọc và transcribe chính xác |
| Đọc ảnh trực tiếp (thủ công có hỗ trợ) | Gõ lại đề bài/lời giải sang Markdown/LaTeX, đối chiếu đáp án |

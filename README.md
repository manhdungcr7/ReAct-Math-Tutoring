# Bộ dữ liệu Số phức — Khóa luận tốt nghiệp

Repo này chứa dữ liệu và script dùng cho khóa luận so sánh khả năng giải toán
Số phức (THPT, chương trình trước 2025) của các mô hình ngôn ngữ lớn (LLM),
cùng bộ dữ liệu câu hỏi mở rộng dùng để đánh giá.

## Cấu trúc repo

```
So_phuc_60_mau.xlsx / .csv          66 câu số phức gốc (trích từ tài liệu ôn
                                     thi TNTHPT), kèm lời giải/đáp án gốc và
                                     kết quả 3 mô hình (ChatGPT, Gemini, Qwen)
                                     giải + người chấm nhận xét.

solve_with_models.py                Gọi API ChatGPT/Gemini để giải 66 câu gốc
grade_answers.py                    Tự động chấm đáp án các model trả về

Sinh_them_cau_hoi/
  scripts/10_common_utils.py        Hàm dùng chung (định dạng LaTeX, sinh số
                                     ngẫu nhiên theo loại nguyên/hữu tỉ/vô tỉ)
  scripts/11..17_generate_dangN.py  Sinh câu hỏi nhân bản cho từng dạng (1-7)
  scripts/20_export_all.py          Gộp 66 câu gốc + toàn bộ câu nhân bản
                                     thành 1 file Excel/CSV
  scripts/21_verify_all.py          Kiểm tra độc lập từng câu nhân bản bằng
                                     sympy (giải lại từ đầu, không dùng lại
                                     công thức đã sinh ra câu hỏi)
  data/dangN_full.json              Dữ liệu câu nhân bản đã sinh (đầu ra của
                                     script 11-17, đầu vào của script 20/21)
  So_phuc_day_du.xlsx / .csv        **File dữ liệu cuối cùng**: 762 câu
                                     (66 gốc + 696 nhân bản), đã kiểm tra

Danh_gia_chuyen_doi_markdown/
  scripts/                          Pipeline chuyển đổi PDF → Markdown/LaTeX
                                     bằng AI đọc ảnh (không OCR) + quy trình
                                     đánh giá chất lượng chuyển đổi 2 người
  Bang_danh_gia_chuyen_doi_markdown.xlsx / .csv
                                     Bảng đánh giá 120 mẫu (66 số phức + 54
                                     câu thuộc 6 chủ đề khác)
```

> File `DAP AN.pdf` (tài liệu ôn thi gốc, có bản quyền) và ảnh các trang PDF
> không được đưa vào repo công khai này.

## Phương pháp sinh câu hỏi nhân bản

1. Phân loại 66 câu số phức gốc thành **7 dạng bài** theo đề + lời giải chuẩn
   (tham khảo cách chia chuyên đề số phức ôn TNTHPT trước 2025).
2. Với mỗi câu gốc, viết một hàm Python (`mau_sttN`, dùng **sympy**) nhận
   tham số số học ngẫu nhiên làm đầu vào, **giải lại thật sự theo đúng
   phương pháp của lời giải gốc** (không phải chỉ thay số vào công thức có
   sẵn) — kể cả các câu phải biện luận nhiều trường hợp, đếm nghiệm, hay giải
   hệ phương trình.
3. Mỗi câu gốc được nhân bản với số lượng bằng nhau cho 3 loại số: **số
   nguyên, số hữu tỉ, số vô tỉ** (viết dạng phân số/căn thức, không dùng số
   thập phân), tổng mỗi dạng ≥ 90 câu.
4. Kiểm tra 2 lớp độc lập trước khi chốt dữ liệu:
   - `21_verify_all.py`: với mỗi câu, tính lại đáp án bằng một cách suy luận
     độc lập (không tái sử dụng logic sinh câu hỏi), so khớp với đáp án đã
     lưu.
   - Đọc tay ngẫu nhiên nhiều đợt (phủ đủ cả 66 câu gốc) để bắt lỗi hình
     thức/trình bày mà kiểm tra tự động không phát hiện được (trùng phương
     án, lỗi hiển thị LaTeX...).

## Phương pháp chuyển đổi PDF → Markdown

Xem chi tiết trong `Danh_gia_chuyen_doi_markdown/scripts/12_prompt_transcribe.txt`
và `Bao_cao_quy_trinh_trich_xuat_du_lieu.md`. Tóm tắt: render trang PDF thành
ảnh (không OCR), dùng AI đọc ảnh trực tiếp để transcribe trung thực (không tự
sửa nội dung dù thấy có vẻ sai), sau đó 2 người review độc lập đối chiếu với
ảnh gốc để phát hiện lỗi chuyển đổi.

## Cách chạy lại pipeline sinh câu hỏi

```bash
cd Sinh_them_cau_hoi/scripts
python 11_generate_dang1_full.py   # ... đến 17_generate_dang7_full.py
python 20_export_all.py            # gộp + kiểm tra + xuất Excel/CSV cuối cùng
```

`20_export_all.py` sẽ tự dừng và báo lỗi (không xuất file) nếu có bất kỳ câu
nhân bản nào không vượt qua kiểm tra độc lập ở `21_verify_all.py`.

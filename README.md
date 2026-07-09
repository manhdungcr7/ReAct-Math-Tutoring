# Bộ script — Khóa luận tốt nghiệp (Số phức)

Repo này chứa **script** dùng cho khóa luận so sánh khả năng giải toán Số
phức (THPT, chương trình trước 2025) của các mô hình ngôn ngữ lớn (LLM), và
để sinh thêm dữ liệu câu hỏi dùng cho đánh giá. Repo chỉ chứa code — không
chứa dữ liệu/kết quả (tài liệu gốc có bản quyền, các file Excel/CSV/JSON kết
quả được lưu và quản lý riêng, không đưa vào đây).

## Cấu trúc repo

```
solve_with_models.py                 Gọi API ChatGPT/Gemini để giải câu hỏi gốc
grade_answers.py                     Tự động chấm đáp án các model trả về

Sinh_them_cau_hoi/scripts/
  10_common_utils.py                 Hàm dùng chung (định dạng LaTeX, sinh số
                                      ngẫu nhiên theo loại nguyên/hữu tỉ/vô tỉ)
  11..17_generate_dangN_full.py      Sinh câu hỏi nhân bản cho từng dạng (1-7)
  20_export_all.py                   Gộp câu gốc + toàn bộ câu nhân bản thành
                                      1 file Excel/CSV
  21_verify_all.py                   Kiểm tra độc lập từng câu nhân bản bằng
                                      sympy (giải lại từ đầu, không dùng lại
                                      logic đã sinh ra câu hỏi)

Danh_gia_chuyen_doi_markdown/scripts/
  1_find_questions.py .. 19_apply_ghichu.py
                                      Pipeline quét PDF, chọn trang, render
                                      ảnh, transcribe bằng Claude API, và quy
                                      trình đánh giá chất lượng chuyển đổi
  12_prompt_transcribe.txt            Prompt dùng để transcribe ảnh sang Markdown
```

## Quy trình chuyển đổi PDF → Markdown

1. Quét toàn bộ text PDF (`Dap An.pdf`) bằng **PyMuPDF**, tìm các câu có từ
   khóa chủ đề (số phức, sau đó mở rộng sang 6 chủ đề khác), chọn ra danh
   sách trang chứa câu hỏi + lời giải liên quan.
2. Render các trang đó thành ảnh PNG.
3. Đưa ảnh kèm prompt cho **Claude API** để transcribe từ ảnh về định dạng
   Markdown/LaTeX và lưu lại.

## Quy trình sinh câu hỏi nhân bản

1. Với mỗi câu gốc, viết 1 hàm Python (dùng thư viện **sympy**) nhận số
   ngẫu nhiên làm đầu vào.
2. Hàm này giải lại theo đúng phương pháp của lời giải gốc (không phải chỉ
   thay số vào công thức có sẵn).
3. Từ đó sinh ra đề bài, 4 phương án, đáp án đúng và lời giải tương ứng với
   bộ số mới.
4. Sau khi sinh, kiểm tra lại bằng script sympy giải lại từ đầu để xác nhận
   đáp án đúng (`21_verify_all.py`), sau đó tự rà soát lại một lượt xem đã
   ổn chưa rồi điền vào cột "Nhận xét".

## Cách chạy lại pipeline sinh câu hỏi

```bash
cd Sinh_them_cau_hoi/scripts
python 11_generate_dang1_full.py   # ... đến 17_generate_dang7_full.py
python 20_export_all.py            # gộp + kiểm tra + xuất Excel/CSV cuối cùng
```

`20_export_all.py` sẽ tự dừng và báo lỗi (không xuất file) nếu có bất kỳ câu
nhân bản nào không vượt qua kiểm tra độc lập ở `21_verify_all.py`.

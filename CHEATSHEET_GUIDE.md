# Hướng dẫn viết cheatsheet

Mỗi file `cdN.pdf` là một bộ ~30 câu luyện tập (tiếng Việt) của một chuyên đề toán/AI/lập trình. Mỗi chuyên đề có một cheatsheet tại `cheatsheets/CDNN_cheatsheet.md` (NN hai chữ số, vd `CD02`).

## Đọc PDF

1. Trích text: `pdftotext -enc UTF-8 -layout cdN.pdf /tmp/cdN.txt` (cài `poppler-utils` nếu thiếu), rồi đọc file text.
2. pdftotext thường làm mất công thức (công thức là hình). Với câu bị thiếu, render trang thành PNG bằng PyMuPDF (`pip install pymupdf`, `page.get_pixmap(dpi=150)`) rồi xem ảnh.
3. Đọc TOÀN BỘ câu hỏi. Một số PDF có sẵn "Đáp án X" và giải thích: dùng đáp án đó làm chuẩn (bỏ qua ghi chú "bản in: ..."). Câu nào không có đáp án thì tự giải; câu điền số thì tính lại bằng Python.

## Định dạng

Markdown, tiếng Việt, thuật ngữ kỹ thuật giữ tiếng Anh, công thức LaTeX `$...$`. Ngắn gọn, mật độ cao, chỉ kiến thức mà đề thực sự kiểm tra, không chép nguyên văn câu hỏi.

- Tiêu đề: mã + tên chuyên đề (lấy từ PDF).
- Bảng "Ý tưởng chính được kiểm tra": mỗi ý 1 dòng, kèm mã câu liên quan (vd `T06-A02`).
- Các mục theo chủ đề: khái niệm cốt lõi, công thức, code/cú pháp mẫu ngắn, kết quả mong đợi.
- "Bẫy thường gặp": phương án nhiễu điển hình và vì sao sai.
- "Tra nhanh": bảng tình huống X → dùng Y.
- Cuối file: "Đáp án nhanh" cho mọi câu, đánh dấu câu nào là đáp án chính thức từ PDF và câu nào tự giải. Câu tự luận/code: khung trả lời hoặc lời giải mẫu.

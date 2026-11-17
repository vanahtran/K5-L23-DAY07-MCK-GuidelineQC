# Báo cáo QC — Nhóm MCK

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Dữ liệu nhãn bounding box 2D phục vụ huấn luyện và kiểm thử bài toán phát hiện đối tượng giao thông thông minh (ITS / ADAS); giới hạn trong 40 ảnh đô thị tuyển chọn rèn luyện ca biên, chưa đại diện cho mọi điều kiện thời tiết thực tế.
- Nguồn và phép sử dụng dữ liệu: Xem data/ATTRIBUTION.csv; ảnh có bản quyền Creative Commons và giấy phép mở tương thích mục đích học thuật / đào tạo.
- Manifest và checksum nhãn cuối: batch_manifest.json; SHA-256 nhãn cuối: 36e66cc254f6c89f04fb6e85f1e760fccaea945c48ec3d97cab66f3eaa9cf134.
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán áp dụng taxonomy v1.0 và guideline_v1.0_agreed.md; khi bàn giao áp dụng taxonomy v1.0 và guideline_v1.1_calibrated.md (giữ nguyên taxonomy 6 lớp, cập nhật giải tỏa 352 ca biên và đóng cờ can_review).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (100% hoàn thành).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 272 cặp box ghép, IoU trung bình 0.944 (ngưỡng >= 0.85 đạt), đồng thuận lớp 98.5% (268/272 cặp, ngưỡng >= 95% đạt).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 35 box chưa ghép của 058, 113 box chưa ghép của 173; 0 box loại do ignore; 80 ca bất đồng vùng ignore_region.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_004.jpg (cả hai cùng gán tuk-tuk vào car và tick can_review do taxonomy thiếu lớp tuk_tuk; chốt tạm gán car ở v1.1).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý toàn bộ 352 ca bất đồng theo calibration_decisions.csv; áp dụng các quy tắc R-01 (ôm sát), R-08 (vùng ignore), R-09 (tuk-tuk); người chốt 058 và người kiểm lại 173 xác nhận đạt.
- Tác động v1.1 lên nhãn và ảnh: Phạm vi hiệu chuẩn là 40 ảnh; chuẩn hóa độ ôm box, gỡ bỏ toàn bộ cờ can_review, thống nhất vùng ignore cụm xa.
- Tham chiếu bằng chứng khi Rework: Không phát sinh Rework; toàn bộ ca sửa được lập hồ sơ tại submission/qc/calibration_decisions.csv và đồng bộ vào labels_calibrated.json.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 173 (Tạ Xuân Anh Vũ), ngày 2026-11-17.
- Đã duyệt / cả lô: 40/40 ảnh đã duyệt (100% lô).
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0% (ngưỡng tối đa cho phép 2.0%).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/420 = 0.0%.
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Đã duyệt 40 ca khó (nhóm ca_kho: 0/40 lỗi); không có lỗi lặp lại.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Lỗi máy chặn được: box tọa độ âm hoặc ngoài biên ảnh (chặn bằng validator); lỗi cần người đọc ảnh: phân biệt tượng nhân vật Predator (img_011.jpg) hoặc tranh quảng cáo (img_018.jpg) với con người thật sống.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Lô nhãn đạt 100% tiêu chí chất lượng (IoU 0.944, Class Agreement 98.5%, 0 lỗi sau calibration); bàn giao cho đội ML Engineering phục vụ huấn luyện mô hình Object Detection baseline Day 08.
- Nếu còn việc: Không còn việc tồn đọng; toàn bộ 352 ca đã đóng.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C001 (img_001.jpg, person ôm sát theo R-01, id 8); ca C057 (img_004.jpg, tuk-tuk gán car theo R-09); ca C180 (img_020.jpg, gương chiếu hậu gán ignore_region theo R-06).

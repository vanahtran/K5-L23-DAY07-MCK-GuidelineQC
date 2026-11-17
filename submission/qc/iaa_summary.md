# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 2
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 272 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 272 | 0.944 | 98.5% | 69.5% | 88.2% | 35/113 | 0 | ĐẠT |
| 058|173 | 40 | 272 | 0.944 | 98.5% | 69.5% | 88.2% | 35/113 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 352 ({'iou_thap': 36, 'sai_thuoc_tinh': 98, 'bo_sot_thua': 121, 'bat_dong_ignore': 80, 'co_can_review': 13, 'sai_lop': 4}), trên 40 ảnh cần calibration.

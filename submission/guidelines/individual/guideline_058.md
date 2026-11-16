# Guideline gán nhãn — v1.0-draft-058 — Thành viên 058

- Phiên bản: v1.0-draft-058
- Ngày: 2026-11-16
- Người viết / duyệt: Thành viên 058 (Điều phối nhóm MCK)

---

## 1. Mục tiêu

Tài liệu này xác định các quy tắc chuẩn hóa nhằm gán nhãn bounding box (hộp chữ nhật 2D) cho bài toán phát hiện đối tượng (2D Object Detection) phục vụ huấn luyện các mô hình thị giác máy tính (như YOLO, Faster R-CNN, DETR).

- **Mục đích ứng dụng**: Dữ liệu phục vụ hệ thống camera giám sát giao thông thông minh (ITS) và hệ thống lái xe tự động / hỗ trợ người lái nâng cao (ADAS / Autonomous Driving) trong môi trường đường phố đô thị phức tạp.
- **Rủi ro an toàn nếu sai / sót**:
  - *Bỏ sót (False Negative)*: Bỏ sót người đi bộ (`person`) hoặc người điều khiển xe máy (`motorcycle`) là lỗi đặc biệt nghiêm trọng (Critical), có thể trực tiếp gây ra tai nạn va chạm gây thương vong trong điều hướng phương tiện tự hành.
  - *Phân loại sai (Misclassification)*: Nhầm lẫn giữa phương tiện trọng tải lớn (`truck`) và ô tô con (`car`) sẽ làm sai lệch mô hình tính toán động học và khoảng cách phanh an toàn.
  - *Gán thừa / Box lỏng (False Positive)*: Vẽ box lên các vật thể tĩnh không có khả năng di chuyển (tượng người, tượng xe, biển quảng cáo) có thể dẫn đến hiện tượng phanh khẩn cấp vô cớ (phantom braking) hoặc kích hoạt cảnh báo giả liên tục.

---

## 2. Lớp và thuộc tính

### Bảng định nghĩa Lớp

Mỗi đối tượng hợp lệ trong ảnh thuộc đúng một lớp duy nhất.

| Lớp | Định nghĩa chuẩn | Ví dụ thực tế | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô 4 bánh chở người từ 9 chỗ ngồi trở xuống (sedan, SUV, hatchback, coupe, mui trần). | Ô tô con đang chạy trên đường, ô tô đỗ bên lề (`img_015.jpg`, `img_024.jpg`). | Xe mô hình đồ chơi (`img_038.jpg`), tượng xe đài kỷ niệm (`img_027.jpg`), xe tuk-tuk 3 bánh (`img_004.jpg`). |
| `truck` | Phương tiện cơ giới chuyên chở hàng hóa, có cabin và thùng hàng (hoặc sàn chở) tách biệt rõ rệt. | Xe tải chở hàng, xe bồn, xe chuyên dụng chở ô tô hai tầng (`img_002.jpg`), xe tải nhỏ thùng hở (`img_036.jpg`). | Xe bán tải cá nhân dạng pickup (`img_001.jpg`), xe van thùng kín (`van`), xe đầu kéo không rơ-moóc. |
| `van` | Phương tiện dạng xe thùng kín chở hàng hoặc chở người (10–16 chỗ) với phần thân vỏ và khoang sau liền khối. | Xe minivan, xe cứu thương dạng van, xe chuyển phát nhanh bưu chính. | Xe SUV 7 chỗ (`car`), xe tải thùng kín rời (`truck`), xe buýt công cộng. |
| `motorcycle` | Phương tiện cơ giới 2 bánh có gắn động cơ (xe máy, xe tay ga, xe mô tô phân khối lớn, xe gắn máy). | Xe máy di chuyển trên đường phố (`img_005.jpg`, `img_026.jpg`), xe máy dựng bên vỉa hè (`img_008.jpg`). | Xe đạp thô sơ, xe xích lô đạp (`img_006.jpg`, `img_009.jpg`), xe ba bánh tuk-tuk (`img_004.jpg`). |
| `person` | Con người thật còn sống ở mọi tư thế (đang đi bộ, đứng, ngồi, nằm, hoặc đang điều khiển phương tiện). | Người đi bộ qua đường (`img_014.jpg`), người bán hàng rong (`img_003.jpg`, `img_010.jpg`), người lái xe máy (`img_026.jpg`). | Tượng điêu khắc người (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh cửa hàng (`img_033.jpg`), ảnh chân dung trên pano/billboard (`img_018.jpg`). |
| `ignore_region` | Vùng không yêu cầu gán chi tiết từng vật; thuật toán bỏ qua vùng này, không tính là lỗi nếu bỏ sót vật bên trong. | Đám đông quá xa, nhóm xe nhỏ tít mù tắp (cao < 20 px), hình ảnh phản chiếu qua gương (`img_020.jpg`), tủ kính đồ chơi (`img_038.jpg`). | Vật thể riêng lẻ, rõ nét, có chiều cao $\ge 20$ px nằm ở cự ly gần hoặc trung bình. |

#### Quy định xử lý vật nằm ngoài Taxonomy hiện tại:
1. **Xe ba bánh (tuk-tuk, xe lam, rickshaw - `img_004.jpg`, `img_035.jpg`)**: Do taxonomy v1.0 chưa có lớp `tuk_tuk`, tạm thời xếp vào `car` nếu có cabin che chắn hoặc `motorcycle` nếu điều khiển bằng ghi-đông; luôn tick cờ `can_review = true` để đưa ra biểu quyết thêm lớp ở buổi họp Pha 2.
2. **Xích lô / Xe đạp (`img_006.jpg`, `img_009.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg`)**: Chỉ gán nhãn `person` cho người đạp và hành khách ngồi trên xe. Bản thân chiếc xe đạp/xích lô KHÔNG gán box phương tiện vì nằm ngoài taxonomy 5 lớp.
3. **Xe buýt (`img_032.jpg`)**: Nếu chưa có lớp `bus`, tạm thời gán `truck` và tick `can_review = true` để đề xuất thêm lớp trong taxonomy v1.1.

---

### Bảng quy định Thuộc tính (Attributes)

Các thuộc tính phải có ngưỡng định lượng đo lường được, không dùng cảm tính:

| Thuộc tính | Kiểu dữ liệu | Giá trị chuẩn | Định nghĩa & Ngưỡng định lượng cụ thể |
|---|---|---|---|
| `occluded_level` | Select | `0` | **Không bị che** (0% diện tích bị che khuất; toàn bộ đường viền vật thể nhìn rõ). |
| | | `1` | **Che nhẹ** (1% – 30% diện tích bị che khuất; phần lớn hình thể vẫn quan sát trọn vẹn). |
| | | `2` | **Che vừa / nhiều** (31% – 70% diện tích bị che khuất bởi vật khác hoặc phương tiện khác). |
| | | `3` | **Che nặng** (> 70% diện tích bị che khuất, chỉ còn lộ một phần nhỏ nhưng vẫn nhận dạng chắc chắn qua ngữ cảnh). |
| `truncated` | Select | `no` | Vật thể nằm trọn 100% trong giới hạn của khung ảnh. |
| | | `yes` | Vật thể bị cắt ngang bởi ít nhất một trong 4 cạnh viền của ảnh (trên, dưới, trái, phải). |
| `can_review` | Checkbox | `false` | Mặc định đối với các ca gán rõ ràng, tuân thủ đúng quy tắc. |
| | | `true` | Tick chọn khi người gán gặp ca lưỡng lự, ranh giới chưa rõ hoặc đối tượng biên cần đưa ra họp nhóm cân nhắc. |

---

## 3. Quy tắc vẽ box

- **R-01 (Độ ôm sát - Tightness)**: Bounding box phải ôm sát các pixel nhìn thấy được (visible bounding box). Không vẽ ước lượng phồng ra ngoài khoảng không gian bị khuất hoàn toàn.
- **R-02 (Người điều khiển xe máy - Motorcyclist)**:
  - Tách thành **2 box độc lập**: 1 box `person` và 1 box `motorcycle`.
  - Box `person`: Ôm trọn người lái (bao gồm mũ bảo hiểm, quần áo, ba lô, cánh tay, chân). Gán thuộc tính `occluded_level = 1` hoặc `2` do nửa thân dưới bị thân xe che.
  - Box `motorcycle`: Ôm trọn toàn bộ thân xe (hai bánh xe, yếm, ghi-đông, đuôi xe, kính chiếu hậu). Gán `occluded_level = 1` hoặc `2` do phần yên và tay lái bị người lái che.
  - Nếu xe chở 2 người (`img_026.jpg`): Gán 1 box `motorcycle` và 2 box `person` độc lập (người lái và người ngồi sau).
- **R-03 (Xe chở xe - Car Carrier - `img_002.jpg`)**:
  - Vẽ 1 box lớn gán nhãn `truck` ôm toàn bộ xe tải chuyên dụng (gồm cabin và giàn sàn xe phía sau).
  - Với từng chiếc ô tô con được chở trên giàn: Gán từng box `car` riêng biệt cho mỗi ô tô con nhìn thấy rõ và đạt kích thước tối thiểu, kèm `occluded_level` tương ứng.
- **R-04 (Vật bị cắt mép ảnh - Truncation)**: Vẽ cạnh của box chạm sát mép ảnh tại vị trí vật bị cắt, không kéo box ra ngoài biên tọa độ ảnh. Bật thuộc tính `truncated = yes`.
- **R-05 (Vật không thật - Tượng, ma-nơ-canh, ảnh in)**:
  - Tuyệt đối **KHÔNG gán** nhãn `person` cho: tượng điêu khắc (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh cửa hàng (`img_033.jpg`), hình ảnh người in trên biển quảng cáo billboard (`img_018.jpg`), búp bê đồ chơi (`img_038.jpg`).
  - Tuyệt đối **KHÔNG gán** nhãn phương tiện (`car`, `truck`) cho: bia mộ/tượng hình xe (`img_027.jpg`), xe mô hình đồ chơi trong tủ kính (`img_038.jpg`).
- **R-06 (Phản chiếu qua gương - `img_020.jpg`)**: Hình ảnh phản chiếu người hoặc xe trong gương chiếu hậu không phải là thực thể vật lý trên mặt đường -> Không gán box phương tiện/người; nếu vùng gương lớn thì vẽ 1 box `ignore_region`.
- **R-07 (Ngưỡng kích thước tối thiểu - Min size threshold)**: Vật thể có chiều cao bounding box nhỏ hơn 20 pixel (theo rule `min_box_height_px: 20` trong `taxonomy.json`) thì **KHÔNG gán box đơn lẻ**; nếu nằm trong cụm đông đúc thì gom chung vào `ignore_region`, nếu đứng riêng rẽ ở đường chân trời xa xôi thì bỏ qua.
- **R-08 (Đám đông và phương tiện quá dày đặc)**: Khi các đối tượng ở khoảng cách quá xa hoặc đan xen dày đặc không thể phân định ranh giới từng cá thể, vẽ một box lớn bao bọc toàn bộ khu vực đó và gán nhãn `ignore_region`.

---

## 4. Bảng Ca Khó (Edge Cases)

Bảng phân loại các ca biên thực tế phát hiện trong 40 ảnh của bộ dữ liệu, phân định giữa ca xử lý theo quy tắc chuẩn và ca cần đưa ra biểu quyết ở Pha 2:

| Mã ảnh | Tình huống phát hiện | Rule ID | Quyết định cá nhân | Lý do & Lập luận |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chuyên dụng 2 tầng chở 8 ô tô con. | `R-03` | Vẽ 1 box `truck` bao toàn bộ xe tải + 8 box `car` cho các xe con trên sàn. | Cung cấp tín hiệu nhận diện cả phương tiện vận tải lẫn hàng hóa là ô tô đang lưu thông. |
| `img_004.jpg` | Hai xe tuk-tuk 3 bánh đỗ trên vỉa hè. | `R-09` | Tạm gán `car`, bật cờ `can_review = true`. | Taxonomy hiện tại chưa có lớp `tuk_tuk`, cần đưa ra họp nhóm Pha 2 để thống nhất thêm lớp hay đưa vào `ignore_region`. |
| `img_006.jpg` | Xích lô đạp chở khách du lịch tại Huế. | `R-02` | Gán 2 box `person` (người đạp xe và hành khách); KHÔNG gán box cho xích lô. | Người tham gia giao thông cần được bảo vệ an toàn; chiếc xích lô thô sơ không thuộc taxonomy 5 lớp. |
| `img_011.jpg` | Tượng nhân vật Predator bên cạnh hàng xe mô tô độ. | `R-05` | KHÔNG gán `person` cho tượng Predator; các xe mô tô thật vẫn gán `motorcycle`. | Tượng trang trí không phải con người thật, tránh làm mô hình kích hoạt phát hiện người giả. |
| `img_017.jpg` | Tượng đài Công tước Wellington cưỡi ngựa đội nón giao thông. | `R-05` | KHÔNG gán bất kỳ box nào (ảnh không có đối tượng hợp lệ). | Tượng điêu khắc bằng đồng không phải người sống; ngựa không nằm trong taxonomy. |
| `img_018.jpg` | Biển quảng cáo chính trị in hình chân dung người rất lớn. | `R-05` | KHÔNG gán `person`. | Hình ảnh 2D trên biển pano tĩnh, không di chuyển trên làn đường giao thông. |
| `img_020.jpg` | Gương chiếu hậu ô tô phản chiếu cầu vồng và bóng xe phía sau. | `R-06` | Vẽ 1 box `ignore_region` bao quanh toàn bộ mặt gương. | Đối tượng ảo trong gương không phải thực thể trên mặt phẳng đường di chuyển. |
| `img_026.jpg` | Hai người (nam và nữ) đèo nhau trên xe máy trong hầm chui. | `R-02` | 3 box độc lập: 1 `motorcycle`, 1 `person` cho người lái (`occluded_level=1`), 1 `person` cho người ngồi sau (`occluded_level=2`). | Đảm bảo tính toán độc lập số lượng người dễ tổn thương trên cùng một phương tiện. |
| `img_027.jpg` | Bia mộ/tượng đài tạc hình xe bán tải pickup bằng đá trong nghĩa trang. | `R-05` | KHÔNG gán box `truck` hay `car` (ảnh không có đối tượng). | Khối đá điêu khắc đứng yên, bề mặt và vật liệu không mang đặc tính xe thật. |
| `img_033.jpg` | Ma-nơ-canh người mẫu đứng trước cửa hàng thời trang. | `R-05` | KHÔNG gán `person`. | Vật thể nhân tạo tĩnh trong nhà/cửa hiệu, gán nhãn sẽ làm sai lệch bộ lọc nhận diện người thật. |
| `img_037.jpg` | Xe bán tải GO Transit gắn mui thùng cao Master 75. | `R-10` | Tạm gán `car`, bật cờ `can_review = true`. | Dạng xe lai giữa bán tải pickup (`car`/`truck`) và thùng kín (`van`); cần đưa ra họp nhóm Pha 2 để thống nhất ranh giới. |
| `img_038.jpg` | Tủ kính cửa hàng bày bộ sưu tập đồ chơi ô tô và búp bê nhỏ. | `R-05`, `R-08` | Vẽ 1 box `ignore_region` bao quanh toàn bộ khu vực tủ kính. | Toàn bộ là đồ chơi mô hình thu nhỏ trưng bày trong nhà, không tham gia giao thông. |

---

## 5. QC và phiên bản

### Ngưỡng kiểm soát chất lượng (QC Thresholds)
Áp dụng theo quy chuẩn được thiết lập trong `submission/taxonomy.json`:
- **Độ ôm khớp IoU tối thiểu**:
  - Đối với vật thể lớn (chiều cao $\ge 50$ px): $\text{IoU} \ge 0.85$.
  - Đối với vật thể nhỏ (chiều cao $< 50$ px): $\text{IoU} \ge 0.70$.
- **Độ đồng thuận Inter-Annotator Agreement (IAA)**:
  - IoU trung bình giữa các annotator $\ge 0.85$.
  - Tỷ lệ trùng khớp lớp (Class Agreement) $\ge 95\%$.
- **Tiêu chuẩn nghiệm thu theo ảnh**: Tỷ lệ ảnh có lỗi $\le 2\%$ (trong lô 40 ảnh chỉ cho phép tối đa 0 ảnh lỗi để ra quyết định **Release**).
- **Cơ chế xử lý ca mơ hồ**: Đánh dấu checkbox `can_review = true` trên bounding box và ghi nhận vào nhật ký kiểm thử để thống nhất ở buổi họp Calibration.

### Lịch sử thay đổi phiên bản (Version History)

| Phiên bản | Thay đổi chính | Lý do & Ca liên quan |
|---|---|---|
| `v0` | Bản khởi đầu sơ sài do giảng viên cung cấp. | Chỉ định nghĩa 5 lớp cơ bản, thuộc tính định tính, chưa có ca khó. |
| `v1.0-draft-058` | Bổ sung định nghĩa chi tiết kèm Ví dụ và Phản ví dụ cho 6 lớp; lượng hóa thuộc tính `occluded_level` theo ngưỡng %; bổ sung 8 quy tắc vẽ box không gian; lập bảng 12 ca khó thực tế từ 40 ảnh. | Hoàn thiện bản đặc tả cá nhân độc lập của thành viên 058 để chuẩn bị cho buổi họp thống nhất Pha 2. |

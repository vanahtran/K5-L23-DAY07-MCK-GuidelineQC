
# Guideline gán nhãn — v0.1 — Individual 222

- Phiên bản: v0.1 (Individual Draft)
- Ngày: 2026-11-16
- Người viết: Member 222
- Người duyệt: Pending group review
- Phạm vi: 40 ảnh đường phố trong `data/images/`
- Công cụ dự kiến: CVAT
- Định dạng đầu ra: COCO 1.0

## 1. Mục tiêu

Guideline này quy định cách gán nhãn người và phương tiện giao thông xuất hiện trong ảnh đường phố nhằm tạo bộ dữ liệu phục vụ huấn luyện và đánh giá mô hình Object Detection.

Mục tiêu là bảo đảm:
- Mỗi đối tượng thuộc một class phù hợp trong taxonomy được cung cấp.
- Bounding box bao quanh chính xác phần đối tượng nhìn thấy.
- Các thuộc tính mô tả mức độ che khuất, tình trạng bị cắt bởi biên ảnh và nhu cầu kiểm tra lại.
- Các thành viên có thể áp dụng quy tắc nhất quán khi gặp cùng một tình huống.

Các lỗi cần hạn chế:
- Gán sai class giữa `car`, `truck` và `van`.
- Bỏ sót người hoặc phương tiện có thể nhận diện.
- Vẽ bounding box quá rộng, quá hẹp hoặc bao gồm vật thể che khuất.
- Nhầm lẫn giữa occlusion và truncation.
- Gán nhãn cho hình ảnh phản chiếu, hình in hoặc vật thể mô phỏng như đối tượng thật.
- Bỏ qua các trường hợp chưa chắc chắn mà không đánh dấu review.

## 2. Lớp và thuộc tính

### 2.1. Classes

Sử dụng 6 classes được cung cấp trong guideline v0. Không tự ý thêm class mới trong giai đoạn gán nhãn cá nhân.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người có cấu trúc thân xe kiểu passenger car, bao gồm sedan, hatchback và SUV. | Sedan, hatchback, SUV | Không gán xe tải chở hàng thuộc `truck`; xe thân kín dạng van thuộc `van`. |
| `truck` | Phương tiện cơ giới có kết cấu xe tải chuyên dùng để vận chuyển hàng hóa, thường có thùng hoặc sàn chở hàng riêng biệt. | Xe tải thùng, xe ben | Không nhầm với SUV hoặc xe van thân kín cỡ nhỏ. |
| `van` | Phương tiện có thân xe dạng hộp kín, kích thước nhỏ hoặc trung bình, thường có khoang hành khách hoặc khoang chở hàng liền với cabin. | Cargo van, minivan dạng van | Không nhầm với xe tải có thùng hàng riêng biệt hoặc ô tô con thông thường. |
| `motorcycle` | Phương tiện cơ giới hai bánh, được điều khiển bằng động cơ, bao gồm xe máy và mô tô. | Xe số, xe tay ga, mô tô | Không gán xe đạp không có động cơ vào class này. |
| `person` | Một cá thể người thật xuất hiện trong ảnh, dù chỉ nhìn thấy một phần cơ thể. | Người đi bộ, người điều khiển xe máy, người ngồi trên xe | Không gán tượng người, hình in, ảnh trên biển quảng cáo hoặc hình người trên màn hình. |
| `ignore_region` | Vùng ảnh chứa các đối tượng mục tiêu không thể phân tách hoặc nhận diện đáng tin cậy để gán từng instance riêng biệt. | Đám đông quá dày, vùng phương tiện chồng lấn nghiêm trọng không thể tách box | Không sử dụng thay cho class đối tượng; không dùng chỉ vì đối tượng nhỏ hoặc khó phân loại. |

#### Quy tắc phân biệt class

**A. `car` và `truck`**
- Ưu tiên phân loại theo cấu trúc và mục đích thiết kế của phương tiện.
- Xe chở người có thân xe kiểu ô tô con, kể cả SUV, thuộc `car`.
- Xe có kết cấu chuyên chở hàng hóa thuộc `truck`.
- Không phân loại chỉ dựa trên kích thước nhìn thấy trong ảnh.

**B. `van` và `truck`**
- `van` có thân xe dạng hộp kín, cabin và khoang thân xe thường liền thành một cấu trúc.
- `truck` có kết cấu xe tải chuyên chở hàng hóa, thường có thùng hoặc sàn chở hàng riêng.
- Nếu ảnh không đủ thông tin để phân biệt hai class, chọn class có căn cứ tốt nhất và bật `can_review`.

**C. `person` và `motorcycle`**
- Người ngồi hoặc điều khiển xe máy vẫn được gán một box `person` riêng nếu nhận diện được.
- Xe máy được gán một box `motorcycle` riêng.
- Không gộp người và xe máy thành một bounding box.

#### Đối tượng không thuộc class nào

- Đối tượng ngoài phạm vi 6 classes, ví dụ xe đạp, xe buýt hoặc biển báo giao thông, không được tự ý gán vào class gần giống.
- Không tự tạo class mới trong quá trình annotation cá nhân.
- Không tạo `ignore_region` cho mọi đối tượng ngoài taxonomy.
- Chỉ sử dụng `ignore_region` cho vùng chứa đối tượng mục tiêu mà không thể phân tách hoặc nhận diện đáng tin cậy.
- Nếu chưa xác định được cách xử lý một tình huống, ghi nhận vào danh sách ca khó và đánh dấu `can_review` khi có box đối tượng phù hợp.

### 2.2. Attributes

Áp dụng 3 attributes từ guideline v0 cho các object classes: `car`, `truck`, `van`, `motorcycle`, `person`.

#### Attribute 1: `occluded_level`

- Kiểu dữ liệu CVAT đề xuất: Select.
- Các giá trị cố định: `0`, `1`, `2`, `3`.
- Ý nghĩa: Mức độ phần đối tượng bị che bởi vật thể khác.
- Tỷ lệ che khuất được ước lượng trên toàn bộ diện tích đối tượng nếu có thể suy ra từ ngữ cảnh ảnh; không tính phần nằm ngoài biên ảnh.

| Giá trị | Tỷ lệ bị che ước lượng | Ý nghĩa |
|---|---|---|
| `0` | 0%–10% | Không bị che hoặc chỉ bị che rất ít; gần như nhìn thấy toàn bộ đối tượng. |
| `1` | Trên 10% đến 30% | Bị che khuất nhẹ; phần lớn đối tượng vẫn nhìn thấy. |
| `2` | Trên 30% đến 60% | Bị che khuất đáng kể; một phần lớn đối tượng không nhìn thấy. |
| `3` | Trên 60% | Bị che khuất nghiêm trọng nhưng phần còn nhìn thấy vẫn đủ căn cứ xác định class. |

Quy tắc:
- Chỉ tính phần bị che bởi vật thể khác.
- Không tính phần bị cắt bởi biên ảnh vào tỷ lệ occlusion.
- Không suy đoán hình dạng hoặc diện tích phần bị che để mở rộng bounding box.
- Nếu đối tượng không đủ căn cứ nhận diện class, không cố gán class chỉ để sử dụng mức `3`; đánh dấu review hoặc xử lý theo quy tắc ignore.
- Các ngưỡng phần trăm trên là quy ước cá nhân đề xuất để tăng tính nhất quán, cần được nhóm xem xét khi thống nhất guideline.

#### Attribute 2: `truncated`

- Kiểu dữ liệu CVAT đề xuất: Checkbox.
- Các giá trị: `no`, `yes`.
- Ý nghĩa: Xác định đối tượng có bị cắt bởi biên của ảnh hay không.

| Giá trị | Ý nghĩa | Quy tắc |
|---|---|---|
| `no` | Đối tượng không bị cắt bởi biên ảnh. | Toàn bộ phạm vi đối tượng nằm trong khung hình. |
| `yes` | Một phần đối tượng nằm ngoài khung hình do bị cắt ở biên ảnh. | Áp dụng khi đối tượng chạm biên ảnh và có dấu hiệu tiếp tục ra ngoài. |

Quy tắc:
- Kiểm tra cả bốn cạnh ảnh: trên, dưới, trái và phải.
- Nếu đối tượng bị cắt bởi bất kỳ cạnh nào, chọn `yes`.
- Nếu đối tượng chỉ bị vật thể khác che khuất nhưng vẫn nằm trong khung ảnh, chọn `no`.
- Khi `truncated=yes`, bounding box chỉ được vẽ trong phạm vi ảnh hiện có.

#### Attribute 3: `can_review`

- Kiểu dữ liệu CVAT đề xuất: Checkbox.
- Các giá trị: `true`, `false`.
- Ý nghĩa: Đánh dấu đối tượng cần người khác kiểm tra lại.

| Giá trị | Ý nghĩa | Quy tắc |
|---|---|---|
| `false` | Không cần review. | Class, vị trí box và thuộc tính có đủ căn cứ. |
| `true` | Cần review. | Có sự không chắc chắn về class, box hoặc cách áp dụng guideline. |

Bật `can_review=true` trong các trường hợp:
- Không phân biệt chắc chắn giữa `car`, `truck` và `van`.
- Đối tượng bị che khuất nhiều.
- Đối tượng quá nhỏ hoặc mờ khiến ranh giới box khó xác định.
- Không xác định được đối tượng có thuộc phạm vi taxonomy hay không.
- Xuất hiện tình huống chưa được quy định rõ trong guideline.

Không bật `can_review` chỉ vì đối tượng bị che khuất hoặc bị cắt mép ảnh nếu cách xử lý đã rõ ràng.

Đối với `ignore_region`, ba thuộc tính trên không áp dụng theo cấu hình cá nhân đề xuất; cần xác nhận cách biểu diễn với nhóm trước khi cấu hình CVAT chính thức.

## 3. Quy tắc vẽ bounding box

### 3.1. Quy tắc chung

- Mỗi đối tượng thuộc taxonomy được gán một bounding box riêng.
- Box có dạng hình chữ nhật, bao sát phần đối tượng thực tế nhìn thấy.
- Không bao gồm vùng nền không thuộc đối tượng.
- Không mở rộng box để bao gồm phần bị che khuất hoặc phần nằm ngoài ảnh.
- Không gộp nhiều đối tượng cùng class vào một box.
- Không gộp người và phương tiện thành một box.

### 3.2. Đối tượng bị che khuất

- Vẽ box theo phạm vi phần đối tượng nhìn thấy.
- Không ước lượng và vẽ thêm phần bị che.
- Nếu đối tượng có nhiều phần nhìn thấy nhưng bị vật khác che ở giữa, box bao quanh phạm vi nhìn thấy của đối tượng.
- Chọn `occluded_level` theo mức độ che khuất.
- Nếu không đủ căn cứ xác định class, bật `can_review`.

### 3.3. Đối tượng bị cắt ở mép ảnh

- Vẽ box đến sát biên ảnh tại vị trí đối tượng bị cắt.
- Không vẽ phần box nằm ngoài ảnh.
- Chọn `truncated=yes`.
- Nếu đối tượng không bị cắt bởi biên ảnh, chọn `truncated=no`.

### 3.4. Phản chiếu, hình in, tượng và đồ chơi

- Không gán nhãn cho hình ảnh phản chiếu trong gương hoặc kính.
- Không gán nhãn người hoặc phương tiện xuất hiện trong tranh, ảnh in, biển quảng cáo hoặc màn hình.
- Không gán nhãn tượng người hoặc mô hình người giả.
- Không gán nhãn đồ chơi có hình dạng người hoặc phương tiện.
- Chỉ gán nhãn đối tượng thực tế xuất hiện trong cảnh đường phố.

### 3.5. Xe chủ (ego vehicle)

- Không gán nhãn phần phương tiện của camera đang ghi hình nếu nó chỉ là bộ phận của chính nền tảng camera và không phải đối tượng mục tiêu độc lập trong cảnh.
- Các phương tiện khác xuất hiện trong cảnh vẫn phải được gán nhãn bình thường.
- Nếu không xác định được một phương tiện là ego vehicle hay phương tiện trong cảnh, ghi nhận để review.

### 3.6. Đối tượng quá nhỏ hoặc mờ

- Vẫn gán nhãn nếu có đủ dấu hiệu trực quan để nhận diện class và xác định box hợp lý.
- Không tự đặt một ngưỡng pixel tối thiểu khi chưa có quy định từ lab.
- Không đoán class đối với đối tượng chỉ xuất hiện như một vùng nhỏ không thể nhận diện.
- Nếu có căn cứ nhận diện nhưng box hoặc class chưa chắc chắn, bật `can_review=true`.

### 3.7. Đám đông và nhiều đối tượng chồng lấn

- Gán một box riêng cho từng người nếu có thể phân biệt từng cá thể.
- Gán một box riêng cho từng phương tiện nếu có thể phân biệt từng phương tiện.
- Không tạo một box lớn bao toàn bộ đám đông khi từng người vẫn có thể được nhận diện và tách riêng.
- Chỉ dùng `ignore_region` cho vùng mà các đối tượng mục tiêu không thể phân tách đáng tin cậy.

### 3.8. Một tình huống có nhiều đối tượng

- Người đang lái xe máy: một box `person`, một box `motorcycle`.
- Người đứng cạnh ô tô: một box `person`, một box `car`.
- Nhiều người trên cùng một xe: gán box riêng cho từng người có thể nhận diện.
- Các phương tiện che khuất lẫn nhau: mỗi phương tiện có thể nhận diện được phải có box riêng.
- Các box có thể chồng lấn khi đối tượng thực tế chồng lấn; không bóp méo box để tránh overlap.

## 4. Ca khó

Các tình huống dưới đây là danh sách cần kiểm tra khi rà soát bộ ảnh, chưa được khẳng định là đã xuất hiện trong ảnh cụ thể.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| Chờ rà soát | Khó phân biệt `car`, `truck`, `van` | CLS-01 | Chọn class có căn cứ tốt nhất; bật `can_review` nếu chưa chắc chắn. | Tránh tự ý thay đổi taxonomy. |
| Chờ rà soát | Phương tiện bị che khuất nhiều | OCC-01 | Vẽ box phần nhìn thấy và chọn `occluded_level` phù hợp. | Không suy đoán phần bị che. |
| Chờ rà soát | Người hoặc xe bị cắt bởi biên ảnh | TRU-01 | Box đến sát biên ảnh và đặt `truncated=yes`. | Phân biệt truncation với occlusion. |
| Chờ rà soát | Người ngồi trên hoặc điều khiển xe máy | BOX-01 | Gán hai box riêng nếu nhận diện được cả người và xe. | Hai đối tượng thuộc hai classes khác nhau. |
| Chờ rà soát | Đám đông không thể tách từng cá thể | IGN-01 | Xem xét `ignore_region`; bật review nếu cách xử lý chưa chắc chắn. | Tránh tạo box không đáng tin cậy. |
| Chờ rà soát | Đối tượng quá nhỏ hoặc mờ | REV-01 | Gán nếu đủ căn cứ; nếu không chắc, đánh dấu review. | Tránh gán nhãn theo phỏng đoán. |
| Chờ rà soát | Hình người hoặc phương tiện trong ảnh in, màn hình, gương | OBJ-01 | Không gán nhãn. | Không phải đối tượng thực tế trong cảnh. |
| Chờ rà soát | Phương tiện nghi là ego vehicle | OBJ-02 | Không gán bộ phận của nền tảng camera; yêu cầu review nếu không rõ. | Tránh nhầm đối tượng ghi hình với đối tượng mục tiêu. |

Sau khi rà soát đủ 40 ảnh:
- Thay các dòng "Chờ rà soát" bằng tên ảnh thực tế `img_001.jpg` đến `img_040.jpg` khi có tình huống tương ứng.
- Ghi lại quyết định thực tế, không tự tạo tình huống không xuất hiện trong dữ liệu.
- Nếu chưa chốt được quyết định, bật `can_review` trên box phù hợp hoặc ghi nhận vấn đề cấp ảnh.
- Dùng Rule ID để liên kết với nhật ký QC.
- Các tình huống mới phát hiện phải được ghi lại để nhóm xem xét ở giai đoạn thống nhất guideline.

## 5. QC và phiên bản

### 5.1. Kiểm tra chất lượng cá nhân

Trước khi hoàn thành annotation, thực hiện các kiểm tra:

| Hạng mục QC | Tiêu chí kiểm tra |
|---|---|
| Class correctness | Mỗi box sử dụng đúng một class trong taxonomy. |
| Missing objects | Không bỏ sót người hoặc phương tiện có thể nhận diện trong phạm vi nhiệm vụ. |
| Bounding box | Box ôm sát phần đối tượng nhìn thấy, không bao gồm nền không cần thiết. |
| Occlusion | `occluded_level` phản ánh mức độ che khuất, không nhầm với truncation. |
| Truncation | `truncated=yes` khi đối tượng bị cắt bởi biên ảnh. |
| Review | Các trường hợp không chắc chắn được đánh dấu `can_review=true`. |
| Duplicate boxes | Không tạo nhiều box trùng lặp cho cùng một đối tượng. |
| Ignore region | Chỉ sử dụng cho vùng thực sự không thể tách instance mục tiêu. |
| Export | Kiểm tra output COCO và đối chiếu số lượng annotation với kết quả trên CVAT. |

### 5.2. Mục tiêu QC cá nhân

- Rà soát toàn bộ 40 ảnh trước khi hoàn thành phần việc cá nhân.
- Không để lại box sai class hoặc box không hợp lệ mà đã nhận diện được trong quá trình tự kiểm tra.
- Tất cả trường hợp chưa chắc chắn phải được đánh dấu để review.
- Không tự ý thay đổi taxonomy hoặc quy tắc chung sau khi nhóm đã thống nhất phiên bản guideline chính thức.

Đây là mục tiêu tự kiểm tra cá nhân, không thay thế ngưỡng QC hoặc tiêu chí IAA chính thức do lab và nhóm quy định.

### 5.3. Xử lý ca mơ hồ

- Người gán nhãn ghi nhận ca khó và Rule ID tương ứng.
- Các trường hợp có `can_review=true` được đưa vào danh sách cần kiểm tra.
- Khi có bất đồng, giữ lại bằng chứng và đề xuất quyết định thay vì tự ý thay đổi quy tắc chung.
- Các thay đổi ảnh hưởng đến class, attributes hoặc cách vẽ box phải được ghi vào lịch sử phiên bản.
- Chỉ áp dụng quy tắc nhóm sau khi có quyết định thống nhất.

### 5.4. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Guideline khởi đầu với 6 classes và 3 attributes. | Guideline starter do lab cung cấp. |
| v0.1 | Bổ sung định nghĩa class, ngưỡng occlusion đề xuất, quy tắc truncation, bounding box và xử lý ca khó. | Làm rõ các điểm có thể dẫn đến cách hiểu khác nhau khi annotation cá nhân. |

**Trạng thái:** Individual Draft — chờ rà soát đủ 40 ảnh và chờ nhóm xem xét trước khi trở thành guideline thống nhất.

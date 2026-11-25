# Báo cáo Day 13 — Nguyễn Tiến Đạt, 2A202603289

## 1. Kết luận trước

- **Thông tin chung:** MSSV `2A202603289`, task `D13-2A202603289`, 10 frame được giao gồm `s01f11`, `s01f56`, `s02f18`, `s02f38`, `s03f07`, `s03f50`, `s04f09`, `s04f42`, `s05f14`, `s05f48`. Môi trường làm việc: Windows 11, trình duyệt Chrome, CVAT `v2.83.0`; làm bài độc lập.
- **Tóm tắt kết quả:** Tổng cộng đã vẽ **59 cuboid** trên 10 frame (54 `Car`, 2 `Bus`, 2 `Cyclist`, 1 `Truck`); hai frame có ít cuboid nhất là `s02f38` (3 cuboid) và `s05f48` (3 cuboid) do cảnh giao thông thưa vật rõ trong phạm vi gần. Phiếu tự kiểm `selfcheck` gắn cờ **2/59 cuboid** (`s01f11 #2` và `s01f11 #3` nghi vấn đáy chìm). Phần tôi tự tin nhất là xác định **hướng yaw** và **phân loại class** của các xe ở gần nhờ đối chiếu đủ 7 camera; phần còn yếu nhất là căn **độ cao đáy hộp** ở những đoạn đường dốc và ước lượng kích thước với **vật ở xa trên 25 m** bị thưa điểm.
- **Cách dùng CVAT 3D:**
  1. **Đổi góc nhìn:** Kéo chuột trái trong khung 3D để xoay (xoay thẳng từ trên xuống để quan sát toàn cảnh), kéo chuột phải để dời khung nhìn, lăn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc nhìn mặc định.
  2. **Vẽ và chỉnh cuboid:** Chọn công cụ vẽ cuboid → chọn label → bấm **Shape** (không dùng Track) → nháy đúp vào cụm điểm của vật để đặt hộp; chọn hộp rồi chỉnh tâm, kích thước, hướng ở khung **Top** và chỉnh đáy chạm mặt đường, chiều cao ở khung **Side/Front** (không xoay ở Side/Front).
  3. **Bật hướng và kiểm tra với camera:** Bật **Appearance → Cuboid orientation** để hiện vạch đỏ ngắn (`+X` của cuboid) chỉ về đầu xe; dùng lệnh `python scripts/lab.py preview` để chiếu các cuboid lên 7 ảnh camera và ảnh BEV trong `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f11` và `s01f56`. Ảnh chụp trong `report/individual/images/`: `t1-s01f11-3d.png`, `t1-s01f11-top.png`, `t1-s01f56-3d.png`, `t1-s01f56-top.png` (ảnh nhìn từ trên xuống có đánh dấu phía trước `−y`, trục `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; `+x`, `+y` chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f11` | Phía trước (`−y`) ở cạnh trên của ảnh nhìn từ trên xuống; `+x` chỉ sang trái, `+y` chỉ xuống dưới (phía sau xe). | Mặt đường tương đối phẳng ở làn xe thu dữ liệu, hơi dốc nhẹ về phía làn ngược chiều bên trái; thấp hơn tâm LiDAR khoảng 1,7 m (`z ≈ −1,7 m`). | ~38 m (xe buýt phía trước, `dist_m = 37,80 m`). | `Car`, `Bus`. | Bên trái và sau-trái (làn đối diện, `x > 0`, `y > 0`) có nhiều xe con đi ngược chiều như `s01f11 #2` (12,65 m), `#5` (24,98 m), `#6` (24,73 m) thấy rõ trên cam05/cam06. | Các cụm điểm rải rác ngoài 40 m phía trước và cụm điểm khuất sau hàng xe bên trái quá thưa, không đủ hình khối để kết luận loại vật. |
| `s01f56` | Phía trước (`−y`) ở cạnh trên của ảnh nhìn từ trên xuống; `+x` chỉ sang trái, `+y` chỉ xuống dưới (phía sau xe). | Mặt đường khá phẳng; thấp hơn tâm LiDAR khoảng 1,75 m (`z ≈ −1,75 m`). | ~20–25 m (các xe con phía trước rõ nhất tới ~16,3 m). | `Car`. | Phía sau (`+y`) có xe con bám đuôi (`s01f56 #2` ở `y = +6,87 m`, thấy trên cam09) và xe bên sau-trái (`s01f56 #1` ở `x = +2,91 m`, `y = +3,76 m`, thấy trên cam05). | Một số cụm điểm nhỏ ở rìa vỉa hè hai bên đường và phía xa trên 35 m bị thưa điểm, khó phân biệt là cột biển báo, cây hay người đi bộ. |

**3 điều chỉ camera cho biết và 3 điều chỉ LiDAR cho biết (từ `s01f11` và `s01f56`):**
- **Chỉ camera cho biết:** (1) Màu sắc, kiểu dáng và chi tiết đầu/đuôi xe (đèn pha, đèn hậu) giúp phân biệt chính xác hướng đầu xe và phân biệt `Car` với `Truck`/`Bus`; (2) Trạng thái đèn tín hiệu, vạch kẻ đường, biển báo giao thông; (3) Nhận diện rõ vật nhỏ hoặc vật có bề mặt ít phản xạ mà LiDAR chỉ có vài điểm.
- **Chỉ LiDAR cho biết:** (1) Tọa độ 3D `(x, y, z)` và khoảng cách chính xác tính bằng mét từ gốc cảm biến đến từng vật; (2) Kích thước hình học thật (dài, rộng, cao theo mét) và độ cao mặt đường (`z ≈ −1,7 m`) không bị méo phối cảnh; (3) Vùng phủ 360° liên tục quanh xe, phát hiện được các vật nằm trong góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước (`−y`) ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f18` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Nhiều xe đi ngược chiều ở phía sau-trái (`x > 0`, `y > 0`). |
| `s02f38` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Cảnh thưa vật ở gần, chỉ có ít xe rõ trong phạm vi 30 m. |
| `s03f07` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Mật độ xe dày ở cả phía trước, phía sau và làn bên trái trong phạm vi 15 m. |
| `s03f50` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Có xe hai bánh (`Cyclist`) di chuyển sát bên phải (`x = −7,49 m`). |
| `s04f09` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Mặt đường phía làn đối diện bên trái có độ cao thấp hơn gốc LiDAR (`z_bottom` khoảng −2,0 đến −2,4 m); có xe buýt lớn bên trái. |
| `s04f42` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Có `Cyclist` phía trước bên phải ở tầm trung (~16,2 m) với cụm điểm nhỏ hơn hẳn ô tô. |
| `s05f14` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Có cả hai chiều xe chạy song song hai bên xe thu dữ liệu ở cự ly gần (6–15 m). |
| `s05f48` | Cạnh trên (`+x` sang trái, `+y` xuống dưới) | Có xe tải (`Truck`) ở rất gần bên phải (`dist_m = 4,80 m`) với mật độ điểm rất dày (9620 điểm) gây bóng che lớn phía sau nó. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh chụp trong `report/individual/images/`: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f11`, idx 1 | `Car` | Đúng (`yaw_deg = 269,2°`, cùng chiều `−y`) | Chạm đường (`z_bottom = −1,727 m`, `ground_gap_m = 0,04 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu camera | Trong khoảng Sedan/hatchback (`4,486 × 1,864 × 1,480 m`) |
| (b) xe buýt hoặc xe tải | `s01f11`, idx 8 | `Bus` | Đúng (`yaw_deg = 281,7°`, hướng đầu xe về `−y`) | Chạm đường (`z_bottom = −1,781 m`, `ground_gap_m = 0,04 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu camera | Trong khoảng xe buýt thường (`10,926 × 2,600 × 3,200 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s03f50`, idx 4 | `Cyclist` | Đúng (`yaw_deg = 272,0°`, cùng chiều `−y`) | Chạm đường (`z_bottom = −1,701 m`, `ground_gap_m = 0,06 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu camera | Trong khoảng xe máy có người (`1,800 × 0,800 × 1,600 m`) |

- **Vòng hiệu chuẩn:** Khi so sánh trên task chung `D13-HC485087`, cuboid ban đầu của tôi dễ bị lệch nhẹ ở chiều rộng (bao thiếu gương hoặc nới hơi rộng) và độ cao đáy hộp khi chỉ nhìn khung Top. Cả lớp thống nhất bám sát guideline: luôn đối chiếu ảnh camera để chốt đầu xe trước khi xoay vạch đỏ `+X`, chỉnh đáy hộp bằng khung Side/Front cho chạm mặt đường, và ưu tiên kích thước thực từ point cloud kết hợp bảng 4.3 thay vì ước lượng theo cảm tính.
- **Các vật bỏ qua vì không đủ cơ sở:** Một số cụm điểm rất thưa ở khoảng cách trên 35–45 m hoặc bị che khuất phần lớn phía sau xe lớn ở `s01f11` và `s02f38` (không thấy đủ 15–20% thể tích, không đủ cơ sở xác định kích thước/class → loại nghi vấn `LOW_VISIBILITY` / `UNCERTAIN_SCOPE`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

Các nguyên nhân dưới đây đều được ghi nhận dưới dạng **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f00` / `t4-1-bong-che.png` / mượn từ task chung `D13-HC485087` | Giả thuyết: tia laser LiDAR truyền thẳng bị thân xe buýt lớn sát bên trái (~4 m) chắn hoàn toàn nên không chiếu tới vùng phía sau nó. | Phía sau thân xe buýt tạo thành một vùng hình quạt gần như không có điểm nào, trong khi hai bên mép góc che vẫn dày điểm. | Không suy đoán vùng trống là đường không có vật; nếu có vật bị che một phần phía sau thì bật thuộc tính `occluded` và vẽ cuboid bao trọn phần suy luận được. |
| 2. Thưa ở xa | `s01f11` / `t4-2-thua-o-xa.png` / task của tôi (`D13-2A202603289`) | Giả thuyết: LiDAR 40 tia có góc phân giải ngang và dọc cố định nên càng xa cảm biến, khoảng cách giữa các tia càng lớn, làm mật độ điểm trên cùng một diện tích bề mặt giảm mạnh. | Xe ở khoảng cách trên 30–40 m chỉ còn vài chục điểm rời rạc, khó thấy rõ biên bốn góc và bánh xe. | Chỉ vẽ khi thấy khoảng 15–20% thể tích hoặc có ảnh camera rõ đủ cơ sở suy luận kết hợp bảng 4.3; nếu không đủ cơ sở thì bỏ qua và ghi nhận `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f56` / `t4-3-phan-xa-thap.jpg` / task của tôi (`D13-2A202603289`) | Giả thuyết: bề mặt tối màu, kính chắn gió hoặc bề mặt nghiêng hấp thụ hoặc phản xạ gương tia laser ra hướng khác thay vì phản hồi về cảm biến. | Một số xe màu tối hoặc vùng kính xe có số điểm ít hơn hẳn và cường độ phản xạ (intensity) thấp hơn so với các xe sáng màu ở cùng khoảng cách. | Không dựa vào intensity để cắt bớt hộp; đối chiếu ảnh camera để xác định hình dáng thật của vật và chỉ ghi nhận nguyên nhân là giả thuyết. |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / mượn từ task chung `D13-HC485087` | Giả thuyết: nhiễu phản xạ đa đường (multipath) từ mặt đường/kính hoặc sai số đo cự ly của tia laser tạo ra các điểm ảo không khớp hình học thật. | Xuất hiện cụm điểm phía sau bên trái nằm sâu dưới mặt đường (ở `z ≈ −13,5 m`, thấp hơn mặt đường quanh đó khoảng 11 m), không gắn với vật thể nào trên ảnh camera. | Không gán nhãn thành vật thể vì đây là điểm nhiễu, không phải đối tượng giao thông. |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu dưới đây lấy từ `submission/D13-2A202603289-bangA.csv` và cột `points_in`, `ground_gap_m` từ `submission/D13-2A202603289-tukiem.csv`.

| Cuboid (frame, idx, class) | `z` (m) | `h` (m) | `z` đáy = `z − h/2` (m) | `dist_m` (m) | `r = √(x²+y²+z²)` (m) | `yaw_deg` (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f11`, idx 1, `Car` | −0,987 m | 1,480 m | −1,727 m | 6,32 m | 6,40 m | 269,2° | Chạm đường (`z` đáy rất sát −1,7 m; `ground_gap_m = 0,04 m`). |
| `s01f11`, idx 4, `Car` | −0,447 m | 1,418 m | −1,155 m | 19,70 m | 19,71 m | 92,9° | `z` đáy cao hơn mốc −1,7 m là 0,545 m (có vẻ lơ lửng so với mặt phẳng chuẩn, nhưng `ground_gap_m = −0,16 m` do mặt đường làn này dốc lên). |
| `s04f09`, idx 4, `Car` | −1,253 m | 1,848 m | −2,177 m | 12,58 m | 12,64 m | 84,2° | `z` đáy thấp hơn mốc −1,7 m là 0,477 m (nhìn theo mốc −1,7 m có vẻ chìm, nhưng `ground_gap_m = 0,25 m` do mặt đường phía trái thấp hơn gốc LiDAR). |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s03f07`, idx 1 (`Car`) có `dist_m = 3,52 m`, `r = 3,66 m`.
  - **Xa nhất:** `s01f11`, idx 8 (`Bus`) có `dist_m = 37,80 m`, `r = 37,80 m`.
- **(b) So kích thước với bảng 4.3:**
  - **Xe hai bánh (`Cyclist`):** `s03f50`, idx 4 có `l = 1,800 m`, `w = 0,800 m`, `h = 1,600 m`. So với bảng 4.3 (xe máy có người: dài 1,8–2,3 m, rộng 0,70–0,95 m, cao 1,40–1,80 m), cả ba chiều đều nằm trong khoảng tham chiếu.
  - **Xe con (`Car`):** `s01f11`, idx 3 có `l = 4,505 m`, `w = 1,992 m`, `h = 1,552 m`. So với bảng 4.3 nhóm Sedan/hatchback (dài 3,8–5,0 m, rộng 1,70–1,90 m, cao 1,40–1,60 m), chiều rộng `1,992 m` hơi vượt ngưỡng 1,90 m (vẫn nằm trong khoảng SUV/minivan 1,80–2,10 m), nguyên nhân là hộp bao gồm cả phần gương chiếu hậu hai bên và thân xe thực tế khá rộng.
- **(c) Hướng `yaw_deg`:**
  - Xe cùng chiều phía trước: `s01f11`, idx 1 (`Car`, `y = −5,490 m`) có `yaw_deg = 269,2°`, rất gần **270°**.
  - Xe ngược chiều: `s01f11`, idx 3 (`Car`, `y = −9,981 m`) có `yaw_deg = 92,9°` (và `s02f18`, idx 1 có `yaw_deg = 91,1°`), rất gần **90°**.

**Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):**

| Cuboid (frame, idx, class) | `dist_m` (m) | Số điểm (`points_in`) |
|---|---|---|
| `s02f18`, idx 1, `Car` | 4,37 m | 4254 điểm |
| `s01f56`, idx 3, `Car` | 10,18 m | 845 điểm |
| `s01f11`, idx 7, `Car` | 24,13 m | 236 điểm |
| `s05f48`, idx 2, `Car` | 30,30 m | 169 điểm |

- **Quy luật mật độ:** Khi khoảng cách xa gấp khoảng **2,4 lần** (từ 10,18 m lên 24,13 m), số điểm trên xe con giảm từ 845 điểm xuống còn 236 điểm (còn khoảng **28%**, tức gần 1/3,6), và ở 30,30 m chỉ còn 169 điểm. Xu hướng suy giảm nhanh theo khoảng cách này phù hợp với đường cong trung vị ở slide bài giảng (xe con: khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m).
- **Tự kiểm sau `preview` và `selfcheck`:**
  - **Các cuboid đã sửa:** (1) `s01f11` (hộp cũ `idx 7`, `dist_m = 0,49 m`): phát hiện vẽ nhầm vào xe thu dữ liệu ngay gốc LiDAR nên đã xoá bỏ, bấm Save và chạy lại `sync`; (2) `s01f11`, idx 8 (`Bus`): căn lại chiều dài (`10,926 m`), góc yaw (`281,7°`) và đáy hộp (`ground_gap_m = 0,04 m`) sau khi đối chiếu `preview` trên cam01/cam03; (3) `s05f48`, idx 3 (`Truck`): điều chỉnh lại class từ `Car` sang `Truck` và mở rộng kích thước (`5,395 × 2,473 × 2,488 m`) cho khớp cụm 9620 điểm trên Side/Front.
  - **Cảnh báo giữ nguyên:** `s01f11 #2` (`ground_gap_m = −0,67 m`) và `s01f11 #3` (`ground_gap_m = −0,43 m`) bị `selfcheck` cảnh báo chìm; tôi giữ nguyên vì trên point cloud mặt đường tại khu vực làn bên trái này dốc và có nhiều điểm lề/vỉa hè làm ước lượng mặt đường cục bộ bị đẩy lên cao hơn thực tế (`z_bottom` thực của hai hộp là −1,578 m và −1,561 m, vẫn cao hơn mốc −1,7 m).
- **Đổi kiểm:** Do làm bài cá nhân tại nhà sau giờ học nên tôi tự đối chiếu chéo toàn bộ các con số ở mục 5 (`s01f11 #1` với `dist_m = 6,32 m`, `z_bottom = −1,727 m`, và `s01f11 #8` với `dist_m = 37,80 m`, `yaw_deg = 281,7°`) trực tiếp với file `submission/D13-2A202603289-bangA.csv` và `submission/D13-2A202603289-tukiem.csv`, kết quả khớp hoàn toàn.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Góc nhìn từ mô hình học máy:**
  - Nhãn 3D đo bằng **mét** trong hệ tọa độ thực thay vì pixel vì mô hình nhận diện 3D cho xe tự lái cần dự đoán chính xác vị trí không gian, khoảng cách và kích thước vật lý của vật thể để lập kế hoạch quỹ đạo và tránh va chạm — điều mà tọa độ pixel 2D không thể hiện trực tiếp do phụ thuộc tiêu cự và méo phối cảnh.
  - Vật thưa điểm cần **quy tắc riêng** (ngưỡng 15–20% thể tích, không đoán bừa) vì nếu người gán nhãn vẽ hộp theo cảm tính lên những cụm điểm quá ít thông tin, nhãn nhiễu (sai tâm, sai kích thước hoặc sai class) sẽ làm mô hình học sai biên độ bất định và dễ sinh dự đoán giả (false positive) khi vận hành thực tế.
- **(b) Cách dùng AI:**
  - **Công cụ:** Antigravity (trợ lý AI tích hợp trong IDE).
  - **Câu hỏi chính:** Rà soát toàn bộ cấu trúc repo, kiểm tra bất thường trong Bảng A và phiếu tự kiểm (`selfcheck`), tính toán kiểm chứng các giá trị `z_bottom`, `r` và hỗ trợ trình bày báo cáo đúng quy chuẩn.
  - **Cách sử dụng kết quả:** Tôi dùng AI để phát hiện hộp vẽ nhầm xe gốc LiDAR ở `s01f11`, kiểm chứng số liệu từ Bảng A/phiếu tự kiểm và chạy kiểm lỗi trước khi nộp; toàn bộ việc vẽ, xoá/sửa cuboid trên CVAT và chụp ảnh bằng chứng đều do tôi tự thực hiện trên máy cá nhân, không tải ảnh camera hay dữ liệu VF lên dịch vụ bên ngoài.
- **(c) Điều còn chưa chắc:**
  - Hai frame `s02f38` và `s05f48` mới có 3 cuboid mỗi frame (ít hơn mức khuyến nghị 5 cuboid/frame), nên tôi chưa chắc liệu còn bỏ sót các vật nhỏ hoặc vật ở góc khuất hai bên trong phạm vi 30 m hay không.
  - Các hộp bị che khuất một phần trong toàn bộ 10 frame hiện đều để `occluded = 0`; tôi cần rà soát kỹ hơn trên ảnh camera để đánh dấu `occluded = 1` cho những xe bị chắn phía sau.

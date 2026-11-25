# Báo cáo Day 13 — Trương Duy Anh, 2A202603027

## 1. Kết luận trước

- **Mã số học viên:** `2A202603027` · **Task CVAT:** `D13-2A202603027`
- **10 frame được giao:** `s01f06`, `s01f47`, `s02f23`, `s02f53`, `s03f03`, `s03f56`, `s04f09`, `s04f44`, `s05f02`, `s05f41`
- **Môi trường:** Windows 11 · Trình duyệt: Google Chrome · Phiên bản CVAT: v2.83.0 · **MSSV bạn ngồi cạnh:** `[điền MSSV bạn ngồi cạnh]`

Tổng cộng tôi đã vẽ **73 cuboid** trên 10 frame (68 `Car`, 3 `Bus`, 1 `Truck`, 1 `Pedestrian`); frame ít nhất là `s05f41` với **3 cuboid** do cảnh có ít phương tiện nằm rõ trong tầm 30–50 m và các cụm điểm ở xa quá thưa không đủ cơ sở gán. Lệnh `selfcheck` cờ **16/73 cuboid**, chủ yếu ở các cảnh báo độ cao đáy (`ground_gap_m`), hộp rỗng/rất ít điểm và sai lệch kích thước ở vật xa. Tôi tin nhất ở phần phân loại **class** và **hướng (`yaw`)** của các phương tiện ở gần (< 20 m) nhờ đối chiếu trực tiếp ảnh camera và cụm điểm dày; phần còn yếu nhất là căn **đáy (`z_bottom`)** trên đoạn đường hơi dốc, cũng như **kích thước và hướng** của các vật ở xa (> 30 m) hoặc bị che khuất một phần.

**Cách dùng CVAT (3 thao tác chính):**
1. **Đổi góc nhìn:** Trong khung 3D (Perspective), kéo chuột trái để xoay (xoay thẳng từ trên xuống để nhìn toàn cảnh BEV), kéo chuột phải để dời khung nhìn, lăn con lăn để phóng to/thu nhỏ, nháy đúp vào vùng trống để về góc nhìn mặc định.
2. **Vẽ và chỉnh cuboid:** Chọn biểu tượng cuboid ở thanh công cụ trái → chọn class → bấm **Shape** (không bấm Track) → đưa hộp tới cụm điểm và nháy đúp để đặt; chọn cuboid rồi chỉnh tâm, chiều dài/rộng và xoay `yaw` ở khung **Top**, chỉnh độ cao và hạ đáy chạm mặt đường ở khung **Side** và **Front** (không xoay ở Side/Front).
3. **Bật mũi tên hướng và xem trên ảnh camera:** Bật **Appearance → Cuboid orientation** để hiển thị vạch đỏ ngắn (`+X` của cuboid) chỉ về đầu xe; sau khi bấm Save, chạy `python scripts/lab.py preview` để xem cuboid chiếu lên 7 ảnh camera trong `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f06` và `s01f47`. Ảnh: `t1-s01f06-3d.png`, `t1-s01f06-top.png`, `t1-s01f47-3d.png`, `t1-s01f47-top.png` (ảnh nhìn từ trên xuống có đánh dấu phía trước `−y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f06` | Phía trước xe (`−y`) ở **cạnh trên**; `+x` chỉ sang **trái**, `+y` chỉ xuống **dưới** (ra sau xe) | Tương đối phẳng; thấp hơn LiDAR khoảng `1,7 m` (`z ≈ −1,7 m`) | ~`30 m` (xe con `s01f06#8` ở `30,04 m`) | `Car`, `Bus` | Có xe con phía sau (`y > 0`, cách `14,35 m` và `28,67 m`, xem qua `cam09`) và xe con, xe buýt ở làn bên trái (`x > 0`, cách `4,81 m` đến `23,55 m`, xem qua `cam05`, `cam06`) | Một số cụm điểm thưa ngoài `35 m` và cụm sát lề đường bị che một phần, không đủ hình dạng để kết luận là phương tiện hay vật tĩnh |
| `s01f47` | Phía trước xe (`−y`) ở **cạnh trên**; `+x` chỉ sang **trái**, `+y` chỉ xuống **dưới** (ra sau xe) | Tương đối phẳng; thấp hơn LiDAR khoảng `1,7 m` (`z ≈ −1,7 m`) | ~`31 m` (xe con `s01f47#9` ở `30,77 m`) | `Car`, `Pedestrian` | Có xe con ngay phía sau (`7,11 m`) và sau-trái (`30,77 m`, xem qua `cam09`), xe ở hai bên trái/phải và 1 người đi bộ phía trước bên phải (`14,07 m`, xem qua `cam07`) | Cụm điểm rời rạc phía xa (> `35 m`) và các cụm nhỏ ven vỉa hè bên phải khó tách biệt giữa người, xe dừng hay cột/cây |

**3 điều chỉ camera cho biết (từ `s01f06` và `s01f47`):**
1. Màu sắc và đặc điểm nhận diện rõ ràng loại phương tiện (phân biệt xe con, xe buýt, người đi bộ ngay cả khi cụm điểm LiDAR bị thưa).
2. Phân biệt chính xác đầu và đuôi xe nhờ đèn pha, đèn hậu và kính chắn gió (đặc biệt khi xe đứng yên hoặc chỉ thấy một mặt trên point cloud).
3. Ngữ cảnh giao thông như vạch kẻ làn, lề đường/vỉa hè và tư thế người đi bộ (đang đi bộ, đứng trên vỉa hè hay dắt xe).

**3 điều chỉ LiDAR cho biết (từ `s01f06` và `s01f47`):**
1. Tọa độ 3D và khoảng cách vật lý chính xác theo đơn vị mét (`x, y, z`, `dist_m`) từ tâm LiDAR đến từng đối tượng.
2. Kích thước hình học thật (dài, rộng, cao bằng mét) và cao độ mặt đường quanh xe (`z ≈ −1,7 m`).
3. Trường nhìn 360° liên tục quanh xe (kể cả góc mù giữa các camera), cho phép nhìn toàn cảnh từ trên xuống (BEV) để định vị vật phía sau và hai bên.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f23` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Mặt đường ở vùng xa phía sau hơi chênh cao độ nên đáy cuboid dễ bị lệch so với mốc `−1,7 m` |
| `s02f53` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Có xe ở xa tới `42,05 m` rất thưa điểm; có xe ngược chiều gần ở làn bên trái |
| `s03f03` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Mật độ xe xung quanh khá dày trong tầm `4–21 m`, nhiều xe song song ở hai bên và phía sau |
| `s03f56` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Có xe buýt lớn ở sau-trái (`28,54 m`) gây bóng che phía sau; xe ở `39,78 m` cực kỳ ít điểm |
| `s04f09` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Đa dạng loại phương tiện (`Car`, `Truck`, `Bus`), xe buýt dài `11,68 m` ở bên trái (`21,14 m`) |
| `s04f44` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Có các phương tiện ở rất xa gần ngưỡng `50 m` (`44,18 m` và `48,30 m`) với cụm điểm rất thưa |
| `s05f02` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Mặt đường ở làn bên trái hơi nghiêng/chênh độ cao so với mặt phẳng quanh xe |
| `s05f41` | Cạnh trên (`−y` lên trên, `+x` sang trái, `+y` xuống dưới) | Số lượng phương tiện rõ trong tầm `30 m` ít hơn hẳn các frame khác (chỉ gán được 3 cuboid) |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f06`, idx `2` | `Car` | Có (`yaw_deg = 269,5°`, chỉ về phía trước `−y`) | Có (`z_bottom = −1,780 m`, `ground_gap_m = 0,02 m`) | Có | Có (`4,44 m × 1,78 m × 1,52 m`, nằm trong chuẩn Sedan/hatchback: dài `3,8–5,0 m`, rộng `1,70–1,90 m`, cao `1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f06`, idx `6` | `Bus` | Có (`yaw_deg = 269,5°`, dọc thân xe về phía trước) | Có (`z_bottom = −1,955 m`, `ground_gap_m = 0,03 m`) | Có | Dài `12,42 m` (`10,0–12,5 m`) và cao `3,33 m` (`3,0–3,5 m`) đúng chuẩn Bus thường; rộng `2,77 m` hơi vượt `2,60 m` khoảng `0,17 m` do bao phần mép gương/nhiễu ở `23,55 m` |
| (c) người đi bộ hoặc xe hai bánh | `s01f47`, idx `8` | `Pedestrian` | Có (`yaw_deg = 272,9°`, theo hướng di chuyển) | Sát nền vỉa hè (`z_bottom = −1,095 m`, `ground_gap_m = 0,22 m` do đứng cao hơn lòng đường) | Có | Dài `0,44 m` (`0,4–0,9 m`) và cao `1,51 m` (`1,50–1,95 m`) đúng chuẩn Người lớn; rộng `0,49 m` nhỏ hơn cận dưới `0,50 m` là `0,01 m` (trong sai số ±10%) |

- **Vòng hiệu chuẩn (`D13-HC485087`):** Khi đối chiếu với cả lớp, cuboid của tôi ban đầu có xu hướng căn rộng hơn một chút ở chiều ngang (bao dư khoảng trống quanh gương/mép) và một số hộp chưa kiểm kỹ mặt đáy ở khung Side/Front nên hơi lệch `z_bottom`. Cả lớp thống nhất theo guideline: luôn căn vạch đỏ ngắn (`+X` của cuboid) theo đúng hướng đầu xe đối chiếu qua ảnh camera, hạ đáy chạm sát mặt đường cục bộ (chỉ chỉnh ở Side/Front, không xoay ở hai khung này), và không nới rộng hộp quá mức ở vật bị che mà giữ kích thước sát thực tế theo bảng 4.3.
- **Các vật bỏ qua vì không đủ cơ sở:**
  - `s01f06`, vùng phía trước xa > `40 m`: cụm điểm quá thưa (< 15% thể tích), không xác định được kích thước và hướng (`LOW_VISIBILITY`, `UNCERTAIN_SCOPE`).
  - `s01f47`, sát lề phải > `25 m`: cụm điểm nhỏ bị che khuất một phần, không phân biệt chắc chắn giữa người đứng yên hay vật tĩnh ven đường (`UNCERTAIN_CLASS`, `LOW_VISIBILITY`).
  - `s05f41`, các cụm điểm mờ ở phía xa > `35 m`: chỉ còn vài điểm lẻ tẻ, không đủ cơ sở suy luận hình hộp nên bỏ qua theo quy tắc 15–20% (`LOW_VISIBILITY`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f00` / `t4-1-bong-che.png` / task chung `D13-HC485087` | **Giả thuyết:** Thân xe buýt kích thước lớn sát bên trái (~`4 m`) chắn hoàn toàn các tia laser LiDAR chiếu xuyên qua, tạo vùng khuất phía sau | Phía sau thân xe buýt xuất hiện một vùng hình quạt trống gần như không có điểm nào, trong khi hai bên ngoài góc che điểm vẫn rất dày | Vùng trống là "chưa quan sát được", không phải đường trống. Nếu có vật nằm một phần trong bóng che, bật thuộc tính `occluded` và vẽ cuboid bao trọn cả phần kích thước suy luận được |
| 2. Thưa ở xa | `s04f44` (cuboid `#8` ở `44,18 m`) / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603027` | **Giả thuyết:** Góc phân kỳ giữa 40 tia LiDAR tăng tuyến tính theo khoảng cách nên ở tầm `30–50 m`, khoảng cách giữa các vòng tia lớn khiến rất ít tia trúng thân xe | Xe con ở `44,18 m` chỉ còn `57` điểm mảnh tập trung ở mặt hướng về LiDAR (hay như `s02f53#6` ở `31,60 m` chỉ còn `12` điểm), không còn thấy rõ đầy đủ 4 cạnh thân xe | Chỉ vẽ khi quan sát được khoảng 15–20% thể tích cuboid hoặc có đủ cơ sở từ ảnh camera và bảng 4.3 để suy luận kích thước; nếu không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f06` (cuboid `#2` ở `4,81 m`) / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603027` | **Giả thuyết:** Bề mặt sơn xe tối màu, bề mặt kính hoặc góc tới tia laser xiên làm năng lượng phản xạ yếu, nhiều tia không đạt ngưỡng hồi tiếp về cảm biến | Dù chỉ cách `4,81 m`, `s01f06#2` chỉ có `162` điểm và intensity thấp, ít hơn rất nhiều so với xe con ở khoảng cách tương đương (như `s03f03#8` ở `3,98 m` có `2090` điểm) | Ghi nhận nguyên nhân ở dạng giả thuyết, không khẳng định chắc chắn; kết hợp ảnh camera và bảng 4.3 để bao trọn thân xe, tránh cắt thiếu kích thước và không nới hộp theo quầng phản xạ mạnh của vật lân cận |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / task chung `D13-HC485087` | **Giả thuyết:** Phản xạ đa đường (multipath) giữa mặt đường ướt/kính/bề mặt kim loại hoặc nhiễu cảm biến làm quãng đường truyền tia laser bị tính dài hơn thực tế | Khi xoay nghiêng khung 3D thấy một cụm khoảng `46` điểm ở phía sau bên trái, cách khoảng `29 m`, nằm ở cao độ `z ≈ −13,5 m` (thấp hơn mặt đường xung quanh khoảng `11 m`) | Đây là điểm sai hình học chắc chắn không thuộc vật thể thật; theo guideline tuyệt đối **không** gán nhãn thành vật |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu trích trực tiếp từ `submission/D13-2A202603027-bangA.csv` và cột `points_in` của `submission/D13-2A202603027-tukiem.csv` (mốc mặt đường khoảng `−1,7 m`, lệch quá `0,3 m` là lơ lửng hoặc chìm):

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s03f03`, idx `8`, `Car` | `−0,934 m` | `1,560 m` | `−1,714 m` | `3,98 m` | `4,09 m` | `267,9°` | **Chạm đường** (lệch mốc `−1,7 m` chỉ `0,014 m` < `0,3 m`; `ground_gap_m = 0,06 m`) |
| `s02f23`, idx `7`, `Car` | `−0,569 m` | `1,560 m` | `−1,349 m` | `27,78 m` | `27,79 m` | `267,2°` | **Lơ lửng** (cao hơn mốc `−1,7 m` là `0,351 m` > `0,3 m`; `selfcheck` báo `ground_gap_m = 0,63 m`) |
| `s05f41`, idx `2`, `Car` | `−1,490 m` | `1,500 m` | `−2,240 m` | `13,70 m` | `13,78 m` | `274,3°` | **Chìm** (thấp hơn mốc `−1,7 m` là `0,540 m` > `0,3 m`; `selfcheck` báo `ground_gap_m = −0,43 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất (trên toàn bảng A):** `s01f47`, idx `3` (`Car`, tâm rất sát vị trí xe thu dữ liệu) có `dist_m = 0,04 m` và `r = 0,88 m`. Nếu xét **vật bên ngoài gần nhất** (loại trừ vùng sát xe thu dữ liệu): `s03f03`, idx `8` (`Car`) có `dist_m = 3,98 m` và `r = 4,09 m`.
  - **Xa nhất:** `s04f44`, idx `7` (`Car`) có `dist_m = 48,30 m` và `r = 48,32 m`.
- **(b) So sánh kích thước `l, w, h` với bảng 4.3:**
  - **Người đi bộ (`s01f47`, idx `8`, `Pedestrian`):** `l = 0,440 m`, `w = 0,490 m`, `h = 1,510 m`. So với bảng 4.3 (Người lớn: dài `0,4–0,9 m`, rộng `0,5–0,9 m`, cao `1,50–1,95 m`), `l` và `h` nằm trong khoảng chuẩn; riêng `w = 0,490 m` nhỏ hơn cận dưới `0,50 m` khoảng `0,010 m` (2%, vẫn trong sai số ±10% của Pedestrian) vì ở khoảng cách `14,07 m` cụm điểm theo phương ngang khá mảnh (`80` điểm).
  - **Xe con (`s01f06`, idx `2`, `Car`):** `l = 4,440 m`, `w = 1,780 m`, `h = 1,520 m` — cả ba chiều đều nằm gọn trong khoảng Sedan/hatchback (`3,8–5,0 m`, `1,70–1,90 m`, `1,40–1,60 m`). Ngược lại, một xe con ở xa như `s02f53`, idx `6` (`dist_m = 31,60 m`, `12` điểm) có `l = 5,530 m`, `w = 2,370 m`, `h = 1,930 m`, trong đó chiều dài và chiều rộng vượt ngưỡng chuẩn (`5,3 m` và `2,10 m`) do cụm điểm quá thưa khiến việc ước lượng mép hộp bị nới rộng hơn thực tế.
- **(c) Góc hướng `yaw_deg`:**
  - **Xe cùng chiều phía trước (`s01f47`, idx `4`, `Car` ở `y = −10,110 m`):** `yaw_deg = 267,2°` — rất gần `270°` (hướng `−y`).
  - **Xe ngược chiều (`s02f53`, idx `2`, `Car` ở `y = −4,150 m`):** `yaw_deg = 88,8°` (hoặc `s02f53`, idx `1` có `yaw_deg = 90,5°`) — rất gần `90°` (hướng `+y`).

**Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):**

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s03f03`, idx `8`, `Car` | `3,98 m` | `2090` |
| `s03f03`, idx `2`, `Car` | `12,37 m` | `786` |
| `s02f53`, idx `4`, `Car` | `23,04 m` | `255` |
| `s04f44`, idx `8`, `Car` | `44,18 m` | `57` |

**Quy luật mật độ và so sánh với slide bài giảng:** Khi khoảng cách tăng gấp ~`2` lần (từ `12,37 m` lên `23,04 m`, và từ `23,04 m` lên `44,18 m`), số điểm LiDAR trong cuboid giảm còn khoảng `1/3` đến `1/4` (từ `786` xuống `255` rồi còn `57` điểm, gần theo quy luật nghịch bình phương khoảng cách). Kết quả này rất sát đường cong trung vị của xe con trên slide bài giảng (`2090` điểm ở `3,98 m` so với ~`1968` ở `0–10 m`; `786` ở `12,37 m` so với ~`718` ở `10–20 m`; `255` ở `23,04 m` so với ~`214` ở `20–30 m`; `57` ở `44,18 m` nằm giữa mức ~`72` ở `30–40 m` và ~`19` ở `60–80 m`).

**Tự kiểm sau `preview` / `selfcheck`:**
- **Hai cuboid đã kiểm và chỉnh sửa:**
  1. `s03f03`, idx `8` (`Car`): rà soát trên `preview` và các khung Top/Side/Front để căn sát kích thước (`5,030 m × 1,910 m × 1,560 m`), hướng `yaw = 267,9°` và đáy `z_bottom = −1,714 m` (`ground_gap_m = 0,06 m`).
  2. `s01f47`, idx `5` (`Car`): đối chiếu `preview` trên `cam09` và chỉnh lại tâm cùng cao độ đáy (`z_bottom = −1,648 m`, `ground_gap_m = 0,16 m`, `1429` điểm) cho ôm khít thân xe phía sau.
- **Các cảnh báo giữ nguyên kèm lý do:**
  - `s04f44`, idx `2` và idx `4` (cờ "điểm sát ngay ngoài mép"): giữ nguyên kích thước hộp ôm sát thân xe vì các điểm ngay ngoài mép thuộc vật/lề đường bên cạnh, nếu nới rộng sẽ bao dư sai guideline.
  - `s02f23`, idx `6` (cờ `ground_gap_m = 0,47 m`) và `s04f09`, idx `3` (cờ `ground_gap_m = 0,49 m`): tạm giữ nguyên sau khi đối chiếu khung Side/Front vì mặt đường cục bộ tại vị trí đó có độ chênh cao nhẹ so với mặt phẳng ước lượng quanh xe.
  - Các cảnh báo hộp rỗng/lệch kích thước/ngược `length < width` còn sót (như `s01f06#3`, `s04f44#7`, `s03f56#5`, `s05f02#5`) là các lỗi thực sự tôi đã phát hiện qua `selfcheck` và ghi nhận trung thực để tiếp tục chỉnh trên CVAT trước khi chốt.

**Đổi kiểm với bạn ngồi cạnh:**
- **MSSV người đổi kiểm:** `[điền MSSV bạn ngồi cạnh]`
- **Hai con số đối chiếu trực tiếp trên bảng A:** `s03f03`, idx `8` (`dist_m = 3,98 m`, `z_bottom = −1,714 m`) và `s04f44`, idx `7` (`dist_m = 48,30 m`, `yaw_deg = 175,8°`).
- **Kết quả:** Khớp hoàn toàn với bảng A trên máy (không cần sửa số trong báo cáo).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Từ phía mô hình học trên nhãn này:**
  - Nhãn 3D phải đo bằng **mét** trong hệ tọa độ thực thay vì pixel vì mô hình nhận thức và lập kế hoạch quỹ đạo của xe tự hành cần biết chính xác vị trí, khoảng cách vật lý và kích thước thật của chướng ngại vật để ước lượng vận tốc, khoảng cách phanh và tránh va chạm — điều mà ảnh 2D theo pixel không thể hiện trực tiếp do biến dạng phối cảnh.
  - Vật thưa điểm cần **quy tắc gán nhãn riêng** (ngưỡng 15–20% thể tích hoặc đủ cơ sở suy luận, không đoán) vì nếu gán bừa theo cảm tính sẽ đưa nhiễu lớn (sai tâm, sai kích thước, sai hướng) vào tập huấn luyện làm mô hình học sai đặc trưng; ngược lại nếu bỏ qua tùy tiện các vật ở xa vẫn còn đủ tín hiệu sẽ làm giảm khả năng phát hiện sớm (recall) của mô hình.
- **(b) Dùng AI:**
  - **Công cụ:** Antigravity 
  - **Câu hỏi chính:** Cách kiểm tra định dạng bài nộp,  và chuẩn hóa trình bày báo cáo theo yêu cầu rubric (không đưa ảnh camera hay dữ liệu VF gốc cho AI).
  - **Cách dùng kết quả:** Dùng các con số đã đối chiếu từ bảng A và phiếu tự kiểm để hoàn thiện báo cáo, tự kiểm tra lại từng mục trước khi chạy `check_submission.py`.
- **(c) Điều còn chưa chắc:**
  - Còn chưa chắc ở việc căn độ cao đáy `z_bottom` trên các frame có mặt đường hơi nghiêng hoặc dốc (như `s02f23`, `s05f02`, `s05f41`) khiến một số hộp bị cờ lơ lửng hoặc chìm.
  - Hai cuboid ở `s01f06` và `s04f44` bị cờ `length < width` (nhầm chiều dài và chiều rộng do hướng hộp xoay lệch 90°), cùng một vài hộp bị cờ rỗng/ít điểm ở xa và hai hộp ở gần gốc tọa độ (`s01f06#1`, `s01f47#3`) dễ trùng với chính xe thu dữ liệu cần được rà soát kỹ hơn trên CVAT.

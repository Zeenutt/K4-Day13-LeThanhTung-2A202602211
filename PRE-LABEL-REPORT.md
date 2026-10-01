# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác. Không commit bản điền này.

## Nhóm và provenance

- Mã nhóm/phòng: 
- Thành viên: xem `TEAMMATES.md`. 
- Trạng thái: `executed-by-group`. 
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: một phiên thao tác trên máy, 2026-10-01 khoảng 22:09 giờ Việt Nam (UTC+7). Windows amd64, Docker Desktop 4.67.0, engine Linux amd64, Python 3.10.7.
- Image tag và image ID; phiên bản repo: tag `day13-pointpillars:lc-20261001-amd64`. Manifest ghi OCI index `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`. Image đã nạp và dùng để infer là config id `sha256:e7b6032b36dfc01b51da2fb29d752942bb51f7e9f5c3fc5d0e73276a7c8e9931`. Repo revision trong manifest: `0831856d921609312d42c7582c366e5a311bb7b1`.
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: `C:\Lab13\input\demo.pcd`, frame_id `demo`, 17238 điểm, SHA256 `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`. Chạy local, không phải frame Robotaxi.
- Checkpoint: PointPillars KITTI có sẵn trong image; hash khớp manifest `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1` (`/opt/PointPillars/pretrained/epoch_160.pth`).
- Phạm vi: front-window, một pass `identity`, không `--full-scene`. Score threshold 0.3.
- Giả định kênh thứ tư/intensity và nguồn z_ground: reflectance gốc bị bỏ trong PCD; adapter dùng kênh hằng; RGB = 0. `z_ground` ước lượng từ scan = 0.075 m.

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy **Số hộp** từ `n_boxes`, **mean_z** từ `mean_z` trong `run-A/B/C/summary.csv`; không tự tính lại hoặc đoán. `mean_z` không phải điểm chất lượng. Mở `side-*.png`, đối chiếu `boxes-*.json` để ghi quan sát. Số hộp không phải đáp án cần khớp nhóm khác.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `C:\ket-qua-nhom-03\run-A\boxes-demo-delta-0-voxel-0.16.json`, `C:\ket-qua-nhom-03\run-A\side-demo-delta-0-voxel-0.16.png`, `C:\ket-qua-nhom-03\run-A\summary.csv` | CSV: `n_boxes=1`, `mean_z=0.330`. JSON: 1 `vehicles`, score 0.322, tâm x=13.154 y=-0.451 z=0.330. Log: `rear=0`. |
| B | 1.73 | 0.16 | 13 | 1.034 | `C:\ket-qua-nhom-03\run-B\boxes-demo-delta-1.73-voxel-0.16.json`, `C:\ket-qua-nhom-03\run-B\side-demo-delta-1.73-voxel-0.16.png`, `C:\ket-qua-nhom-03\run-B\summary.csv` | CSV: `n_boxes=13`, `mean_z=1.034`. JSON: 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian`. Score 0.318–0.933. z từ 0.698 đến 1.426. Log: `rear=0`. |
| C | 1.73 | 0.32 | 6 | 1.091 | `C:\ket-qua-nhom-03\run-C\boxes-demo-delta-1.73-voxel-0.32.json`, `C:\ket-qua-nhom-03\run-C\side-demo-delta-1.73-voxel-0.32.png`, `C:\ket-qua-nhom-03\run-C\summary.csv` | CSV: `n_boxes=6`, `mean_z=1.091`. JSON: cả 6 hộp đều `pedestrian`. Score 0.301–0.808. z từ 0.841 đến 1.378. Không còn `vehicles`. Log: `rear=0`. |

- A/B — chỉ đổi delta: A có 1 hộp; B có 13 hộp. `C:\ket-qua-nhom-03\run-A\summary.csv` và `C:\ket-qua-nhom-03\run-B\summary.csv` khác ở `n_boxes` (1 so với 13) và `mean_z` (0.330 so với 1.034). Ảnh Side A chỉ một hộp xe thấp, score 0.322; Side B có nhiều hộp xe nằm trên đám điểm, score tới 0.933. Đây là chạy lại model trên input khác, không chỉ dịch hộp cũ; nếu chỉ cộng 1.73 m vào z hộp A thì z thành 2.060, trong khi z các hộp B chỉ từ 0.698 đến 1.426. Điều còn chưa chắc là hộp nào của B, nếu có, ứng với cùng vật thể với hộp A, vì không có nhãn chuẩn.
- B/C — chỉ đổi pillar: B có 13 hộp; C có 6 hộp. `C:\ket-qua-nhom-03\run-B\summary.csv` và `C:\ket-qua-nhom-03\run-C\summary.csv` khác ở `n_boxes` (13 so với 6); `mean_z` gần nhau (1.034 và 1.091). Số lượng/lớp/vị trí thay đổi như sau: B có 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian`; C chỉ còn 6 `pedestrian`, không có xe. Ví dụ xe B tại x=8.094, score 0.933 không còn trong C. Có đủ bằng chứng để kết luận tốt hơn không? Không. Nhiều hộp hơn hoặc score cao hơn không chứng minh B đúng hơn; không có ground truth.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào? Chỉ pass phía trước, `rear=0`. Ảnh Side là x theo z, không thấy y và không thấy yaw. Cửa sổ hình khoảng z −4 đến 4 m và x −20 đến 70 m. Hộp miss phía sau, lệch ngang, hoặc sai yaw có thể không lộ trên hình Side.
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp? Không JSON nào trong `C:\ket-qua-nhom-03` đủ cơ sở để import CVAT hay job Robotaxi. Prediction là demo KITTI, reflectance không thật. Ca QC ghi `training_only`. Cần đối chiếu nhiều view và ảnh camera trước khi sửa hộp trên frame được giao; không nhập prediction demo này vào frame khác.

## Ca QC có kiểm soát — không import CVAT

Nguồn là prediction B `C:\ket-qua-nhom-03\run-B\boxes-demo-delta-1.73-voxel-0.16.json`, SHA256 `16f30b0833ec5978320d71a102bf54df825d76c8d8264e563a7b7b40bee4cf61`. Helper `pipeline-qc-cases.py` tạo biến đổi có chủ đích từ prediction này, trừ `delta + z_ground` = 1.805 m. Không phải kết quả inference riêng và không phải nhãn đúng.

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0 / 13 | 0 | Không | Giữ bản sao prediction. Không dùng làm đáp án. | `C:\ket-qua-nhom-03\qc-cases\case-correct.json`, `C:\ket-qua-nhom-03\qc-cases\side-correct.png` |
| case-batch-z | 13 / 13 | −1.805 m trên mọi hộp | Không, chỉ z | Dừng cả batch. Kiểm transform/pipeline vì phép z ngược bị bỏ. Không sửa từng hộp. | `C:\ket-qua-nhom-03\qc-cases\case-batch-z.json`: mọi z giảm đúng 1.805 m. Ảnh Side các hộp nằm dưới đường z=0. |
| case-one-box-z | 1 / 13 | −1.805 m, hộp đầu `vehicles` x=8.094, z 0.921 → −0.884 | Không, chỉ z của hộp đó | Không dừng cả batch. Kiểm hộp này trên nhiều view. 12 hộp còn lại giữ nguyên. | `C:\ket-qua-nhom-03\qc-cases\case-one-box-z.json`, `C:\ket-qua-nhom-03\qc-cases\side-one-box-z.png` |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng.

## Nhận xét cá nhân

- Vai trò đã làm: vận hành container, kiểm JSON/cấu hình, xem hình Side, ghi log.
- Một quan sát có dẫn file: `C:\ket-qua-nhom-03\run-B\summary.csv` ghi 13 hộp, mean_z 1.034; `C:\ket-qua-nhom-03\run-C\summary.csv` ghi 6 hộp, mean_z 1.091, và JSON lượt C chỉ còn pedestrian. Ảnh Side B có nhiều hộp xe trên đám điểm; ảnh `C:\ket-qua-nhom-03\qc-cases\side-batch-z.png` kéo cả cụm xuống dưới đường z=0.
- Phép z thuận: `z_model = z_source - z_ground - delta`. Phép ngược: `z_source = z_model + delta + z_ground`. Quên phép ngược trên hộp đã ở hệ nguồn tương đương trừ 1.805 m, đúng ca batch-z.
- Quyết định lỗi batch: nếu mọi hộp lệch cùng một lượng z thì dừng pipeline, không import CVAT. Nếu một hộp lệch thì soi riêng hộp đó trên nhiều view.
- Điều chưa chắc: không có ground truth nên không kết luận B đúng hơn C trên đường thật. Runner gốc `student-bundle.py` không tự `passed` trên Docker Desktop này vì inspect digest OCI index thất bại; inference dùng image config đã đối chiếu checkpoint.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
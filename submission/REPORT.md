# Báo cáo Day 08 — 2A202603169 - Nguyễn Minh Hiếu

> Báo cáo tổng hợp model/recipe, Human QC, SAM, Calibration và quyết định chất lượng. Các tỷ lệ đều ghi mẫu số và nguồn. **REWORK** là quyết định trên bộ nhãn hiện tại; chưa có bằng chứng đạt chuẩn Release. Các điểm `[CẦN XÁC MINH]` phải được bổ sung từ log/ảnh gốc, không tự đoán.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng khi merge | Vì sao (run_id) |
|---|---|---|---|
| `person`, `bicycle`, `motorbike`, `car`, `bus`, `truck` | YOLO26m | Mặc định `0.30` | `r01`: 215 box/8 ảnh ở conf 0.25, 1.28 s/ảnh; `r02`: 139 box/8 ảnh ở conf 0.50, 0.45 s/ảnh. Tăng ngưỡng giảm 76/215 = 35,3% box nhưng có nguy cơ mất vật nhỏ. |
| `three_wheeler` | YOLOE-26s, prompt gồm `tuk-tuk`, `cyclo`, `cargo tricycle` | `0.15` | `r03`: 8 box/8 ảnh pilot, 0.18 s/ảnh; `threshold_sweep.md` đề xuất 0.15, recall 7/7 = 100% và precision khoảng 78% trên mẫu Gold cho lớp này. |

**Nguồn:** `submission/model_log.csv` (`r01`–`r04`), `submission/qc/threshold_sweep.md`, `work/prelabel/merged_review/pre.coco.json`.

- **Lệnh `merge_prelabels.py` đã chạy (nguyên văn):** `[CẦN XÁC MINH từ lịch sử Colab hoặc notebook; thông số xác nhận được: --thr "three_wheeler=0.15"; conf mặc định 0.30]`. Không ghi một lệnh phỏng đoán thành lệnh thực nghiệm.
- **Pipeline:** YOLO26m chạy trên 36 ảnh tạo 1.669 box ứng viên ở conf 0.05; YOLOE tạo 125 box ứng viên ở conf 0.05; tổng 1.794 box trước merge. Kết quả sau merge 551 box/28 ảnh (24 Batch + 4 Gold): 288 `person`, 143 `motorbike`, 69 `car`, 30 `three_wheeler`, 14 `bicycle`, 5 `truck`, 2 `bus`. **Nguồn:** log CP3–CP4 và `work/prelabel/merged_review/pre.coco.json`.
- **SAM:** Chọn SAM 2.1 Base (`sam2.1_b.pt`) để sinh mask từ gợi ý box/click, chạy trên Google Colab CUDA do MX350 2 GB có hạn chế VRAM. Trong `r04`, SAM sinh 16 mask/16 box trên 2 ảnh, tốc độ 1,78 giây/ảnh. Ở Part 5 còn so sánh thêm MobileSAM, không tuyên bố phiên bản nào luôn tốt hơn nếu chưa đánh giá độc lập. **Nguồn:** `submission/model_log.csv`, file tại `submission/masks/`.
- **Lỗi không sửa bằng đổi ngưỡng:** Xe ba bánh bị nhận thành `car` hoặc `truck` là sai nhãn/thiếu lớp trong taxonomy detector, không thể chỉ thay confidence để bổ sung lớp. Dùng YOLOE prompt `three_wheeler` và duyệt lại bằng người; với box sai lớp thì sửa category trong CVAT. Ví dụ cần duyệt bổ sung từng xuất hiện ở `img_018.jpg` (box ID 189 trong bản trung gian, không đồng nhất ID bản Release).
- **Box điểm cao mà sai / điểm thấp mà đúng:** `[CẦN XÁC MINH: tên ảnh + ID + score lấy trực tiếp từ COCO pre-label; hiện log gửi chưa có confidence đủ để kết luận]`.
- **Giới hạn:** Ngưỡng được chọn trên chính 4 ảnh Gold nên chỉ số trên Gold là *in-sample*, có thể lạc quan. 4 ảnh ít và phân bố lớp lệch; không coi đây là kiểm định độc lập. **Nguồn:** `submission/qc/threshold_sweep.md`, `submission/qc/review_stats.md`.

## 2. AI error profile (Part 4)

**Lượt duyệt độc lập đầu tiên:**

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 464 | 464/472 = 98,3% | 472 box máy |
| edited | 2 | 2/472 = 0,4% | 472 box máy |
| deleted | 6 | 6/472 = 1,3% | 472 box máy |
| added | 1 | 1/467 = 0,2% | 467 đối tượng sau duyệt trong vùng đánh giá |

**Nguồn:** `submission/qc/review_stats.md`. Script loại 22 đối tượng gần vùng bẫy khỏi bảng sai số model.

**Lượt QC bổ sung V3 sau khi xem phản hồi** (không tính là đánh giá mù mới): accepted 463/472 = 98,1%; edited 2/472 = 0,4%; deleted 7/472 = 1,5%; added 3/468 = 0,6%. Trong đó 1 trường hợp deleted + added là đổi lớp và số vật sót thật xấp xỉ 2 theo contract script. **Nguồn:** `submission/qc/review_stats_v3.md`.

**Lớp khó nhất trong số liệu V3:** `three_wheeler` có 6/7 box bị xóa, 2 box chỉnh sửa và 1 box thêm; các trường hợp xe ba bánh chồng lấn, xe hàng phức tạp dễ gây false positive hoặc lệch box. Ví dụ `img_013.jpg`, `img_017.jpg`; quan sát QC còn ghi lỗi box ở `img_003.jpg`. Đó là nhận xét trên mẫu, không chứng minh chất lượng toàn bộ lớp trong tự nhiên.

**Máy so với Gold:** TP = 37, FP = 28, FN = 10; precision = 37/(37+28) = **37/65 = 56,9%**; recall = 37/(37+10) = **37/47 = 78,7%**. Có 4 ảnh Gold; script dùng 47 đối tượng hợp lệ, dù lượt validator ban đầu từng ghi 50 annotation (cần đối chiếu bộ lọc khi giải thích chênh lệch). **Nguồn:** `submission/qc/review_stats.md`.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Lần kiểm tra độc lập đầu:** phát hiện **2/12 = 16,7%** bẫy, dưới yêu cầu ≥80% (tương đương ít nhất 10/12). Theo loại: `extra` 2/3, `missing` 0/3, `wrong_class` 0/3, `loose` 0/3. **Nguồn:** `submission/qc/review_stats.md`.
- **Bẫy lọt:** nhiều lỗi thiếu vật, sai lớp và box lỏng không được nhận ra. Quy trình đã chưa thực hiện đầy đủ bước *quét ảnh trước khi nhìn box máy*, sau đó kiểm tra lớp và độ khít từng box; đây là giả thuyết giải thích kết quả, không thể xác minh hành vi duyệt chỉ qua script.
- **Accepted cao không đồng nghĩa QC tốt:** 464/472 = 98,3% box máy được giữ nguyên nhưng mới phát hiện 2/12 bẫy. Việc chấp nhận nhiều box dễ tạo thiên kiến dựa vào dự đoán sẵn có. Bản V3 sửa bổ sung sau phản hồi vẫn 2/12 và **không được trình bày như phép thử độc lập mới**. **Nguồn:** `review_stats.md`, `review_stats_v3.md`.

## 4. Máy có đáng dùng không?

| Cách làm | Thời gian | Số ảnh | Trung bình |
|---|---:|---:|---:|
| Gán tay Gold | 30 phút | 4 ảnh | **30/4 = 7,50 phút/ảnh** |
| Duyệt Pre-label | 100 phút | 24 ảnh | **100/24 = 4,17 phút/ảnh** |

**Nguồn:** `submission/time_log.csv` (dòng `gold_manual`, `batch_review` sau cập nhật). So sánh tham khảo: (7,50 − 4,1667)/7,50 ≈ **44,4%** ít thời gian/ảnh hơn. Kết luận này **không** bao gồm đầy đủ chi phí suy luận, chỉnh sửa hậu QC hoặc bảo đảm chất lượng; 4 ảnh Gold và 24 ảnh Batch khác nhau về độ khó, và người duyệt Gold trước đã quen taxonomy.

**Quyết định về sử dụng máy:** Có, để *tạo nhãn sơ bộ* ở các lớp thông dụng và bổ sung `three_wheeler` bằng YOLOE; **không** dùng để tự Release. Precision máy trên Gold chỉ 37/65 = 56,9%; recall 37/47 = 78,7%; QC bắt 2/12 = 16,7% bẫy. Duyệt bằng người phải độc lập và kiểm tra lại theo checklist.

**Ba quy tắc giảm thiên kiến có thể kiểm tra:**
1. Với **24/24** ảnh Batch, quét toàn ảnh không nhìn box để tìm vật bị bỏ sót, sau đó mới xem box AI; đánh dấu hoàn tất checklist theo ảnh.
2. Với **100% box** trong ảnh QC, kiểm tra `category` và bốn cạnh box so với vật thật; ghi lỗi theo ID và ảnh ví dụ, không chấp nhận chỉ bằng confidence.
3. Chốt lượt review đầu trước khi mở đáp án bẫy; lưu báo cáo độc lập và bản Rework riêng, không ghi đè 2/12 thành một kết quả mù mới.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Số click | Có gồm người lái/hàng hóa? | Lỗi quan sát | File trong `submission/masks/` |
|---|---|---:|---|---|---|
| `img_013.jpg` | SAM 2.1 Base | 3 (2 dương, 1 âm) | Có dính một phần người lái | Vùng mask lan sang áo/chân người lái; cần thêm click âm | `tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 Base | 3 (2 dương, 1 âm) | Không thấy dính đáng kể trong ảnh kiểm | Biên gầm/bánh xe chưa hoàn toàn khít | `tuktuk_img004_sam2.jpg` |
| `img_017.jpg` | SAM 2.1 Base | 3 (2 dương, 1 âm) | Không thấy dính đáng kể trong ảnh kiểm | Có vùng khuyết dưới gầm/cạnh bánh | `tuktuk_img017_sam2.jpg` |
| `img_013.jpg` | MobileSAM | 3 (2 dương, 1 âm) | Tách phần lớn người lái tốt hơn bản SAM 2.1 hiện tại | Có lỗi nhỏ gần sàn/bánh; mask gồm 2 polygon | `tuktuk_img013_mobile.jpg` |

Ở cùng `img_013.jpg`, SAM 2.1 tạo **317.255 pixel = 45,4% ảnh, 1 polygon**; MobileSAM **245.541 pixel = 35,2% ảnh, 2 polygon** (theo log Colab). Diện tích nhỏ hơn không tự động đúng hơn; quan sát trực quan mới cho biết MobileSAM giảm phần dính người lái trong thử nghiệm này.

**Guideline §4.2:** Mask phương tiện chỉ gồm khung, mái, thân, bánh; người lái (`person`) và hàng hóa nhô ngoài phải tách. Điểm âm hỗ trợ nhưng một điểm có thể chưa đủ; cần thêm click âm tại vùng người lái hoặc dùng box prompt và duyệt mask sau chạy. **Nguồn:** `guideline/guideline_v1.0_label_contract.md`, `submission/masks/*.jpg`, log Part 5. **CVAT Interactor:** `[CẦN XÁC MINH: chưa có log chứng minh sử dụng interactor trong CVAT; mask ở đây chạy offline qua Colab]`.

## 6. Calibration (Part 6)

Có **3 quyết định/3 dòng** trong `submission/qc/calibration_decisions.csv`:
- **CAL-01 — `loi_may`, `img_013.jpg|img_017.jpg`:** Không dùng một box bao nhiều xe ba bánh độc lập; kiểm tra instance và các cạnh box.
- **CAL-02 — `cau_hoi_mo`, `img_003.jpg`:** Box xe ba bánh chở hàng bao bộ phận phương tiện (khung, mái, thân, bánh), không mở rộng để bao hàng nhô ngoài.
- **CAL-03 — `can_review`, `img_018.jpg|img_021.jpg`:** Quét ảnh trước khi chấp nhận pre-label, kiểm tra riêng `person` và `motorbike`; ca nhỏ/che khuất chưa rõ phải đánh dấu để xem lại.

**Nguồn:** `submission/qc/calibration_decisions.csv` và `submission/guidelines/guideline_v1.1_calibrated.md` (đã bổ sung CAL-01, CAL-02, CAL-03 vào bản sao guideline v1.0). Các thay đổi cần được đối chiếu lại với chính v1.0 để phân biệt **guideline thiếu quy tắc**, **người áp dụng sai** và **lỗi model**; không tự nhận mọi ca máy sai là guideline mơ hồ.

## 7. QC và quyết định (Part 6)

**Thiết kế mẫu:** Script `qc_sample.py --n 5 --seed 42` lấy **8/24 ảnh batch = 33,3%** gồm **3 ảnh `can_review` + 5 ảnh ngẫu nhiên**, với **201/478 đối tượng = 42,1%** số annotation Release Candidate. Ảnh trong mẫu: `img_003` (21), `img_006` (24), `img_013` (11), `img_014` (19), `img_017` (6), `img_018` (49), `img_021` (39), `img_035` (32). Người kiểm: tôi, đồng thời đã tham gia duyệt nhãn — **không độc lập hoàn toàn**. **Nguồn:** `submission/qc/qc_sample.csv`.

**Phiếu kiểm trực quan hiện tại** ghi **2 lỗi/201 đối tượng = 1,0%** và **2 ảnh có lỗi/8 ảnh = 25,0%**. Hai lỗi được ghi là `sai_vi_tri_kich_thuoc` trên `img_003.jpg` (#21, mái che ngoài box) và `img_017.jpg` (#143, box tràn qua xe khác). Các ảnh còn lại ghi `khong` theo lượt đối chiếu hiện có, **không có nghĩa toàn bộ vật nhỏ/che khuất đều đã được chứng nhận chính xác**. Cần xác nhận cuối trong CVAT trước khi sửa tọa độ. **Nguồn:** `submission/qc/qc_review.csv`, `submission/qc/qc_report_auto.md`.

**Giới hạn mẫu nhỏ:** Với 2/8 ảnh lỗi, **cận trên Wilson 95% khoảng 59%**; không thể chứng minh chuẩn ảnh lỗi ≤2% từ mẫu 8 ảnh. Script kiểm được tên file, taxonomy, tọa độ trong biên, schema, so sánh bộ nhãn và thống kê phiếu; không tự biết box có ôm sát vật thật, lớp có đúng ngoài đời, hoặc vật bị cả model và người cùng bỏ sót. **Nguồn:** `qc_report_auto.md`, `docs/CALIBRATION_QC.md`.

**Quyết định: REWORK — 2026-11-19.** Lý do: tỷ lệ ảnh lỗi **2/8 = 25,0% > 2%**, bắt bẫy **2/12 = 16,7% < 80%** và còn nghi vấn ảnh đông/nhỏ. Chưa được Release. Cần xác nhận/sửa hai box QC trong CVAT, export `submission/labels/qc_fixed.json` nếu sửa, chạy lại `build_release.py`, tái lấy mẫu QC theo hướng dẫn và giữ rõ vòng trước/vòng sau. Nếu có vấn đề guideline chưa chốt hoặc ảnh hỏng không dùng được, quyết định theo cây **Hold → Rework → Release**, không cố ép Release.

## 8. Ghi chú cá nhân

- **Dùng AI:** YOLO26m tạo box lớp thông dụng; YOLOE-26s bổ sung `three_wheeler`; SAM 2.1 và MobileSAM tạo mask từ click; ChatGPT hỗ trợ phân tích log, định dạng COCO, đề xuất sửa nhãn và viết nháp báo cáo. Tôi kiểm tra bằng ảnh gốc/overlay, CVAT, validation scripts và kết quả QC; những gợi ý AI không được xem là QC độc lập.
- **Gold có bị nhiễm không:** Gold được tạo/commit trước các lần chạy model CP2–CP3, nhưng `[CẦN XÁC MINH với quá trình gán thực tế: đã/không xem gợi ý AI trước khi chốt 4 ảnh Gold]`. Không khẳng định “hoàn toàn độc lập” nếu không có bằng chứng.
- **Máy/GPU/CVAT:** Windows 11, Ubuntu WSL2, Intel Core i7, RAM 16 GB, NVIDIA MX350 2 GB VRAM; chạy suy luận chính trên Google Colab GPU CUDA; CVAT Local Docker v2.81.0. **Nguồn:** môi trường lab và log công việc.
- **Vướng mắc:** quyền chạy gói Python ở Windows, thiết lập `.venv` WSL, chuẩn COCO ZIP cho CVAT, chuyển file từ Colab về WSL, xử lý các phiên bản annotation, hạn chế VRAM MX350.
- **Cache:** Các thử nghiệm model ghi trong log được thực hiện trên Colab; không dùng `data/cache/` thay cho các lần chạy đã báo cáo.
- **Lỗi/bug script:** Không ghi nhận bug của `qc_report.py`; script chỉ đọc `review_stats.json` của lượt đầu và báo cáo gợi ý **REWORK**. Các vấn đề gặp phải chủ yếu là đường dẫn/file import/export và chất lượng nhãn; không đồng nhất với bug phần mềm.
- **Lần sau:** Tôi sẽ tách lượt quét ảnh không nhìn nhãn AI, lập checklist missing/class/bbox cho từng ảnh và tổ chức một lượt kiểm độc lập trước khi chốt nhãn để giảm automation bias.

# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 472; đối tượng sau duyệt: 468; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 463 | 463/472 = 98.1% | 472 box máy |
| edited (sửa) | 2 | 2/472 = 0.4% | 472 box máy |
| deleted (xoá) | 7 | 7/472 = 1.5% | 472 box máy |
| added (vẽ thêm) | 3 | 3/468 = 0.6% | 468 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 2.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 250 | 0 | 1 | 0 |
| bicycle | 9 | 0 | 0 | 0 |
| motorbike | 127 | 0 | 0 | 0 |
| car | 55 | 0 | 0 | 0 |
| bus | 2 | 0 | 0 | 2 |
| truck | 3 | 0 | 0 | 0 |
| three_wheeler | 17 | 2 | 6 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 47 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 28 | 10 | 56.9% (KTC 95%: 45–68%) | 78.7% (KTC 95%: 65–88%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 2/12 = 16.7% (KTC 95%: 5–45%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 0/3
  - extra: 2/3
  - missing: 0/3
  - wrong_class: 0/3
  - lọt: img_026.jpg (loose), img_031.jpg (loose), img_034.jpg (missing), img_029.jpg (wrong_class), img_018.jpg (wrong_class), img_027.jpg (missing), img_006.jpg (wrong_class), img_022.jpg (extra), img_013.jpg (missing), img_008.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).

# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 201 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 2/201 = 1.0% | 201 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/8 = 25.0% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 464/472 = 98.3% / 2/472 = 0.4% / 6/472 = 1.3% | 472 box máy |
| Máy: added | 1/467 = 0.2% | 467 đối tượng sau duyệt |
| Bắt lỗi bẫy | 2/12 = 16.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 2 | img_003.jpg, img_017.jpg |

## Giới hạn thống kê
- 2 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **59%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 25.0% > 2%
- bắt bẫy 17% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.

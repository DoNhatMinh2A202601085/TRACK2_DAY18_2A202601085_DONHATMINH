# Day 18 Reflection

Trong "Top 5 Lakehouse Anti-Patterns", team chúng tôi dễ vướng nhất vào **anti-pattern #3: Không vacuum orphan files sau khi job crash** (NB6 đã chứng minh).

**Lý do:**
- Pipeline ingest của team có thể chạy đồng thời nhiều writer, khi job bị kill giữa chừng → file parquet bị ghi dở nằm lại trong storage mà không vào `_delta_log`
- `VACUUM` không thu hồi các file này (chỉ sweep file đã tombstone trong log)
- Team chưa có script dọn orphan tự động — chỉ phát hiện khi hóa đơn S3 tăng bất thường
- Trong production, ước tính ~5-10% storage là orphan files không được track

Lab này cho thấy cần kết hợp 3 bước: (1) list file trên storage, (2) trừ tập file trong Delta log, (3) xoá phần hiệu. Đây là anti-pattern phổ biến nhất trong team vì mọi người tin tưởng `VACUUM` đã xử lý hết.

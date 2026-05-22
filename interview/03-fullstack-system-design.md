# Fullstack & System Design Questions

Mục tiêu của phần này là kiểm tra tư duy thiết kế hệ thống ở mức 2+ năm mà không phụ thuộc quá nhiều vào nghiệp vụ cụ thể. Interviewer nên giải thích ngắn bối cảnh trước khi hỏi và cho ứng viên quyền đặt câu hỏi làm rõ.

Nguyên tắc chấm:

- Không yêu cầu ứng viên biết domain LMS/contest trước.
- Ưu tiên cách ứng viên phân rã bài toán, đặt câu hỏi, nhận diện edge case và trade-off.
- Có thể đổi tên entity sang domain quen thuộc với ứng viên: task, order, booking, course, document, ticket.
- Nếu ứng viên chưa từng làm đúng use case, hỏi cách họ sẽ tiếp cận thay vì chấm rớt ngay.

## 1. Thiết kế màn hình danh sách có filter và phân quyền

### Bài toán

Thiết kế một màn hình quản trị hiển thị danh sách bản ghi. Người dùng có thể tìm kiếm, lọc, sort, phân trang và xem chi tiết. Dữ liệu thuộc nhiều tenant, mỗi user chỉ được xem dữ liệu mình có quyền.

Ví dụ entity có thể là `Course`, `User`, `Order`, `Ticket`, `Document`.

### Câu hỏi

1. Frontend và backend cần thống nhất API contract như thế nào?
2. Query params nên gồm những gì: search, filter, sort, page/cursor?
3. Backend cần validate và giới hạn filter/sort ra sao để tránh query nguy hiểm?
4. MongoDB cần index như thế nào cho search/filter/sort phổ biến?
5. Offset pagination và cursor pagination khác nhau thế nào? Khi nào chọn mỗi loại?
6. Frontend quản lý loading, empty, error, permission denied state thế nào?
7. Nếu user đổi filter liên tục, làm sao tránh request cũ ghi đè request mới?
8. Redis cache có phù hợp không? Nếu có, cache key và invalidation thế nào?

### Tín hiệu tốt

- Luôn scope query theo `tenantId` và permission từ backend.
- Không tin filter/sort raw từ client một cách mù quáng.
- Biết dùng pagination, projection, index, giới hạn page size.
- Frontend xử lý race condition, loading state và URL state tốt.
- Biết cache list query khó invalidation hơn cache detail/static metadata.

## 2. Thiết kế action có thể bị gọi trùng

### Bài toán

Thiết kế một API thực hiện action quan trọng trên một bản ghi, ví dụ:

- Mark một item là completed.
- Submit một form.
- Confirm một booking.
- Publish một document.
- Apply một coupon.

Client có thể gửi request trùng do double click, retry, mạng chập chờn hoặc mobile sync.

### Câu hỏi

1. API endpoint và request/response nên thiết kế thế nào?
2. Backend cần validate những gì trước khi đổi trạng thái?
3. Làm sao đảm bảo action idempotent?
4. Data model cần unique key, status transition hoặc audit log không?
5. Nếu action thành công rồi nhưng client timeout, request retry sẽ trả gì?
6. Frontend nên disable button, show pending state và handle retry thế nào?
7. Khi nào cần dùng transaction? MongoDB transaction có trade-off gì?
8. Có event async nào nên publish qua RabbitMQ sau khi action thành công không?

### Tín hiệu tốt

- Nhận ra duplicate request là case production rất thường gặp.
- Có idempotency key, unique constraint hoặc status transition rõ ràng.
- Không chỉ rely vào frontend disable button.
- Phân biệt trạng thái action thành công, đang xử lý, thất bại có thể retry, thất bại không nên retry.
- Event async chỉ publish sau khi state chính đã được commit.

## 3. Thiết kế upload và xử lý file

### Bài toán

Thiết kế tính năng upload file từ frontend lên hệ thống. File sau khi upload cần được lưu trữ, validate và có thể xử lý bất đồng bộ như tạo thumbnail, scan, parse metadata hoặc gửi notification.

### Câu hỏi

1. Upload file nên đi qua backend hay dùng signed URL lên S3? Trade-off là gì?
2. Backend cần validate gì: file size, type, extension, owner, tenant, quota?
3. Metadata file nên lưu trong MongoDB như thế nào?
4. Frontend hiển thị progress, cancel, retry, upload failed thế nào?
5. Xử lý hậu kỳ nên synchronous hay asynchronous?
6. RabbitMQ message nên chứa những field nào?
7. Nếu worker xử lý file fail, retry và DLQ nên thiết kế ra sao?
8. Làm sao tránh user tenant A truy cập file tenant B?
9. File URL nên public, private hay signed? Khi nào dùng CDN?

### Tín hiệu tốt

- Biết signed URL giảm tải backend nhưng vẫn cần authorization và metadata.
- Không tin MIME/type từ client hoàn toàn.
- Có trạng thái file: uploading, uploaded, processing, ready, failed.
- Có tenant/user ownership rõ ràng.
- Worker xử lý idempotent, có retry giới hạn và DLQ.

## 4. Thiết kế notification/email async

### Bài toán

Sau một action của user, hệ thống cần gửi email hoặc notification. Không muốn request chính bị chậm vì phụ thuộc email provider.

### Câu hỏi

1. Request chính nên làm gì và phần nào đưa qua queue?
2. Message gửi qua RabbitMQ nên gồm data đầy đủ hay chỉ gồm ID để worker fetch lại?
3. Làm sao đảm bảo không gửi notification trùng?
4. Retry policy nên như thế nào với lỗi tạm thời và lỗi vĩnh viễn?
5. Nếu email provider rate limit, hệ thống xử lý sao?
6. Frontend có cần biết notification đã gửi chưa không?
7. Monitoring/alert nào cần có cho queue và worker?

### Tín hiệu tốt

- Tách request chính khỏi tác vụ chậm/không ổn định.
- Có idempotency/deduplication cho notification.
- Không retry vô hạn.
- Biết dùng DLQ, backoff, alert backlog/processing latency.
- Phân biệt trạng thái business chính và trạng thái side effect.

## 5. Multi-tenant

1. Multi-tenant ảnh hưởng gì đến API design, database query và authorization?
2. Làm sao đảm bảo user tenant A không đọc được dữ liệu tenant B?
3. Frontend nên lấy và lưu tenant context như thế nào?
4. Cache Redis cần include tenantId trong key không? Vì sao?
5. Background job cần truyền tenant context như thế nào?
6. Log/monitoring nên có field nào để debug theo tenant?

Tín hiệu tốt:

- Mọi query nhạy cảm đều scope theo tenant.
- Không tin tenantId từ client một cách mù quáng.
- Cache, queue message, logs đều có tenant context.
- Có suy nghĩ về data isolation và operational support.

## 6. API contract và phối hợp frontend/backend

1. Khi backend thay đổi response shape, làm sao giảm rủi ro làm hỏng frontend?
2. API list nên trả pagination metadata gì?
3. Error response nên có structure gì để frontend render đúng?
4. Frontend có nên tự ghép nhiều API để dựng page không? Khi nào nên tạo BFF/aggregate endpoint?
5. Làm sao xử lý optimistic update khi API có thể fail?
6. Làm sao version API hoặc rollout thay đổi lớn?

Tín hiệu tốt:

- Có contract rõ cho success/error/pagination.
- Biết giữ backward compatibility hoặc rollout theo version/feature flag.
- Frontend không phụ thuộc vào message lỗi khó parse.
- Biết khi nào cần aggregate endpoint để giảm waterfall hoặc tránh lộ logic phức tạp ra client.

## 7. Performance end-to-end

1. Một trang dashboard load chậm, em debug từ frontend đến backend như thế nào?
2. Làm sao phân biệt chậm do network, frontend render, API, database hay third-party?
3. Những metric nào nên đo ở frontend?
4. Những metric nào nên đo ở backend?
5. Nếu API trả payload rất lớn, em xử lý ra sao?
6. Nếu MongoDB query nhanh nhưng page vẫn chậm, em kiểm tra gì?
7. Nếu Redis cache hit rate thấp, em kiểm tra gì?

Tín hiệu tốt:

- Biết dùng browser performance/network tools.
- Biết đo từng đoạn thay vì đoán.
- Có strategy pagination, projection, lazy loading, caching.
- Biết tránh waterfall request và unnecessary render.
- Biết payload size và render cost cũng có thể là bottleneck.

## 8. Reliability và incident thinking

1. Production vừa deploy xong bị lỗi login diện rộng. Em làm gì trong 10 phút đầu?
2. RabbitMQ queue backlog tăng liên tục. Em kiểm tra gì?
3. Redis latency tăng, API bắt đầu timeout. Em xử lý thế nào?
4. Một migration MongoDB chạy lâu gây ảnh hưởng production. Em sẽ làm gì?
5. Làm sao viết một postmortem hữu ích sau incident?

Tín hiệu tốt:

- Ưu tiên giảm tác động cho user trước.
- Biết rollback/disable feature flag nếu có.
- Biết xem log, metric, recent changes.
- Không chỉ fix code, còn nghĩ đến prevention.

## 9. Ownership và nhiệt huyết

1. Feature nào em tự hào nhất trong 6-12 tháng gần đây? Vì sao?
2. Có lần nào em chủ động cải thiện codebase/process mà không được giao trực tiếp không?
3. Khi review code của người khác, em thường chú ý điều gì?
4. Khi bị review nhiều comment, em phản ứng thế nào?
5. Em học công nghệ mới bằng cách nào? Gần đây học gì và áp dụng ở đâu?
6. Nếu được vào team, trong 30 ngày đầu em muốn hiểu những gì?

Tín hiệu tốt:

- Có ví dụ cụ thể, không nói chung chung.
- Có tinh thần ownership nhưng không ôm việc thiếu phối hợp.
- Biết nhận feedback và điều chỉnh.
- Có động lực học vì giải quyết vấn đề thật, không chỉ chạy theo trend.


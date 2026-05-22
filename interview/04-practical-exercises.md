# Practical Exercises

Các bài dưới đây có thể dùng cho live coding 45-60 phút hoặc take-home 3-6 giờ. Nên chọn một bài phù hợp với seniority và phần cần kiểm chứng.

## 1. Live coding backend: Course progress API

### Yêu cầu

Thiết kế và implement API:

- `POST /courses/:courseId/lessons/:lessonId/complete`
- Ghi nhận một học viên đã hoàn thành bài học.
- Không tạo dữ liệu trùng nếu user gửi request nhiều lần.
- Trả về progress hiện tại của khóa học.

### Có thể đơn giản hóa

- Dùng in-memory array nếu không muốn setup MongoDB trong buổi live.
- Có thể pseudo-code repository nếu mục tiêu là đánh giá tư duy.

### Cần quan sát

- Có validate user đã enroll course không.
- Có kiểm tra lesson thuộc course không.
- Có xử lý duplicate request không.
- Có tách controller/business logic hợp lý không.
- Có nghĩ đến tenant scope không.
- Có test case hoặc ít nhất nêu test case chính không.

### Follow-up

1. Nếu course có 5.000 lessons, tính progress thế nào cho nhanh?
2. Nếu cần phát event khi hoàn thành toàn khóa, em làm gì?
3. Nếu API bị gọi từ mobile offline sync và có request cũ gửi lại, xử lý sao?

## 2. Live coding frontend: Course progress UI

### Yêu cầu

Tạo component hiển thị danh sách lesson và progress:

- Loading, error, empty state.
- Lesson đã hoàn thành/chưa hoàn thành.
- Button mark complete.
- Optimistic update hoặc pessimistic update đều được, nhưng phải giải thích trade-off.

### Cần quan sát

- State đặt đúng chỗ.
- Không mutate state trực tiếp.
- Có xử lý request pending/double click.
- Có rollback hoặc error handling nếu optimistic update fail.
- UI không phụ thuộc vào dữ liệu luôn hoàn hảo.

### Follow-up

1. Nếu API chậm 3 giây, UX nên thế nào?
2. Nếu user không có quyền học course, frontend và backend xử lý ra sao?
3. Nếu component được dùng ở nhiều page, em tách như thế nào?

## 3. Debugging exercise: API chậm

### Bối cảnh

Endpoint `GET /courses/:id/students` đang chậm ở production. Khi course có nhiều học viên, page mất 8-12 giây mới load.

### Hỏi ứng viên

1. Em cần thêm thông tin gì?
2. Em kiểm tra frontend hay backend trước?
3. Backend cần log/metric gì?
4. MongoDB query cần kiểm tra gì?
5. API response có thể tối ưu gì?
6. Frontend render table lớn có thể tối ưu gì?
7. Giải pháp tạm thời và dài hạn là gì?

### Tín hiệu tốt

- Không vội kết luận.
- Biết chia nhỏ thời gian: browser, network, API, DB, render.
- Nhắc đến pagination, projection, index, query explain.
- Nhắc đến virtualization nếu table rất lớn.

## 4. Debugging exercise: Queue backlog

### Bối cảnh

RabbitMQ queue xử lý email certificate tăng backlog từ vài trăm lên vài chục nghìn message sau một campaign.

### Hỏi ứng viên

1. Em kiểm tra gì đầu tiên?
2. Làm sao biết bottleneck ở producer, broker, consumer hay email provider?
3. Có nên scale consumer ngay không?
4. Retry policy có thể gây vấn đề gì?
5. Làm sao tránh gửi email trùng?
6. Sau incident, em cải thiện gì?

### Tín hiệu tốt

- Kiểm tra rate publish/consume, consumer error, retry loop, DLQ.
- Có idempotency khi gửi email.
- Không scale mù nếu bottleneck là provider rate limit.
- Có alert backlog và processing latency.

## 5. Take-home: Mini LMS module

### Yêu cầu

Xây dựng một module nhỏ gồm:

- Trang danh sách khóa học.
- Trang chi tiết khóa học gồm danh sách bài học.
- API lấy course detail và update lesson progress.
- Dữ liệu có thể dùng MongoDB thật hoặc mock repository.
- Có README hướng dẫn chạy.

### Kỳ vọng

- Không cần UI cầu kỳ, nhưng phải rõ trạng thái loading/error/empty.
- Code rõ ràng, dễ đọc, không over-engineering.
- Có ít nhất vài test hoặc mô tả test case nếu không đủ thời gian.
- Có xử lý authorization/tenant ở mức mô phỏng.

### Tiêu chí chấm

- Correctness: feature chạy đúng yêu cầu.
- Code structure: dễ hiểu, đúng trách nhiệm.
- Error handling: xử lý lỗi API/form/state.
- Data modeling: có suy nghĩ về progress và duplicate.
- UX: trạng thái giao diện đầy đủ.
- Communication: README rõ, biết nêu trade-off.

## 6. Take-home: Refactor and review

### Yêu cầu

Đưa ứng viên một đoạn code cố tình có vấn đề:

- Controller chứa quá nhiều business logic.
- Query MongoDB thiếu tenant filter.
- API không pagination.
- Frontend mutate Redux state trực tiếp.
- Component thiếu loading/error state.

Yêu cầu ứng viên:

- Review vấn đề.
- Refactor một phần quan trọng.
- Viết ngắn gọn lý do thay đổi.

### Cần quan sát

- Có phát hiện lỗi security/tenant không.
- Có ưu tiên vấn đề nghiêm trọng trước không.
- Có refactor vừa đủ không.
- Có giữ backward compatibility không.


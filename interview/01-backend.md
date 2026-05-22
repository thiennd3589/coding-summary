# Backend Interview Questions

## 1. Kinh nghiệm dự án thật

1. Hãy mô tả một API/backend feature em trực tiếp thiết kế hoặc implement gần đây nhất.
   - Follow-up: request flow đi qua những layer nào?
   - Follow-up: em chịu trách nhiệm phần nào, phần nào do người khác làm?
   - Tín hiệu tốt: nói rõ controller/service/repository/model, validation, error handling, test, deploy.

2. Một bug backend khó nhất em từng debug là gì?
   - Follow-up: em phát hiện từ log, report của user, monitoring hay test?
   - Follow-up: root cause là gì và fix có ảnh hưởng gì đến dữ liệu cũ không?
   - Tín hiệu tốt: có quy trình điều tra, biết khoanh vùng, có prevention sau fix.

3. Khi nhận một requirement backend chưa rõ, em thường làm gì trước khi code?
   - Tín hiệu tốt: hỏi về input/output, permission, tenant, edge case, backward compatibility, migration.

## 2. LoopBack 4

1. Trong LoopBack, controller, service, repository và model nên chia trách nhiệm như thế nào?
   - Tín hiệu tốt: controller mỏng, business logic ở service/use-case, repository lo data access.

2. Dependency injection trong LoopBack dùng để giải quyết vấn đề gì?
   - Follow-up: em từng inject repository/service/config/current user như thế nào?

3. Em xử lý validation request trong LoopBack ra sao?
   - Follow-up: validation nào nên ở schema/model, validation nào nên ở business logic?

4. Khi cần thêm một API mới có authorization theo role/tenant, em sẽ thiết kế các bước nào?
   - Tín hiệu tốt: authentication, current user profile, tenant scope, permission guard, query filter.

5. Em từng gặp vấn đề gì với repository pattern hoặc relation trong LoopBack chưa?
   - Follow-up: cách xử lý N+1 query, include relation, pagination.

6. Nếu một endpoint đang chậm, em sẽ kiểm tra theo thứ tự nào?
   - Tín hiệu tốt: log duration từng layer, query plan/index, external calls, serialization, payload size.

## 3. MongoDB

1. Khi thiết kế schema MongoDB, khi nào em embed document, khi nào em reference?
   - Tín hiệu tốt: dựa trên access pattern, cardinality, update frequency, document size.

2. Index trong MongoDB hoạt động thế nào? Khi nào index có thể làm hệ thống chậm hơn?
   - Follow-up: compound index nên sắp xếp field theo logic nào?

3. Làm sao debug một query MongoDB chậm?
   - Tín hiệu tốt: explain plan, scan count, index usage, projection, pagination strategy.

4. Offset pagination và cursor pagination khác nhau thế nào?
   - Follow-up: trong feed/course list có dữ liệu lớn, em chọn cách nào?

5. Em xử lý migration dữ liệu MongoDB như thế nào để không làm downtime?
   - Tín hiệu tốt: backward compatible code, batch migration, idempotent script, monitoring.

6. Multi-tenant với MongoDB có những cách lưu dữ liệu nào?
   - Tín hiệu tốt: tenantId field, database per tenant, collection per tenant, trade-off isolation/cost/ops.

## 4. Redis

1. Em đã dùng Redis cho use case nào: cache, session, rate limit, lock, queue?
   - Follow-up: TTL, invalidation và cache key được thiết kế thế nào?

2. Cache invalidation thường khó ở điểm nào?
   - Tín hiệu tốt: stale data, write-through/write-around, event invalidation, versioned key.

3. Nếu Redis bị down, hệ thống nên hành xử thế nào?
   - Tín hiệu tốt: degrade gracefully, fallback DB, timeout ngắn, circuit breaker nếu cần.

4. Distributed lock bằng Redis có rủi ro gì?
   - Tín hiệu tốt: TTL, lock ownership, clock/time issue, duplicate execution, idempotency.

5. Em thiết kế rate limit login/API như thế nào?
   - Follow-up: key theo IP/user/tenant, window algorithm, response khi limit exceeded.

## 5. RabbitMQ

1. Khi nào nên dùng message queue thay vì xử lý synchronous trong request?
   - Tín hiệu tốt: long-running jobs, retry, decoupling, load smoothing, eventual consistency.

2. Producer, exchange, queue, binding, consumer là gì?
   - Follow-up: direct/topic/fanout exchange khác nhau thế nào?

3. Làm sao đảm bảo job không bị xử lý mất dữ liệu khi consumer crash?
   - Tín hiệu tốt: ack/nack, durable queue, persistent message, retry, dead-letter queue.

4. Message bị xử lý trùng thì sao?
   - Tín hiệu tốt: idempotency key, unique constraint, status transition, deduplication.

5. Em thiết kế retry policy cho email/payment/report generation như thế nào?
   - Tín hiệu tốt: retry có giới hạn, backoff, DLQ, alert, không retry lỗi validation.

## 6. AWS và vận hành

1. Em đã từng deploy hoặc vận hành service trên AWS nào?
   - Follow-up: EC2, ECS, Lambda, S3, CloudFront, RDS/DocumentDB, ElastiCache, SQS/SNS, CloudWatch.

2. Một API production lỗi 500 tăng đột biến, em kiểm tra gì đầu tiên?
   - Tín hiệu tốt: recent deploy, logs, metrics, dependency health, DB/Redis/RabbitMQ, rollback plan.

3. Em lưu file upload trên AWS như thế nào?
   - Tín hiệu tốt: S3, signed URL, content type/size validation, virus scan nếu cần, CDN, permission.

4. Secret/config nên quản lý ra sao giữa local/staging/production?
   - Tín hiệu tốt: env vars, secret manager, không commit secret, rotation.

5. Làm sao giảm rủi ro khi deploy backend?
   - Tín hiệu tốt: migration compatible, health check, canary/rolling deploy, rollback, monitoring.

## 7. Security và API quality

1. Authentication và authorization khác nhau thế nào?
2. Em xử lý permission theo role/tenant/resource owner như thế nào?
3. Các lỗi security phổ biến trong REST API là gì?
   - Tín hiệu tốt: broken auth, IDOR, injection, mass assignment, rate limit, sensitive logs.
4. API error response nên thiết kế ra sao để frontend dễ xử lý?
5. Khi nào nên dùng transaction? MongoDB transaction có trade-off gì?


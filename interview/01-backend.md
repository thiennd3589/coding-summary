# Backend Interview Questions

Mục tiêu phần backend là tìm ứng viên có kinh nghiệm xây dựng và vận hành API production, không chỉ biết một framework cụ thể. Nếu ứng viên chưa từng dùng LoopBack, vẫn có thể đánh giá tốt qua Express, NestJS, Spring, Laravel, Django, Rails hoặc stack backend khác.

Nguyên tắc hỏi:

- Hỏi framework-neutral trước, framework-specific sau.
- Với mỗi câu trả lời, hỏi tiếp bằng ví dụ thật: "Em đã gặp case này chưa?", "Khi đó em làm gì?", "Kết quả ra sao?".
- Không chấm thấp chỉ vì ứng viên chưa biết LoopBack, trừ khi vị trí cần vào làm LoopBack ngay và không có thời gian ramp-up.
- Ưu tiên ứng viên hiểu request lifecycle, data consistency, security, observability và trade-off.

## 1. Kinh nghiệm dự án thật

1. Hãy mô tả một API/backend feature em trực tiếp thiết kế hoặc implement gần đây nhất.
   - Follow-up: request flow đi qua những layer nào?
   - Follow-up: em chịu trách nhiệm phần nào, phần nào do người khác làm?
   - Tín hiệu tốt: nói rõ route/controller, service/use-case, repository/data access, validation, error handling, test, deploy.

2. Một bug backend khó nhất em từng debug là gì?
   - Follow-up: em phát hiện từ log, report của user, monitoring hay test?
   - Follow-up: root cause là gì và fix có ảnh hưởng gì đến dữ liệu cũ không?
   - Tín hiệu tốt: có quy trình điều tra, biết khoanh vùng, có prevention sau fix.

3. Một lần em phải thay đổi data model hoặc API contract là gì?
   - Follow-up: làm sao tránh làm hỏng client cũ?
   - Tín hiệu tốt: backward compatibility, migration plan, versioning, rollout từng bước.

4. Khi nhận một requirement backend chưa rõ, em thường làm gì trước khi code?
   - Tín hiệu tốt: hỏi về input/output, permission, tenant, edge case, backward compatibility, migration, expected scale.

## 2. API design và request lifecycle

1. Một request đi từ client đến database thường qua những bước nào trong backend?
   - Tín hiệu tốt: routing, auth, validation, business logic, data access, transaction, response mapping, logging.

2. Controller/handler, service/use-case và repository/data access nên chia trách nhiệm như thế nào?
   - Tín hiệu tốt: handler mỏng, business logic ở service/use-case, data access tách riêng, không để logic nghiệp vụ rải rác.

3. Em thiết kế REST API tạo/sửa/xóa/list một resource như thế nào?
   - Follow-up: status code, response body, pagination, filter, sort, validation error.

4. API error response nên thiết kế ra sao để frontend dễ xử lý?
   - Tín hiệu tốt: stable error code, message cho user/dev, field errors, correlation/request id.

5. Làm sao xử lý backward compatibility khi API response cần đổi field?
   - Tín hiệu tốt: add field trước, deprecate sau, versioning nếu cần, contract tests, rollout có monitoring.

6. Khi nào nên tạo aggregate endpoint/BFF thay vì để frontend gọi nhiều API nhỏ?
   - Tín hiệu tốt: giảm waterfall, gom permission logic, tối ưu payload, tránh overfetching/underfetching.

## 3. Validation, business rules và consistency

1. Validation nào nên ở request schema, validation nào thuộc business logic?
   - Tín hiệu tốt: type/format/range ở schema, rule phụ thuộc DB/permission/state ở business layer.

2. Thiết kế API submit một action có thể bị gọi trùng do retry/double click như thế nào?
   - Tín hiệu tốt: idempotency key, unique constraint, status transition, không chỉ rely vào frontend disable button.

3. Nếu request thành công ở backend nhưng client timeout và retry, API nên trả gì?
   - Tín hiệu tốt: trả trạng thái hiện tại hoặc kết quả cũ theo idempotency key.

4. Khi nào cần transaction? Nếu database không hỗ trợ transaction tốt, em xử lý consistency thế nào?
   - Tín hiệu tốt: atomic update, status machine, outbox pattern, compensating action, idempotent retry.

5. Làm sao tránh race condition khi nhiều request cùng cập nhật một resource?
   - Tín hiệu tốt: optimistic locking/version, atomic operation, unique index, transaction, queue serialize nếu cần.

6. Em thiết kế audit log cho action quan trọng như thế nào?
   - Tín hiệu tốt: actor, resource, action, before/after hoặc diff, timestamp, request id, tenant id.

## 4. Authentication, authorization và security

1. Authentication và authorization khác nhau thế nào?
2. Em xử lý permission theo role, tenant và resource owner như thế nào?
3. IDOR là gì? Làm sao tránh user đọc/sửa resource không thuộc quyền của mình?
4. Các lỗi security phổ biến trong REST API là gì?
   - Tín hiệu tốt: broken auth, injection, mass assignment, insecure direct object reference, rate limit, sensitive logs.

5. Token/session nên lưu và rotate như thế nào?
   - Follow-up: access token, refresh token, httpOnly cookie, logout, revoke token.

6. Em thiết kế rate limit login/API như thế nào?
   - Follow-up: key theo IP/user/tenant, window algorithm, response khi limit exceeded.

7. Làm sao tránh log lộ thông tin nhạy cảm?
   - Tín hiệu tốt: mask token/password/PII, structured logs, log level phù hợp.

## 5. Database và MongoDB

1. Khi thiết kế schema document database, khi nào embed document, khi nào reference?
   - Tín hiệu tốt: dựa trên access pattern, cardinality, update frequency, document size.

2. Index hoạt động thế nào? Khi nào index có thể làm hệ thống chậm hơn?
   - Follow-up: compound index nên sắp xếp field theo logic nào?

3. Làm sao debug một query chậm?
   - Tín hiệu tốt: explain plan, scan count, index usage, projection, pagination strategy.

4. Offset pagination và cursor pagination khác nhau thế nào?
   - Follow-up: list dữ liệu lớn có sort/filter thì chọn cách nào?

5. Em xử lý migration dữ liệu production như thế nào để không downtime?
   - Tín hiệu tốt: backward compatible code, batch migration, idempotent script, monitoring, rollback plan.

6. Multi-tenant với database có những cách lưu dữ liệu nào?
   - Tín hiệu tốt: tenantId field, database per tenant, collection/table per tenant, trade-off isolation/cost/ops.

7. Nếu cần soft delete, em thiết kế query/index/unique constraint như thế nào?
   - Tín hiệu tốt: deletedAt/deletedBy, partial unique index nếu phù hợp, audit và restore policy.

## 6. Redis và caching

1. Em đã dùng Redis cho use case nào: cache, session, rate limit, lock, queue?
   - Follow-up: TTL, invalidation và cache key được thiết kế thế nào?

2. Cache invalidation thường khó ở điểm nào?
   - Tín hiệu tốt: stale data, write-through/write-around, event invalidation, versioned key.

3. Khi nào không nên cache?
   - Tín hiệu tốt: dữ liệu thay đổi liên tục, permission phức tạp, invalidation đắt hơn query, dữ liệu nhạy cảm.

4. Nếu Redis bị down, hệ thống nên hành xử thế nào?
   - Tín hiệu tốt: degrade gracefully, fallback DB, timeout ngắn, circuit breaker nếu cần.

5. Distributed lock bằng Redis có rủi ro gì?
   - Tín hiệu tốt: TTL, lock ownership, clock/time issue, duplicate execution, idempotency.

6. Cache key trong hệ thống multi-tenant cần chú ý gì?
   - Tín hiệu tốt: include tenant/user/permission context khi cần, tránh leak dữ liệu giữa tenant.

## 7. Queue, async jobs và RabbitMQ

1. Khi nào nên dùng message queue thay vì xử lý synchronous trong request?
   - Tín hiệu tốt: long-running jobs, retry, decoupling, load smoothing, eventual consistency.

2. Producer, exchange, queue, binding, consumer là gì?
   - Follow-up: direct/topic/fanout exchange khác nhau thế nào?

3. Làm sao đảm bảo job không bị mất dữ liệu khi consumer crash?
   - Tín hiệu tốt: ack/nack, durable queue, persistent message, retry, dead-letter queue.

4. Message bị xử lý trùng thì sao?
   - Tín hiệu tốt: idempotency key, unique constraint, status transition, deduplication.

5. Em thiết kế retry policy cho email/payment/report generation như thế nào?
   - Tín hiệu tốt: retry có giới hạn, backoff, DLQ, alert, không retry lỗi validation.

6. Nếu queue backlog tăng liên tục, em kiểm tra gì?
   - Tín hiệu tốt: publish rate, consume rate, worker error, provider rate limit, retry loop, resource saturation.

7. Khi action chính thành công nhưng publish message fail thì xử lý thế nào?
   - Tín hiệu tốt: outbox pattern, retry publisher, transaction boundary rõ, reconciliation job.

## 8. File upload và external services

1. Upload file nên đi qua backend hay dùng signed URL lên S3? Trade-off là gì?
2. Backend cần validate gì trước/sau khi upload file?
   - Tín hiệu tốt: size, content type, extension, owner, tenant, quota, malware scan nếu cần.

3. Metadata file nên lưu thế nào?
   - Tín hiệu tốt: storage key, owner, tenant, status, size, checksum, createdAt, access policy.

4. Khi gọi third-party API chậm hoặc lỗi, backend nên xử lý gì?
   - Tín hiệu tốt: timeout, retry có giới hạn, circuit breaker, fallback, async processing, observability.

5. Làm sao tránh duplicate side effect khi retry third-party call?
   - Tín hiệu tốt: provider idempotency key nếu có, local idempotency, reconciliation.

## 9. Observability, debugging và performance

1. Một API production lỗi 500 tăng đột biến, em kiểm tra gì đầu tiên?
   - Tín hiệu tốt: recent deploy, logs, metrics, dependency health, DB/Redis/RabbitMQ, rollback plan.

2. Một endpoint đang chậm, em sẽ kiểm tra theo thứ tự nào?
   - Tín hiệu tốt: request duration từng layer, query plan/index, external calls, serialization, payload size.

3. Log tốt cho backend cần có những field nào?
   - Tín hiệu tốt: timestamp, level, request id, user id, tenant id, route, status, duration, error code.

4. Metric nào nên theo dõi cho API service?
   - Tín hiệu tốt: latency p50/p95/p99, error rate, throughput, saturation, DB latency, queue depth.

5. Làm sao debug memory leak hoặc CPU cao ở backend?
   - Tín hiệu tốt: reproduce, profiling, heap snapshot, dependency check, recent changes, workload pattern.

6. Nếu API trả payload rất lớn, em tối ưu thế nào?
   - Tín hiệu tốt: pagination, projection, compression, aggregate endpoint, streaming nếu phù hợp.

## 10. AWS và vận hành

1. Em đã từng deploy hoặc vận hành service trên AWS nào?
   - Follow-up: EC2, ECS, Lambda, S3, CloudFront, RDS/DocumentDB, ElastiCache, SQS/SNS, CloudWatch.

2. Secret/config nên quản lý ra sao giữa local/staging/production?
   - Tín hiệu tốt: env vars, secret manager, không commit secret, rotation.

3. Làm sao giảm rủi ro khi deploy backend?
   - Tín hiệu tốt: migration compatible, health check, canary/rolling deploy, rollback, monitoring.

4. Health check tốt nên kiểm tra gì và không nên kiểm tra gì?
   - Tín hiệu tốt: readiness/liveness, dependency check có timeout, không làm query nặng.

5. Nếu deploy xong phát hiện bug nghiêm trọng, em quyết định rollback hay hotfix dựa trên gì?
   - Tín hiệu tốt: mức độ ảnh hưởng, thời gian fix, data corruption risk, rollback safety, communication.

## 11. Framework-specific optional: LoopBack 4

Chỉ dùng phần này nếu muốn kiểm tra khả năng match trực tiếp với stack hiện tại. Nếu ứng viên chưa dùng LoopBack, có thể hỏi họ mapping sang framework đã dùng.

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


# Interview Scorecard

Scorecard này dùng để chấm nhất quán giữa nhiều interviewer. Nên ghi bằng chứng cụ thể, không chỉ ghi cảm tính.

## Thang điểm

- 1: Yếu, không đạt kỳ vọng 2+ năm.
- 2: Có nền tảng nhưng thiếu kinh nghiệm thực tế hoặc cần kèm nhiều.
- 3: Đạt kỳ vọng, có thể làm việc độc lập với task vừa phải.
- 4: Tốt, hiểu trade-off, xử lý vấn đề thực tế vững.
- 5: Rất tốt, có chiều sâu, ownership mạnh, có thể dẫn dắt phần việc.

## Rubric tổng quát

| Năng lực | 1-2 điểm | 3 điểm | 4-5 điểm |
| --- | --- | --- | --- |
| Backend | Nắm API/framework hời hợt, khó giải thích code mình làm | Thiết kế API, data access, validation cơ bản ổn | Hiểu performance, consistency, queue/cache, failure handling |
| Frontend | Chỉ làm UI theo mẫu, yếu state/error handling | Làm component/page hoàn chỉnh, biết quản lý state | Tối ưu UX/performance, hiểu SSR/CSR/cache/security |
| Database | Chưa hiểu index/schema/query trade-off | Biết schema, index, pagination cơ bản | Debug query tốt, hiểu migration, multi-tenant, scale |
| System design | Thiết kế rời rạc, bỏ qua edge case | Có flow end-to-end hợp lý | Biết trade-off, idempotency, async, observability |
| Debugging | Đoán lỗi, thiếu quy trình | Biết tái hiện, đọc log, kiểm tra từng phần | Khoanh vùng nhanh, biết metric, prevention sau fix |
| Ownership | Chờ giao việc, ít phản biện | Chủ động trong task được giao | Chủ động cải thiện hệ thống/process, giao tiếp rõ |
| Communication | Trả lời mơ hồ, khó làm rõ vai trò | Diễn đạt rõ phần mình làm | Biết đặt câu hỏi, giải thích trade-off, nhận feedback tốt |

## Câu hỏi bắt buộc nên có

1. Feature gần nhất em trực tiếp làm là gì? Hãy mô tả end-to-end.
2. Bug production hoặc bug khó nhất em từng xử lý là gì?
3. Một quyết định kỹ thuật em từng phải trade-off là gì?
4. Nếu API/page chậm, em debug theo thứ tự nào?
5. Em từng chủ động cải thiện điều gì trong dự án?

## Tín hiệu kinh nghiệm thật

- Nhớ được dữ liệu, flow, edge case, lỗi đã gặp.
- Nói được lý do chọn giải pháp, không chỉ nói "team em dùng vậy".
- Biết phần mình làm và phần mình không làm.
- Có ví dụ về deploy, monitoring, rollback, migration hoặc incident.
- Biết user/business impact của feature hoặc bug.

## Tín hiệu có thể đang phóng đại kinh nghiệm

- Không giải thích được feature mình nói là owner.
- Chỉ trả lời bằng keyword: "Redis cache", "RabbitMQ async", "Next SSR" nhưng không nêu trade-off.
- Không biết bug nào đáng nhớ dù nói đã làm production lâu.
- Không phân biệt được lỗi do frontend/backend/database.
- Né tránh câu hỏi follow-up bằng câu trả lời chung chung.

## Tín hiệu nhiệt huyết đúng nghĩa

- Có dự án/cải tiến tự chủ động làm.
- Theo dõi chất lượng sản phẩm sau khi release.
- Muốn hiểu domain, user, và metric thành công.
- Có thói quen học qua việc giải quyết vấn đề thật.
- Nhận feedback bình tĩnh, hỏi lại để hiểu đúng.

## Red flags

- Không quan tâm security/authorization khi nói về multi-tenant.
- Cho rằng frontend check permission là đủ.
- Không xử lý duplicate/retry khi nói về queue/payment/progress.
- Không biết cách debug ngoài việc "console.log".
- Không viết hoặc không nghĩ đến test cho logic quan trọng.
- Đổ lỗi cá nhân khi nói về incident hoặc conflict.

## Gợi ý kết luận sau phỏng vấn

### Strong hire

- Điểm trung bình 4+.
- Có bằng chứng kinh nghiệm production.
- Có ownership và giao tiếp rõ.
- Có thể tự xử lý feature fullstack vừa/lớn.

### Hire

- Điểm trung bình khoảng 3.
- Có nền tảng thực tế, thiếu một vài mảng nhưng học được.
- Phù hợp nếu team có mentoring/review tốt.

### Lean no hire

- Điểm trung bình dưới 3.
- Trả lời thiếu chiều sâu, cần kèm nhiều.
- Chưa đủ chắc cho vị trí 2+ năm nếu team cần người tự chủ.

### No hire

- Không chứng minh được kinh nghiệm đã nêu.
- Bỏ qua vấn đề bảo mật/dữ liệu nghiêm trọng.
- Giao tiếp khó rõ trách nhiệm hoặc thiếu trung thực.

## Mẫu ghi chú phỏng vấn

```text
Candidate:
Role level:
Interviewer:
Date:

Backend score:
Frontend score:
Database score:
System design score:
Debugging score:
Ownership score:
Communication score:

Evidence:
- 

Concerns:
- 

Recommendation:
Strong hire / Hire / Lean no hire / No hire
```


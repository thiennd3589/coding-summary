# Frontend Interview Questions

## 1. Kinh nghiệm dự án thật

1. Hãy mô tả một feature frontend em trực tiếp làm từ đầu đến cuối.
   - Follow-up: em xử lý data fetching, loading, error, empty state, permission như thế nào?
   - Tín hiệu tốt: nói rõ component structure, state ownership, API contract, UX edge cases.

2. Một bug frontend khó nhất em từng debug là gì?
   - Follow-up: bug do state, render, browser, API, cache hay CSS?
   - Tín hiệu tốt: biết dùng DevTools, React DevTools, network tab, reproduction steps.

3. Khi nhận design/requirement chưa rõ, em làm gì trước khi implement?
   - Tín hiệu tốt: hỏi state, responsive, validation, permission, error case, data shape.

## 2. React

1. Khi nào component re-render? Làm sao phát hiện re-render không cần thiết?
   - Tín hiệu tốt: state/props/context change, React DevTools profiler, memoization đúng chỗ.

2. `useEffect` thường bị dùng sai ở đâu?
   - Follow-up: dependency array, cleanup, derived state, race condition khi fetch.

3. Controlled và uncontrolled component khác nhau thế nào?
   - Follow-up: form lớn nên chọn cách nào, vì sao?

4. Khi nào dùng `useMemo`, `useCallback`, `React.memo`?
   - Tín hiệu tốt: không lạm dụng, dùng khi có chi phí render/tính toán hoặc reference stability cần thiết.

5. Em thiết kế component reusable như thế nào để không bị over-engineering?
   - Tín hiệu tốt: bắt đầu từ use case thật, API đơn giản, composition, tránh props quá nhiều.

6. Error boundary giải quyết vấn đề gì? Nó không bắt được loại lỗi nào?

## 3. Next.js

1. SSR, SSG, ISR và CSR khác nhau thế nào? Khi nào chọn mỗi loại?
   - Tín hiệu tốt: SEO, personalization, freshness, performance, infra cost.

2. Với Next.js App Router, server component và client component khác nhau thế nào?
   - Follow-up: khi nào bắt buộc phải dùng `"use client"`?

3. Em xử lý authentication trong Next.js như thế nào?
   - Tín hiệu tốt: cookie/httpOnly, middleware/server-side guard, refresh token, tránh lưu token nhạy cảm trong localStorage nếu có lựa chọn tốt hơn.

4. Data fetching ở server và client khác nhau thế nào về cache, security và UX?

5. Em tối ưu performance page Next.js như thế nào?
   - Tín hiệu tốt: bundle analysis, dynamic import, image optimization, cache, avoid waterfall, Core Web Vitals.

6. Khi route cần permission theo tenant/role, frontend nên xử lý gì và backend vẫn phải xử lý gì?
   - Tín hiệu tốt: frontend để UX/navigation, backend là source of truth.

## 4. Redux và state management

1. Khi nào nên dùng Redux thay vì local state hoặc React Query/SWR?
   - Tín hiệu tốt: shared client state phức tạp, workflow nhiều màn hình, predictable updates.

2. Redux store nên chứa dữ liệu gì và không nên chứa dữ liệu gì?
   - Tín hiệu tốt: tránh duplicate server cache nếu đã có query library, tránh non-serializable data.

3. Selector dùng để làm gì? Memoized selector có lợi ích gì?

4. Em tổ chức async flow với Redux Toolkit như thế nào?
   - Follow-up: loading/error state, cancellation, stale request, optimistic update.

5. Một bug do state bị stale hoặc race condition, em xử lý thế nào?

## 5. Forms, validation và UX

1. Em thiết kế form tạo khóa học/bài học có nhiều field như thế nào?
   - Tín hiệu tốt: validation schema, dirty state, autosave nếu cần, error summary, permission.

2. Validation nào nên ở frontend, validation nào bắt buộc backend kiểm tra lại?

3. Làm sao xử lý submit double click hoặc request trùng?
   - Tín hiệu tốt: disable pending, idempotency nếu cần, optimistic/pessimistic UI.

4. Empty state, loading state và error state tốt cần có gì?

5. Em xử lý responsive layout cho dashboard/table/filter như thế nào?

## 6. Frontend security và quality

1. XSS là gì? React giúp gì và không giúp gì?
   - Follow-up: khi nào `dangerouslySetInnerHTML` nguy hiểm?

2. CSRF là gì? Khi dùng cookie auth cần chú ý gì?

3. Làm sao tránh lộ thông tin nhạy cảm ở frontend logs, source map, localStorage?

4. Em viết test frontend ở những mức nào?
   - Tín hiệu tốt: unit component, integration flow, e2e cho critical path, mock API hợp lý.

5. Khi production có bug UI chỉ xảy ra với một user/tenant, em debug thế nào?


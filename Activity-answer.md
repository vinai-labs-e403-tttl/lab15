# Trả lời Activity-question.md - FlowBot VinBus Route Chatbot

## 1. Mục tiêu cuối ngày của nhóm

Đến cuối ngày, nhóm hoàn thành:

1. Worksheet 0-5 dựa trên agent FlowBot trong repo.
2. Mini project proposal 7 slide cho FlowBot.
3. Kết luận về deployment, cost, reliability và Track Phase 2.
4. Phiếu peer review cho các đội khác trong zone.

## 2. Checklist chuẩn bị đầu ngày

- [x] Nhóm có 5 thành viên.
- [x] Chủ đề đã chọn: FlowBot - chatbot tìm tuyến bus VinBus bằng ngôn ngữ tự nhiên.
- [x] Vai trò đề xuất:
  - Product lead: phụ trách user pain, value proposition, proposal flow.
  - Architect: phụ trách FastAPI, LangGraph, frontend/backend architecture.
  - Cost lead: ước lượng token/API/compute/logging/human review.
  - Reliability lead: phụ trách timeout, fallback, monitoring, route correctness.
  - Presenter: gom nội dung thành slide/poster và thuyết trình.
- [x] Nơi ghi chép chung: file markdown/slide chung từ repo.

## 3. Buổi sáng - hoạt động và đầu ra

### 3.1. Mini reflection

3-5 ý tổng hợp về hiện trạng năng lực:

- Nhóm đã build được prototype end-to-end: React frontend -> FastAPI backend -> LangGraph agent -> OpenAI tool calling -> Google Maps Transit API.
- Nhóm mạnh về AI application và rapid prototyping: có chat UI, API `/chat`, conversation id, system prompt và tool route thật.
- Nhóm có nền tảng về product thinking: đã xác định user pain, route precision, correction rate, failure modes và ROI trong spec.
- Điểm yếu hiện tại là production readiness: CORS đang mở `*`, conversation store còn in-memory, chưa có logging/monitoring đầy đủ, API response backend và frontend chưa đồng bộ route cards.
- Hướng muốn đi sâu: deployment enterprise, cost control, reliability cho provider timeout và dữ liệu route thay đổi.

## Worksheet 0 - Learning Timeline

### 1. Kỹ năng nhóm tự tin nhất

- Xây AI agent có tool calling bằng LangGraph và GPT-4o-mini.
- Tích hợp API bên ngoài, cụ thể là Google Maps Routes API v2 với transit mode.
- Xây full-stack prototype gồm FastAPI backend và React/TypeScript frontend.

### 2. Sản phẩm/agent nhóm đã làm

FlowBot là chatbot tìm tuyến xe bus VinBus bằng ngôn ngữ tự nhiên. User nhập câu hỏi như "Đi từ Hồ Hoàn Kiếm đến Văn Miếu bằng bus như thế nào?", agent trích xuất điểm đi/điểm đến, gọi tool tìm route, rồi trả về tuyến bus, trạm lên/xuống, thời gian và số trạm dừng.

### 3. Chủ đề xuyên suốt cả ngày

Biến FlowBot từ prototype hackathon thành AI transit assistant có thể deploy cho môi trường enterprise của VinBus/XanhSM/VinGroup.

### Câu hỏi bắt buộc

**Sản phẩm này giải quyết bài toán gì?**

Người dùng mất thời gian tìm tuyến bus vì giao diện bản đồ phức tạp, phải tap nhiều bước và dễ nhầm trạm lên/xuống. FlowBot giải quyết bằng hỏi đáp tự nhiên và gọi tool route thật thay vì để LLM tự bịa.

**Ai là người dùng chính?**

- Commuters đi bus hằng ngày cần tìm route nhanh.
- Tourist/người mới đến Hà Nội hoặc TP.HCM chưa quen tuyến bus.
- Về enterprise, user nội bộ có thể là bộ phận CSKH/operation cần trả lời câu hỏi về tuyến nhanh hơn.

**Vì sao chủ đề này phù hợp để phân tích deployment và cost?**

Vì hệ thống phụ thuộc vào LLM API, Google Maps API, route database, latency real-time và độ đúng của dữ liệu giao thông. Sai route có tác động thật tới user, nên cần phân tích deployment, cost, reliability, fallback và monitoring.

## Worksheet 1 - Enterprise Deployment Clinic

### 1. Bối cảnh tổ chức/khách hàng

Khách hàng giả định là VinBus/XanhSM hoặc một đơn vị vận hành giao thông công cộng muốn nhúng chatbot vào app để hỗ trợ tìm tuyến bus.

### 2. Dữ liệu hệ thống đụng đến

- Câu hỏi của user: origin, destination, thời gian đi, preference như ít đi bộ/ít chuyển tuyến.
- Conversation id và lịch sử hỏi đáp ngắn hạn.
- Route data: VinBus local routes, trạm dừng, lịch trình, tần suất, tuyến nội khu.
- Dữ liệu từ Google Maps Routes API: route, distance, duration, transit steps.
- Log vận hành: latency, lỗi API, correction feedback, query bị fallback.
- Nếu Phase 2 có login/location: vị trí hiện tại, địa điểm nhà/công ty, lịch sử tìm kiếm.

### 3. Mức độ nhạy cảm của dữ liệu

- Route data: domain-specific, không quá nhạy cảm nhưng có giá trị vận hành.
- Query origin/destination: có thể là dữ liệu cá nhân vì tiết lộ nơi user sống, làm việc, di chuyển.
- Location realtime nếu bật GPS: nhạy cảm cao, cần xin consent, giảm lưu trữ và mã hóa.
- Log/correction: nhạy cảm trung bình, cần ẩn danh user id và có chính sách retention.

### 4. Ba ràng buộc enterprise lớn nhất

1. Bảo vệ dữ liệu vị trí và hành trình cá nhân của user.
2. Độ chính xác của route và khả năng audit khi bot gợi sai.
3. Phụ thuộc provider ngoài: OpenAI và Google Maps có thể timeout, tăng giá, quota limit hoặc policy thay đổi.

### 5. Chọn mô hình deployment

Chọn **Hybrid**.

### 6. Hai lý do

- Frontend/backend và route database nội bộ nên chạy trong cloud/VPC của doanh nghiệp để kiểm soát log, auth, audit trail và dữ liệu route.
- LLM và Maps API có thể dùng managed external API ở giai đoạn MVP để tiết kiệm thời gian, nhưng cần gateway, rate limit, cache, masking và fallback để giảm rủi ro.

### Câu hỏi gợi ý

**Có cần audit trail không?**

Có. Cần log query đã ẩn danh, tool input/output, route được đề xuất, latency, error, fallback và feedback "sai tuyến" để debug route safety.

**Dữ liệu có được rời khỏi tổ chức không?**

Ở MVP, origin/destination sẽ rời khỏi tổ chức khi gọi OpenAI/Google Maps. Enterprise nên giảm thiểu dữ liệu gửi đi, masking user id, không gửi PII không cần thiết, và có DPA/contract với provider.

**Có cần tích hợp hệ thống cũ/phương thức phê duyệt không?**

Có. Cần tích hợp app VinBus, route/GTFS data, analytics/logging, CSKH ticketing và quy trình phê duyệt thay đổi prompt/routing logic.

**Nếu trả lời sai thì ai bị ảnh hưởng đầu tiên?**

User đi bus bị ảnh hưởng đầu tiên: đi sai trạm, muộn giờ, mất niềm tin. Sau đó là CSKH và uy tín thương hiệu.

## Worksheet 2 - Cost Anatomy Lab

### 1. Ước lượng usage

MVP demo:

- 50 user/ngày.
- 3 request/user/ngày.
- Khoảng 150 request/ngày.
- Peak traffic: 10 request/phút trong giờ cao điểm demo.

Pilot enterprise:

- 500 user/ngày.
- 3 request/user/ngày.
- Khoảng 1,500 request/ngày.
- Peak traffic: 100 request/phút nếu nhúng vào app thật.

Growth:

- 5,000 user/ngày.
- 3 request/user/ngày.
- Khoảng 15,000 request/ngày.
- Peak traffic: 500-1,000 request/phút vào giờ cao điểm.

### 2. Ước lượng token nếu dùng LLM API

Mỗi request:

- Input: 500-900 tokens gồm system prompt, history ngắn, user query, tool schema.
- Output: 150-300 tokens.
- Tool result: 300-800 tokens nếu có nhiều route.
- Tổng ước lượng: 1,000-2,000 tokens/request.

### 3. Các lớp cost

- LLM token cost: GPT-4o-mini hoặc model tương đương cho parsing/tool calling/format response.
- Google Maps Routes API cost: mỗi request route có thể tính phí riêng và có quota.
- Compute backend: FastAPI/Uvicorn, autoscaling, load balancer.
- Frontend hosting/CDN.
- Storage: route DB, logs, feedback, conversation metadata.
- Observability: metrics, log aggregation, tracing, alerting.
- Human review: xem correction log, đánh giá route sai, cập nhật prompt/data.
- Maintenance: refresh route data, fix API changes, security patches.

### 4. Tính sơ bộ cost MVP

Với 150 request/ngày:

- LLM cost: thấp nếu dùng GPT-4o-mini, ước lượng vài cent đến dưới 1 USD/ngày tùy giá/token.
- Google Maps API: có thể là cost chính vì mỗi route query gọi API ngoài.
- Compute/logging cho MVP: có thể chạy 1 instance nhỏ, ước lượng 1-3 USD/ngày nếu deploy cloud đơn giản.
- Human review: 15-30 phút/ngày xem log/correction trong giai đoạn pilot.

Kết luận MVP: cost kỹ thuật chính nằm ở API provider và compute nhỏ; hidden cost lớn hơn là human review và data maintenance.

### 5. Nếu user tăng 5x/10x

- Google Maps API cost tăng gần tuyến tính theo số route query.
- LLM token cost tăng tuyến tính, nhưng có thể giảm bằng caching/model routing.
- Logging/observability tăng theo traffic.
- Human review không nên tăng tuyến tính; cần sampling và dashboard.
- Bottleneck có thể là rate limit/timeout của Google Maps và latency p95.

### Câu hỏi bắt buộc

**Cost driver lớn nhất là gì?**

Google Maps route query và LLM calls trên mỗi request. Nếu mỗi câu hỏi đều gọi cả OpenAI và Google Maps, cost tăng trực tiếp theo traffic.

**Hidden cost dễ bị quên nhất là gì?**

Human review và maintenance route data. Nếu VinBus thay đổi tuyến, prompt/tool đúng nhưng data cũ vẫn làm bot trả lời sai.

**Chỗ nào đang ước lượng quá lạc quan?**

Giả định mỗi user chỉ hỏi 3 câu/ngày và mỗi câu hỏi chỉ cần 1 tool call. Thực tế user có thể hỏi lại nhiều lần, nhập địa điểm mơ hồ, hoặc provider timeout làm phát sinh retry.

## Worksheet 3 - Cost Optimization Debate

### Chiến lược 1: Semantic caching

- Tiết kiệm: LLM token và Google Maps API cho các query lặp lại như "VinUni đến Ocean Park", "Hồ Gươm đến Văn Miếu".
- Lợi ích: giảm cost, giảm latency, tăng ổn định vào giờ cao điểm.
- Trade-off: route/time có thể thay đổi, cache sai có thể gây route cũ.
- Thời điểm áp dụng: làm ngay ở MVP/pilot, với TTL ngắn và invalidate khi route data đổi.

### Chiến lược 2: Model routing

- Tiết kiệm: LLM cost bằng cách dùng model nhỏ cho query đơn giản, model mạnh cho query mơ hồ/đa ràng buộc.
- Lợi ích: chỉ trả tiền cao khi cần reasoning phức tạp.
- Trade-off: cần classifier/router, có rủi ro route query sai sang model quá yếu.
- Thời điểm áp dụng: sau khi có log 1-2 tuần để biết query nào đơn giản/phức tạp.

### Chiến lược 3: Local route lookup trước, Google Maps fallback sau

- Tiết kiệm: Google Maps API call cho các tuyến VinBus nội khu có trong `vinbus_local_routes.json`.
- Lợi ích: nhanh, rẻ, kiểm soát dữ liệu nội bộ, giảm phụ thuộc provider.
- Trade-off: local keyword matching dễ miss/sai nếu user viết khác tên địa điểm; cần update DB liên tục.
- Thời điểm áp dụng: làm ngay cho các route nội khu đã biết, mở rộng sang GTFS/GTFS-Realtime ở Phase 2.

### Chiến lược làm ngay và để sau

- Làm ngay: semantic caching + local route lookup trước.
- Để sau: model routing nâng cao khi có traffic và log thật.

## Worksheet 4 - Scaling & Reliability Tabletop

### Tình huống 1: Traffic tăng đột biến

- Tác động user: response chậm, loading lâu, timeout, nhiều user gửi lại request.
- Phản ứng ngắn hạn: rate limit theo session/IP, queue request, cache query phổ biến, scale backend instance, đặt timeout rõ ràng.
- Giải pháp dài hạn: autoscaling, CDN frontend, Redis cache, async job cho query không cần realtime, dashboard peak traffic.
- Metric monitor: RPS, p50/p95 latency, error rate, timeout rate, cache hit rate, CPU/memory.

### Tình huống 2: Provider timeout OpenAI/Google Maps

- Tác động user: bot không trả route, trả lời chậm, hoặc bị lỗi kết nối.
- Phản ứng ngắn hạn: retry có backoff, circuit breaker, trả fallback "hiện chưa lấy được route realtime", gợi ý kiểm tra app VinBus/bản đồ chính thức.
- Giải pháp dài hạn: provider abstraction, fallback sang local route DB/GTFS, multi-provider cho LLM nếu cần, quota monitoring.
- Metric monitor: provider latency, provider error code, retry count, circuit breaker open rate, quota usage.

### Tình huống 3: Response chậm

- Tác động user: user đang đi trên đường sẽ bỏ cuộc, gửi lại nhiều lần, mất tin tưởng.
- Phản ứng ngắn hạn: hiện loading state rõ, stream response nếu có, giới hạn history/token, cắt bớt tool call không cần thiết.
- Giải pháp dài hạn: cache, precompute popular routes, optimize prompt, tách parsing và route lookup, return route card trước rồi explanation sau.
- Metric monitor: time to first byte, end-to-end latency, token/request, tool duration, abandonment rate.

### Fallback proposal

Thứ tự fallback:

1. Local VinBus route DB nếu match được origin/destination.
2. Cached route gần nhất với TTL và hiện cảnh báo "cần kiểm tra lại".
3. Rule-based flow hỏi lại điểm đi/điểm đến nếu query mơ hồ.
4. Human escalation/CSKH nếu user báo sai tuyến hoặc cần hỗ trợ khẩn.
5. Link sang app/bản đồ chính thức khi provider lỗi.

## Worksheet 5 - Skills Map & Track Direction

### Tự chấm 1-5 theo mảng

Bảng để điền trong nhóm:

| Thành viên | Business/Product | Infra/Data/Ops | AI Engineering/Application |
|---|---:|---:|---:|
| Product lead | 4 | 2 | 3 |
| Architect | 3 | 4 | 4 |
| Cost lead | 4 | 3 | 3 |
| Reliability lead | 3 | 4 | 3 |
| Presenter | 4 | 2 | 3 |

### Điểm mạnh của nhóm

- Mạnh nhất: AI Engineering/Application và product prototype.
- Tương đối tốt: full-stack integration và API tool calling.
- Cần bổ sung: DevOps/observability, security/privacy, data pipeline cho route data.

### Track Phase 2 phù hợp nhất

Đề xuất: **AI Engineering/Application + Infra/Data/Ops**.

Lý do: FlowBot không chỉ cần prompt hay UI đẹp; giá trị thật nằm ở route correctness, tool reliability, cache, monitoring, GTFS/GTFS-Realtime pipeline và enterprise deployment.

### 2-3 kỹ năng cần bù

- Production backend/DevOps: Docker, CI/CD, autoscaling, secrets management, structured logging.
- Data engineering cho transit: GTFS/GTFS-Realtime, route versioning, data refresh, route validation.
- AI reliability/evaluation: test set route queries, precision/correction metrics, fallback design, prompt/tool regression tests.

## 4. Buổi chiều - mini project proposal

### 4.1. Chia zone và vai trò

- Zone: điền theo sắp xếp lớp.
- Presenter: 1 thành viên nói tốt, nắm product story.
- Q&A: Architect + Reliability lead trả lời câu hỏi kỹ thuật/cost.
- Format nộp bài: 7 slide.

### 4.2. Sprint 1 - 7 khối nội dung proposal

1. **Project overview:** FlowBot giúp user tìm tuyến bus VinBus bằng chat tự nhiên.
2. **Enterprise context:** deploy cho VinBus/XanhSM/app giao thông, user data có location và route query.
3. **Deployment choice:** Hybrid - backend/data/log trong enterprise cloud, OpenAI/Google Maps qua gateway có rate limit/cache.
4. **Cost analysis:** cost chính là LLM, Google Maps API, compute, logging, human review; growth 10x làm provider cost và latency tăng mạnh.
5. **Optimization plan:** local route lookup, semantic caching, model routing.
6. **Reliability plan:** timeout, retry, circuit breaker, cache fallback, human escalation, monitoring p95/error/correction.
7. **Track recommendation:** Phase 2 theo AI Engineering/Application + Infra/Data/Ops.

### 4.3. Cấu trúc slide đề xuất

**Slide 1 - Tên dự án, user, bài toán**

FlowBot - VinBus Route Chatbot. User là commuter/tourist/người mới đi bus. Pain: bản đồ phức tạp, khó tìm tuyến, sợ đi sai trạm.

**Slide 2 - Bối cảnh enterprise và ràng buộc**

VinBus/XanhSM cần chatbot nhúng vào app. Ràng buộc: privacy location, audit trail, route accuracy, provider dependency, integration với route data.

**Slide 3 - Kiến trúc triển khai đề xuất**

React frontend -> API gateway -> FastAPI backend -> LangGraph agent -> local route DB/cache -> Google Maps fallback -> OpenAI model. Logs/metrics vào monitoring stack.

**Slide 4 - Cost anatomy**

Cost gồm LLM token, Google Maps API, compute, storage/logging, human review, maintenance route data. Cost driver lớn: provider calls/request.

**Slide 5 - Cost optimization**

Làm ngay: semantic caching và local route DB trước. Sau đó: model routing và prompt compression. Khi volume lớn: xem self-host/smaller model cho parsing đơn giản.

**Slide 6 - Reliability & scaling**

Rủi ro: traffic spike, provider timeout, response chậm, route data cũ. Giải pháp: autoscaling, rate limit, retry/backoff, circuit breaker, cache fallback, GTFS refresh, correction log.

**Slide 7 - Track đề xuất và next step**

Track Phase 2: AI Engineering/Application + Infra/Data/Ops. Next step: đồng bộ backend/frontend response, thêm logging, cache, route eval test set, Docker deploy, GTFS data pipeline.

## 5. Checklist nộp cuối ngày

- [x] Worksheet 0-5 của nhóm.
- [x] Nội dung mini project proposal 7 slide.
- [ ] Phiếu chấm các đội khác trong zone.
- [x] Kết luận Track Phase 2: AI Engineering/Application + Infra/Data/Ops.

## 6. Kết luận ngắn gọn

FlowBot đã là working prototype có AI, tool calling và route API thật. Để lên enterprise, nhóm nên ưu tiên Hybrid deployment, route correctness, audit logging, provider fallback và cost optimization bằng cache/local route lookup. Phase 2 nên tập trung vào AI Engineering/Application kết hợp Infra/Data/Ops vì độ tin cậy của dữ liệu route quan trọng hơn việc chỉ cải thiện prompt.

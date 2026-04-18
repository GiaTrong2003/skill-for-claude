# Kế hoạch luyện phỏng vấn lên chính thức — Viettel

> Tài liệu luyện tập phản xạ phỏng vấn trong 3 tuần. Tập trung vào 3 trụ chính: **Hệ thống lớn / Kiến trúc**, **Xử lý lỗi & Incident**, **Deal lương**. Kèm bộ câu hỏi mẫu, khung trả lời, và cách luyện nói hàng ngày.

## Mục lục
1. [Nguyên tắc luyện tập](#0-nguyên-tắc-luyện-tập)
2. [Phần 1 — Hệ thống lớn & Kiến trúc](#phần-1--hệ-thống-lớn--kiến-trúc)
3. [Phần 2 — Xử lý lỗi & Incident](#phần-2--xử-lý-lỗi--incident)
4. [Phần 3 — Deal lương](#phần-3--deal-lương)
5. [Behavioral & câu hỏi ngược](#behavioral--câu-hỏi-ngược-lại-interviewer)
6. [Lịch luyện 3 tuần](#lịch-luyện-3-tuần)
7. [Checklist ngày phỏng vấn](#checklist-ngày-phỏng-vấn)

---

## 0. Nguyên tắc luyện tập

### Khung trả lời vạn năng — **CPRRL**
Mọi câu kỹ thuật hay behavioral đều nhét vào 5 bước này, nói trong **90–120 giây**:

- **C — Context (bối cảnh)**: hệ thống gì, scale bao nhiêu, vai trò của bạn.
- **P — Problem (vấn đề)**: đề bài cụ thể, ràng buộc.
- **R — Reasoning (lập luận)**: các phương án, trade-off, lý do chọn.
- **R — Result (kết quả)**: số liệu đo được (QPS, latency, % giảm lỗi, thời gian tiết kiệm).
- **L — Lesson (bài học)**: nếu làm lại sẽ khác gì.

> Không có số liệu → câu trả lời yếu. Luôn cố gắng gắn con số.

### Quy trình luyện mỗi ngày (45–60 phút)
1. **10 phút**: đọc/ghi chú lý thuyết chủ đề hôm nay.
2. **25 phút**: tự đặt 3–5 câu hỏi, **nói thành tiếng**, **bấm giờ 2 phút/câu**, **ghi âm**.
3. **10 phút**: nghe lại bản ghi, ghi ra 3 lỗi (lắp bắp, thiếu số liệu, lạc đề, quá dài, quá ngắn).
4. **10 phút**: viết lại 1 câu trả lời tốt nhất thành script, đọc lại trước khi đi ngủ.

### Luật bất di bất dịch
- **Luôn kéo về dự án thật bạn đang làm ở Viettel.** Interviewer chính thức sẽ soi kỹ.
- **Không biết thì nói không biết**, rồi đề xuất cách sẽ tìm hiểu. Chém gió bị bắt = mất điểm nặng.
- **Hỏi lại để làm rõ trước khi trả lời thiết kế hệ thống** — đây là điểm cộng lớn.

---

## Phần 1 — Hệ thống lớn & Kiến trúc

### 1.1 Kiến thức phải nắm (checklist tự chấm)

| Chủ đề | Phải trả lời được |
|---|---|
| **Load Balancer** | L4 vs L7; round-robin / least-conn / consistent hash; sticky session khi nào dùng |
| **Reverse Proxy / CDN** | Nginx vs HAProxy; khi nào đặt CDN; cache-control header |
| **Database** | SQL vs NoSQL; ACID vs BASE; index B-tree vs hash; EXPLAIN; deadlock |
| **Replication** | Master-slave, master-master; đồng bộ vs bất đồng bộ; split-brain |
| **Sharding** | Range / Hash / Geo; hot partition; rebalancing |
| **Cache** | Cache-aside, write-through, write-back; TTL; cache stampede; thundering herd |
| **Message Queue** | Kafka vs RabbitMQ vs Redis Stream; at-least-once vs exactly-once; DLQ |
| **Microservices** | Service discovery; API gateway; saga pattern; 2PC vs eventual consistency |
| **Consistency** | CAP; PACELC; eventual / strong / read-your-writes |
| **Networking** | TCP vs UDP; HTTP/1.1 vs 2 vs 3; gRPC; WebSocket; DNS |
| **Scaling** | Vertical vs horizontal; stateless services; session store |
| **Domain Viettel** | OCS/BSS/CRM/billing/charging real-time, telco-specific nếu đang làm mảng này |

### 1.2 Bộ 15 câu hỏi kỹ thuật mẫu

**Dễ — khởi động**
1. Phân biệt SQL và NoSQL. Khi nào chọn cái nào?
2. Index hoạt động thế nào? Có trường hợp nào index làm chậm query không?
3. Cache-aside là gì? Rủi ro của nó?
4. Giải thích CAP theorem bằng ví dụ thực tế của bạn.
5. Vì sao cần message queue? Cho 1 case bạn đã dùng.

**Trung bình — lập luận trade-off**
6. Thiết kế API rate limiter cho 1 triệu user, 1000 request/giây mỗi user. Chọn thuật toán gì?
7. DB đang chậm, CPU DB 90%. Bạn điều tra và xử lý thế nào theo thứ tự?
8. Service A gọi B gọi C. C chậm dần rồi chết. Hậu quả và cách phòng?
9. Khi nào tách monolith thành microservices? Khi nào KHÔNG nên tách?
10. Hệ thống đang 10k user, dự báo lên 1 triệu trong 6 tháng. Bạn làm gì trước?

**Khó — thiết kế hệ thống**
11. Thiết kế hệ thống nạp tiền điện thoại real-time (domain Viettel): yêu cầu độ trễ < 200ms, không mất giao dịch, 50k TPS peak.
12. Thiết kế hệ thống gửi SMS OTP cho toàn bộ user Viettel: peak 200k SMS/phút, không gửi trùng, không mất.
13. Thiết kế hệ thống tính cước (charging) real-time cho cuộc gọi: độ chính xác 100%, báo cáo theo giờ.
14. Thiết kế notification service đa kênh (SMS/email/push): retry, ưu tiên, rate limit theo user.
15. Thiết kế log aggregation cho 500 microservices, 10TB log/ngày, query được trong 5 giây.

### 1.3 Khung trả lời thiết kế hệ thống — **7 bước**

Dùng khi gặp câu dạng "thiết kế X":

```
1. Làm rõ yêu cầu (2 phút)
   - Functional: user làm gì? input/output?
   - Non-functional: QPS? latency? dung lượng? SLA?
   - Ràng buộc: budget, stack có sẵn, compliance

2. Ước lượng quy mô (1 phút)
   - DAU/MAU → QPS
   - Kích thước data → storage 1 năm
   - Bandwidth in/out

3. API chính (1 phút)
   - REST / gRPC / async
   - 3–5 endpoint cốt lõi

4. Mô hình dữ liệu (2 phút)
   - Bảng / collection chính
   - Quan hệ, index

5. Kiến trúc high-level (5 phút)
   - Vẽ: Client → LB → Service → Cache/DB/Queue
   - Giải thích luồng của 1 request điển hình

6. Deep-dive 1–2 điểm (5 phút)
   - Interviewer thường soi: bottleneck, DB schema, cache strategy
   - Luôn nêu trade-off

7. Xử lý scale & lỗi (2 phút)
   - Điểm chết: single point of failure?
   - Cách mở rộng 10x
   - Monitoring
```

### 1.4 Mẫu trả lời ngắn — "Thiết kế rate limiter"

> **C**: "Ở dự án X em làm, API gateway phải bảo vệ service thanh toán khỏi request spam.
> **P**: Yêu cầu 1M user, mỗi user 1000 req/s, tổng chịu tải 100k TPS, cho phép sai số 1%.
> **R**: Em cân nhắc 3 thuật toán — fixed window đơn giản nhưng có vấn đề burst ở ranh giới; sliding window log chính xác nhưng tốn bộ nhớ O(n); **token bucket** em chọn vì cho phép burst ngắn mà vẫn giới hạn trung bình, chỉ cần lưu 2 giá trị (tokens, last_refill) mỗi user. Lưu ở Redis với Lua script để atomic. Nếu Redis chết thì fallback local in-memory kèm cảnh báo.
> **R**: Sau deploy, 99p latency +2ms, chặn được 30% request spam, service backend giảm 40% CPU.
> **L**: Nếu làm lại, em sẽ dùng Redis Cluster ngay từ đầu thay vì single-node rồi mới scale."

### 1.5 Cách luyện phản xạ cho phần này
- **Vẽ trên giấy A4, nói thành tiếng**. Không được gõ máy khi luyện.
- **Đặt đồng hồ 15 phút cho 1 bài thiết kế**. Hết giờ dừng, review.
- **Ghi âm, đếm số lần "ờ", "à", "kiểu như"** — mục tiêu < 3 lần / 2 phút.

---

## Phần 2 — Xử lý lỗi & Incident

### 2.1 Kiến thức phải nắm

| Chủ đề | Key point |
|---|---|
| **Retry** | Exponential backoff + jitter; tối đa 3–5 lần; chỉ retry idempotent operation |
| **Timeout** | Mọi network call phải có timeout; timeout tầng trên > tầng dưới |
| **Circuit breaker** | 3 trạng thái closed/open/half-open; Hystrix / Resilience4j |
| **Bulkhead** | Cô lập resource pool để 1 service chết không kéo cả hệ thống |
| **Idempotency** | Key idempotent cho POST; dedupe ở consumer queue |
| **Graceful degradation** | Trả dữ liệu cũ / chức năng cắt giảm thay vì fail toàn bộ |
| **Observability** | 3 pillar: log (structured, correlation ID), metric (RED/USE), trace (OpenTelemetry) |
| **SLI/SLO/SLA** | SLI là cái đo, SLO là mục tiêu nội bộ, SLA là cam kết với KH |
| **Incident process** | Detect → Triage → Mitigate → Fix → Post-mortem |
| **On-call** | Run-book, severity level (Sev1–Sev4), escalation |

### 2.2 Bộ 12 câu hỏi sự cố mẫu

1. Kể 1 sự cố production nghiêm trọng nhất bạn từng xử lý. (2 phút)
2. Nửa đêm có alert DB connection pool full. Bạn làm gì trong 5 phút đầu?
3. Service chính bị chậm bất thường từ 8h sáng. Quy trình điều tra?
4. Ai đó deploy xong thì 500 error tăng vọt. Rollback hay fix forward?
5. Log file bị đầy ổ cứng, server không ghi log được nữa. Phòng thế nào?
6. Message queue bị dồn 1 triệu message chưa consume. Làm gì?
7. API bên thứ 3 bạn phụ thuộc bị chết 2 giờ. Hệ thống bạn phản ứng thế nào?
8. Data trong DB bị sai do bug — làm sao xác định phạm vi và sửa?
9. Redis cache bị flush sạch lúc peak. Hiện tượng gì xảy ra và cách phòng?
10. Có memory leak ở Java service. Cách phát hiện và fix?
11. Bạn từng cause sự cố nào chưa? Học được gì?
12. Post-mortem là gì? Khi nào cần? Nội dung gồm gì?

### 2.3 Khung kể chuyện sự cố — **STAR++ (thêm Metric + Lesson)**

Chuẩn bị **3 câu chuyện sự cố** có thật, mỗi câu nói được trong 2 phút:

```
S — Situation: hệ thống gì, scale, lúc nào
T — Task: vai trò của bạn, áp lực (VD: đang ngủ, 2h sáng, Sev1)
A — Action:
   1. Bước đầu tiên: detect bằng gì (alert/monitoring/user báo)
   2. Triage: hypothesis ban đầu
   3. Mitigate: giảm thiểu (rollback / restart / traffic shift)
   4. Root cause: tìm ra nguyên nhân thực
   5. Fix: vá code/config
R — Result: giảm downtime còn bao nhiêu phút, ảnh hưởng X user, không mất tiền
M — Metric: 5 phút MTTD (mean time to detect), 15 phút MTTR
L — Lesson: thêm alert mới, thêm test, thêm run-book, đổi kiến trúc
```

### 2.4 Mẫu kể sự cố (script tham khảo)

> "Tháng 3/2025, hệ thống nạp tiền online, peak 30k TPS. 2h sáng em bị paging — API nạp tiền latency p99 nhảy từ 150ms lên 3 giây, error rate 8%.
> (T) Em là on-call primary.
> (A) Đầu tiên em xem dashboard Grafana — thấy DB write latency tăng, nhưng CPU DB chỉ 40%. Em check slow query log, phát hiện 1 query INSERT mất 2 giây. Grep correlation ID sang service, thấy là luồng ghi log giao dịch. Hypothesis: bảng log bị lock. Em check process list MySQL — đúng là có 1 transaction treo 5 phút do deploy trước đó 6 tiếng đã sửa code mà quên commit ở 1 nhánh exception. Mitigate: em kill session treo, lại chạy được. Sau đó em đọc code, thấy block `catch` thiếu rollback, em raise hotfix PR trong 20 phút.
> (R) Tổng downtime 35 phút, khoảng 2000 giao dịch fail nhưng đều được auto-retry sau khi khôi phục, không mất tiền khách.
> (M) MTTD 4 phút (nhờ alert p99), MTTR 35 phút.
> (L) Em thêm 2 thứ: (1) alert transaction treo > 30 giây, (2) lint rule bắt buộc mọi `catch` phải có rollback hoặc rethrow. Post-mortem em chia sẻ cả team."

### 2.5 Luyện phản xạ phần này
- **Viết trước 3 câu chuyện**, mỗi câu 150–200 từ. Học thuộc 80%, 20% ứng biến.
- **Kể cho bạn bè nghe, bắt họ hỏi vặn**. Câu vặn hay nhất: "Nếu Redis không fix được thì sao?", "Sao không dùng X thay vì Y?".
- Tập **im lặng 3 giây suy nghĩ** trước khi trả lời. Trả lời vội = lộ không chuẩn bị.

---

## Phần 3 — Deal lương

### 3.1 Chuẩn bị trước — 4 việc

1. **Khảo sát thị trường**
   - ITviec, TopDev Salary Report, VietnamWorks.
   - Hỏi 2–3 đồng nghiệp cùng level trong/ngoài Viettel (kín đáo).
   - Xác định range cho level bạn đang hướng tới (VD: Senior Dev ở Viettel 2026).

2. **Chuẩn bị 3 con số**
   - **Floor (đáy)**: mức tối thiểu bạn chấp nhận, dưới thì từ chối.
   - **Target (mong muốn)**: mức bạn thực sự muốn. Thường = floor × 1.2.
   - **Wow (ước mơ)**: mức cao nhất hợp lý. Thường = target × 1.15.
   - Quy tắc: khi nói range, đưa **[target, wow]**. Không bao giờ lộ floor.

3. **Liệt kê "đạn"** — 3–5 thành tựu có số
   - "Em dẫn dắt migration DB giảm 40% chi phí infra, tiết kiệm ~X triệu/năm."
   - "Em fix memory leak làm service ổn định từ 95% lên 99.9% uptime."
   - "Em mentor 2 junior, họ đã lên mid trong 1 năm."

4. **Tính tổng thu nhập (TC — Total Compensation)**
   - Lương cứng × 12
   - Thưởng KPI / 13 / Tết
   - Phụ cấp (ăn, xăng, điện thoại, OT)
   - Cổ phiếu / ESOP (nếu có)
   - Bảo hiểm, khám sức khỏe, đào tạo
   - So sánh giữa các offer nên so TC, không so lương cứng.

### 3.2 Chiến thuật đàm phán — 5 quy tắc vàng

1. **Không bao giờ nói con số trước**
   - Câu đáp: *"Em muốn hiểu rõ scope công việc và kỳ vọng của anh/chị trước, sau đó anh/chị có thể cho em biết range mà Viettel đang định cho vị trí này không ạ?"*

2. **Nếu bị ép nói**
   - Câu đáp: *"Dựa trên khảo sát thị trường cho level này em thấy range khoảng **[target] — [wow]** triệu. Em tin mình có thể mang lại giá trị xứng với mức đó dựa trên [kể 1 thành tựu]."*

3. **Khi họ đưa offer**
   - **KHÔNG đồng ý ngay, dù có hợp lý.** Luôn xin 24–48 giờ suy nghĩ.
   - Câu đáp: *"Em cảm ơn anh/chị, đây là cơ hội em rất quan tâm. Anh/chị cho em 1–2 ngày để em xem xét kỹ và trao đổi lại được không ạ?"*

4. **Counter-offer**
   - Nếu offer < target: counter về target + chút đệm.
   - Câu đáp: *"Em rất muốn nhận vị trí này. Mức lương em đang mong là [target × 1.05]. Lý do là [dẫn thành tựu + khảo sát thị trường]. Anh/chị có thể cân nhắc giúp em không?"*
   - **Luôn đi kèm lý do**, không chỉ đưa số.

5. **Đàm phán đa chiều**
   - Không được lương cứng thì đàm phán: thưởng ký (sign-on), review sớm (6 tháng thay vì 12), cấp bậc cao hơn, thêm ngày phép, học phí, laptop/thiết bị.

### 3.3 Bộ 10 câu hỏi deal lương hay gặp + cách trả lời

**Q1. "Lương hiện tại của em bao nhiêu?"**
> *"Hiện tại em đang ở mức [con số thật hoặc TC]. Tuy nhiên em mong muốn đánh giá vị trí mới dựa trên giá trị em mang lại và scope công việc, không chỉ so với lương cũ ạ."*

**Q2. "Mức lương em mong muốn là bao nhiêu?"**
> *"Dạ em mong mức [target] — [wow] triệu tùy theo scope và package tổng thể. Em sẵn sàng trao đổi thêm khi hiểu rõ hơn về kỳ vọng vị trí."*

**Q3. "Sao em lại xin mức đó, cao hơn mức cũ khá nhiều?"**
> *"Dạ vì [1] khảo sát thị trường cho level này đang ở mức đó; [2] em đã có thêm [X năm / dự án Y / thành tựu Z] so với khi em được định lương lần trước; [3] scope công việc ở vị trí chính thức cũng mở rộng hơn, bao gồm [trách nhiệm mới]."*

**Q4. "Mức em xin cao hơn budget bên anh. Em giảm xuống được không?"**
> *"Em hiểu anh đang có ràng buộc. Anh có thể chia sẻ mức bên mình đang có là bao nhiêu để em cân nhắc không ạ? Hoặc nếu lương cứng chưa đạt, mình có thể trao đổi về các phần khác như thưởng, review cycle sớm, hoặc sign-on bonus không ạ?"*

**Q5. "Em có đang phỏng vấn chỗ nào khác không?"**
> Nếu CÓ: *"Dạ em đang trong quá trình trao đổi với 1–2 công ty nữa, nhưng Viettel là ưu tiên số 1 của em vì [lý do]."*
> Nếu KHÔNG: *"Hiện em đang tập trung cho cơ hội này vì em thực sự muốn gắn bó. Em có theo dõi thị trường để hiểu mức định giá của bản thân."*
> **Tránh nói dối có offer ảo.** Bị check ra là mất tất cả.

**Q6. "Nếu anh offer đúng mức em xin, em có ký luôn không?"**
> *"Em rất cảm ơn. Đây là quyết định quan trọng nên em mong anh cho em 1–2 ngày để xem kỹ hợp đồng và trao đổi với gia đình trước khi chốt ạ."*

**Q7. "Em thấy mình xứng đáng mức đó ở điểm gì?"**
> Kể ngay 2 thành tựu đo được + 1 kỹ năng đặc biệt. Không nói "em chăm chỉ, em học nhanh" — quá chung.

**Q8. "Ngoài lương em quan tâm gì nhất?"**
> *"Em quan tâm 3 thứ theo thứ tự: (1) scope công việc và cơ hội học hỏi, (2) team và lead, (3) package tổng thể gồm cả thưởng và lộ trình thăng tiến."*

**Q9. "Sau 3 năm em muốn ở đâu?"**
> Ghép mục tiêu cá nhân với con đường Viettel cho sẵn (Senior → Tech Lead / Architect). Tránh nói "em chưa biết".

**Q10. "Tại sao nên chọn em thay vì ứng viên khác?"**
> Công thức: **Domain + Kỹ thuật + Soft skill**.
> *"Em đã làm [domain Viettel — billing/OCS/…] 2 năm, nắm nghiệp vụ. Về kỹ thuật em mạnh [stack X], đã giải bài [Y]. Ngoài ra em [mentor/dẫn dắt/đã làm với cross-team] nên có thể đóng góp nhanh."*

### 3.4 Những lỗi chết người khi deal lương
- Nói con số trước interviewer.
- Đồng ý offer đầu tiên ngay lập tức.
- Nói "bao nhiêu cũng được, miễn được nhận".
- Khoe offer ảo không có thật.
- Than thở lý do cá nhân (nhà nghèo, con ốm) — không chuyên nghiệp.
- Đàm phán xong rồi còn xin thêm — mất tín nhiệm.
- Không đàm phán vì sợ mất lòng — thiệt cả đời.

### 3.5 Luyện phản xạ phần này
- **Role-play với bạn bè** ít nhất 3 lần, đóng vai cả 2 bên.
- **Đứng trước gương nói con số** với giọng tự tin, không run. Con số càng lớn càng phải nói chậm, rõ, không ngập ngừng.
- **Viết ra 3 câu chốt yêu thích** và học thuộc.

---

## Behavioral & câu hỏi ngược lại interviewer

### Behavioral — 10 câu kinh điển
1. Giới thiệu bản thân (60 giây).
2. Điểm mạnh — điểm yếu (mỗi bên 2 điểm, có ví dụ).
3. Vì sao em muốn chính thức / gắn bó Viettel?
4. Kế hoạch 3 năm tới?
5. Xung đột với đồng nghiệp — kể và cách xử lý.
6. Lần gần nhất em thất bại / bị sai — học được gì?
7. Em làm việc độc lập hay team hơn?
8. Áp lực cao nhất em từng chịu?
9. Quyết định khó nhất em từng ra về kỹ thuật?
10. Nếu bất đồng với lead về giải pháp, em làm gì?

### Câu hỏi bạn NÊN hỏi lại interviewer
**Về vị trí:**
- "Một ngày làm việc điển hình ở vị trí này trông như thế nào?"
- "Thách thức lớn nhất team đang gặp là gì?"
- "Anh kỳ vọng 90 ngày đầu em sẽ đạt được gì?"

**Về team & văn hóa:**
- "Quy trình code review và deploy của team thế nào?"
- "Team xử lý production incident ra sao? Có run-book và on-call rotation không?"
- "Team có hoạt động chia sẻ kỹ thuật nội bộ không?"

**Về lộ trình:**
- "Lộ trình thăng tiến từ vị trí này là gì? Thời gian trung bình?"
- "Chu kỳ đánh giá performance và điều chỉnh lương như thế nào?"
- "Công ty hỗ trợ gì cho việc học tập (khóa học, hội nghị, chứng chỉ)?"

**Cuối cuộc:**
- "Có phần nào trong hồ sơ của em anh còn băn khoăn để em giải thích thêm không ạ?" ← **câu cực mạnh**, cho cơ hội vá lỗ hổng.
- "Bước tiếp theo của quy trình là gì và khi nào em có thể nhận phản hồi?"

---

## Lịch luyện 3 tuần

### Tuần 1 — Kiến trúc nền tảng
| Ngày | Chủ đề | Output cuối ngày |
|---|---|---|
| T2 | LB + Reverse proxy + CDN | Vẽ sơ đồ 1 hệ thống đang làm, nói 5 phút |
| T3 | DB: SQL/NoSQL, index, replication | Trả lời 3 câu Q1–Q3 phần 1 |
| T4 | Cache + cache pattern | Tự kể 1 tình huống cache stampede |
| T5 | Message queue, async | Q5 + vẽ luồng pub/sub |
| T6 | Microservices, API gateway | Q9 + trade-off thật |
| T7 | Mock interview 45' | Ghi âm, review |
| CN | Nghe lại, viết 3 điểm cần sửa | Script 1 bài thiết kế |

### Tuần 2 — Xử lý lỗi & Incident
| Ngày | Chủ đề | Output |
|---|---|---|
| T2 | Retry / Timeout / Circuit breaker | Q2 + Q7 phần 2 |
| T3 | Observability (log/metric/trace) | Đọc & hiểu dashboard Grafana của dự án |
| T4 | **Viết 3 câu chuyện sự cố** | 3 script, mỗi cái 2 phút |
| T5 | HA, SLA/SLO, DR | Q5 + câu SLA của Viettel |
| T6 | Security cơ bản | Câu AuthN/AuthZ, rate limit |
| T7 | Mock 45' — tập trung kể sự cố | Ghi âm |
| CN | Review + refine 3 story | Học thuộc 80% |

### Tuần 3 — System design + Deal lương + Behavioral
| Ngày | Chủ đề | Output |
|---|---|---|
| T2 | Thiết kế 1 bài lớn (chọn Q11 hoặc Q12) | Vẽ + nói 20 phút |
| T3 | Capacity planning, bottleneck | Ước lượng QPS/storage 1 bài |
| T4 | Domain Viettel chuyên sâu | Note riêng theo mảng đang làm |
| T5 | **Deal lương** — role-play 3 lần | Học thuộc 10 câu phần 3.3 |
| T6 | Behavioral + câu hỏi ngược | Script tự giới thiệu 60s |
| T7 | Full mock 60' (kỹ thuật + behavioral + lương) | Ghi âm |
| CN | Review tổng, chuẩn bị ngày PV | Checklist ngày PV |

---

## Checklist ngày phỏng vấn

### Đêm trước
- [ ] In CV ra giấy, ghi chú 3 thành tựu có số sẵn ra rìa.
- [ ] Đọc lại 3 câu chuyện sự cố, 10 câu deal lương, script giới thiệu 60s.
- [ ] Chuẩn bị 5 câu hỏi ngược lại interviewer.
- [ ] Kiểm tra đường đi / link meeting, test mic + camera.
- [ ] Ngủ đủ giấc. Không học thêm kiến thức mới — sẽ rối.

### Sáng hôm đó
- [ ] Ăn sáng nhẹ. Uống nước đủ.
- [ ] Đến sớm 15 phút (online thì vào sớm 5 phút).
- [ ] Mặc lịch sự (Viettel thiên về formal).
- [ ] Tắt thông báo điện thoại.
- [ ] 3 phút hít thở sâu trước khi vào — giảm run.

### Trong phỏng vấn
- [ ] Tự tin bắt tay / chào, nhìn mắt interviewer.
- [ ] **Im lặng 2–3 giây suy nghĩ trước khi trả lời** câu khó.
- [ ] Dùng khung CPRRL cho mọi câu.
- [ ] Không biết → thừa nhận + đề xuất cách tìm hiểu.
- [ ] Ghi chú câu hỏi quan trọng (xin phép trước).
- [ ] Cuối buổi: hỏi câu ngược + hỏi "next step và timeline".

### Sau phỏng vấn
- [ ] Trong 1 giờ: ghi lại tất cả câu đã được hỏi + cách đã trả lời.
- [ ] Gửi email/tin nhắn cảm ơn trong 24h.
- [ ] Nếu có offer: **không đồng ý ngay**, xin 1–2 ngày.
- [ ] Nếu trượt: xin feedback, cập nhật danh sách câu hỏi cho lần sau.

---

## Phụ lục — Template ghi âm tự đánh giá

Mỗi bản ghi nghe lại tự chấm 5 tiêu chí:

| Tiêu chí | 1–5 | Ghi chú |
|---|---|---|
| Có khung CPRRL rõ không | | |
| Có số liệu cụ thể không | | |
| Tốc độ nói (không quá nhanh/chậm) | | |
| Số lần "ờ, à" trong 2 phút | | Mục tiêu < 3 |
| Độ dài (mục tiêu 90–120s) | | |

Tuần 1 điểm trung bình, tuần 3 phải ≥ 4/5 mỗi tiêu chí.

---

**Chúc may mắn!** Luyện đúng phương pháp 3 tuần, khi vào phỏng vấn thật sẽ thấy mọi câu đều "đã gặp rồi" — đó là đích đến.

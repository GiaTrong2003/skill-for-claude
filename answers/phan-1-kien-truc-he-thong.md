# Phần 1 — Hệ thống lớn & Kiến trúc — Bộ câu trả lời

> **Format chung** cho mỗi câu:
> - **Ý chính** — danh sách đánh số, chắc và an toàn.
> - **Kéo về thực tế** — 1 đoạn ngắn, có thể neo vào dự án **Hợp đồng điện tử** (tích hợp **Viettel-CA, USB token, SmartCA**) hoặc kiến thức chung nếu dự án không match.
> - **> Clarify:** câu hỏi lại interviewer (chỉ khi cần).
> - **> Nếu chưa chắc:** cách thừa nhận + đề xuất tìm hiểu (chỉ khi cần).

---

## Câu 1. Phân biệt SQL và NoSQL. Khi nào chọn cái nào?

**Ý chính**:
1. **Mô hình**: SQL có schema cứng, bảng–hàng–cột, quan hệ rõ. NoSQL có 4 họ — document (Mongo), key-value (Redis), column (Cassandra), graph (Neo4j).
2. **Nhất quán**: SQL theo **ACID**, transaction mạnh. NoSQL phần lớn theo **BASE**, eventual consistency.
3. **Scale**: SQL scale dọc dễ, scale ngang khó. NoSQL sinh ra để scale ngang qua replica set + sharding.
4. **Truy vấn**: SQL có JOIN, aggregation phong phú. NoSQL hạn chế JOIN, thường denormalize.
5. **Chọn SQL khi**: nghiệp vụ có quan hệ, cần transaction, cần báo cáo phức tạp — ngân hàng, hợp đồng, đơn hàng.
6. **Chọn NoSQL khi**: schema thay đổi liên tục, scale rất lớn, ghi nhiều hơn đọc, hoặc cần mô hình đặc thù — log, IoT, feed, cache.

**Kéo về thực tế**: Trong hệ thống Hợp đồng điện tử em làm, em dùng **PostgreSQL** cho metadata hợp đồng & chữ ký (cần ACID — 1 hợp đồng có 3–5 bên ký, service crash giữa chừng phải rollback sạch), **MongoDB** cho audit log (schema biến thiên theo loại event, ghi nhiều đọc ít, giữ 10 năm), **Elasticsearch** cho full-text search nội dung hợp đồng. Không có lựa chọn "SQL vs NoSQL" tổng thể — mỗi loại data quyết riêng.

> **Clarify**: "Anh muốn em so sánh tổng quát hay đi sâu 1 loại NoSQL cụ thể (document / graph / column) ạ?"
> **Nếu chưa chắc** (vd: graph DB): "Em chưa hands-on Neo4j production, em hiểu nguyên lý trên paper. Em sẽ PoC nhỏ + hỏi team Viettel đã dùng để học từ thực tế."

---

## Câu 2. Index hoạt động thế nào? Có trường hợp nào index làm chậm query không?

**Ý chính**:
1. **Bản chất**: Index là cấu trúc phụ, lưu cột được đánh index + con trỏ về hàng gốc, giúp tìm nhanh hơn full scan.
2. **Các loại phổ biến**: **B-tree** (O(log n), dùng cho `=`, range, `ORDER BY`, `LIKE 'abc%'`), **hash** (O(1) equality), **composite** `(a,b,c)` tuân thủ left-most prefix, **covering index** chứa luôn cột SELECT → index-only scan.
3. **Khi index LÀM CHẬM**:
   - **Write amplification**: mỗi INSERT/UPDATE phải cập nhật tất cả index → tăng latency, tăng WAL.
   - **Selectivity thấp**: cột chỉ 2 giá trị (`is_active`) → optimizer bỏ qua.
   - **Hàm trên cột**: `WHERE LOWER(email)=?` → không dùng được index thường.
   - **Implicit cast**: truyền string vào cột int → index bị bỏ.
   - **`OR`, `NOT IN`, `!=`** thường không tận dụng index.
   - **Bảng nhỏ** (< vài nghìn hàng): full scan nhanh hơn.
   - **Index bloat**: cần `REINDEX` định kỳ.

**Kéo về thực tế**: Bảng `signature` của dự án ~200M hàng, partition theo tháng. API "list hợp đồng của tenant theo trạng thái" p95 nhảy từ 200ms lên 2s. Em chạy `EXPLAIN ANALYZE` → seq scan do index cũ `(tenant_id)` selectivity thấp (1 tenant chiếm 30% data). Em đổi thành composite `(tenant_id, status, created_at DESC)` cover luôn sort, đồng thời DROP index `(status)` đơn lẻ — đỡ write 15%. p95 về 50ms.

> **Clarify**: "Anh muốn em nói về Postgres, MySQL, hay nguyên lý chung ạ?"
> **Nếu chưa chắc** (vd: Oracle): "Em chưa debug index Oracle, nhưng B-tree là chung. Em sẽ học `DBMS_XPLAN` để đọc plan tương đương."

---

## Câu 3. Cache-aside là gì? Rủi ro của nó?

**Ý chính**:
1. **Cache-aside (lazy loading)**: app đọc cache trước, miss thì đọc DB rồi tự ghi cache. Khi update, **invalidate** (xóa key) thay vì write cache.
2. **Ưu điểm**: đơn giản; cache chết thì app vẫn chạy (chậm), không ép mọi đường qua cache.
3. **4 rủi ro kinh điển**:
   - **Stale data** — TTL chưa hết nhưng DB đã đổi.
   - **Cache stampede (thundering herd)** — 1 key nóng hết TTL cùng lúc, nhiều request cùng miss → đập DB.
   - **Cache penetration** — key không tồn tại ở cả cache lẫn DB, attacker lợi dụng quét.
   - **Cache avalanche** — nhiều key TTL cố định hết hạn cùng lúc.
4. **Cách phòng**:
   - Stale: invalidate khi update + TTL ngắn; hoặc versioned key (`user:v2:123`).
   - Stampede: **single-flight lock** (Redis Lua / mutex) — chỉ 1 request đi DB, các request khác chờ.
   - Penetration: cache cả giá trị `null` với TTL ngắn; hoặc bloom filter.
   - Avalanche: TTL có **jitter** (`600s ± 60s random`).

**Kéo về thực tế**: Dự án em cache thông tin **tenant (doanh nghiệp)** và **template hợp đồng** trên Redis. 1 lần Redis bị restart, service xác thực đập DB 10k QPS, DB CPU 95%. Em thêm single-flight bằng Lua + TTL có jitter + invalidate qua Kafka event khi admin cập nhật tenant + cache cả miss. Lần restart kế tiếp DB chỉ tăng từ 500 lên 800 QPS.

> **Clarify**: "Anh muốn em so sánh với write-through / write-behind luôn không ạ?"
> **Nếu chưa chắc** (vd: refresh-ahead): "Em chưa triển khai refresh-ahead production. Nếu cần, em sẽ PoC bằng Caffeine (in-process) hoặc Redis Function."

---

## Câu 4. Giải thích CAP theorem bằng ví dụ thực tế.

**Ý chính**:
1. **Phát biểu**: Hệ phân tán không thể đồng thời đạt 3 thuộc tính — **C**onsistency (mọi node thấy cùng giá trị), **A**vailability (mọi request có response), **P**artition tolerance (vẫn chạy khi mạng chia cắt).
2. **Thực tế mạng luôn có thể phân vùng** → **P là bắt buộc**. Câu hỏi thật: chọn CP hay AP khi partition xảy ra?
   - **CP**: từ chối request để không trả data sai — Zookeeper, etcd, HBase, Postgres sync replication.
   - **AP**: vẫn trả lời, đồng bộ sau — Cassandra, DynamoDB (mặc định), Riak.
3. **PACELC** mở rộng: ngay cả khi không có partition (**E**lse), vẫn phải trade-off **L**atency vs **C**onsistency. Spanner là PC/EC, MongoDB mặc định PA/EC.

**Kéo về thực tế**: Dự án em chia 3 mảng data, mỗi mảng chọn khác nhau:
- **Hợp đồng & chữ ký → CP**. Tuyệt đối không để 2 node thấy 2 trạng thái khác nhau (A thấy "đã ký", B thấy "chưa ký" → hợp đồng có thể bị khiếu nại pháp lý). Postgres sync replication, nếu replica mất kết nối → write reject, user retry.
- **Audit log → AP**. Thà log muộn 5s còn hơn block user → Kafka → ES eventual consistency.
- **Cache tenant → AP**. Chấp nhận stale 10 phút để đổi latency.

Bài học: không có "hệ thống CAP đồng nhất", từng loại data phải quyết riêng.

> **Clarify**: "Anh muốn em nói cả PACELC luôn không ạ?"
> **Nếu chưa chắc** (Spanner/CockroachDB): "Em chưa vận hành NewSQL dạng này, em đọc qua paper Spanner/TrueTime. Sẽ học thêm nếu Viettel có cluster tương tự."

---

## Câu 5. Vì sao cần message queue? Cho 1 case bạn đã dùng.

**Ý chính**:
1. **Decoupling**: producer không cần biết consumer, 2 bên scale & deploy độc lập.
2. **Asynchronous**: producer không chờ consumer xử lý xong → API trả về nhanh.
3. **Load leveling**: hấp thụ burst — peak 10k/s, consumer xử lý đều 2k/s.
4. **Durability & retry**: message không mất khi consumer chết, retry tự động, có **DLQ**.
5. **Pub/Sub & fan-out**: 1 event → nhiều consumer song song.
6. **Khi KHÔNG nên dùng**: request–response latency < 50ms; 1 producer 1 consumer cùng service (in-memory đủ); team chưa có người vận hành (MQ thêm monitoring, lag, DLQ, ordering).

**Kéo về thực tế**: Dự án em — sau khi 1 hợp đồng được ký, cần làm 7 việc: verify chữ ký, gọi TSA lấy timestamp, gen PDF cuối, email + SMS các bên, push notification, index ES, ghi audit. Ban đầu làm đồng bộ → p99 10s, user ký lại → duplicate. Em tách thành Kafka topic `contract.signed`, API chỉ publish event rồi trả về 150ms. 7 consumer group nghe cùng topic, mỗi việc scale riêng + DLQ riêng. Chọn Kafka (không RabbitMQ) vì cần replay được (audit), partition theo `contract_id` giữ ordering, throughput cao khi report cuối tháng. Dùng **outbox pattern** (ghi DB + event cùng transaction, daemon đẩy Kafka) để tránh "commit DB OK nhưng publish fail".

> **Clarify**: "Anh muốn em so sánh Kafka / RabbitMQ / Redis Stream luôn không ạ?"
> **Nếu chưa chắc** (Pulsar): "Em chưa vận hành Pulsar, em biết nó tách compute/storage, có tier-ed storage. Nguyên lý pub/sub giống nên em tin học nhanh."

---

## Câu 6. Thiết kế API rate limiter cho 1 triệu user, 1000 req/s mỗi user. Chọn thuật toán gì?

**Ý chính**:
1. **Xác định yêu cầu**: tổng tải ~10⁹ req/s lý thuyết (không thực tế, thường peak ~100k–1M TPS); cho phép burst hay cứng? độ chính xác?
2. **4 thuật toán chính**:
   - **Fixed window**: đếm theo cửa sổ cố định (1s, 1m). Đơn giản nhưng bị "burst ở ranh giới" — user có thể đẩy 2× limit ngay biên.
   - **Sliding window log**: lưu timestamp từng request → chính xác nhưng O(n) bộ nhớ mỗi user.
   - **Sliding window counter**: xấp xỉ 2 fixed window kề nhau theo tỷ lệ → nhẹ và chính xác hơn fixed.
   - **Token bucket**: bucket có dung lượng N, refill `r` token/s. Cho phép burst ngắn, giới hạn trung bình. Chỉ lưu 2 giá trị/user: `tokens`, `last_refill`.
3. **Chọn**: **Token bucket** là lựa chọn cân bằng nhất cho API gateway công cộng.
4. **Triển khai**: lưu ở **Redis** với **Lua script** để atomic (đọc+giảm+ghi trong 1 op). Key: `rl:{user_id}`. TTL vài phút để dọn dead user.
5. **Phân tán**: Redis Cluster shard theo user_id; nếu Redis chết → fallback local in-memory kèm cảnh báo (fail-open hay fail-close tùy nghiệp vụ — thanh toán nên fail-close, feed có thể fail-open).
6. **Response**: trả `429 Too Many Requests` + header `X-RateLimit-Remaining`, `Retry-After`.

**Kéo về thực tế**: Dự án em cần rate limit API gọi OTP ký hợp đồng để chống spam SMS (SMS tốn tiền). Em đặt token bucket 5 token/số điện thoại/5 phút, Lua script trên Redis. Nếu user spam, trả 429 kèm `Retry-After`. Thêm tầng rate limit theo IP để chống attacker dùng nhiều số.

> **Clarify**: "Anh có muốn em giới hạn theo user, theo IP, hay theo API key ạ?" — ảnh hưởng đến key trong Redis.
> **Nếu chưa chắc**: "Nếu yêu cầu distributed chính xác tuyệt đối, em sẽ đọc thêm về leaky bucket và envoy rate limit service."

---

## Câu 7. DB đang chậm, CPU DB 90%. Bạn điều tra và xử lý thế nào theo thứ tự?

**Ý chính** — quy trình 6 bước, từ không xâm lấn đến xâm lấn:
1. **Mitigate trước khi điều tra sâu** (nếu đang incident Sev1): bật rate limit ở tầng trước, shed tải non-critical, hoặc failover sang replica để người dùng có thể dùng tiếp.
2. **Nhìn dashboard**: active connections, slow query count, lock wait, disk I/O, network, replication lag. Xác định CPU do **CPU user** (query nặng) hay **CPU sys** (context switch/IO wait).
3. **Top offender**: `SELECT` từ `pg_stat_activity` / `SHOW PROCESSLIST` → sort theo duration, tìm query/transaction treo lâu.
4. **Slow query log + `EXPLAIN ANALYZE`** trên query nặng nhất: thiếu index? plan đổi? bảng vừa grow lớn?
5. **Check lock**: deadlock? long transaction không commit (thường do code quên `commit` trong block `catch`)?
6. **Check write load**: có job batch / ETL đang chạy không đúng giờ không?
7. **Tác động ngoài**: traffic bất thường, bot, DDoS ở tầng trên?
8. **Xử lý theo thứ tự rủi ro thấp → cao**:
   - Kill query/session treo (rủi ro thấp).
   - Thêm index / rewrite query (cần test).
   - Scale replica để shed read.
   - Nếu không xong: failover, rollback deploy gần nhất.

**Kéo về thực tế**: Có lần dự án em DB CPU 90% lúc 9h sáng. Em vào `pg_stat_activity` thấy 1 session INSERT treo 5 phút. Trace sang code, thấy block `catch` thiếu rollback sau deploy hôm trước. Em kill session → CPU về 40%. Sau đó raise hotfix PR thêm rollback + alert transaction treo > 30s.

> **Clarify**: "Đây là tình huống giả định hay anh muốn em dẫn 1 case em thật sự đã gặp ạ?"
> **Nếu chưa chắc**: Không nên nói "em không biết" — luôn dẫn được ít nhất 3 bước đầu (dashboard, top query, slow log) vì đó là kỹ năng cơ bản.

---

## Câu 8. Service A gọi B gọi C. C chậm dần rồi chết. Hậu quả và cách phòng?

**Ý chính**:
1. **Hậu quả nếu không có phòng hộ**:
   - C chậm → B giữ connection lâu → B cạn **thread pool / connection pool** → B cũng chậm.
   - A tiếp tục gọi B → A cũng cạn pool → **cascading failure** kéo cả chuỗi sập.
   - Thêm **retry không jitter** từ A → đập C càng nặng (**retry storm**) khi C mới hồi phục.
2. **Các pattern phòng hộ**:
   - **Timeout** bắt buộc ở mỗi hop (tầng trên > tầng dưới) — không bao giờ để gọi "không timeout".
   - **Retry có giới hạn + exponential backoff + jitter**; chỉ retry operation **idempotent**.
   - **Circuit breaker** (closed/open/half-open): khi C fail liên tục → open, A/B dừng gọi trong X giây, sau đó half-open thử 1 request.
   - **Bulkhead**: tách pool — ví dụ thread pool riêng cho nhóm gọi C, fail không làm ngạt phần còn lại.
   - **Fallback / graceful degradation**: C chết thì B trả data cũ từ cache, hoặc bỏ qua phần không critical.
   - **Load shedding**: khi upstream quá tải, reject sớm request mới thay vì chờ treo.
3. **Observability đi kèm**: trace (OpenTelemetry), metric RED (Rate/Error/Duration) cho mỗi hop, alert khi error rate > ngưỡng.

**Kéo về thực tế**: Trong flow ký qua SmartCA hoặc USB token, service ký của em gọi sang gateway CA. Gateway CA đôi khi chậm ở giờ cao điểm. Em đặt timeout 8s (vì ký USB token có OTP push mobile mất thời gian thật), circuit breaker với Resilience4j — ngưỡng 50% lỗi trong 20 request thì open 30s. Khi open, service trả lỗi rõ "CA đang gián đoạn, thử lại sau X giây" thay vì treo. Bulkhead tách pool riêng cho luồng ký khỏi luồng xem/tải hợp đồng — CA chết không làm chết API list hợp đồng.

> **Clarify**: "Anh muốn em đi sâu vào 1 pattern không — timeout, circuit breaker, hay bulkhead ạ?"
> **Nếu chưa chắc** (vd: service mesh): "Nếu team Viettel dùng Istio/Linkerd, một số pattern này có thể làm ở mesh thay vì code. Em chưa hands-on Istio production, sẽ học thêm."

---

## Câu 9. Khi nào tách monolith thành microservices? Khi nào KHÔNG nên tách?

**Ý chính**:
1. **Nên tách khi**:
   - Các module có **tốc độ thay đổi khác nhau** rõ rệt (module A deploy 10 lần/ngày, B deploy 1 lần/tháng).
   - **Yêu cầu scale khác nhau**: module ký cần nhiều CPU, module report cần nhiều RAM → scale riêng tiết kiệm.
   - **SLA khác nhau**: phần thanh toán SLA 99.99%, phần thống kê 99% — tách để sự cố phần thấp không kéo phần cao.
   - **Team lớn** (>2 pizza team): nhiều đội cùng sửa monolith → conflict, deploy dẫm chân.
   - **Công nghệ khác nhau**: 1 phần Python ML, 1 phần Java business → tách để stack hợp lý.
2. **KHÔNG nên tách khi**:
   - **Domain chưa rõ** — tách sai boundary tốn gấp 10 lần sửa hơn tách đúng. Nguyên tắc Sam Newman: "start with a modular monolith".
   - **Team nhỏ** (< 10 người) — overhead vận hành microservices (CI, monitoring, tracing, service mesh) lớn hơn lợi ích.
   - **Transaction phức tạp cross-domain** — chuyển sang saga / 2PC làm phức tạp code và khó debug.
   - **Latency-sensitive** — network hop thêm 5–50ms, chuỗi 5 service = 100ms chỉ riêng IO.
   - **Chưa có observability đủ** — microservices không có distributed tracing là thảm họa khi debug.
3. **Cách tách đúng**: bắt đầu bằng **modular monolith** (các module có boundary rõ, share nothing), khi boundary ổn định mới carve-out từng module thành service (strangler pattern).

**Kéo về thực tế**: Hệ thống em tách 4 service chính: `contract-service` (CRUD + workflow), `signing-service` (gọi Viettel-CA / SmartCA / USB token — khác SLA và dependency ngoài), `notification-service`, `audit-service`. Không tách nhỏ hơn vì team em ~15 người, tách nhỏ hơn là overhead thừa. Phần billing-internal em giữ trong contract-service vì transaction luôn chung 1 DB Postgres, tách ra thì phải saga phức tạp.

> **Clarify**: "Anh đang nghĩ đến 1 hệ thống cụ thể để em phân tích không ạ?"
> **Nếu chưa chắc**: "Bản thân em chưa chứng kiến 1 dự án đi từ microservices ngược về monolith — em biết qua bài của Prime Video/Amazon. Nếu Viettel có case tương tự, em rất muốn học."

---

## Câu 10. Hệ thống đang 10k user, dự báo 1 triệu user trong 6 tháng. Bạn làm gì trước?

**Ý chính** — thứ tự ưu tiên:
1. **Xác định bottleneck hiện tại** (đừng scale mù): load test đẩy ngưỡng → tìm điểm vỡ đầu tiên (DB? cache? network? service X?).
2. **Ước lượng mục tiêu**: QPS đỉnh, storage 1 năm, bandwidth, số concurrent connection.
3. **Các hạng mục theo impact / effort**:
   - **Stateless hóa service** (session ra Redis, config ra config server) → scale ngang tự do. (impact cao, effort vừa)
   - **Thêm cache** cho các endpoint đọc nhiều (cache-aside + single-flight). (nhanh thấy hiệu quả)
   - **Tối ưu DB**: review index, thêm read replica, bật connection pooler (pgbouncer). (effort vừa)
   - **CDN** cho asset tĩnh + API response cacheable.
   - **Message queue hóa** các luồng có thể async (notification, analytics, audit).
   - **Auto-scaling** + health check + graceful shutdown.
4. **Sharding DB**: chỉ làm khi read replica không đủ (single-writer bottleneck). Chọn shard key cẩn thận (tenant_id thường hợp cho SaaS).
5. **Quan sát & kỷ luật**: dashboard RED/USE, SLO rõ, load test định kỳ (canary traffic 10% / 50% / 100%).
6. **Tổ chức**: chia team theo domain khi service tách, training on-call, run-book.

**Kéo về thực tế**: Dự án em đã trải qua giai đoạn similar — khi chuẩn bị mở cho tenant bán lẻ. Em làm theo thứ tự: (1) load test tìm bottleneck → thấy bottleneck ở API list hợp đồng (chưa có composite index) và API verify OTP (chưa cache tenant). (2) Sửa 2 điểm này trước, không scale gì cả — p95 đã cải thiện 5×. (3) Sau đó mới thêm read replica, pgbouncer, CDN asset. (4) Không vội sharding — cuối cùng chỉ cần partitioning bảng `signature` theo tháng là đủ.

> **Clarify**: "Anh có thông tin cụ thể về tỉ lệ đọc/ghi và mô hình truy cập không ạ? Nó ảnh hưởng đến thứ tự ưu tiên."
> **Nếu chưa chắc** (vd: sharding): "Em chưa tự sharding DB production quy mô lớn; em hiểu nguyên lý (range/hash/geo, rebalancing, cross-shard transaction). Em sẽ học từ team đã làm nếu cần."

---

## Câu 11. Thiết kế hệ thống nạp tiền điện thoại real-time (latency < 200ms, không mất giao dịch, 50k TPS peak).

> Đây là domain Viettel (không thuộc dự án HĐ điện tử). Em trả lời theo kiến thức chung + nguyên lý em đã áp dụng ở flow thanh toán trong dự án mình.

**Ý chính** — 7 bước thiết kế:
1. **Làm rõ yêu cầu**:
   - Input: số thuê bao + mệnh giá + phương thức (thẻ cào / ví / ngân hàng).
   - Output: kết quả thành công/thất bại; 99.99% thành công trong 200ms; không được mất giao dịch.
   - Idempotency bắt buộc (client retry mà không nạp 2 lần).
2. **Ước lượng**: 50k TPS × 86400s ≈ 4.3 tỷ giao dịch/ngày. Trung bình record ~500 byte → ~2 TB/ngày thô.
3. **API**:
   - `POST /topup` với header `Idempotency-Key: {uuid}`.
   - Body: `{msisdn, amount, channel, ref}`.
4. **Kiến trúc luồng**:
   - API Gateway → topup-service (stateless, horizontal scale).
   - Kiểm tra idempotency key ở Redis trước (nếu đã xử lý → trả kết quả cũ).
   - Ghi **WAL / event log** vào Kafka (topic `topup.request`, partition theo `msisdn`) NGAY — đây là "điểm không mất".
   - Luồng đồng bộ: gọi **OCS (Online Charging System)** qua Diameter / gRPC để nạp; đồng thời trừ ví / charge thẻ.
   - Ghi kết quả vào DB (Postgres cho bản ghi nghiệp vụ) + publish `topup.completed`.
5. **Mô hình dữ liệu**:
   - Bảng `topup` partition theo ngày: `id, msisdn, amount, channel, status, ref, idempotency_key, created_at, updated_at`.
   - Index `(idempotency_key)` unique + `(msisdn, created_at)`.
6. **Đảm bảo không mất**:
   - Kafka `acks=all`, replication factor 3.
   - Outbox pattern: DB + Kafka trong cùng transaction logic.
   - Reconciliation job chạy mỗi 5 phút: đối soát Kafka ↔ DB ↔ OCS, bù các giao dịch lệch.
7. **Đạt latency < 200ms**:
   - Idempotency check ở Redis < 2ms.
   - OCS call < 100ms (cần OCS in-region, connection pool warm).
   - DB write < 20ms (SSD, synchronous commit nhưng group commit).
   - Dự phòng: circuit breaker nếu OCS chậm → trả "pending" và xử lý async (báo user qua SMS/app).
8. **Mở rộng & lỗi**:
   - Scale horizontal topup-service theo autoscaler.
   - Multi-region active-active cho HA, routing theo vùng thuê bao.
   - DLQ cho các giao dịch fail, dashboard theo tỉ lệ mất/lệch.

**Điểm nhấn em sẽ nói**: "Ràng buộc 'không mất giao dịch' là ràng buộc khó nhất — em sẽ nhấn vào outbox pattern + reconciliation job, vì đó là thứ thực sự đảm bảo, không phải 'thiết kế đẹp'."

> **Clarify bắt buộc**: "Anh cho em clarify — 'không mất giao dịch' là không mất **request** (OK trả pending được) hay không được phép trừ tiền sai (chặt hơn)? Anh có cần em tính cả rollback khi OCS charge OK mà ví trừ fail không?"
> **Nếu chưa chắc** (vd: Diameter protocol): "Em chưa code Diameter trực tiếp, em biết OCS thường expose API gRPC/REST nội bộ. Nếu Viettel dùng Diameter, em sẽ học thêm — nguyên lý charging chung em nắm."

---

## Câu 12. Thiết kế hệ thống gửi SMS OTP cho toàn bộ user Viettel: peak 200k SMS/phút, không gửi trùng, không mất.

**Ý chính**:
1. **Yêu cầu**:
   - 200k/phút ≈ 3.3k/s peak.
   - Idempotent: cùng `request_id` chỉ gửi 1 SMS.
   - Không mất: producer crash hoặc SMSC slow đều không được làm mất.
   - Đúng thứ tự không yêu cầu (OTP là 1 lần, không phụ thuộc trước đó).
2. **Luồng**:
   - `POST /otp/send` nhận `{phone, purpose, request_id, ttl}`.
   - Dedupe bằng Redis `SETNX otp:req:{request_id}` TTL 5 phút → trả về OTP cũ nếu trùng.
   - Sinh OTP (6 số), lưu `otp:{phone}:{purpose}` TTL ~3 phút cùng counter verify.
   - Đẩy message vào **Kafka topic `sms.send`** partition theo `phone` (tránh spam 1 số nhiều OTP song song).
   - Consumer đọc → gọi **SMSC / SMS Gateway** → ghi trạng thái `sent/failed` vào DB.
3. **Chống spam & rate limit**: token bucket theo số điện thoại (max 5 OTP/5 phút), theo IP, theo `purpose`. Trả 429 nếu vượt.
4. **Retry & DLQ**:
   - SMSC fail có code rõ ràng (retriable: timeout, network; non-retriable: số sai, blacklist) → retry 3 lần backoff hoặc vào DLQ.
   - DLQ có dashboard + job retry thủ công / tự động mỗi giờ.
5. **Không gửi trùng** ở tầng ngoài Kafka:
   - Consumer idempotent: kiểm tra `sms:sent:{request_id}` trước khi call SMSC.
   - Lưu receipt từ SMSC để tránh double.
6. **Observability**: metric tỷ lệ gửi thành công, delivery delay (nếu SMSC trả delivery report), số OTP verify fail (chỉ số UX quan trọng).
7. **Mở rộng**:
   - Partition Kafka tăng khi throughput tăng.
   - Nhiều SMSC provider với routing (failover theo giá + success rate theo nhà mạng).
   - Geo-replication Kafka nếu cần DR.

**Kéo về thực tế**: Dự án em gửi OTP để ký hợp đồng (ký đơn giản). Em đã áp mô hình này ở quy mô nhỏ hơn — peak ~500 OTP/phút. Idempotency-Key là trụ cột (user nhấn ký 2 lần, SMS chỉ 1). Rate limit theo `phone` cực quan trọng — không có thì có user bị brute force OTP tới 100 lần/phút.

> **Clarify**: "Anh cho em hỏi — 200k/phút là đỉnh tức thời (burst) hay sustained? Và có yêu cầu delivery confirmation từ SMSC không ạ? Hai thứ này đổi thiết kế queue & DB khá nhiều."
> **Nếu chưa chắc** (vd: SMPP protocol): "Em chưa làm việc trực tiếp với SMPP — nếu Viettel tự chạy SMSC, em sẽ học. Em đã dùng abstraction qua REST của nhà cung cấp SMS."

---

## Câu 13. Thiết kế hệ thống tính cước (charging) real-time cho cuộc gọi: độ chính xác 100%, báo cáo theo giờ.

> Domain Viettel thuần. Em trả lời theo kiến thức chung về charging telco.

**Ý chính**:
1. **Yêu cầu**:
   - Độ chính xác 100% — không được sai tiền khách, không được dư/thiếu cho nhà mạng.
   - Real-time: trừ tiền / cấp quota theo giây gọi.
   - Báo cáo theo giờ: tổng hợp dữ liệu sẵn sàng truy vấn sau mỗi giờ.
2. **Khái niệm cốt lõi**:
   - **CDR (Call Detail Record)** sinh ra từ switch / softswitch với `call_id, caller, callee, start, end, duration, cell, …`.
   - **OCS (Online Charging)**: trừ quota/tiền online theo giây (Gy interface trong 3GPP).
   - **OFCS (Offline Charging)**: tạo CDR sau cuộc gọi, chạy rating offline (Gz interface).
3. **Luồng** — gộp 2 nhánh:
   - Bắt đầu gọi: OCS reserve quota (ví dụ 60s) cho thuê bao; nếu ví < 0 → reject.
   - Trong cuộc gọi: OCS re-authorize khi quota sắp hết (reserve tiếp hoặc cut-off).
   - Kết thúc: trả quota thừa về ví; sinh CDR đẩy vào Kafka topic `cdr.raw`.
   - Rating engine đọc CDR → tra **tariff plan / gói cước** → sinh CDR đã rating đẩy sang `cdr.rated`.
   - Billing aggregator đọc `cdr.rated` → aggregate theo giờ/ngày vào DB/data warehouse cho báo cáo.
4. **Đảm bảo 100% chính xác**:
   - Exactly-once effectively: Kafka + idempotent consumer, mỗi CDR có `call_id` unique.
   - **Reconciliation**: đối soát `OCS reservation log` ↔ `CDR raw` ↔ `CDR rated` ↔ tổng trừ ví. Nếu lệch, chạy job bù.
   - Tariff plan lưu versioned (hiệu lực theo thời gian) — rating phải dùng đúng version tại thời điểm gọi.
   - Audit log mọi thay đổi gói cước + mọi điều chỉnh tay.
5. **Mô hình dữ liệu**:
   - CDR lưu dạng **append-only** trên HDFS / Iceberg / ClickHouse — partition theo giờ.
   - OCS dùng in-memory grid (GridGain / Hazelcast) cho latency µs.
   - Billing aggregate lưu trong Postgres / ClickHouse cho BI.
6. **Báo cáo theo giờ**:
   - ClickHouse với materialized view rollup theo giờ → truy vấn tổng cước, top thuê bao, lỗi rating.
7. **HA & mở rộng**:
   - OCS active-active multi-node, replication đồng bộ ví để tránh âm.
   - CDR pipeline có thể xử lý chậm miễn không mất.
   - DR: Kafka multi-datacenter, tariff config đồng bộ.

**Điểm nhấn**: "100% chính xác = reconciliation + idempotency + tariff versioning. Không có design nào đảm bảo chỉ bằng 'ghi 1 lần duy nhất' — phải có job đối soát chạy liên tục."

> **Clarify**: "Anh muốn em tập trung vào phần OCS real-time hay phần rating/aggregation báo cáo ạ? Hai phần kiến trúc khác nhau rõ."
> **Nếu chưa chắc** (vd: Diameter Gy/Gz): "Em chưa code Gy/Gz interface. Em hiểu khái niệm reserve–use–return của OCS ở mức logic. Nếu team Viettel có course nội bộ về 3GPP charging em rất muốn học."

---

## Câu 14. Thiết kế notification service đa kênh (SMS/email/push): retry, ưu tiên, rate limit theo user.

**Ý chính**:
1. **Yêu cầu**: đa kênh, retry có kiểm soát, ưu tiên (OTP > marketing), rate limit theo user (không spam), audit đã gửi gì cho ai.
2. **API & data model**:
   - `POST /notify` body: `{user_id, channels: [sms,email,push], template_id, variables, priority, ttl, idempotency_key}`.
   - Bảng `notification`: `id, user_id, template_id, channel, status, priority, scheduled_at, sent_at, tries, last_error, request_id`.
   - Bảng `template` versioned; bảng `user_preferences` (user chọn kênh nào + mute giờ nào).
3. **Luồng**:
   - API nhận → validate + dedupe idempotency key → ghi DB `PENDING` → push vào Kafka topic theo priority (`notif.high`, `notif.normal`, `notif.low`).
   - Mỗi priority có consumer group riêng với concurrency khác nhau — high nhiều worker hơn.
   - Worker kiểm tra user preferences + rate limit → gọi provider tương ứng (SMS gateway, SMTP/SendGrid, FCM/APNs).
4. **Retry**:
   - Lỗi retriable (timeout, 5xx): exponential backoff 2s, 8s, 30s, 2m; tối đa 5 lần; sau đó DLQ.
   - Lỗi non-retriable (400 "invalid phone"): fail ngay, ghi reason, không retry.
   - Retry dùng Kafka **delay topic** hoặc scheduled job, không retry trong memory worker (worker crash = mất).
5. **Ưu tiên**:
   - Topic riêng là cơ chế đơn giản và hiệu quả nhất — tránh priority queue "thuần" vì khó debug.
   - Budget resource: nếu high priority quá tải, low priority bị hoãn (chấp nhận được).
6. **Rate limit theo user**:
   - Token bucket Redis theo `user_id + channel` — ví dụ SMS marketing tối đa 3/ngày, OTP miễn trừ.
   - Quiet hours: không gửi push từ 22h–7h trừ khi `priority=high`.
7. **Template & i18n**:
   - Template engine (Handlebars / Freemarker), versioned, preview trước khi publish.
   - Render ở worker (không ở client) để có audit đúng nội dung đã gửi.
8. **Observability**:
   - Delivery rate theo kênh; lag của topic; số retry / DLQ; click rate (nếu tracking).
   - Alert khi delivery rate 1 kênh giảm > 10% — thường là provider down.
9. **Mở rộng**: nhiều provider mỗi kênh, routing theo cost + success rate; WebHook delivery report ghi đè status.

**Kéo về thực tế**: Dự án em có cần gửi mail cho các bên sau mỗi hành động (mời ký, đã ký, đã hoàn tất, hết hạn). Em xây notification-service riêng với Kafka topic priority 3 mức, template versioned, user có thể tắt kênh. Phần quan trọng nhất em học được: **audit phải lưu đã gửi gì thực sự** (không chỉ "đã push event") — vì có trường hợp khách hàng khiếu nại "tôi không nhận được email mời ký", mình cần bản log.

> **Clarify**: "Anh có yêu cầu exactly-once delivery không, hay at-least-once chấp nhận được ạ? Ảnh hưởng đến dedupe ở provider layer."
> **Nếu chưa chắc**: "Em chưa xử lý push notification scale rất lớn (>10M DAU); em hiểu FCM/APNs có giới hạn TPS, cần batch. Sẽ học thêm khi cần."

---

## Câu 15. Thiết kế log aggregation cho 500 microservices, 10TB log/ngày, query được trong 5 giây.

**Ý chính**:
1. **Yêu cầu**: 10 TB/ngày ≈ 116 MB/s sustained, peak có thể 3–5×. Query p95 < 5s với khoảng thời gian rộng.
2. **Kiến trúc 3 tầng**:
   - **Collect**: mỗi service log ra **stdout** (12-factor) → sidecar / node agent (Fluent Bit, Vector, Filebeat) → forward.
   - **Transport**: **Kafka** làm buffer giữa collector và storage. Topic `logs.raw` partition theo service.
   - **Storage & index**: tách nóng/lạnh:
     - Hot (7 ngày): Elasticsearch / OpenSearch — index full-text, truy vấn nhanh.
     - Warm (30–90 ngày): Elasticsearch index đã forcemerge + ít replica.
     - Cold (>90 ngày): object storage (S3/MinIO) + Loki hoặc ClickHouse — rẻ, chậm hơn.
3. **Schema log**:
   - Bắt buộc **JSON structured**: `timestamp, level, service, trace_id, span_id, user_id, message, fields...`.
   - `trace_id` từ OpenTelemetry để correlate cross-service.
4. **Đạt query < 5s**:
   - Index ES với mapping phù hợp: timestamp là `@timestamp`, service / level là `keyword` (không analyzed).
   - ILM (Index Lifecycle Management): rotate theo ngày, shard theo volume (~50 GB/shard).
   - Hot node dùng SSD NVMe + RAM lớn; query luôn có filter `@timestamp` + `service` để cắt shard.
   - Dashboard Kibana có saved query thường dùng → cache.
5. **Chi phí & kỷ luật**:
   - Sampling log `DEBUG` trong production (keep 10%).
   - **Không log PII** (số CCCD, OTP, chữ ký) → mask ở pipeline trước khi vào ES.
   - Retention theo loại log: access log 7 ngày, audit log 10 năm (yêu cầu luật nếu có).
6. **HA & throughput**:
   - Kafka replication 3, retention 3 ngày (đệm trong lúc ES bảo trì).
   - ES nhiều node, shard allocation balance.
   - Backpressure: nếu ES chậm, Kafka consumer chậm, nhưng collector không drop (Kafka đệm).
7. **Observability cho chính hệ log**:
   - Metric lag của consumer ES, số docs rejected, disk ES, latency query.
   - Alert khi lag > 5 phút — thường là sign của mapping explosion hoặc node chết.

**Kéo về thực tế**: Dự án em quy mô nhỏ hơn (~50 GB log/ngày) nhưng dùng đúng mô hình này: Filebeat → Kafka → Logstash → ES (hot 7 ngày) + S3 (cold 10 năm cho audit hợp đồng). Bài học: **bắt buộc log JSON + trace_id** ngay từ đầu — sau này mới thêm là tốn hàng tháng refactor. Em cũng đã bị "mapping explosion" khi 1 service log field dynamic vào user_id.xxx — ES tạo 100k field, cluster gần sập. Fix bằng `dynamic: strict` cho index log.

> **Clarify**: "500 microservices có phải Viettel internal không anh? Em muốn hỏi về độ đồng đều log format — nếu chưa chuẩn hóa thì phần collect / parse sẽ là phần tốn công nhất."
> **Nếu chưa chắc** (vd: Loki/ClickHouse as main store): "Em chưa vận hành Loki/ClickHouse làm primary log store. Em biết Loki index metadata thôi (không full-text) nên rẻ nhưng tìm theo nội dung chậm. Nếu Viettel đã chọn stack đó, em sẽ học theo."

---

> **Hết 15 câu Phần 1.** Nếu bạn OK với độ dài và chất lượng, mình sang **Phần 2 — Xử lý lỗi & Incident** (12 câu).

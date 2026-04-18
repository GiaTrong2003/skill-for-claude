# Phần 2 — Xử lý lỗi & Incident — Bộ câu trả lời

> **Format chung**: **Ý chính** (đánh số) → **Kéo về thực tế** (HĐ điện tử hoặc kiến thức chung) → `> Clarify` / `> Nếu chưa chắc` khi cần.
>
> **Khung kể sự cố (STAR++)**: Situation → Task → Action (detect → triage → mitigate → root cause → fix) → Result → Metric (MTTD/MTTR) → Lesson.

---

## Câu 1. Kể sự cố production nghiêm trọng nhất bạn từng xử lý.

**Ý chính** — cấu trúc kể trong 2 phút:
1. **Context ngắn gọn** (1 câu): hệ thống gì, lúc nào, scale, role của mình.
2. **Impact cụ thể có số**: X user ảnh hưởng, Y phút downtime, Z giao dịch fail.
3. **Detect** nhờ đâu (alert / monitoring / user báo) — đưa MTTD.
4. **Triage & mitigate**: giả thuyết đầu → hành động giảm thiểu (rollback / restart / traffic shift).
5. **Root cause**: tìm ra bằng cách nào — công cụ, log, metric.
6. **Fix**: vá code/config, thời gian vá.
7. **Lesson**: alert mới, test mới, run-book, thay đổi kiến trúc.

**Kéo về thực tế**: Tháng 10/2025, peak giờ hành chính ~30k hợp đồng/giờ. 9h sáng em bị paging — API ký hợp đồng p99 từ 180ms lên 3s, error rate 8%. Em là on-call.
- **Detect** sau 4 phút nhờ alert p99. Nhìn Grafana thấy DB write latency tăng, nhưng CPU DB chỉ 40% → không phải overload.
- **Triage**: check `pg_stat_activity` → 1 session INSERT treo 5 phút trên bảng `signature`. Grep correlation ID về service, thấy là luồng ghi chữ ký sau khi gọi Viettel-CA thành công.
- **Mitigate**: kill session treo, hệ thống phục hồi ngay (~30s).
- **Root cause**: đọc code, block `catch` trong flow gọi Viettel-CA thiếu `rollback` khi throw exception phi thường (connection reset) → transaction treo giữ lock bảng.
- **Fix**: hotfix PR thêm rollback + log chi tiết, 20 phút merge & deploy.
- **Result & Metric**: MTTD 4 phút, MTTR 35 phút, ~2000 giao dịch fail nhưng retry tự động sau khi khôi phục, không mất hợp đồng nào.
- **Lesson**: (1) thêm alert "transaction treo > 30s" ở Postgres, (2) lint rule bắt buộc mọi `catch` phải có `rollback` hoặc `rethrow`, (3) post-mortem chia sẻ toàn team.

> **Clarify**: "Anh muốn em kể sự cố về mặt tech sâu hay về mặt phối hợp người / quy trình ạ?" — cho phép chọn 1 câu chuyện phù hợp nhất.
> **Nếu chưa có nhiều sự cố lớn**: "Em chưa gặp sự cố Sev1 đúng nghĩa vì team em small, nhưng em đã xử lý [sự cố Sev2]. Em sẽ kể câu đó và chia sẻ cách em nghĩ nếu gặp Sev1."

---

## Câu 2. Nửa đêm có alert DB connection pool full. Bạn làm gì trong 5 phút đầu?

**Ý chính** — thứ tự 5 phút đầu, **mitigate trước điều tra sâu**:
1. **Phút 0–1 — Xác nhận alert không phải false positive**: mở dashboard DB connections, error rate API → có ảnh hưởng user thật không?
2. **Phút 1–2 — Đánh giá impact**: bao nhiêu service ảnh hưởng, API nào đang 5xx, user-facing hay internal?
3. **Phút 2–3 — Mitigation nhanh** (song song với điều tra):
   - Kill idle/sleeping connections (Postgres `pg_terminate_backend` cho `state='idle in transaction'` > 1 phút).
   - Tăng tạm `max_connections` / mở rộng pool app (nếu có không gian memory).
   - Hoặc restart app service đang leak connection (identify qua dashboard per-service).
4. **Phút 3–4 — Triage root cause**:
   - Có **deploy** trong 1h gần đây không? → rollback là option đầu.
   - Có **job batch / ETL** chạy không đúng giờ không?
   - Có **slow query** đang treo connection không? → `pg_stat_activity`.
   - Có **spike traffic** bất thường không? (DDoS, bot).
5. **Phút 4–5 — Escalate nếu chưa ổn**: báo lead / DBA on-call, mở war-room. Không cố gồng 1 mình quá 15 phút.
6. **Sau ổn định**: mở ticket điều tra sâu, post-mortem.

**Phòng trước cho tương lai**:
- Kết nối phải có **timeout + max lifetime** (HikariCP `maxLifetime` < DB idle timeout).
- **Leak detection threshold** ở pool để log stack trace connection giữ quá lâu.
- Alert sớm ở 70% pool, không đợi 100%.

**Kéo về thực tế**: Dự án em từng bị similar lúc 2h sáng — cron job export báo cáo tháng quên dùng `LIMIT`, 1 query scan cả bảng 200M hàng, giữ 20 connection 15 phút. Em kill query, pool phục hồi trong 1 phút. Sau đó thêm `statement_timeout = 30s` làm default cho role `report`, job dùng role riêng.

> **Clarify**: "Lúc đó em là on-call primary hay có DBA trực cùng ạ?" — ảnh hưởng mức độ em tự xử lý vs escalate.

---

## Câu 3. Service chính bị chậm bất thường từ 8h sáng. Quy trình điều tra?

**Ý chính** — "8h sáng" là gợi ý quan trọng (traffic pattern):
1. **Hypothesis dựa trên thời điểm**: 8h thường là giờ người dùng bắt đầu → traffic tăng đột biến sau đêm ít traffic → **cold cache**, **connection pool warm-up**, **JIT chưa tối ưu**, **autoscaler chưa kịp thêm pod**.
2. **4 hướng điều tra song song**:
   - **Traffic**: QPS có tăng bất thường không, phân bố endpoint có đổi không?
   - **Resource**: CPU/RAM/GC/disk IO của service — có bottleneck rõ không?
   - **Downstream**: DB / cache / service ngoài (Viettel-CA / SmartCA) — latency của từng hop (qua trace).
   - **Deploy / config**: đã có thay đổi nào đêm qua (cron deploy, config rollout)?
3. **Công cụ**: Grafana dashboard RED (Rate/Error/Duration), APM (Jaeger/Tempo), `pg_stat_statements`, log error tăng đột biến.
4. **Hay gặp ở "8h"**:
   - **Cache stampede** khi key TTL qua đêm hết cùng lúc — fix jitter TTL.
   - **Autoscale lag**: HPA phản ứng chậm, pod mới chưa warm → thêm warm-up probe + min replicas cao hơn.
   - **GC spike** do object tích tụ đêm (cron job, background task không clean).
   - **DB plan đổi** sau nightly `ANALYZE` — statistics mới làm optimizer chọn plan tệ.
   - **Batch job nặng** chạy 7h–8h chia sẻ DB/cache → tách workload.
5. **Mitigate tức thời**: tăng replica tay, bật rate limit, failover replica, rollback config đêm qua.

**Kéo về thực tế**: Dự án em có lần service `contract-service` chậm mỗi sáng thứ 2. Truy ra là cron job gen báo cáo tuần chạy 7:30 sáng share cùng DB. Em move job sang replica read-only + chạy 3h sáng. Giải quyết gọn, không cần scale tăng.

> **Clarify**: "Chậm là p95 hay p99? Có cụ thể endpoint nào không, hay toàn service chậm đều ạ?" — câu trả lời định hướng điều tra.

---

## Câu 4. Ai đó deploy xong thì 500 error tăng vọt. Rollback hay fix forward?

**Ý chính** — nguyên tắc quyết định:
1. **Mặc định là ROLLBACK** nếu đạt các điều kiện:
   - Không xác định được root cause trong 5 phút.
   - Impact user thật (không phải internal metric lẻ).
   - Có cách rollback an toàn — nghĩa là **không có migration DB breaking** (ví dụ drop column, rename).
2. **Fix forward KHI**:
   - Root cause đã rõ trong 1–2 phút + fix < 10 phút.
   - Rollback không khả thi: migration đã chạy, data format mới đã ghi.
   - Fix forward rủi ro thấp hơn rollback (ví dụ 1 tham số config sai, sửa là xong).
3. **Nguyên tắc phòng**:
   - **Expand/contract migration**: tách làm 2 deploy — deploy 1 chỉ thêm (expand), deploy 2 xóa cái cũ (contract) — để rollback luôn được.
   - **Feature flag**: bật/tắt feature mới qua flag, không cần rollback code.
   - **Canary deploy**: 5% → 25% → 100%, phát hiện sớm ở 5%.
4. **Quy trình khi rollback**:
   - Thông báo war-room ngay.
   - Rollback artifact (image version cũ) trước, đừng cố "fix quick" trên master.
   - Sau khi ổn, tách branch để điều tra — không vội redeploy.

**Kéo về thực tế**: Dự án em có quy ước: ai deploy là người quyết rollback. Rule em hay dùng: **"Nếu sau 5 phút chưa rõ root cause → rollback, không thương lượng"**. Có lần em tưởng biết nguyên nhân (chỉ là missing env var), định fix forward, hóa ra còn kèm 1 bug config khác → rollback xong mới ngồi fix 2 bug trong 30 phút, thay vì cố gồng sửa trên prod 1 tiếng.

> **Clarify**: "Tình huống này có migration DB kèm deploy không ạ?" — yếu tố quyết định rollback khả thi.
> **Bẫy cần tránh**: đừng trả lời "em sẽ phân tích rồi mới quyết" một cách chung chung — interviewer muốn nghe **quyết định nhanh** và nguyên tắc ra quyết định.

---

## Câu 5. Log file bị đầy ổ cứng, server không ghi log được nữa. Phòng thế nào?

**Ý chính** — tách **mitigate khẩn** và **phòng lâu dài**:
1. **Mitigate tức thời**:
   - Xóa file log cũ (giữ lại file đang ghi để không rotate giữa chừng).
   - `truncate -s 0` với file log đang ghi thay vì `rm` (tránh inode vẫn giữ space).
   - Restart service nếu file descriptor bị kẹt.
2. **Phòng lâu dài — 5 lớp**:
   - **Log rotation**: `logrotate` rotate theo size (100 MB) + age (7 ngày), nén `.gz`, xóa khi vượt quota.
   - **Centralized logging**: log đẩy về **Fluent Bit → Kafka → ES/Loki**, KHÔNG giữ trên server app. Server chỉ buffer ngắn (file hoặc stdout tail).
   - **Alert sớm**: disk usage > 70% → alert warning, > 85% → page.
   - **Retention policy theo loại log**: access log 3–7 ngày trên local, audit log đẩy sang S3 (giữ theo luật), debug log chỉ bật khi cần.
   - **Log hygiene**: log JSON structured, không log PII (OTP, chữ ký raw, CCCD); sample `DEBUG`/`INFO` ở production.
3. **Anti-pattern cần tránh**:
   - Log ra 1 file duy nhất không rotate.
   - Log trong vòng lặp nóng (thousands of events/s) → gây chính nó đầy disk.
   - Dùng `cat` / `tail` debug trên server không có rotation.

**Kéo về thực tế**: Dự án em từng có service log mọi request body (bao gồm PDF hợp đồng base64) vì 1 dev enable `DEBUG` tạm mà quên tắt. 1 ngày đầy 50 GB disk. Sau đó em: (1) đổi log format strict JSON schema, (2) redact rule bắt buộc trước khi ghi log, (3) alert disk 70% cho mỗi node, (4) đẩy log trung tâm qua Fluent Bit để local chỉ giữ buffer.

> **Clarify**: "Server này là VM hay container ạ? Container thường có stdout bị giới hạn bởi Docker log driver, phòng khác hơn chút."

---

## Câu 6. Message queue bị dồn 1 triệu message chưa consume. Làm gì?

**Ý chính** — 4 bước:
1. **Triage root cause** trước khi tăng consumer mù:
   - Consumer **crash loop**? (check logs, OOM, exception)
   - Consumer **chậm** do downstream (DB slow, external API chậm)?
   - Producer **spike** bất thường?
   - Message **poison** làm consumer retry vô hạn?
2. **Mitigate tùy root cause**:
   - Nếu crash: fix bug / revert deploy consumer → restart.
   - Nếu chậm do downstream: tạm scale consumer hoặc **batch processing** (thay vì 1 msg / call, gộp 100 msg).
   - Nếu producer spike: rate limit / tạm hoãn producer non-critical.
   - Nếu poison message: chuyển vào **DLQ** ngay, đừng để nó block partition.
3. **Tăng throughput consumer** (nếu CPU/IO còn dư):
   - Tăng **partition** (Kafka) — nhưng phải đảm bảo ordering vẫn đúng theo key.
   - Tăng **consumer instance** trong group.
   - Tăng `max.poll.records` + batch commit.
4. **Đối soát sau khi tiêu thụ xong**:
   - Check tổng sent vs tổng processed; các message DLQ có cần retry thủ công không.
   - Post-mortem: thêm alert consumer lag > 5 phút, run-book.

**Kéo về thực tế**: Dự án em có lần backlog 200k message topic `contract.signed` vì consumer gọi TSA (timestamp authority) chậm do TSA bị tấn công slowloris. Em tạm thời **bypass TSA** (ghi message vào buffer topic `contract.signed.pending-tsa`) để unblock các consumer khác (email, audit), sau đó chạy riêng consumer TSA với retry backoff. Backlog tiêu trong 2 giờ.

**Câu vặn hay gặp**: "Nếu tăng consumer không giúp gì thì sao?" → **bottleneck không phải consumer** mà là **downstream** (DB/API). Tăng consumer chỉ làm downstream chết nhanh hơn. Phải sửa downstream hoặc batch.

> **Clarify**: "Queue là Kafka, RabbitMQ hay SQS ạ? Vì cách scale khác nhau (Kafka giới hạn bởi partition, RabbitMQ giới hạn bởi consumer prefetch)."

---

## Câu 7. API bên thứ 3 bạn phụ thuộc bị chết 2 giờ. Hệ thống bạn phản ứng thế nào?

**Ý chính**:
1. **Phát hiện nhanh**:
   - Health check + synthetic probe gọi third-party mỗi 30s.
   - Alert khi error rate > 5% hoặc latency > threshold.
2. **Hạn chế lan rộng**:
   - **Circuit breaker open** — dừng gọi third-party, trả lỗi nhanh thay vì treo.
   - **Bulkhead**: tách pool gọi third-party, không cho nó chiếm thread pool chung.
   - **Timeout ngắn** (3–5s), không chờ.
3. **Graceful degradation** (tùy nghiệp vụ):
   - Trả dữ liệu **cached** nếu có.
   - **Queue request** để xử lý sau (nếu async chấp nhận được).
   - **Trả lỗi rõ ràng** cho user: "Dịch vụ X đang gián đoạn, xin thử lại sau N phút" — KHÔNG để user thấy generic 500.
   - **Fallback provider**: nếu có 2 nhà cung cấp, switch sang cái còn lại.
4. **Chủ động với nhà cung cấp**:
   - Liên hệ qua channel support / status page của họ.
   - Nếu có SLA breach, ghi nhận để claim sau.
5. **Sau khi phục hồi**:
   - Reconcile: các request đã queue/pending cần retry theo backoff jitter (tránh retry storm ngay khi họ sống lại).
   - Post-mortem: có cần giảm dependency, tìm provider dự phòng, cache rộng hơn không?

**Kéo về thực tế**: Dự án em phụ thuộc **Viettel-CA** và **SmartCA** cho luồng ký. Có lần SmartCA bảo trì không thông báo trước, down 90 phút. Em có sẵn 2 fallback: (1) user có USB token → redirect sang flow USB, (2) user chỉ có SmartCA → hiển thị banner "SmartCA đang gián đoạn, vui lòng thử sau N phút" + gửi notification khi CA khôi phục. Không mất hợp đồng nào, chỉ delay 90 phút. Post-mortem: thêm status page nội bộ hiển thị health các CA, thêm auto-notification khi CA khôi phục.

> **Clarify**: "Third-party đó có SLA cam kết không? Và có provider thay thế được không ạ? Hai yếu tố này quyết định strategy là 'chờ + degrade' hay 'failover'."

---

## Câu 8. Data trong DB bị sai do bug — làm sao xác định phạm vi và sửa?

**Ý chính**:
1. **Dừng bleeding trước** — deploy fix cho bug ngay, dù sửa data sau. Đừng để data sai tiếp tục ghi vào.
2. **Xác định phạm vi**:
   - **Git blame / commit log** → tìm commit gây bug, biết được thời điểm deploy ra prod.
   - **Audit log / event log** → các record được ghi/sửa trong khoảng thời gian đó.
   - **Reverse engineer**: viết query tìm pattern data sai (ví dụ `status='SIGNED'` nhưng không có record trong bảng `signature`).
3. **Dry-run trước khi sửa**:
   - Viết script SELECT ra danh sách record sẽ bị đụng.
   - Review với team / lead.
   - Backup bảng liên quan (`pg_dump` / snapshot).
4. **Sửa data**:
   - Trong **transaction**, update theo batch (1k–10k record / batch) để không lock bảng lâu.
   - Log lại before/after mỗi record vào bảng audit riêng — phòng trường hợp cần rollback.
   - Rate-limit batch (ví dụ sleep 100ms giữa batch) để không ngạt replication.
5. **Verify**:
   - Sau fix, chạy lại query pattern data sai → kỳ vọng 0 record.
   - Random spot-check 10–20 record bằng tay.
6. **Thông báo**:
   - Nếu data sai ảnh hưởng user (ví dụ hiển thị sai trạng thái) → notify và post-mortem công khai.
   - Nếu là data nội bộ → ghi chú trong post-mortem.

**Kéo về thực tế**: Dự án em từng có bug: khi hủy hợp đồng, script không clear trường `last_signed_by`, khiến UI vẫn hiển thị "đã ký bởi X" dù hợp đồng đã hủy. Em: (1) fix code, (2) query tìm tất cả hợp đồng `status='CANCELLED'` nhưng còn `last_signed_by != null` — ra ~800 record, (3) dry-run UPDATE có WHERE chặt + LIMIT 100 trước, (4) chạy batch trong transaction, log vào `data_fix_audit`, (5) verify bằng query đối chứng. Toàn bộ 40 phút.

> **Clarify**: "Data bị sai là lệch 1 trường cụ thể hay nhiều bảng liên quan ạ? Nhiều bảng thì phải xử lý theo thứ tự foreign key."
> **Nguyên tắc cứng**: KHÔNG UPDATE trực tiếp trên production không có WHERE chặt. Luôn SELECT trước, dry-run trước.

---

## Câu 9. Redis cache bị flush sạch lúc peak. Hiện tượng gì xảy ra và cách phòng?

**Ý chính**:
1. **Hiện tượng**:
   - **Thundering herd** vào DB — mọi request đều cache miss.
   - DB CPU/IO vọt lên → latency API tăng 10–100×.
   - Nếu DB quá tải → cascade: API timeout → upstream retry → càng ngạt.
2. **Nguyên nhân Redis bị flush**:
   - Ops chạy `FLUSHALL` / `FLUSHDB` nhầm (hay gặp).
   - Redis restart (maxmemory chưa persist).
   - Failover sang replica chưa warm.
   - Eviction policy `allkeys-lru` khi đầy memory.
3. **Cách phòng**:
   - **Single-flight**: 1 key miss, chỉ 1 request đi DB, các request khác chờ (Redis Lua / distributed mutex).
   - **TTL có jitter**: tránh hết hạn đồng loạt.
   - **Pre-warm sau restart**: chạy script warm các key top-N trước khi nhận traffic.
   - **L2 cache** (in-process Caffeine / Guava) làm lớp đệm giữa app và Redis.
   - **Rate limit tầng ngoài** để nếu Redis chết, DB không bị ngạt quá ngưỡng.
   - **Redis Cluster** với replica đồng bộ — flush 1 node không mất tất cả.
   - **ACL**: disable `FLUSHALL` cho account production, chỉ admin có key riêng.
4. **Mitigate khi đang xảy ra**:
   - Bật rate limit ở gateway (giảm 50% traffic).
   - Scale DB read replica tạm.
   - Pre-warm các key nóng trực tiếp từ script đồng bộ.

**Kéo về thực tế**: Dự án em đã mô tả ở **Câu 3 Phần 1** — lần Redis restart làm DB ngạt. Giải pháp em làm: single-flight Lua + TTL jitter + invalidate qua Kafka event. Lần flush kế tiếp DB chỉ tăng từ 500 → 800 QPS (không phải 10k).

> **Clarify**: "Redis là standalone hay cluster ạ? Cluster nửa node chết còn phân nửa data."
> **Câu vặn hay gặp**: "Nếu single-flight cũng timeout thì sao?" → fallback: trả stale data nếu có (từ L2 cache), hoặc trả lỗi rõ kèm `Retry-After`, không treo user.

---

## Câu 10. Có memory leak ở Java service. Cách phát hiện và fix?

**Ý chính**:
1. **Phát hiện**:
   - **Metric heap** tăng đều, không giảm sau GC — triệu chứng kinh điển.
   - **Full GC tần suất cao** nhưng heap không giảm → generation cũ đầy object còn reference.
   - **OutOfMemoryError** cuối cùng, nhưng mình nên bắt sớm.
   - Monitor: Prometheus JVM exporter, hoặc APM (New Relic / Datadog).
2. **Công cụ chẩn đoán**:
   - **Heap dump** (`jmap -dump:format=b,file=heap.hprof <pid>` hoặc `-XX:+HeapDumpOnOutOfMemoryError`).
   - Phân tích với **Eclipse MAT** hoặc **VisualVM**: tìm dominator tree, object lớn nhất.
   - **async-profiler** / **JFR** cho allocation profiling.
   - `jstat -gc` theo dõi GC real-time.
3. **Pattern leak phổ biến**:
   - **Collection tĩnh** (static `Map`) không bao giờ clear.
   - **Listener / callback** đăng ký mà không unregister.
   - **ThreadLocal** không `remove()` trong pool thread → leak theo pool size.
   - **Cache không có bounded size** (HashMap thay vì Caffeine).
   - **Connection / Stream / ResultSet** không close — leak native memory lẫn heap.
   - **Classloader leak** (hot-reload framework, reload không unload class).
4. **Fix**:
   - Clear reference khi không dùng.
   - Dùng `WeakHashMap` / `SoftReference` cho cache không critical.
   - Bounded cache (Caffeine với max size + TTL).
   - try-with-resources cho mọi Closeable.
5. **Mitigate tạm**:
   - Restart service định kỳ (cron) nếu chưa fix kịp.
   - Tăng heap (chỉ dời hẹn, không fix).
   - Scale ngang để giảm load mỗi instance.

**Kéo về thực tế**: Dự án em có service xử lý PDF hợp đồng dùng thư viện PDF bên thứ ba. Em phát hiện heap tăng đều 100 MB/ngày. Heap dump với MAT → dominator là `PdfReader` giữ font cache global. Bug là thư viện đó cache font theo class loader, mỗi request tạo loader mới (do dynamic class) → leak. Fix bằng cách (1) share PdfReader singleton, (2) thêm bounded cache manual. Heap stable sau đó.

> **Clarify**: "Java version mấy ạ? Java 8 và 17+ có khác về GC (G1 mặc định vs CMS/Parallel) — ảnh hưởng cách diagnose."
> **Nếu chưa chắc**: "Em chưa dùng JFR continuous profiling production; em biết khái niệm, nếu team Viettel đã bật, em sẽ đọc report từ đó."

---

## Câu 11. Bạn từng cause sự cố nào chưa? Học được gì?

**Ý chính** — interviewer đánh giá **trung thực + khả năng tự nhận trách nhiệm + bài học**. Không được nói "em chưa cause sự cố gì".
1. **Cấu trúc 90 giây**:
   - 1 câu ngắn thừa nhận: "Có, em từng cause [sự cố X]".
   - Context: hệ thống gì, scope.
   - Em làm sai ở đâu (rõ ràng, không đổ lỗi).
   - Hậu quả (có số).
   - Cách em handle ngay khi phát hiện (detect + mitigate).
   - Bài học + thay đổi hành vi sau đó.
2. **Nguyên tắc**:
   - Chọn sự cố **đủ lớn để có bài học**, nhưng không quá thảm khốc (mất dữ liệu khách hàng vĩnh viễn).
   - **Thừa nhận thẳng** cái sai của mình, đừng đổ lỗi quy trình / team.
   - Bài học phải **hành động cụ thể** đã làm, không chung chung "em cẩn thận hơn".
   - Kết ở tone tích cực, thể hiện đã trưởng thành.

**Kéo về thực tế** (script mẫu):
> "Có, năm ngoái em cause 1 sự cố khoảng Sev2. Em chạy migration thêm cột `signer_email_verified` cho bảng `signer` ở production, quên NOT NULL default. Ứng dụng đọc column đó xong gặp NullPointerException cho mọi request list hợp đồng. Lỗi lan ra trong 3 phút.
> Em là người merge PR + chạy migration, không có ai review. Em phát hiện qua alert p99 trong 2 phút, rollback bằng cách `ALTER TABLE ... SET DEFAULT false` + fill `UPDATE ... WHERE signer_email_verified IS NULL`. Tổng downtime 12 phút.
> Bài học em rút ra 3 điều cụ thể, em đã làm ngay sau incident: (1) mọi migration production bắt buộc 2 reviewer — em đưa vào CI check. (2) Em viết checklist 'migration đi kèm backfill script' làm standard. (3) Em không bao giờ merge + deploy cùng thời điểm nữa, tách 30 phút để quan sát."

> **Clarify** (khi cần nhiều thời gian): "Anh muốn em kể 1 sự cố tech em cause, hay 1 quyết định thiết kế em về sau thấy sai ạ?"
> **Tuyệt đối không**: "em rất cẩn thận nên chưa cause sự cố gì". Interviewer hiểu 1 là nói dối, 2 là chưa từng chạm vào production thật.

---

## Câu 12. Post-mortem là gì? Khi nào cần? Nội dung gồm gì?

**Ý chính**:
1. **Định nghĩa**: Tài liệu viết sau incident (Sev1/Sev2 bắt buộc, Sev3 khuyến khích) để phân tích toàn bộ sự cố, học bài học, ra action items chống tái diễn.
2. **Nguyên tắc tối quan trọng — BLAMELESS**:
   - Trọng tâm là **hệ thống / quy trình**, không phải người.
   - Không "ai là người làm sai" mà "quy trình / công cụ nào đã để 1 người duy nhất có thể gây sự cố này".
   - Người cause sự cố thường là người hiểu nhất → nên chủ trì hoặc tham gia viết, không bị punish.
3. **Nội dung chuẩn — 8 mục**:
   - **Summary** (3–5 câu): chuyện gì xảy ra, ai bị ảnh hưởng, hậu quả.
   - **Timeline**: mốc thời gian UTC — detect lúc X, mitigate lúc Y, fix lúc Z.
   - **Impact**: số user ảnh hưởng, số giao dịch fail, thiệt hại tiền (nếu tính được), SLA breach không.
   - **Root cause**: nguyên nhân gốc (thường dùng **5 Whys** hoặc fishbone).
   - **Contributing factors**: các yếu tố phụ làm incident nặng hơn (thiếu alert, run-book sai, on-call chưa training).
   - **What went well**: detect nhanh, mitigate đúng, teamwork tốt — nhấn để tạo tinh thần.
   - **What went poorly**: alert chậm, không có run-book, hand-off lộn xộn.
   - **Action items**: list hành động cụ thể, **có owner + deadline + priority** (P0/P1/P2). Thiếu action items = post-mortem thất bại.
4. **Khi nào bắt buộc**:
   - Sev1 (major outage, user-facing).
   - Sev2 (partial outage, workaround được).
   - Data loss / security incident, bất kể severity.
   - Near-miss đáng học.
5. **Quy trình**:
   - Trong 24–48h sau incident: draft.
   - Meeting review với team + stakeholder.
   - Publish rộng (toàn company) để các team khác học.
   - Track action items đến khi close.

**Kéo về thực tế**: Dự án em có template post-mortem chuẩn. Mỗi post-mortem đóng thường mất 3–5 action items. Em đặc biệt thích **What went well** — làm tinh thần team lên, và giúp nhận ra những thứ đang làm tốt mà không ai để ý. Sự cố em cause ở Câu 11 em là người chủ trì viết post-mortem, team review, ra 3 action items — cả 3 đã close trong 1 tháng.

> **Clarify**: "Viettel có template post-mortem nội bộ không anh? Em muốn follow theo nếu có, vì mỗi công ty có format hơi khác."
> **Câu vặn hay gặp**: "Post-mortem blameless có đồng nghĩa với 'không ai chịu trách nhiệm' không?" → Không. Blameless là về **cách phân tích**, không phải về **accountability**. Người cause vẫn cần tự nhận, học, và cam kết đổi hành vi — nhưng công cụ kỷ luật là training & process, không phải đổ lỗi.

---

> **Hết 12 câu Phần 2.** File tiếp theo: `phan-3-deal-luong.md`.

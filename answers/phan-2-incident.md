# Phần 2 — Xử lý lỗi & Incident — Bộ câu trả lời

> **Format chung**: **Ý chính** (đánh số) → **Kéo về thực tế** → **Câu xoáy đáp xoay** → `> Clarify` / `> Nếu chưa chắc` khi cần.
>
> **Khung kể sự cố (STAR++)**: Situation → Task → Action (detect → triage → mitigate → root cause → fix) → Result → Metric (MTTD/MTTR) → Lesson.
>
> **Quy ước**: thuật ngữ khó có giải thích bằng **ngôn ngữ đời thường** ngay sau — đọc kèm sẽ hiểu ngay khi nói.

---

## Bảng thuật ngữ tham chiếu nhanh — Phần 2

| Thuật ngữ | Giải thích đời thường |
|---|---|
| **MTTD** | Mean Time To Detect — trung bình bao lâu từ khi xảy ra đến khi phát hiện ra. |
| **MTTR** | Mean Time To Recover — trung bình bao lâu từ khi phát hiện đến khi hệ thống ổn. |
| **p99** | 99% request nhanh hơn con số này, chỉ 1% chậm hơn — đo "đuôi dài" của latency. |
| **Connection pool** | Bể kết nối — tập hợp kết nối DB giữ sẵn để tái dùng, không phải mở mới mỗi lần. |
| **SHOW PROCESSLIST** | Lệnh MariaDB liệt kê mọi connection + query đang chạy, thời gian treo. Chi tiết hơn: `information_schema.PROCESSLIST`, `INNODB_TRX`, `INNODB_LOCKS`. |
| **KILL / KILL QUERY** | Lệnh MariaDB giết connection hoặc chỉ query (giữ connection). Dùng khi có session treo lock bảng. |
| **performance_schema** | Schema nội bộ MariaDB chứa metric runtime: digest query, lock, IO wait — tương đương `pg_stat_*` của Postgres. |
| **max_statement_time** | Setting MariaDB giới hạn thời gian tối đa 1 query chạy (giây). Tránh query treo vô hạn. |
| **Correlation ID** | ID duy nhất gắn vào 1 request từ đầu đến cuối, dùng để trace log qua nhiều service. |
| **Idempotent** | Bất biến lặp lại — gọi 1 lần hay 5 lần với cùng key vẫn ra cùng kết quả. |
| **DLQ** | Dead Letter Queue — hàng đợi chứa message không xử lý được sau nhiều retry. |
| **Poison message** | Message làm consumer crash mỗi lần xử lý — nếu không có DLQ thì retry vô hạn, chặn cả partition. |
| **Circuit breaker** | "Cầu chì" — khi downstream lỗi nhiều, tự ngắt gọi trong X giây để tránh cháy lan. |
| **Blameless post-mortem** | Viết báo cáo sự cố tập trung vào hệ thống/quy trình, không đổ lỗi người. |
| **5 Whys** | Hỏi "Tại sao?" liên tiếp 5 lần để đào root cause thật, không dừng ở triệu chứng. |
| **WAL / Redo log / Binlog** | Write-Ahead Log — log ghi trước khi commit. MariaDB InnoDB dùng **redo log** (file `ib_logfile*`, để khôi phục crash) + **binlog** (binary log — để replicate sang slave). |
| **Thundering herd** | "Đàn trâu xô cửa" — nhiều request cùng miss cache 1 lúc, cùng đập DB. |

---

## Câu 1. Kể sự cố production nghiêm trọng nhất bạn từng xử lý.

**Ý chính** — cấu trúc kể trong 2 phút:
1. **Context ngắn gọn** (1 câu): hệ thống gì, lúc nào, scale, role của mình.
2. **Impact cụ thể có số**: X user ảnh hưởng, Y phút downtime, Z giao dịch fail.
3. **Detect** nhờ đâu (alert / monitoring / user báo) — đưa MTTD (trung bình bao lâu phát hiện ra).
4. **Triage & mitigate**: giả thuyết đầu → hành động giảm thiểu (rollback / restart / traffic shift).
5. **Root cause**: tìm ra bằng cách nào — công cụ, log, metric.
6. **Fix**: vá code/config, thời gian vá.
7. **Lesson**: alert mới, test mới, run-book, thay đổi kiến trúc.

**Kéo về thực tế**: Tháng 10/2025, peak giờ hành chính ~30k hợp đồng/giờ. 9h sáng em bị paging — API ký hợp đồng **p99** (99% request nhanh hơn con số này, chỉ 1% chậm hơn) từ 180ms lên 3s, error rate 8%. Em là on-call.
- **Detect** sau 4 phút nhờ alert p99. Nhìn Grafana thấy DB write latency tăng, nhưng CPU DB chỉ 40% → không phải overload.
- **Triage**: chạy `SHOW FULL PROCESSLIST` + `SELECT * FROM information_schema.INNODB_TRX ORDER BY trx_started` (xem transaction đang chạy + thời gian bắt đầu) → 1 session INSERT treo 5 phút trên bảng `signature`. Grep **correlation ID** (ID duy nhất gắn vào request từ đầu đến cuối để trace) về service, thấy là luồng ghi chữ ký sau khi gọi Viettel-CA thành công.
- **Mitigate**: kill session treo, hệ thống phục hồi ngay (~30s).
- **Root cause**: đọc code, block `catch` trong flow gọi Viettel-CA thiếu `rollback` khi throw exception phi thường (connection reset) → transaction treo giữ lock bảng.
- **Fix**: hotfix PR thêm rollback + log chi tiết, 20 phút merge & deploy.
- **Result & Metric**: **MTTD** (trung bình bao lâu phát hiện) 4 phút, **MTTR** (trung bình bao lâu phục hồi) 35 phút, ~2000 giao dịch fail nhưng retry tự động sau khi khôi phục, không mất hợp đồng nào.
- **Lesson**: (1) thêm alert "transaction treo > 30s" qua query `information_schema.INNODB_TRX WHERE TIMESTAMPDIFF(SECOND, trx_started, NOW()) > 30`, (2) lint rule bắt buộc mọi `catch` phải có `rollback` hoặc `rethrow`, (3) post-mortem chia sẻ toàn team.

**Câu xoáy đáp xoay**:
- *"MTTD 4 phút là nhờ đâu — alert tự động hay user báo?"* → Alert p99 > 1.5s đã được set từ trước, Grafana dashboard on-call luôn mở. User chưa kịp báo thì alert đã trigger rồi. Đây là lý do em rất coi trọng đặt alert ngưỡng cảnh báo, không chờ ngưỡng nguy hiểm.
- *"Sau khi kill session thì sao biết thật sự không mất dữ liệu?"* → Transaction chưa commit → tự rollback sạch. Em verify bằng cách count record trong bảng `signature` trước và sau → cùng số. Retry tự động theo **idempotency key** (bất biến lặp lại — cùng key gọi lại vẫn ra cùng kết quả) → không tạo duplicate.
- *"Nếu kill session không ổn, hệ thống vẫn treo — em làm gì tiếp?"* → Escalate ngay. Tạm rate limit API ký xuống 20%, tắt luồng Viettel-CA cũ, chuyển user sang USB token flow còn hoạt động. Mục tiêu số 1 là giảm blast radius (phạm vi ảnh hưởng), không phải tìm root cause khi đang cháy.
- *"Bài học 'bắt buộc catch có rollback' — em enforce thế nào trong CI?"* → Thêm lint rule custom (SpotBugs / PMD pattern) flag mọi catch block trong service layer không có `rollback/rethrow`. CI fail nếu vi phạm. Kết hợp với code review checklist.

> **Clarify**: "Anh muốn em kể sự cố về mặt tech sâu hay về mặt phối hợp người / quy trình ạ?" — cho phép chọn 1 câu chuyện phù hợp nhất.
> **Nếu chưa có nhiều sự cố lớn**: "Em chưa gặp sự cố Sev1 đúng nghĩa vì team em small, nhưng em đã xử lý [sự cố Sev2]. Em sẽ kể câu đó và chia sẻ cách em nghĩ nếu gặp Sev1."

---

## Câu 2. Nửa đêm có alert DB connection pool full. Bạn làm gì trong 5 phút đầu?

**Ý chính** — thứ tự 5 phút đầu, **mitigate trước điều tra sâu**:
1. **Phút 0–1 — Xác nhận alert không phải false positive**: mở dashboard DB connections, error rate API → có ảnh hưởng user thật không?
2. **Phút 1–2 — Đánh giá impact**: bao nhiêu service ảnh hưởng, API nào đang 5xx, user-facing hay internal?
3. **Phút 2–3 — Mitigation nhanh** (song song với điều tra):
   - Kill idle/sleeping connections. MariaDB: tìm trong `SHOW PROCESSLIST` các thread `Command='Sleep'` và `Time > 60`, hoặc `information_schema.INNODB_TRX` với `trx_state='RUNNING'` mà thread `Sleep` — rồi `KILL <thread_id>`. Với bulk: script select ra list id, loop KILL. Có thể set `wait_timeout` thấp hơn (mặc định 28800s → giảm xuống 600s) để DB tự dọn.
   - Tăng tạm `max_connections` / mở rộng pool app (nếu có không gian memory).
   - Hoặc restart app service đang leak connection (identify qua dashboard per-service).
4. **Phút 3–4 — Triage root cause**:
   - Có **deploy** trong 1h gần đây không? → rollback là option đầu.
   - Có **job batch / ETL** chạy không đúng giờ không?
   - Có **slow query** đang treo connection không? → `SHOW FULL PROCESSLIST` hoặc `information_schema.PROCESSLIST ORDER BY TIME DESC`.
   - Có **spike traffic** bất thường không? (DDoS, bot).
5. **Phút 4–5 — Escalate nếu chưa ổn**: báo lead / DBA on-call, mở war-room. Không cố gồng 1 mình quá 15 phút.
6. **Sau ổn định**: mở ticket điều tra sâu, post-mortem.

**Phòng trước cho tương lai**:
- Kết nối phải có **timeout + max lifetime** (**HikariCP** `maxLifetime` — thời gian sống max của connection trong pool — phải nhỏ hơn DB idle timeout, nếu không DB tự đóng connection cũ mà app vẫn giữ cái đã chết → exception).
- **Leak detection threshold** ở pool để log stack trace connection giữ quá lâu.
- Alert sớm ở 70% pool, không đợi 100%.

**Kéo về thực tế**: Dự án em từng bị similar lúc 2h sáng — cron job export báo cáo tháng quên dùng `LIMIT`, 1 query scan cả bảng 200M hàng, giữ 20 connection 15 phút. Em `KILL QUERY <id>` cho các thread đó, pool phục hồi trong 1 phút. Sau đó thêm `SET max_statement_time = 30` (MariaDB, đơn vị giây) làm default cho user `report`, job dùng user riêng có `max_statement_time` cao hơn.

**Câu xoáy đáp xoay**:
- *"Nếu kill connection mà pool vẫn full ngay lập tức thì sao?"* → Có leak đang diễn ra active. Kiểm tra ngay per-service dashboard → service nào đang tăng connection → restart service đó + đặt `max_statement_time` global/user level. Kiểm tra code service đó có chỗ nào open connection mà không close trong try-finally/try-with-resources không.
- *"Tăng max_connections có downside không?"* → Có — MariaDB dùng **thread-per-connection model**: mỗi connection = 1 thread OS, tốn ~256KB thread stack + buffer per-session (~2–3MB). Nếu tăng vượt RAM → OOM (Out of Memory, hết bộ nhớ → crash) hoặc context-switch overhead cao. Tốt hơn: (1) giới hạn pool ở app, (2) dùng **MariaDB thread pool plugin** (`thread_handling=pool-of-threads`) — không phải thread-per-connection mà share thread — scale tốt hơn ở connection count cao, (3) đặt **MaxScale** hoặc **ProxySQL** làm connection proxy (tương tự PgBouncer của Postgres, multiplex connection).
- *"maxLifetime phải nhỏ hơn DB idle timeout — cụ thể con số thế nào với MariaDB?"* → Công thức: `HikariCP maxLifetime < MariaDB wait_timeout`. MariaDB mặc định `wait_timeout=28800s` (8h) — quá dài cho production. Em set xuống `600s` (10 phút), và HikariCP `maxLifetime=580s`. Thêm `idleTimeout=300s` ở pool để trả connection sớm. Nếu có firewall/NAT ở giữa cắt idle connection sớm hơn, phải tính cả đó.

> **Clarify**: "Lúc đó em là on-call primary hay có DBA trực cùng ạ?" — ảnh hưởng mức độ em tự xử lý vs escalate.

---

## Câu 3. Service chính bị chậm bất thường từ 8h sáng. Quy trình điều tra?

**Ý chính** — "8h sáng" là gợi ý quan trọng (traffic pattern):
1. **Hypothesis dựa trên thời điểm**: 8h thường là giờ người dùng bắt đầu → traffic tăng đột biến sau đêm ít traffic → **cold cache** (cache trống sau khi service restart), **connection pool warm-up** (pool chưa mở đủ connection), **JIT chưa tối ưu** (JIT — Just-In-Time compiler của JVM, mã chạy lần đầu chậm hơn mãi về sau), **autoscaler chưa kịp thêm pod**.
2. **4 hướng điều tra song song**:
   - **Traffic**: QPS (queries per second) có tăng bất thường không, phân bố endpoint có đổi không?
   - **Resource**: CPU/RAM/GC/disk IO của service — có bottleneck rõ không?
   - **Downstream**: DB / cache / service ngoài (Viettel-CA / SmartCA) — latency của từng hop (qua trace phân tán).
   - **Deploy / config**: đã có thay đổi nào đêm qua (cron deploy, config rollout)?
3. **Công cụ**: Grafana dashboard **RED** (Rate — số request, Error — tỉ lệ lỗi, Duration — thời gian xử lý), APM (Jaeger/Tempo cho distributed tracing), `performance_schema.events_statements_summary_by_digest` (MariaDB — thống kê query theo template digest, tương đương `pg_stat_statements`) hoặc **slow query log** (`long_query_time=0.5s`), log error tăng đột biến.
4. **Hay gặp ở "8h"**:
   - **Cache stampede** khi key **TTL** (Time-To-Live, thời gian sống của key) qua đêm hết cùng lúc — fix jitter TTL (thêm số ngẫu nhiên vào TTL).
   - **Autoscale lag**: **HPA** (Horizontal Pod Autoscaler — tự động thêm bớt pod) phản ứng chậm 2–3 phút, pod mới chưa warm → thêm warm-up probe + min replicas cao hơn.
   - **GC spike** do object tích tụ đêm (cron job, background task không clean).
   - **DB plan đổi** sau khi statistics được refresh (MariaDB InnoDB tự recalc khi data đổi ~10% với `innodb_stats_auto_recalc=ON`, hoặc sau `ANALYZE TABLE` thủ công → optimizer có thể chọn plan khác → query chậm bất ngờ).
   - **Batch job nặng** chạy 7h–8h chia sẻ DB/cache → tách workload.
5. **Mitigate tức thời**: tăng replica tay, bật rate limit, failover replica, rollback config đêm qua.

**Kéo về thực tế**: Dự án em có lần service `contract-service` chậm mỗi sáng thứ 2. Truy ra là cron job gen báo cáo tuần chạy 7:30 sáng share cùng DB. Em move job sang replica read-only + chạy 3h sáng. Giải quyết gọn, không cần scale tăng.

**Câu xoáy đáp xoay**:
- *"Cache stampede lúc 8h — em detect qua metric gì cụ thể?"* → Tỉ lệ cache hit/miss ratio đột ngột giảm về 0 + DB QPS (số query mỗi giây) tăng đột biến cùng lúc service chậm. Redis có metrics `keyspace_hits` vs `keyspace_misses` — visualize trên Grafana thấy ngay. Nếu hit rate từ 95% xuống 20% đúng 8h → stampede.
- *"HPA phản ứng chậm 2–3 phút — cách giải quyết ngoài tăng minReplicas?"* → Dùng **KEDA** (Kubernetes Event-Driven Autoscaling — scale dựa trên event như message queue length thay vì chỉ CPU), phản ứng nhanh hơn theo metric business. Hoặc **Predictive scaling** — schedule thêm pod trước 7:50 dựa vào historical pattern.
- *"DB plan đổi sau ANALYZE — cụ thể em debug thế nào với MariaDB?"* → Query `performance_schema.events_statements_summary_by_digest` so sánh `avg_timer_wait` của query digest hôm nay vs hôm qua. Nếu 1 digest đột ngột tăng 10×, chạy `EXPLAIN FORMAT=JSON <query>` hoặc `ANALYZE SELECT ...` (MariaDB 10.1+ — chạy thật + show plan). Nếu optimizer chọn full table scan thay vì index → dùng optimizer hint `SELECT /*+ INDEX(t, idx_name) */` hoặc `FORCE INDEX (idx_name)` tạm để test. Fix dài hạn: `ANALYZE TABLE <table>` refresh stats, hoặc `ALTER TABLE ... STATS_SAMPLE_PAGES=64` để InnoDB lấy nhiều sample hơn.

> **Clarify**: "Chậm là **p95** hay **p99** (percentile — 95% hay 99% request nhanh hơn con số này)? Có cụ thể endpoint nào không, hay toàn service chậm đều ạ?" — câu trả lời định hướng điều tra.

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
   - **Expand/contract migration** (tách migration thành 2 deploy — deploy 1 chỉ thêm column/index "expand", deploy 2 xóa cái cũ "contract" — để lúc nào cũng rollback được): deploy 1 ADD COLUMN nullable, deploy 2 migrate data + switch code, deploy 3 DROP COLUMN cũ.
   - **Feature flag** (bật/tắt feature qua config, không cần rollback code): bật/tắt feature mới qua flag, không cần rollback code.
   - **Canary deploy** (deploy cho 5% user trước, quan sát, rồi mới 100%): 5% → 25% → 100%, phát hiện sớm ở 5%.
4. **Quy trình khi rollback**:
   - Thông báo war-room ngay.
   - Rollback artifact (image version cũ) trước, đừng cố "fix quick" trên master.
   - Sau khi ổn, tách branch để điều tra — không vội redeploy.

**Kéo về thực tế**: Dự án em có quy ước: ai deploy là người quyết rollback. Rule em hay dùng: **"Nếu sau 5 phút chưa rõ root cause → rollback, không thương lượng"**. Có lần em tưởng biết nguyên nhân (chỉ là missing env var), định fix forward, hóa ra còn kèm 1 bug config khác → rollback xong mới ngồi fix 2 bug trong 30 phút, thay vì cố gồng sửa trên prod 1 tiếng.

**Câu xoáy đáp xoay**:
- *"Expand/contract — cho ví dụ cụ thể với trường hợp đổi tên column?"* → Deploy 1: ADD COLUMN `email_new VARCHAR` (nullable). Code ghi vào cả `email_old` và `email_new`. Deploy 2: backfill `email_new` từ `email_old`, đổi code đọc từ `email_new`. Deploy 3: DROP COLUMN `email_old`. Bất kỳ deploy nào fail đều rollback được vì code cũ vẫn đọc `email_old` còn đó.
- *"Canary 5% — em monitor metric nào để quyết định tăng lên 25%?"* → Error rate của canary pod phải tương đương hoặc thấp hơn baseline. p99 latency không tăng hơn 10%. Nếu có business metric (ví dụ số hợp đồng ký thành công/phút) thì so sánh proportional. Sau 5–10 phút stable → promote.
- *"Feature flag — em dùng tool gì? Có overhead không?"* → Em dùng database table `feature_flags` + cache Redis TTL 60s. Overhead ~1ms/request (1 Redis read). Với Viettel scale lớn có thể dùng self-hosted **Unleash** hoặc tích hợp với config service nội bộ.

> **Clarify**: "Tình huống này có migration DB kèm deploy không ạ?" — yếu tố quyết định rollback khả thi.
> **Bẫy cần tránh**: đừng trả lời "em sẽ phân tích rồi mới quyết" một cách chung chung — interviewer muốn nghe **quyết định nhanh** và nguyên tắc ra quyết định.

---

## Câu 5. Log file bị đầy ổ cứng, server không ghi log được nữa. Phòng thế nào?

**Ý chính** — tách **mitigate khẩn** và **phòng lâu dài**:
1. **Mitigate tức thời**:
   - Xóa file log cũ (giữ lại file đang ghi để không rotate giữa chừng).
   - `truncate -s 0` với file log đang ghi thay vì `rm` (**truncate** giảm size về 0 ngay lập tức mà không cần restart process, disk space giải phóng ngay; còn **rm** xóa inode nhưng file descriptor giữ space cho đến khi process đóng).
   - Restart service nếu file descriptor bị kẹt.
2. **Phòng lâu dài — 5 lớp**:
   - **Log rotation**: `logrotate` rotate theo size (100 MB) + age (7 ngày), nén `.gz`, xóa khi vượt quota.
   - **Centralized logging** (log tập trung): log đẩy về **Fluent Bit** (log forwarder nhẹ < 1MB RAM, chạy trên mỗi pod) → **Kafka** → **Elasticsearch/Loki**, KHÔNG giữ trên server app. Server chỉ buffer ngắn.
   - **Alert sớm**: disk usage > 70% → alert warning, > 85% → page.
   - **Retention policy** theo loại log: access log 3–7 ngày trên local, audit log đẩy sang S3 (giữ theo luật), debug log chỉ bật khi cần.
   - **Log hygiene** (vệ sinh log): log JSON structured, không log PII (OTP, chữ ký raw, CCCD); **sample** `DEBUG`/`INFO` ở production (không log 100% request — chỉ log 1/100 để tiết kiệm volume).
3. **Anti-pattern cần tránh**:
   - Log ra 1 file duy nhất không rotate.
   - Log trong vòng lặp nóng (thousands of events/s) → gây chính nó đầy disk.
   - Dùng `cat` / `tail` debug trên server không có rotation.

**Kéo về thực tế**: Dự án em từng có service log mọi request body (bao gồm PDF hợp đồng base64) vì 1 dev enable `DEBUG` tạm mà quên tắt. 1 ngày đầy 50 GB disk. Sau đó em: (1) đổi log format strict JSON schema, (2) redact rule bắt buộc trước khi ghi log, (3) alert disk 70% cho mỗi node, (4) đẩy log trung tâm qua Fluent Bit để local chỉ giữ buffer.

**Câu xoáy đáp xoay**:
- *"truncate -s 0 khác rm ở chỗ nào, khi nào dùng cái nào?"* → `rm` xóa inode nhưng nếu process còn giữ file descriptor, OS vẫn chiếm disk space đó cho đến khi process đóng. `truncate -s 0` làm file size = 0 ngay lập tức, OS giải phóng disk, process tiếp tục ghi vào file đó từ vị trí 0 mà không cần restart. Dùng `truncate` khi cần giải phóng disk khẩn mà không muốn restart service.
- *"Fluent Bit vs Logstash — khi nào dùng cái nào?"* → Fluent Bit nhẹ (< 1MB RAM, viết bằng C), dùng làm **forwarder** trên mỗi pod/VM để thu log và đẩy lên. Logstash nặng (~300MB+), dùng làm **aggregator** trung tâm để xử lý/transform phức tạp trước khi vào ES. Pipeline điển hình: `Fluent Bit (pod)` → `Kafka` → `Logstash (enrichment)` → `Elasticsearch`.
- *"Sample log — cụ thể implement thế nào trong Java?"* → SLF4J với `MDC` + custom filter: tạo Logback `SamplingFilter` — mỗi N request chỉ log 1 (dùng `AtomicLong counter % N == 0`). Luôn log ERROR/WARN đầy đủ, chỉ sample INFO/DEBUG. Hoặc dùng OpenTelemetry sampling để consistent với trace.

> **Clarify**: "Server này là VM hay container ạ? Container thường có stdout bị giới hạn bởi Docker log driver, phòng khác hơn chút."

---

## Câu 6. Message queue bị dồn 1 triệu message chưa consume. Làm gì?

**Ý chính** — 4 bước:
1. **Triage root cause** trước khi tăng consumer mù:
   - Consumer **crash loop**? (check logs, OOM — hết bộ nhớ, exception)
   - Consumer **chậm** do downstream (DB slow, external API chậm)?
   - Producer **spike** bất thường?
   - **Poison message** (message làm consumer crash mỗi lần xử lý) → retry vô hạn, block partition?
2. **Mitigate tùy root cause**:
   - Nếu crash: fix bug / revert deploy consumer → restart.
   - Nếu chậm do downstream: tạm scale consumer hoặc **batch processing** (thay vì 1 msg / call, gộp 100 msg).
   - Nếu producer spike: rate limit / tạm hoãn producer non-critical.
   - Nếu poison message: chuyển vào **DLQ** (Dead Letter Queue — hàng đợi chứa message không xử lý được) ngay, đừng để nó block partition.
3. **Tăng throughput consumer** (nếu CPU/IO còn dư):
   - Tăng **partition** (Kafka — mỗi partition là 1 luồng song song) — nhưng phải đảm bảo ordering vẫn đúng theo key.
   - Tăng **consumer instance** trong group.
   - Tăng `max.poll.records` + batch commit.
4. **Đối soát sau khi tiêu thụ xong**:
   - Check tổng sent vs tổng processed; các message DLQ có cần retry thủ công không.
   - Post-mortem: thêm alert consumer lag > 5 phút, run-book.

**Kéo về thực tế**: Dự án em có lần backlog 200k message topic `contract.signed` vì consumer gọi TSA (timestamp authority — cơ quan cấp dấu thời gian cho chữ ký) chậm do TSA bị tấn công slowloris (tấn công làm server mở nhiều connection nhưng gửi dữ liệu cực chậm). Em tạm thời **bypass TSA** (ghi message vào buffer topic `contract.signed.pending-tsa`) để unblock các consumer khác (email, audit), sau đó chạy riêng consumer TSA với retry backoff. Backlog tiêu trong 2 giờ.

**Câu xoáy đáp xoay**:
- *"Poison message — em detect thế nào nếu không có monitoring?"* → Consumer lag tăng mà consumer restart liên tục + error log cùng message ID lặp lại. Thủ công: xem `offset` của partition không tăng dù consumer đang chạy. Với Kafka: `kafka-consumer-groups.sh --describe` xem lag per partition. Fix: set `max.delivery.count` (số lần retry tối đa), sau đó route sang **DLQ**.
- *"Tăng partition Kafka — có thay đổi được sau khi đã có data không?"* → Được tăng số partition, nhưng không giảm. Tuy nhiên khi tăng partition, **key → partition mapping thay đổi** (vì `hash(key) % num_partitions` khác đi) → ordering theo key bị phá vỡ. Với message đã có trong partition cũ vẫn xử lý đúng, nhưng message mới cùng key có thể vào partition khác. Cần assess business impact trước khi tăng.
- *"bypass TSA rồi reconcile sau — em đảm bảo không miss message nào?"* → Ghi vào topic `contract.signed.pending-tsa` với trạng thái `TSA_PENDING`. Consumer riêng xử lý retry với **exponential backoff + jitter** (chờ gấp đôi mỗi lần + thêm random để tránh retry đồng loạt). Alert nếu message `TSA_PENDING` > 1 giờ không clear → human intervention. Sau xong reconcile count: tổng `TSA_DONE` + `TSA_FAILED` = tổng ban đầu.

> **Clarify**: "Queue là Kafka, RabbitMQ hay SQS ạ? Vì cách scale khác nhau — Kafka giới hạn bởi số partition, RabbitMQ giới hạn bởi consumer prefetch count."

---

## Câu 7. API bên thứ 3 bạn phụ thuộc bị chết 2 giờ. Hệ thống bạn phản ứng thế nào?

**Ý chính**:
1. **Phát hiện nhanh**:
   - Health check + synthetic probe (cron gọi test request mỗi 30s) gọi third-party.
   - Alert khi error rate > 5% hoặc latency > threshold.
2. **Hạn chế lan rộng**:
   - **Circuit breaker open** (cầu chì — khi downstream lỗi nhiều, tự ngắt gọi trong X giây) — dừng gọi third-party, trả lỗi nhanh thay vì treo. 3 trạng thái: CLOSED (bình thường) → OPEN (đã ngắt, fail fast) → HALF-OPEN (thử 1 request xem đã recover chưa).
   - **Bulkhead** (vách ngăn — tách pool resource để 1 phần chết không kéo cả hệ thống): tách pool thread gọi third-party, không cho nó chiếm pool chung.
   - **Timeout ngắn** (3–5s), không chờ vô hạn.
3. **Graceful degradation** (xuống cấp có kiểm soát — tức giảm chức năng nhưng vẫn hoạt động):
   - Trả dữ liệu **cached** nếu có.
   - **Queue request** để xử lý sau (nếu async chấp nhận được).
   - **Trả lỗi rõ ràng** cho user: "Dịch vụ X đang gián đoạn, xin thử lại sau N phút" — KHÔNG để user thấy generic 500.
   - **Fallback provider**: nếu có 2 nhà cung cấp, switch sang cái còn lại.
4. **Chủ động với nhà cung cấp**:
   - Liên hệ qua channel support / status page của họ.
   - Nếu có SLA breach, ghi nhận để claim sau.
5. **Sau khi phục hồi**:
   - Reconcile: các request đã queue/pending cần retry theo **backoff jitter** (tránh retry storm ngay khi họ sống lại — hàng nghìn request đồng loạt đập vào).
   - Post-mortem.

**Kéo về thực tế**: Dự án em phụ thuộc **Viettel-CA** và **SmartCA** cho luồng ký. Có lần SmartCA bảo trì không thông báo trước, down 90 phút. Em có sẵn 2 fallback: (1) user có USB token → redirect sang flow USB, (2) user chỉ có SmartCA → hiển thị banner "SmartCA đang gián đoạn, vui lòng thử sau N phút" + gửi notification khi CA khôi phục. Không mất hợp đồng nào, chỉ delay 90 phút.

**Câu xoáy đáp xoay**:
- *"Circuit breaker 3 trạng thái — threshold để chuyển là gì?"* → CLOSED → OPEN: khi error rate > X% trong cửa sổ Y giây (ví dụ: 5 lỗi trong 10s). OPEN → HALF-OPEN: sau `resetTimeout` (ví dụ 30s). HALF-OPEN → CLOSED/OPEN: nếu 1 test request thành công → CLOSED; nếu fail → về OPEN. Dùng thư viện **Resilience4j** (Java) để config các threshold này.
- *"Nếu circuit breaker OPEN mà user vẫn cần kết quả realtime — xử lý sao?"* → Trả lỗi rõ ràng ngay với status code phù hợp (503 Service Unavailable) + `Retry-After` header cho biết khi nào thử lại. Không bao giờ để user chờ timeout. Với flow thanh toán/ký không thể queue → reject ngay, hiển thị message thân thiện.
- *"Retry storm sau khi CA recover — cụ thể exponential backoff + jitter là gì?"* → Mỗi lần retry: `delay = min(base * 2^attempt, maxDelay) + random(0, jitter)`. Ví dụ: attempt 1 = 1s + 0.5s random, attempt 2 = 2s + 1s random, attempt 3 = 4s + 2s random... Jitter phòng nhiều consumer cùng retry đúng lúc CA vừa recover.

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
   - Viết script `SELECT` ra danh sách record sẽ bị đụng.
   - Review với team / lead.
   - Backup bảng liên quan (`mysqldump --single-transaction <db> <table>` cho bảng nhỏ, `mariabackup` cho full/incremental backup ở production scale, hoặc snapshot disk).
4. **Sửa data**:
   - Trong **transaction** (giao dịch — tất cả hoặc không gì), `UPDATE` theo batch (1k–10k record / batch) để không lock bảng lâu.
   - Log lại before/after mỗi record vào bảng `data_fix_audit` — phòng trường hợp cần rollback.
   - **Rate-limit batch** (ví dụ sleep 100ms giữa batch) để không ngạt **replication** (quá trình đồng bộ dữ liệu từ master sang slave).
5. **Verify**:
   - Sau fix, chạy lại query pattern data sai → kỳ vọng 0 record.
   - Random spot-check 10–20 record bằng tay.
6. **Thông báo**:
   - Nếu data sai ảnh hưởng user → notify và post-mortem công khai.
   - Nếu là data nội bộ → ghi chú trong post-mortem.

**Kéo về thực tế**: Dự án em từng có bug: khi hủy hợp đồng, script không clear trường `last_signed_by`, khiến UI vẫn hiển thị "đã ký bởi X" dù hợp đồng đã hủy. Em: (1) fix code, (2) query tìm tất cả hợp đồng `status='CANCELLED'` nhưng còn `last_signed_by != null` — ra ~800 record, (3) dry-run UPDATE có WHERE chặt + LIMIT 100 trước, (4) chạy batch trong transaction, log vào `data_fix_audit`, (5) verify bằng query đối chứng. Toàn bộ 40 phút.

**Câu xoáy đáp xoay**:
- *"Backup bảng trước khi fix — dùng cách nào với MariaDB?"* → 3 cách: (1) `mysqldump --single-transaction --skip-lock-tables <db> <table> > backup.sql` — không lock bảng (InnoDB snapshot qua transaction consistent read), phù hợp bảng nhỏ–trung (< 10GB). (2) `CREATE TABLE backup_signature_20260419 LIKE signature; INSERT INTO backup_signature_20260419 SELECT * FROM signature WHERE <điều kiện>` — nhanh, trong DB luôn, rollback dễ. (3) **mariabackup** (thay `xtrabackup` của MySQL) — full/incremental backup ở production scale, hỗ trợ point-in-time recovery qua binlog. (4) Snapshot disk EBS/GCP nếu có quyền infra.
- *"Sleep 100ms giữa batch — tại sao không chạy càng nhanh càng tốt?"* → `UPDATE` lớn tạo nhiều **binlog + redo log** (MariaDB — binlog để replicate sang slave, redo log cho crash recovery) → **replication lag** (slave bị tụt lại sau master). Slave bị lag → query đọc từ slave trả về stale data. Check qua `SHOW SLAVE STATUS` cột `Seconds_Behind_Master`. Sleep 100ms giữa batch cho slave kịp sync. Cũng tránh contention disk với production traffic và giảm áp lực InnoDB **purge thread** (cleanup undo log).
- *"5 Whys để tìm root cause — cho ví dụ cụ thể với bug này?"* → (1) Data sai "last_signed_by không clear khi cancel" — Tại sao? (2) Cancel handler thiếu clear field → Tại sao? (3) Code review không catch → Tại sao? (4) Không có test case cho state transition ACTIVE→CANCELLED → Tại sao? (5) Thiếu quy định test coverage cho state machine. Root cause thật: thiếu test standard cho state transition, không phải lỗi cẩu thả cá nhân.

> **Clarify**: "Data bị sai là lệch 1 trường cụ thể hay nhiều bảng liên quan ạ? Nhiều bảng thì phải xử lý theo thứ tự foreign key."
> **Nguyên tắc cứng**: KHÔNG UPDATE trực tiếp trên production không có WHERE chặt. Luôn SELECT trước, dry-run trước.

---

## Câu 9. Redis cache bị flush sạch lúc peak. Hiện tượng gì xảy ra và cách phòng?

**Ý chính**:
1. **Hiện tượng**:
   - **Thundering herd** ("đàn trâu xô cửa") vào DB — mọi request đều cache miss.
   - DB CPU/IO vọt lên → latency API tăng 10–100×.
   - Nếu DB quá tải → cascade: API timeout → upstream retry → càng ngạt.
2. **Nguyên nhân Redis bị flush**:
   - Ops chạy `FLUSHALL` / `FLUSHDB` nhầm (hay gặp).
   - Redis restart (maxmemory chưa persist).
   - Failover sang replica chưa warm.
   - **Eviction policy** `allkeys-lru` khi đầy memory (LRU — Least Recently Used, tự xóa key ít dùng nhất khi hết RAM).
3. **Cách phòng**:
   - **Single-flight** (1 key miss, chỉ 1 request đi DB, các request khác chờ): Redis Lua script hoặc distributed mutex (khóa phân tán).
   - **TTL có jitter**: tránh hết hạn đồng loạt — `TTL = base + random(0, jitter)`.
   - **Pre-warm sau restart**: chạy script warm các key top-N trước khi nhận traffic.
   - **L2 cache in-process** (cache trong RAM của JVM — Caffeine/Guava): lớp đệm giữa app và Redis, cực nhanh (nanoseconds) vì không qua network.
   - **Rate limit tầng ngoài** để nếu Redis chết, DB không bị ngạt quá ngưỡng.
   - **Redis Cluster** với replica đồng bộ — flush 1 node không mất tất cả.
   - **ACL** (Access Control List): disable `FLUSHALL` cho account production, chỉ admin có key riêng.
4. **Mitigate khi đang xảy ra**:
   - Bật rate limit ở gateway (giảm 50% traffic).
   - Scale DB read replica tạm.
   - Pre-warm các key nóng trực tiếp từ script đồng bộ.

**Kéo về thực tế**: Dự án em đã mô tả ở **Câu 3 Phần 1** — lần Redis restart làm DB ngạt. Giải pháp em làm: single-flight Lua + TTL jitter + invalidate qua Kafka event. Lần flush kế tiếp DB chỉ tăng từ 500 → 800 QPS (không phải 10k).

**Câu xoáy đáp xoay**:
- *"Single-flight Lua script — cụ thể implement thế nào?"* → Script Lua chạy atomic (nguyên tử — không bị interrupt) trong Redis: `SETNX lock_key 1 EX 5` (SET if Not eXists với TTL 5s). Nếu thành công → thread này đi DB. Các thread khác SETNX fail → `WAIT key value` hoặc poll ngắn → sau khi thread đầu ghi cache xong, thread sau đọc cache thay vì DB. Tổng số request DB = 1 thay vì N.
- *"TTL jitter — code Java cụ thể thế nào?"* → `long ttl = BASE_TTL + ThreadLocalRandom.current().nextLong(0, JITTER_MAX)`. Ví dụ `BASE_TTL = 3600s, JITTER_MAX = 600s` → TTL rải đều trong 3600–4200s. Không dùng `Random` thường (thread-unsafe), dùng `ThreadLocalRandom`.
- *"L2 cache in-process — downside gì? Khi nào không nên dùng?"* → Downside: (1) mỗi pod có bản riêng → stale nếu data đổi trên Redis (inconsistency giữa các pod), (2) tốn heap JVM. Không nên dùng cho data thay đổi thường xuyên (session user, giỏ hàng). Phù hợp cho data ít thay đổi (config, feature flag, master data, public key certificate).

> **Clarify**: "Redis là standalone hay cluster ạ? Cluster nửa node chết còn phân nửa data."
> **Câu vặn hay gặp**: "Nếu single-flight cũng timeout thì sao?" → Fallback: trả **stale data** nếu có (từ L2 cache), hoặc trả lỗi rõ kèm `Retry-After`, không treo user.

---

## Câu 10. Có memory leak ở Java service. Cách phát hiện và fix?

**Ý chính**:
1. **Phát hiện**:
   - **Metric heap** tăng đều, không giảm sau **GC** (Garbage Collection — thu gom rác, tự giải phóng memory không còn dùng) — triệu chứng kinh điển.
   - **Full GC tần suất cao** nhưng heap không giảm → generation cũ đầy object còn reference.
   - **OutOfMemoryError** cuối cùng, nhưng nên bắt sớm qua metric heap.
   - Monitor: Prometheus JVM exporter, hoặc APM (New Relic / Datadog).
2. **Công cụ chẩn đoán**:
   - **Heap dump** (`jmap -dump:format=b,file=heap.hprof <pid>` hoặc flag `-XX:+HeapDumpOnOutOfMemoryError`).
   - Phân tích với **Eclipse MAT** hoặc **VisualVM**: tìm **dominator tree** (cây thống trị — object nào đang giữ nhiều memory nhất), object lớn nhất.
   - **async-profiler** / **JFR** (Java Flight Recorder) cho allocation profiling (xem code nào đang cấp phát memory nhiều nhất).
   - `jstat -gc` theo dõi GC real-time.
3. **Pattern leak phổ biến**:
   - **Collection tĩnh** (`static Map`) không bao giờ clear.
   - **Listener / callback** đăng ký mà không unregister.
   - **ThreadLocal** không `remove()` trong pool thread → leak theo pool size (thread pool giữ reference đến object cũ mãi mãi).
   - **Cache không có bounded size** (HashMap không giới hạn thay vì Caffeine có max size + TTL).
   - **Connection / Stream / ResultSet** không close — leak native memory lẫn heap.
4. **Fix**:
   - Clear reference khi không dùng. Dùng `try-with-resources` cho mọi Closeable.
   - **WeakHashMap** / **SoftReference** cho cache không critical (**Soft** — GC thu khi memory tight; **Weak** — GC thu ngay khi không còn strong reference).
   - Bounded cache (Caffeine với max size + TTL).
5. **Mitigate tạm**:
   - Restart service định kỳ (cron) nếu chưa fix kịp — không phải fix, chỉ dời hẹn.
   - Scale ngang để giảm load mỗi instance.

**Kéo về thực tế**: Dự án em có service xử lý PDF hợp đồng dùng thư viện PDF bên thứ ba. Em phát hiện heap tăng đều 100 MB/ngày. Heap dump với MAT → dominator là `PdfReader` giữ font cache global. Bug là thư viện đó cache font theo class loader, mỗi request tạo loader mới (do dynamic class) → leak. Fix bằng cách (1) share `PdfReader` singleton, (2) thêm bounded cache manual. Heap stable sau đó.

**Câu xoáy đáp xoay**:
- *"Heap dump trong production có rủi ro gì không?"* → Có — `jmap` pause JVM trong thời gian dump (vài giây đến vài phút tùy heap size). Với production traffic → visible latency spike. Em dùng flag `-XX:+HeapDumpOnOutOfMemoryError` (tự dump khi OOM, không pause chủ động) hoặc **async-profiler** (ít invasive, không full pause). Nếu phải dump chủ động, làm lúc traffic thấp.
- *"ThreadLocal không remove() — trong thread pool cụ thể leak thế nào?"* → Thread pool tái sử dụng thread. Nếu `ThreadLocal.remove()` không được gọi sau mỗi request, thread next request trên cùng thread đó vẫn thấy value cũ (sai dữ liệu) VÀ object cũ không bị GC thu (vì thread đang chạy có strong reference). Với pool 200 thread, leak tích tụ theo số lượng request × size của object.
- *"WeakReference khác SoftReference khác StrongReference — khi nào dùng cái nào cho cache?"* → **Strong** (bình thường): GC không thu bao giờ trừ khi mình clear reference. **Soft**: GC thu khi JVM sắp OOM — tốt cho cache (GC sẽ tự evict khi memory tight). **Weak**: GC thu ngay khi không còn strong reference nào khác — `WeakHashMap` dùng cho mapping mà key có thể bị GC mà không cần cleanup thủ công.

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

**Câu xoáy đáp xoay**:
- *"CI check 2 reviewer migration — implement thế nào, người ta có thể bypass không?"* → GitHub **CODEOWNERS** file: gán DBA/lead là required reviewer cho pattern `src/main/resources/db/migration/**`. Kết hợp với **branch protection rule**: require 2 approvals, dismiss stale reviews, include administrators. Không có reviewer đúng approving thì CI block merge, không bypass được (trừ admin force merge — nhưng audit log ghi lại).
- *"Merge + deploy tách 30 phút — quy trình cụ thể thế nào?"* → Merge PR vào main → CI build + test (5–10 phút) → artifact push lên registry → **deploy staging + smoke test** tự động → chờ 15–30 phút monitor staging metrics → deploy prod. Không deploy prod ngay sau merge để CI alert còn thời gian bubble up.
- *"Lần sau em có thể cause sự cố tương tự không?"* → Luôn có thể cause loại sự cố khác. Em không cam kết "không bao giờ lại cause sự cố" — điều đó không thực tế. Em cam kết: (1) không cause cùng loại 2 lần, (2) khi cause thì phát hiện nhanh hơn (MTTD thấp hơn) và phục hồi nhanh hơn (MTTR thấp hơn).

> **Clarify** (khi cần nhiều thời gian): "Anh muốn em kể 1 sự cố tech em cause, hay 1 quyết định thiết kế em về sau thấy sai ạ?"
> **Tuyệt đối không**: "em rất cẩn thận nên chưa cause sự cố gì". Interviewer hiểu 1 là nói dối, 2 là chưa từng chạm vào production thật.

---

## Câu 12. Post-mortem là gì? Khi nào cần? Nội dung gồm gì?

**Ý chính**:
1. **Định nghĩa**: Tài liệu viết sau incident (Sev1/Sev2 bắt buộc, Sev3 khuyến khích) để phân tích toàn bộ sự cố, học bài học, ra action items chống tái diễn.
2. **Nguyên tắc tối quan trọng — BLAMELESS** (không đổ lỗi người):
   - Trọng tâm là **hệ thống / quy trình**, không phải người.
   - Không "ai là người làm sai" mà "quy trình / công cụ nào đã để 1 người duy nhất có thể gây sự cố này".
   - Người cause sự cố thường là người hiểu nhất → nên chủ trì hoặc tham gia viết, không bị punish.
3. **Nội dung chuẩn — 8 mục**:
   - **Summary** (3–5 câu): chuyện gì xảy ra, ai bị ảnh hưởng, hậu quả.
   - **Timeline**: mốc thời gian UTC — detect lúc X, mitigate lúc Y, fix lúc Z.
   - **Impact**: số user ảnh hưởng, số giao dịch fail, thiệt hại tiền (nếu tính được), SLA (cam kết dịch vụ với khách hàng) breach không.
   - **Root cause**: nguyên nhân gốc (thường dùng **5 Whys** — hỏi "Tại sao?" 5 lần liên tiếp — hoặc fishbone diagram).
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

**Câu xoáy đáp xoay**:
- *"5 Whys có nhược điểm gì không?"* → Có — đơn tuyến (đào 1 nhánh duy nhất), bỏ qua **contributing factors** song song. Với sự cố phức tạp, dùng **Ishikawa diagram** (fishbone — đầu cá là sự cố, xương cá là các nguyên nhân theo nhóm: people/process/tool/environment) để map nhiều nguyên nhân cùng lúc. 5 Whys tốt cho sự cố đơn giản, rõ 1 nguyên nhân.
- *"Action items bị orphan không ai follow — em phòng thế nào?"* → Mỗi action item phải có: **1 owner cụ thể** (không phải "team"), **deadline cụ thể**, **mức ưu tiên P0/P1/P2**. Ghi vào Jira/tracker ngay trong meeting review, không chỉ để trong doc Confluence. Review weekly ở sprint planning cho đến khi close. Metric: % action items close trong deadline.
- *"Post-mortem blameless có đồng nghĩa với 'không ai chịu trách nhiệm' không?"* → Không. Blameless là về **cách phân tích** (tập trung vào system/process, không đổ lỗi người), không phải về **accountability** (trách nhiệm cá nhân). Người cause vẫn cần tự nhận, học, và cam kết đổi hành vi — nhưng công cụ là training & process improvement, không phải kỷ luật hay mắng.
- *"MTTD / MTTR — em có target cụ thể cho hệ thống mình không?"* → Em đặt: Sev1 MTTD < 5 phút (alert + paging tự động), MTTR < 30 phút. Sev2 MTTD < 15 phút, MTTR < 2 giờ. Đo bằng timestamp trong post-mortem, aggregate hàng tháng. Nếu MTTD vượt target → review alert coverage. Nếu MTTR vượt → review run-book và on-call training.

> **Clarify**: "Viettel có template post-mortem nội bộ không anh? Em muốn follow theo nếu có, vì mỗi công ty có format hơi khác."

---

> **Hết 12 câu Phần 2.** File tiếp theo: `phan-3-deal-luong.md`.

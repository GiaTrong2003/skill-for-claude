# Phần 1 — Hệ thống lớn & Kiến trúc — Bộ câu trả lời

> **Format chung** cho mỗi câu:
> - **Ý chính** — danh sách đánh số, chắc và an toàn.
> - **Kéo về thực tế** — 1 đoạn ngắn, có thể neo vào dự án **Hợp đồng điện tử** (tích hợp **Viettel-CA, USB token, SmartCA**) hoặc kiến thức chung.
> - **Câu xoáy đáp xoay** — những câu follow-up sếp hay hỏi để test hiểu sâu + cách xử lý.
> - **> Clarify:** câu hỏi lại interviewer (khi cần).
> - **> Nếu chưa chắc:** cách thừa nhận + đề xuất tìm hiểu.
>
> **Quy ước**: thuật ngữ khó sẽ có giải thích trong ngoặc bằng **ngôn ngữ đời thường** ngay bên cạnh — đọc thuộc đi kèm sẽ hiểu ngay khi nói.

---

## Bảng thuật ngữ tham chiếu nhanh

| Thuật ngữ | Giải thích đời thường |
|---|---|
| **p95 / p99** | Percentile — 95% (99%) request nhanh hơn con số này, chỉ 5% (1%) chậm hơn. Đo "đuôi" của latency, quan trọng hơn trung bình. |
| **QPS / TPS** | Query/Transaction per second — số request/giao dịch mỗi giây. |
| **ACID** | Atomicity (tất cả hoặc không gì), Consistency (dữ liệu luôn đúng ràng buộc), Isolation (các transaction không giẫm chân nhau), Durability (commit rồi không mất) — đặc tính transaction của DB quan hệ. |
| **BASE** | Basically Available, Soft state, Eventual consistency — ngược với ACID, ưu tiên "luôn trả lời được", chấp nhận dữ liệu lệch tạm. |
| **Eventual consistency** | Nhất quán cuối — các node có thể lệch nhau trong vài giây/phút, rồi sẽ đồng bộ lại. |
| **B-tree** | Cây cân bằng nhiều nhánh, cấu trúc lưu trữ giúp tìm theo khóa nhanh O(log n). |
| **Selectivity** | Độ "lọc" của 1 cột — giá trị càng đa dạng, index càng hiệu quả. Cột chỉ 2 giá trị → selectivity thấp. |
| **Seq scan** | Sequential scan — quét toàn bộ bảng từ đầu, không dùng index. |
| **WAL** | Write-Ahead Log — ghi log trước khi commit data, giúp khôi phục khi crash. |
| **TTL** | Time-To-Live — thời gian sống của 1 key trong cache, hết hạn là tự xóa. |
| **Thundering herd** | "Đàn trâu xô cửa" — nhiều request cùng miss cache 1 lúc → cùng đập DB. |
| **Idempotent** | Bất biến lặp lại — gọi API 1 lần hay 5 lần với cùng key vẫn ra cùng kết quả, không tạo duplicate. |
| **DLQ** | Dead Letter Queue — hàng đợi chứa message không xử lý được sau nhiều retry, để điều tra thủ công. |
| **Circuit breaker** | "Cầu chì" — khi downstream lỗi nhiều, tự ngắt gọi trong X giây để tránh cháy lan. |
| **Bulkhead** | "Vách ngăn" — tách pool resource để 1 phần chết không kéo cả hệ thống. |
| **Outbox pattern** | Ghi event vào bảng `outbox` cùng transaction với data → daemon riêng đẩy event đi, đảm bảo 2 bên đồng bộ. |
| **Exponential backoff + jitter** | Chờ gấp đôi mỗi lần retry + thêm random để tránh retry đồng loạt. |
| **SLI / SLO / SLA** | SLI là **số đo** (p99 latency), SLO là **mục tiêu nội bộ** (p99 < 200ms 99.9% thời gian), SLA là **cam kết với khách hàng** (thường lỏng hơn SLO). |
| **MTTD / MTTR** | Mean Time To Detect / Recover — trung bình bao lâu để phát hiện / phục hồi sự cố. |

---

## Câu 1. Phân biệt SQL và NoSQL. Khi nào chọn cái nào?

**Ý chính**:
1. **Mô hình**: SQL có schema cứng, bảng–hàng–cột, quan hệ rõ. NoSQL có 4 họ — document (Mongo), key-value (Redis), column (Cassandra), graph (Neo4j).
2. **Nhất quán**: SQL theo **ACID** (Atomicity-Consistency-Isolation-Durability, tức giao dịch hoặc thành công toàn bộ hoặc fail sạch), transaction mạnh. NoSQL phần lớn theo **BASE** (Basically Available-Soft state-Eventual consistency, tức ưu tiên trả lời được, dữ liệu lệch tạm chấp nhận được).
3. **Scale**: SQL scale dọc (nâng cấp máy) dễ, scale ngang (thêm node) khó. NoSQL sinh ra để scale ngang qua replica set (bản sao) + sharding (chia nhỏ data).
4. **Truy vấn**: SQL có JOIN, aggregation phong phú. NoSQL hạn chế JOIN, thường **denormalize** (nhân bản data ra nhiều chỗ để đọc nhanh).
5. **Chọn SQL khi**: nghiệp vụ có quan hệ, cần transaction, cần báo cáo phức tạp — ngân hàng, hợp đồng, đơn hàng.
6. **Chọn NoSQL khi**: schema thay đổi liên tục, scale rất lớn, ghi nhiều hơn đọc, hoặc cần mô hình đặc thù — log, IoT (thiết bị gửi dữ liệu liên tục), feed mạng xã hội, cache.

**Kéo về thực tế**: Trong hệ thống Hợp đồng điện tử em làm, em dùng **PostgreSQL** cho metadata hợp đồng & chữ ký (cần ACID — 1 hợp đồng có 3–5 bên ký, service crash giữa chừng phải rollback sạch), **MongoDB** cho audit log (schema biến thiên theo loại event, ghi nhiều đọc ít, giữ 10 năm), **Elasticsearch** (bộ search engine chuyên xử lý full-text) cho tìm nội dung hợp đồng. Không có lựa chọn "SQL vs NoSQL" tổng thể — mỗi loại data quyết riêng.

**Câu xoáy đáp xoay**:
- *"Postgres có JSONB thì sao không dùng luôn cho audit log, tránh phải học Mongo?"* → Đúng, JSONB là lựa chọn tốt nếu team đã thạo Postgres. Em vẫn chọn Mongo vì lúc đó team có sẵn kinh nghiệm, và volume ghi audit rất lớn — Mongo ghi nhanh hơn JSONB ở scale này (ít overhead MVCC — Multi-Version Concurrency Control, cơ chế Postgres quản lý nhiều phiên bản cùng lúc). Nếu làm lại, em sẽ PoC cả 2 để đo trước khi quyết.
- *"Em nói ACID mạnh — isolation level em đặt là gì?"* → Mặc định Postgres là `READ COMMITTED` (1 transaction chỉ thấy data đã commit của transaction khác). Với luồng ký cần chặt hơn, em dùng `REPEATABLE READ` (trong 1 transaction luôn thấy snapshot nhất quán) để tránh **phantom read** (đọc lại cùng query ra kết quả khác).
- *"Mongo từ 4.0 đã có multi-document transaction, em biết không?"* → Biết, nhưng có ràng buộc — chỉ trong 1 replica set hoặc sharded cluster cùng shard, và overhead cao. Em chưa dùng vì nếu cần transaction thì đã dùng Postgres.
- *"Nếu 1 ngày phải sharding Postgres em làm sao?"* → 2 cách: (1) **Citus extension** (Postgres shard tự động), (2) sharding tay ở application layer với shard key hợp lý (em sẽ chọn `tenant_id`). Rủi ro lớn nhất là **cross-shard transaction** — cần dùng saga (chuỗi giao dịch nhỏ bù trừ) thay vì 2PC (two-phase commit, chậm và dễ block).

> **Clarify**: "Anh muốn em so sánh tổng quát hay đi sâu 1 loại NoSQL cụ thể ạ?"
> **Nếu chưa chắc** (vd: graph DB): "Em chưa hands-on Neo4j production, em hiểu nguyên lý trên paper. Em sẽ PoC nhỏ + hỏi team Viettel đã dùng để học từ thực tế."

---

## Câu 2. Index hoạt động thế nào? Có trường hợp nào index làm chậm query không?

**Ý chính**:
1. **Bản chất**: Index là cấu trúc phụ, lưu cột được đánh index + con trỏ về hàng gốc, giúp tìm nhanh hơn **full scan** (quét toàn bảng).
2. **Các loại phổ biến**: **B-tree** (cây cân bằng, O(log n), dùng cho `=`, range, `ORDER BY`, `LIKE 'abc%'`), **hash** (O(1) equality), **composite (index ghép)** `(a,b,c)` tuân thủ **left-most prefix** (phải query bắt đầu từ cột trái), **covering index** (chứa luôn cột SELECT → **index-only scan**, không cần đi về bảng gốc).
3. **Khi index LÀM CHẬM**:
   - **Write amplification** (khuếch đại ghi): mỗi INSERT/UPDATE phải cập nhật tất cả index → tăng latency, tăng **WAL** (Write-Ahead Log, log ghi trước khi commit).
   - **Selectivity thấp** (độ lọc kém): cột chỉ 2 giá trị (`is_active`) → optimizer bỏ qua vì full scan còn rẻ hơn.
   - **Hàm trên cột**: `WHERE LOWER(email)=?` → không dùng được index thường. Phải dùng **functional index** (index trên biểu thức).
   - **Implicit cast** (ép kiểu ngầm): truyền string vào cột int → index bị bỏ.
   - **`OR`, `NOT IN`, `!=`** thường không tận dụng index.
   - **Bảng nhỏ** (< vài nghìn hàng): full scan nhanh hơn.
   - **Index bloat** (phình): sau nhiều DELETE/UPDATE, cây phình lên dù ít data thật → cần `REINDEX` định kỳ.

**Kéo về thực tế**: Bảng `signature` của dự án ~200M hàng, partition (chia bảng) theo tháng. API "list hợp đồng của tenant theo trạng thái" **p95** (percentile 95 — 95% request nhanh hơn, 5% chậm hơn) nhảy từ 200ms lên 2s. Em chạy `EXPLAIN ANALYZE` (lệnh Postgres in ra kế hoạch thực thi thật + thời gian mỗi bước) → **seq scan** (quét tuần tự toàn bảng) do index cũ `(tenant_id)` selectivity thấp (1 tenant chiếm 30% data). Em đổi thành composite `(tenant_id, status, created_at DESC)` cover luôn sort, đồng thời DROP index `(status)` đơn lẻ — đỡ write 15%. p95 về 50ms.

**Câu xoáy đáp xoay**:
- *"Composite index `(a, b, c)` có dùng được cho `WHERE b=? AND c=?` không? Tại sao?"* → **Không**, vì left-most prefix — query phải bắt đầu từ `a`. Nếu cần `(b, c)` thì tạo index riêng `(b, c)`, không tận dụng composite cũ được.
- *"Em tạo index trên bảng 200M hàng — không sợ lock bảng lâu à?"* → Dùng `CREATE INDEX CONCURRENTLY` (Postgres) — không khóa bảng cho write, nhưng chậm gấp 2–3× và tốn disk tạm. Em chạy ngoài giờ cao điểm, monitor `pg_stat_progress_create_index`. Với MySQL thì dùng `ALGORITHM=INPLACE, LOCK=NONE`.
- *"Làm sao em biết index nào đang không được dùng để drop?"* → Postgres có view `pg_stat_user_indexes` — cột `idx_scan=0` sau 1 tháng nghĩa là không có query nào dùng. Tuy nhiên phải cẩn thận: index có thể dùng cho enforcement unique constraint hoặc cho query chạy hàng tháng (report cuối tháng).
- *"Nếu đang peak mà không thể DROP index thì em handle sao?"* → (1) `DROP INDEX CONCURRENTLY` — không lock bảng, (2) hoặc chờ giờ thấp điểm, (3) hoặc set index thành `invisible` (MySQL 8+) / đổi `indisvalid=false` (Postgres) để optimizer không dùng mà vẫn giữ cấu trúc → test impact trước khi drop thật.
- *"Partial index là gì? Có cứu được case selectivity thấp không?"* → **Partial index** chỉ đánh index tập con thỏa điều kiện — ví dụ `CREATE INDEX ... WHERE status='ACTIVE'`. Dùng khi phần lớn query chỉ quan tâm 1 giá trị của cột selectivity thấp → index nhỏ, hiệu quả cao.

> **Clarify**: "Anh muốn em nói về Postgres, MySQL, hay nguyên lý chung ạ?"
> **Nếu chưa chắc** (vd: Oracle): "Em chưa debug index Oracle, nhưng B-tree là chung. Em sẽ học `DBMS_XPLAN` để đọc plan tương đương."

---

## Câu 3. Cache-aside là gì? Rủi ro của nó?

**Ý chính**:
1. **Cache-aside (lazy loading — nạp lười)**: app đọc cache trước, miss thì đọc DB rồi tự ghi cache. Khi update, **invalidate** (xóa key) thay vì write cache.
2. **Ưu điểm**: đơn giản; cache chết thì app vẫn chạy (chậm hơn), không ép mọi đường qua cache.
3. **4 rủi ro kinh điển**:
   - **Stale data** (dữ liệu cũ) — TTL chưa hết nhưng DB đã đổi.
   - **Cache stampede / thundering herd** ("đàn trâu xô cửa") — 1 key nóng hết TTL cùng lúc, nhiều request cùng miss → đập DB.
   - **Cache penetration** (xuyên thủng cache) — key không tồn tại ở cả cache lẫn DB, attacker lợi dụng quét.
   - **Cache avalanche** (tuyết lở) — nhiều key TTL cố định hết hạn cùng lúc.
4. **Cách phòng**:
   - Stale: invalidate khi update + TTL ngắn; hoặc **versioned key** (gắn số phiên bản vào tên key, ví dụ `user:v2:123`).
   - Stampede: **single-flight lock** (chỉ 1 request đi DB, các request khác chờ) — Redis Lua script.
   - Penetration: cache cả giá trị `null` với TTL ngắn; hoặc **bloom filter** (cấu trúc xác suất kiểm tra nhanh "key có thể tồn tại không").
   - Avalanche: TTL có **jitter** (`600s ± 60s random` để không hết hạn đồng loạt).

**Kéo về thực tế**: Dự án em cache thông tin **tenant (doanh nghiệp)** và **template hợp đồng** trên Redis. 1 lần Redis bị restart, service xác thực đập DB 10k QPS, DB CPU 95%. Em thêm single-flight bằng Lua + TTL có jitter + invalidate qua Kafka event khi admin cập nhật tenant + cache cả miss. Lần restart kế tiếp DB chỉ tăng từ 500 lên 800 QPS.

**Câu xoáy đáp xoay**:
- *"Single-flight Lua script em viết ra sao?"* → Pseudo: `SET lock_key NX EX 5` (chỉ set nếu chưa có, TTL 5s). Nếu thành công → worker đi DB và ghi cache. Nếu fail (key đã có lock) → sleep 50ms, retry đọc cache. Lua đảm bảo atomic (không bị race giữa check và set).
- *"Tại sao không dùng write-through để tránh stale luôn?"* → Write-through mỗi write đều ghi cả cache và DB — đảm bảo đồng bộ nhưng: (1) slow hơn (double write), (2) cache bị lãng phí với data write-once-read-rarely, (3) phức tạp khi cache chết. Cache-aside đơn giản hơn, phù hợp data đọc nhiều ghi ít.
- *"Invalidate qua Kafka — nếu Kafka lag thì data stale trên vài instance, xử lý sao?"* → Chấp nhận eventual consistency vài giây. Nếu nghiệp vụ cần chặt (ví dụ permission change) → 2 lớp: invalidate qua Kafka + TTL ngắn (30–60s) để đảm bảo worst-case không quá 60s. Nếu thật sự cần real-time → đọc thẳng DB, không cache.
- *"Cache null 30s — nếu attacker tạo 1M key ngẫu nhiên thì Redis ngợp không?"* → Có, rủi ro memory. Phòng bằng: (1) bloom filter chặn trước khi vào cache (false positive chấp nhận được), (2) rate limit theo IP, (3) bound max size bằng `maxmemory-policy=allkeys-lru`.
- *"Em nói 'Redis chết thì app vẫn chạy' — nhưng DB có đủ tải không?"* → Thường KHÔNG. Phải có **L2 cache** (in-process, ví dụ Caffeine Java) làm lớp đệm, hoặc tầng trước có rate limit để cắt tải tự động khi Redis mất. Hệ thống thiết kế "chịu được Redis chết" là hiếm — phần lớn thực tế là "Redis chết → mình chết theo".

> **Clarify**: "Anh muốn em so sánh với write-through / write-behind luôn không ạ?"
> **Nếu chưa chắc** (vd: refresh-ahead): "Em chưa triển khai refresh-ahead production. Nếu cần, em sẽ PoC bằng Caffeine (in-process) hoặc Redis Function."

---

## Câu 4. Giải thích CAP theorem bằng ví dụ thực tế.

**Ý chính**:
1. **Phát biểu**: Hệ phân tán (nhiều node) không thể đồng thời đạt 3 thuộc tính — **C**onsistency (mọi node thấy cùng giá trị), **A**vailability (mọi request có response), **P**artition tolerance (vẫn chạy khi mạng chia cắt).
2. **Thực tế mạng luôn có thể phân vùng** (đứt cáp, switch chết) → **P là bắt buộc**. Câu hỏi thật: chọn CP hay AP khi partition xảy ra?
   - **CP**: từ chối request để không trả data sai — Zookeeper, etcd, HBase, Postgres sync replication.
   - **AP**: vẫn trả lời, đồng bộ sau → **eventual consistency** (nhất quán cuối) — Cassandra, DynamoDB, Riak.
3. **PACELC** (mở rộng CAP): ngay cả khi không có Partition (**E**lse), vẫn phải trade-off **L**atency vs **C**onsistency. Spanner là PC/EC, MongoDB mặc định PA/EC.

**Kéo về thực tế**: Dự án em chia 3 mảng data, mỗi mảng chọn khác nhau:
- **Hợp đồng & chữ ký → CP**. Tuyệt đối không để 2 node thấy 2 trạng thái khác nhau (A thấy "đã ký", B thấy "chưa ký" → hợp đồng có thể bị khiếu nại pháp lý). Postgres **sync replication** (ghi xong cả primary lẫn replica mới coi là commit), nếu replica mất kết nối → write reject, user retry.
- **Audit log → AP**. Thà log muộn 5s còn hơn block user → Kafka → ES eventual consistency.
- **Cache tenant → AP**. Chấp nhận stale 10 phút để đổi latency.

Bài học: không có "hệ thống CAP đồng nhất", từng loại data phải quyết riêng.

**Câu xoáy đáp xoay**:
- *"Postgres sync replication có phải CP thật không? Nếu chỉ có 1 replica sync mà replica đó mất thì sao?"* → **Không phải CP tuyệt đối** — nếu primary sống nhưng replica chết, primary vẫn write (chỉ log warning) trừ khi cấu hình `synchronous_commit=remote_write` và `synchronous_standby_names` yêu cầu 'ANY 1'. Muốn CP đúng nghĩa cần **quorum** (số đông) — ví dụ Postgres Patroni + 3 node, write cần commit ở đa số.
- *"Mạng chia cắt giữa 2 datacenter, em handle như thế nào?"* → Tùy policy — (1) **fencing**: datacenter thiểu số tự stop nhận write (tránh split-brain — 2 primary cùng nhận write), (2) **witness node** ở DC thứ 3 để quyết ai là primary, (3) nếu business chấp nhận AP thì cho cả 2 DC nhận write và conflict-resolve sau (CRDT hoặc last-write-wins).
- *"Kafka là CP hay AP?"* → **Mặc định AP** (với `acks=1`) — nhanh nhưng có thể mất message khi broker chết. Với `acks=all` + `min.insync.replicas=2` → **CP hơn** (không mất message nhưng có thể reject nếu không đủ replica sync). Kafka cho phép **chọn theo topic**.
- *"Quorum trong DB là gì?"* → Quy tắc "đa số" — với cluster N node, write cần W node ack, read cần R node — thỏa `W + R > N` thì đảm bảo read thấy write mới nhất. VD N=3, W=2, R=2: sống sót khi 1 node chết.
- *"CRDT là gì?"* → **Conflict-free Replicated Data Type** — cấu trúc data thiết kế sao cho 2 node update song song ghép lại luôn ra kết quả đúng (counter, set, ordered list). Dùng trong AP system, ví dụ Riak, Redis CRDT, collaborative editor (Google Docs dùng OT/CRDT).

> **Clarify**: "Anh muốn em nói cả PACELC luôn không ạ?"
> **Nếu chưa chắc** (Spanner/CockroachDB): "Em chưa vận hành NewSQL, em đọc qua paper Spanner/TrueTime. Sẽ học thêm nếu Viettel có cluster tương tự."

---

## Câu 5. Vì sao cần message queue? Cho 1 case bạn đã dùng.

**Ý chính**:
1. **Decoupling** (tách cặp): producer không cần biết consumer, 2 bên scale & deploy độc lập.
2. **Asynchronous** (bất đồng bộ): producer không chờ consumer xử lý xong → API trả về nhanh.
3. **Load leveling** (san tải): hấp thụ burst — peak 10k/s, consumer xử lý đều 2k/s.
4. **Durability & retry** (bền bỉ & thử lại): message không mất khi consumer chết, retry tự động, có **DLQ** (dead letter queue — nơi chứa message fail vĩnh viễn để điều tra).
5. **Pub/Sub & fan-out**: 1 event → nhiều consumer song song.
6. **Khi KHÔNG nên dùng**: request–response latency < 50ms; 1 producer 1 consumer cùng service (in-memory đủ); team chưa có người vận hành (MQ thêm monitoring, lag, DLQ, ordering).

**Kéo về thực tế**: Dự án em — sau khi 1 hợp đồng được ký, cần làm 7 việc: verify chữ ký, gọi TSA (Time Stamp Authority — đơn vị cấp timestamp có giá trị pháp lý), gen PDF cuối, email + SMS các bên, push notification, index ES, ghi audit. Ban đầu làm đồng bộ → **p99** (99% request nhanh hơn mức này) 10s, user ký lại → duplicate. Em tách thành Kafka topic `contract.signed`, API chỉ publish event rồi trả về 150ms. 7 consumer group nghe cùng topic, mỗi việc scale riêng + DLQ riêng. Chọn Kafka (không RabbitMQ) vì cần **replay được** (đọc lại message cũ) cho audit/rebuild index, **partition** (chia message theo key) theo `contract_id` giữ ordering, throughput cao khi report cuối tháng. Dùng **outbox pattern** (ghi DB + event cùng transaction, daemon đẩy Kafka) để tránh "commit DB OK nhưng publish fail".

**Câu xoáy đáp xoay**:
- *"Outbox pattern cụ thể implement thế nào?"* → Tạo bảng `outbox(id, aggregate_id, event_type, payload, created_at, sent_at)`. Business code ghi data + INSERT vào outbox trong cùng transaction. Một daemon (Debezium CDC — Change Data Capture, bắt thay đổi DB real-time; hoặc poll query) đọc outbox `WHERE sent_at IS NULL`, publish Kafka, update `sent_at`. Đảm bảo **at-least-once** (ít nhất 1 lần) — consumer phải idempotent.
- *"Consumer xử lý xong nhưng crash TRƯỚC khi commit offset, thì sao?"* → Message sẽ được xử lý lại sau khi restart → **duplicate processing**. Đó là lý do consumer phải **idempotent** — dùng dedupe key (VD `signature_id`), check đã xử lý chưa trước khi ghi.
- *"Kafka `exactly-once` — em có bật không?"* → Có thể bật với **transactional producer + read-committed consumer**, nhưng: (1) overhead ~20% throughput, (2) chỉ exactly-once trong phạm vi Kafka (producer→Kafka→consumer Kafka), không kéo dài sang DB ngoài. Em KHÔNG bật — thay vào đó làm consumer idempotent, rẻ hơn và đủ.
- *"DLQ retry thủ công vs tự động?"* → Với lỗi **retriable** (timeout, 5xx) em retry tự động 3–5 lần backoff. Vào DLQ rồi thì thường là lỗi **non-retriable** (data corrupt, bug code) → retry vô ích, cần người xem. Em có dashboard DLQ + alert khi > 100 message, SRE/dev review, fix bug hoặc bypass record lỗi.
- *"Tại sao Kafka mà không RabbitMQ cho case này?"* → (1) **Replay**: Kafka giữ message theo retention (7 ngày), có thể reset consumer đọc lại. RabbitMQ message xóa khi ack. (2) **Ordering per-key**: Kafka partition theo `contract_id` đảm bảo thứ tự cho cùng 1 hợp đồng. RabbitMQ không đảm bảo ordering khi có nhiều consumer. (3) **Throughput**: Kafka 100k msg/s dễ, RabbitMQ ~10k/s. Đổi lại RabbitMQ routing mạnh hơn (exchange pattern) nhưng case em không cần.
- *"Nếu Kafka broker chết hết thì sao?"* → Outbox pattern cứu — event vẫn trong DB, daemon sẽ retry publish khi Kafka sống lại. Không mất data, chỉ delay.

> **Clarify**: "Anh muốn em so sánh Kafka / RabbitMQ / Redis Stream luôn không ạ?"
> **Nếu chưa chắc** (Pulsar): "Em chưa vận hành Pulsar, em biết nó tách compute/storage, có tier-ed storage. Nguyên lý pub/sub giống nên em tin học nhanh."

---

## Câu 6. Thiết kế API rate limiter cho 1 triệu user, 1000 req/s mỗi user. Chọn thuật toán gì?

**Ý chính**:
1. **Xác định yêu cầu**: tổng tải lý thuyết rất lớn (không thực tế, thường peak ~100k–1M TPS); cho phép burst hay cứng? độ chính xác?
2. **4 thuật toán chính**:
   - **Fixed window**: đếm theo cửa sổ cố định (1s, 1m). Đơn giản nhưng "burst ở ranh giới" — user có thể đẩy 2× limit ngay biên window.
   - **Sliding window log**: lưu timestamp từng request → chính xác nhưng **O(n) memory** mỗi user (tốn RAM).
   - **Sliding window counter**: xấp xỉ 2 fixed window kề nhau theo tỷ lệ → nhẹ và chính xác hơn fixed.
   - **Token bucket** (thùng token): bucket có N token, refill `r` token/s. Cho phép burst ngắn, giới hạn trung bình. Chỉ lưu 2 giá trị/user: `tokens`, `last_refill`.
3. **Chọn**: **Token bucket** là lựa chọn cân bằng nhất cho API gateway công cộng.
4. **Triển khai**: lưu ở **Redis** với **Lua script** để atomic (đọc+giảm+ghi trong 1 op, không bị race). Key: `rl:{user_id}`. TTL vài phút để dọn user không active.
5. **Phân tán**: Redis Cluster shard theo user_id; nếu Redis chết → fallback local in-memory kèm cảnh báo (**fail-open** = cho qua hết, hay **fail-close** = chặn hết, tùy nghiệp vụ — thanh toán nên fail-close, feed có thể fail-open).
6. **Response**: trả `429 Too Many Requests` + header `X-RateLimit-Remaining`, `Retry-After` (bao lâu nữa thử lại được).

**Kéo về thực tế**: Dự án em cần rate limit API gọi OTP ký hợp đồng để chống spam SMS (SMS tốn tiền). Em đặt token bucket 5 token/số điện thoại/5 phút, Lua script trên Redis. Nếu user spam, trả 429 kèm `Retry-After`. Thêm tầng rate limit theo IP để chống attacker dùng nhiều số.

**Câu xoáy đáp xoay**:
- *"Lua script có đảm bảo atomic trong Redis Cluster (nhiều shard) không?"* → Chỉ atomic **trong cùng 1 shard**. Nếu key thuộc nhiều shard khác nhau → Lua fail. Giải pháp: dùng **hash tag** `rl:{user_id}` (curly braces) để ép mọi key liên quan cùng user vào 1 shard. Key rate limit 1 user luôn ở 1 shard → an toàn.
- *"Token bucket refill liên tục hay rời rạc?"* → **Tính lười (lazy refill)** — không có cron refill. Mỗi request đến: tính `tokens_add = (now - last_refill) * rate`, cộng vào `tokens` (cap ở max), update `last_refill=now`. Chỉ tính khi có request → tiết kiệm CPU.
- *"Nếu Redis chết, fail-open hay fail-close?"* → **Tùy nghiệp vụ + API**:
  - OTP / login / thanh toán → **fail-close** (an toàn hơn, chấp nhận false positive chặn 1 chút).
  - API feed / xem hợp đồng → **fail-open** (UX ưu tiên, risk spam thấp).
  - Nên log rõ "Redis unavailable → fallback mode" để ops biết.
- *"1 user có nhiều device, rate limit theo user_id hay device_id?"* → Đa tầng: (1) per-device để công bằng giữa device, (2) per-user để chống abuse tổng, (3) per-IP để chống bot. 3 tầng kiểm tra cùng lúc, vượt 1 tầng là chặn.
- *"Em không nghĩ token bucket mỗi user = 1 key Redis sẽ quá tải à? 1M user = 1M key."* → 1M key-value nhỏ (~100 byte) = ~100MB, Redis xử lý tốt. Vấn đề thật là **throughput QPS Redis** (mỗi request 1 Lua call) — cần **Redis Cluster** + shard theo user_id để phân tải.
- *"Distributed rate limit chính xác tuyệt đối có được không?"* → Khó và đắt. Envoy có **Global Rate Limit Service** dùng Redis làm authoritative — nhưng mỗi request mất 1 round-trip. Trade-off: chấp nhận chính xác ±5% để tiết kiệm latency. Thường ai cần "chính xác tuyệt đối" là over-engineering.

> **Clarify**: "Anh có muốn em giới hạn theo user, theo IP, hay theo API key ạ?"
> **Nếu chưa chắc**: "Nếu yêu cầu chính xác tuyệt đối, em sẽ đọc thêm về leaky bucket và envoy rate limit service."

---

## Câu 7. DB đang chậm, CPU DB 90%. Bạn điều tra và xử lý thế nào theo thứ tự?

**Ý chính** — quy trình 6 bước, từ không xâm lấn đến xâm lấn:
1. **Mitigate trước khi điều tra sâu** (nếu đang incident **Sev1** — severity 1, nghiêm trọng nhất): bật rate limit tầng trước, **shed tải** (từ chối bớt request non-critical), hoặc failover sang replica để user vẫn dùng được.
2. **Nhìn dashboard**: active connections, slow query count, lock wait, disk I/O, network, **replication lag** (độ trễ giữa primary và replica). Xác định CPU do **CPU user** (query nặng) hay **CPU sys** (context switch/IO wait — chờ disk).
3. **Top offender**: `SELECT * FROM pg_stat_activity` / `SHOW PROCESSLIST` → sort theo duration, tìm query/transaction treo lâu.
4. **Slow query log + `EXPLAIN ANALYZE`** trên query nặng nhất: thiếu index? **plan đổi** (optimizer chọn đường thực thi tệ hơn)? bảng vừa grow lớn?
5. **Check lock**: deadlock? long transaction không commit (thường do code quên `commit` trong block `catch`)?
6. **Check write load**: có job batch / ETL (Extract-Transform-Load, pipeline dữ liệu) chạy không đúng giờ không?
7. **Tác động ngoài**: traffic bất thường, bot, DDoS ở tầng trên?
8. **Xử lý theo thứ tự rủi ro thấp → cao**:
   - Kill query/session treo (rủi ro thấp).
   - Thêm index / rewrite query (cần test).
   - Scale replica để shed read.
   - Nếu không xong: failover, rollback deploy gần nhất.

**Kéo về thực tế**: Có lần dự án em DB CPU 90% lúc 9h sáng. Em vào `pg_stat_activity` thấy 1 session INSERT treo 5 phút. Trace sang code, thấy block `catch` thiếu rollback sau deploy hôm trước. Em kill session → CPU về 40%. Sau đó raise hotfix PR thêm rollback + alert transaction treo > 30s.

**Câu xoáy đáp xoay**:
- *"Nếu `pg_stat_activity` không cho thấy query nặng — CPU vẫn cao thì điều tra sao?"* → (1) `pg_stat_statements` (extension gộp query theo template, thống kê tổng thời gian) — tìm query nhiều người gọi (mỗi lần rẻ nhưng tổng nặng). (2) `perf top` trên OS — xem CPU tốn vào process gì (có thể VACUUM/autovacuum chạy). (3) IOPS disk — có thể IO wait cao mà tưởng CPU.
- *"pg_stat_statements có overhead không?"* → Có, ~5% CPU. Nhưng ở production nên bật — lợi ích chẩn đoán lớn hơn chi phí.
- *"Phân biệt lock wait vs query nặng thế nào?"* → `pg_stat_activity` có cột `wait_event_type` = `Lock` → đang chờ lock. Xem cột `query` và `pid` của blocker qua `pg_blocking_pids(pid)`. Nếu `wait_event_type=NULL` hoặc `Client`/`IO` → query đang chạy thật.
- *"Failover sang replica — liệu có mất data không?"* → Tùy replication mode. **Async replication** có thể mất vài giây data mới nhất (replica chưa nhận kịp). **Sync replication** không mất nhưng replica phải up-to-date. Failover cần decision: "ưu tiên no data loss" → chờ replica sync rồi promote; "ưu tiên availability" → promote ngay, accept data loss.
- *"Kill query giữa production có an toàn không?"* → `pg_cancel_backend(pid)` (signal nhẹ — ngắt query hiện tại, giữ connection) an toàn. `pg_terminate_backend(pid)` (giết connection) nặng hơn nhưng chỉ giết connection đó, không crash DB. Nguy cơ: query đang trong transaction, kill thì rollback → data không bị sai (ACID). Nhưng nếu là `VACUUM` hay `CREATE INDEX` quan trọng, kill là phí công.
- *"Plan đổi đột ngột là sao?"* → Optimizer dựa vào statistics (`pg_stats`) chọn plan. Sau nightly `ANALYZE`, statistics mới có thể làm optimizer chọn plan tệ hơn (bảng bigger, histogram mới). Fix: `ALTER TABLE ... SET STATISTICS 1000` (tăng độ chi tiết), hoặc hint plan (Oracle), hoặc dùng `pg_hint_plan` extension.

> **Clarify**: "Đây là tình huống giả định hay anh muốn em dẫn case em thật sự đã gặp ạ?"

---

## Câu 8. Service A gọi B gọi C. C chậm dần rồi chết. Hậu quả và cách phòng?

**Ý chính**:
1. **Hậu quả nếu không có phòng hộ**:
   - C chậm → B giữ connection lâu → B cạn **thread pool / connection pool** → B cũng chậm.
   - A tiếp tục gọi B → A cũng cạn pool → **cascading failure** (lỗi lan chuỗi) kéo cả chuỗi sập.
   - Thêm **retry không jitter** từ A → đập C càng nặng (**retry storm** — bão retry) khi C mới hồi phục.
2. **Các pattern phòng hộ**:
   - **Timeout** bắt buộc ở mỗi hop (tầng trên > tầng dưới) — không bao giờ để gọi "không timeout".
   - **Retry có giới hạn + exponential backoff + jitter** (chờ gấp đôi mỗi lần + thêm ngẫu nhiên); chỉ retry operation **idempotent**.
   - **Circuit breaker** (cầu chì: closed/open/half-open): khi C fail liên tục → open, A/B dừng gọi trong X giây, sau đó half-open thử 1 request.
   - **Bulkhead** (vách ngăn): tách pool — ví dụ thread pool riêng cho nhóm gọi C, fail không làm ngạt phần còn lại.
   - **Fallback / graceful degradation** (giảm dần khéo léo): C chết thì B trả data cũ từ cache, hoặc bỏ qua phần không critical.
   - **Load shedding** (từ chối tải): khi upstream quá tải, reject sớm request mới thay vì chờ treo.
3. **Observability** (khả năng quan sát) đi kèm: **distributed tracing** (OpenTelemetry — theo dấu 1 request qua nhiều service), metric **RED** (Rate / Error / Duration) cho mỗi hop, alert khi error rate > ngưỡng.

**Kéo về thực tế**: Trong flow ký qua SmartCA hoặc USB token, service ký của em gọi sang gateway Viettel-CA. Gateway đôi khi chậm ở giờ cao điểm. Em đặt timeout 8s (vì ký USB token có OTP push mobile mất thời gian thật), circuit breaker với Resilience4j (thư viện Java) — ngưỡng 50% lỗi trong 20 request thì open 30s. Khi open, service trả lỗi rõ "CA đang gián đoạn, thử lại sau X giây" thay vì treo. Bulkhead tách pool riêng cho luồng ký khỏi luồng xem/tải hợp đồng — CA chết không làm chết API list hợp đồng.

**Câu xoáy đáp xoay**:
- *"Circuit breaker threshold em đặt thế nào? Dựa trên số liệu gì?"* → Không có số magic. Em theo nguyên tắc: (1) **sliding window** tối thiểu 20 request (ít hơn dễ false positive do nhiễu), (2) **error rate** 50% — cao để tránh breaker open lúc nhẹ nhàng, (3) **open duration** 30s — đủ để upstream phục hồi, không quá dài để user không chờ quá lâu. Sau đó monitor false positive rate và tune.
- *"Khi circuit open, user thấy gì? UX thế nào?"* → Không trả `500 Internal Server Error` generic. Trả lỗi cấu trúc: `{"code": "CA_UNAVAILABLE", "message": "Dịch vụ ký đang gián đoạn, vui lòng thử lại sau 30 giây", "retry_after": 30}`. Frontend hiển thị banner + disable nút ký + countdown. Users accept "biết tại sao chờ" hơn là "treo không hiểu".
- *"Bulkhead pool size tính thế nào?"* → Little's Law: `concurrency = throughput × latency`. VD 100 req/s × 200ms = 20 concurrent. Pool size = 20–25 (dư 25% buffer). Quá lớn → tốn memory, quá nhỏ → bottleneck false. Monitor `pool_active` và `pool_queue_length` để tune.
- *"Nếu C chỉ chậm (không lỗi), circuit breaker có work không?"* → **Không**, vì breaker đếm lỗi. Cần **slow call detection** — coi call > X ms là "soft failure". Resilience4j có `slowCallDurationThreshold` — call chậm cũng tính vào error rate.
- *"Retry với exponential backoff — tại sao cần jitter?"* → Không có jitter: 1000 client cùng fail, cùng sleep 1s, cùng retry → C vừa sống dậy lại bị đập 1000 QPS cùng lúc → chết tiếp (**retry storm**). Jitter: mỗi client sleep `1s + random(0, 500ms)` → trải đều → C phục hồi được.
- *"Service mesh (Istio/Linkerd) có thay thế hết những pattern này không?"* → Có thể làm **timeout, retry, circuit breaker, mTLS** ở tầng mesh, ngoài code. Lợi: ngôn ngữ-neutral, thống nhất. Hại: thêm layer phức tạp, learning curve, debug khó hơn. Trade-off: team đã lớn, đa ngôn ngữ → mesh hợp; team nhỏ → library trong code đủ.

> **Clarify**: "Anh muốn em đi sâu vào 1 pattern không — timeout, circuit breaker, hay bulkhead ạ?"
> **Nếu chưa chắc** (service mesh): "Em chưa hands-on Istio production, sẽ học thêm."

---

## Câu 9. Khi nào tách monolith thành microservices? Khi nào KHÔNG nên tách?

**Ý chính**:
1. **Nên tách khi**:
   - Các module có **tốc độ thay đổi khác nhau** rõ rệt (A deploy 10 lần/ngày, B deploy 1 lần/tháng).
   - **Yêu cầu scale khác nhau**: module ký cần nhiều CPU, module report cần nhiều RAM → scale riêng tiết kiệm.
   - **SLA khác nhau**: phần thanh toán SLA 99.99%, phần thống kê 99% — tách để sự cố phần thấp không kéo phần cao.
   - **Team lớn** (>2 pizza team, tức >12 người) — nhiều đội cùng sửa monolith → conflict, deploy dẫm chân.
   - **Công nghệ khác nhau**: 1 phần Python ML, 1 phần Java business → tách để stack hợp lý.
2. **KHÔNG nên tách khi**:
   - **Domain chưa rõ** — tách sai boundary tốn gấp 10 lần sửa hơn tách đúng. Nguyên tắc Sam Newman: "start with a modular monolith".
   - **Team nhỏ** (< 10 người) — overhead vận hành microservices (CI, monitoring, tracing, service mesh) lớn hơn lợi ích.
   - **Transaction phức tạp cross-domain** — chuyển sang **saga** (chuỗi giao dịch nhỏ bù trừ) / **2PC** (two-phase commit — giao dịch 2 pha, chậm và dễ block) làm phức tạp code và khó debug.
   - **Latency-sensitive** — network hop thêm 5–50ms, chuỗi 5 service = 100ms chỉ riêng IO.
   - **Chưa có observability đủ** — microservices không có **distributed tracing** (OpenTelemetry/Jaeger) là thảm họa khi debug.
3. **Cách tách đúng**: bắt đầu bằng **modular monolith** (các module có boundary rõ, không share state ngầm), khi boundary ổn định mới carve-out từng module thành service (**strangler pattern** — bóp chết dần monolith cũ).

**Kéo về thực tế**: Hệ thống em tách 4 service chính: `contract-service` (CRUD + workflow), `signing-service` (gọi Viettel-CA / SmartCA / USB token — khác SLA và dependency ngoài), `notification-service`, `audit-service`. Không tách nhỏ hơn vì team em ~15 người, tách nhỏ hơn là overhead thừa. Phần billing-internal em giữ trong contract-service vì transaction luôn chung 1 DB Postgres, tách ra thì phải saga phức tạp.

**Câu xoáy đáp xoay**:
- *"Modular monolith cụ thể tổ chức code thế nào?"* → (1) Mỗi module 1 package riêng (`com.company.contract`, `com.company.signing`), (2) **Public API** rõ — chỉ được gọi qua interface public, không gọi internal class, (3) **Database schema riêng** mỗi module (schema Postgres hoặc table prefix) — không JOIN cross-module, (4) Enforcement bằng **ArchUnit** (Java library kiểm tra kiến trúc lúc test) hoặc lint rule.
- *"Strangler pattern em đã dùng chưa? Kể case."* → Có, lúc tách `notification-service` ra khỏi `contract-service`. Bước (1) viết `notification-service` mới, (2) `contract-service` có flag `use_new_notification` — false mặc định, (3) bật flag 5% traffic → 25% → 50% → 100%, (4) sau 2 tuần stable, xóa code notification khỏi monolith. Tổng 3 tuần, không downtime.
- *"Saga pattern — choreography vs orchestration?"* → **Choreography** (vũ đạo tự phát): mỗi service publish event, service khác react. Phi tập trung, dễ mở rộng, nhưng khó track luồng (nhìn vào là không biết ai gọi ai). **Orchestration** (nhạc trưởng điều phối): có 1 orchestrator (ví dụ Temporal/Camunda) điều phối tất cả step. Dễ track, debug, rollback; nhưng SPOF (Single Point Of Failure — điểm đơn gây sập) và ít linh hoạt. Em dùng choreography cho flow đơn giản (contract signed → notify + audit), orchestration khi workflow có >5 step và cần human approval.
- *"Service mesh cần thiết khi có bao nhiêu service?"* → Kinh nghiệm: <10 service → library trong code đủ. 10–50 → bắt đầu đau, có thể mesh. >50 → mesh gần như bắt buộc để giữ sanity. Không phải số cứng — phụ thuộc team có kỹ năng ops không.
- *"Team 5 người mà 1 phần Python ML, 1 phần Java business — em khuyên tách không?"* → Tách vì **ngôn ngữ khác nhau** là lý do hợp lý nhất. Nhưng cẩn trọng: (1) boundary đơn giản (ML service có API sync/async rõ ràng), (2) không dùng transaction cross-service, (3) chịu overhead deploy 2 pipeline. Team 5 người mà đã chọn 2 ngôn ngữ thì đã chấp nhận trade-off đó.
- *"Prime Video từ microservices về monolith — em có biết không?"* → Có, 2023 Prime Video Monitoring team blog — họ gộp các step function + lambda về 1 ECS monolith để giảm cost 90%. Bài học: **microservices không phải silver bullet** (thuốc tiên) — khi mỗi step là function nhỏ chạy lẻ tẻ, serialization giữa các bước ăn hết tiền. Đặc thù case, không nên generalize "microservices tệ".

> **Clarify**: "Anh đang nghĩ đến 1 hệ thống cụ thể để em phân tích không ạ?"

---

## Câu 10. Hệ thống đang 10k user, dự báo 1 triệu user trong 6 tháng. Bạn làm gì trước?

**Ý chính** — thứ tự ưu tiên:
1. **Xác định bottleneck hiện tại** (đừng scale mù): load test (kiểm thử tải) đẩy ngưỡng → tìm điểm vỡ đầu tiên (DB? cache? network? service X?).
2. **Ước lượng mục tiêu**: **QPS** đỉnh, storage 1 năm, bandwidth, số concurrent connection.
3. **Các hạng mục theo impact / effort**:
   - **Stateless hóa service** (session ra Redis, config ra config server) → scale ngang tự do. (impact cao, effort vừa)
   - **Thêm cache** cho các endpoint đọc nhiều (cache-aside + single-flight). (nhanh thấy hiệu quả)
   - **Tối ưu DB**: review index, thêm **read replica** (bản sao chỉ đọc), bật **connection pooler** (pgbouncer — gom connection từ nhiều client thành ít connection đến DB). (effort vừa)
   - **CDN** (Content Delivery Network — mạng phân phối nội dung, cache ở edge gần user) cho asset tĩnh + API response cacheable.
   - **Message queue hóa** các luồng có thể async (notification, analytics, audit).
   - **Auto-scaling** (tự động thêm/bớt server theo tải) + health check + graceful shutdown.
4. **Sharding DB**: chỉ làm khi read replica không đủ (**single-writer bottleneck** — primary duy nhất nghẽn). Chọn shard key cẩn thận (`tenant_id` thường hợp cho SaaS).
5. **Quan sát & kỷ luật**: dashboard RED/USE (Rate-Error-Duration / Utilization-Saturation-Errors), **SLO** (Service Level Objective — mục tiêu nội bộ) rõ, load test định kỳ (canary traffic 10% / 50% / 100%).
6. **Tổ chức**: chia team theo domain khi service tách, training on-call, run-book (sổ tay xử lý sự cố).

**Kéo về thực tế**: Dự án em đã trải qua giai đoạn similar — khi chuẩn bị mở cho tenant bán lẻ. Em làm theo thứ tự: (1) load test tìm bottleneck → thấy bottleneck ở API list hợp đồng (chưa có composite index) và API verify OTP (chưa cache tenant). (2) Sửa 2 điểm này trước, không scale gì cả — p95 đã cải thiện 5×. (3) Sau đó mới thêm read replica, pgbouncer, CDN asset. (4) Không vội sharding — cuối cùng chỉ cần **partitioning** (chia bảng theo tháng) bảng `signature` là đủ.

**Câu xoáy đáp xoay**:
- *"Load test em dùng tool gì?"* → **k6** (JS script, dễ viết scenario) hoặc **Gatling** (Scala, power cao). JMeter là classic nhưng nặng. k6 em hay dùng vì dễ CI/CD. Với production traffic, em thêm **shadow traffic** (copy traffic thật sang stage) — real-world hơn synthetic.
- *"Chưa có observability tốt, em làm gì trước khi scale?"* → **Không scale.** Scale mù tốn tiền + không biết có work không. Bước 0 bắt buộc: (1) metric cơ bản (RED mỗi endpoint), (2) distributed tracing (ít nhất correlation ID — mã liên kết để trace 1 request qua nhiều service), (3) alert SLO. Mất 2–3 tuần nhưng cứu 6 tháng debug sau.
- *"Sharding theo tenant_id có vấn đề gì?"* → (1) **Hot shard** — 1 tenant lớn chiếm nhiều traffic → shard đó nóng, shard khác nhàn. (2) **Cross-tenant query** (report tổng hợp) khó — phải query nhiều shard rồi merge. (3) **Rebalancing** khi tenant grow — phải migrate tenant sang shard khác, rủi ro downtime. Giảm thiểu: sub-shard tenant lớn, hoặc dùng **consistent hashing** (băm nhất quán — giúp rebalance ít đau khi thêm/bớt node).
- *"Cost tăng theo 100× user — em quản trị thế nào?"* → (1) Measure trước, tối ưu query/index/cache để **cost không linear với user** — ideal là sub-linear. (2) **Tier-ed storage**: hot data Postgres SSD, warm data S3 cold. (3) **Autoscaler với max cap** để không bill runaway. (4) **Unit economics**: `cost per user` tracking hàng tháng — nếu tăng đột biến → alert cost.
- *"Khi nào em KHÔNG dùng auto-scaling?"* → (1) Workload có pattern ổn định (biết trước peak), (2) startup time của pod quá lâu (2 phút) → autoscaler không kịp, (3) workload stateful (cần careful rebalancing). Thay bằng scheduled scaling hoặc overprovisioning có tính toán.
- *"10× user nhưng chỉ 3× traffic (nhiều user passive) thì chiến lược khác không?"* → Khác nhiều. Ưu tiên: (1) cache mạnh cho passive users (họ đọc nhiều ghi ít), (2) lazy load data, (3) có thể giữ DB hiện tại lâu hơn — bottleneck di sang storage/bandwidth. Điều tra pattern user trước khi tối ưu.

> **Clarify**: "Anh có thông tin cụ thể về tỉ lệ đọc/ghi và mô hình truy cập không ạ?"

---

## Câu 11. Thiết kế hệ thống nạp tiền điện thoại real-time (latency < 200ms, không mất giao dịch, 50k TPS peak).

> Đây là domain Viettel (không thuộc dự án HĐ điện tử). Em trả lời theo kiến thức chung + nguyên lý em đã áp dụng ở flow thanh toán trong dự án mình.

**Ý chính** — 7 bước thiết kế:
1. **Làm rõ yêu cầu**:
   - Input: số thuê bao + mệnh giá + phương thức (thẻ cào / ví / ngân hàng).
   - Output: kết quả thành công/thất bại; 99.99% thành công trong 200ms; không được mất giao dịch.
   - **Idempotency** (bất biến lặp) bắt buộc — client retry mà không nạp 2 lần.
2. **Ước lượng**: 50k TPS × 86400s ≈ 4.3 tỷ giao dịch/ngày. Trung bình record ~500 byte → ~2 TB/ngày thô.
3. **API**:
   - `POST /topup` với header `Idempotency-Key: {uuid}`.
   - Body: `{msisdn, amount, channel, ref}`.
4. **Kiến trúc luồng**:
   - API Gateway → topup-service (stateless, horizontal scale).
   - Kiểm tra idempotency key ở Redis trước (nếu đã xử lý → trả kết quả cũ).
   - Ghi **WAL / event log** vào Kafka (topic `topup.request`, partition theo `msisdn`) NGAY — đây là "điểm không mất".
   - Luồng đồng bộ: gọi **OCS (Online Charging System — hệ thống tính cước online real-time)** qua Diameter / gRPC để nạp; đồng thời trừ ví / charge thẻ.
   - Ghi kết quả vào DB (Postgres cho bản ghi nghiệp vụ) + publish `topup.completed`.
5. **Mô hình dữ liệu**:
   - Bảng `topup` partition theo ngày: `id, msisdn, amount, channel, status, ref, idempotency_key, created_at, updated_at`.
   - Index `(idempotency_key)` unique + `(msisdn, created_at)`.
6. **Đảm bảo không mất**:
   - Kafka `acks=all` (chờ tất cả replica ack), **replication factor 3**.
   - **Outbox pattern**: DB + Kafka trong cùng transaction logic.
   - **Reconciliation job** (đối soát) chạy mỗi 5 phút: so Kafka ↔ DB ↔ OCS, bù các giao dịch lệch.
7. **Đạt latency < 200ms**:
   - Idempotency check ở Redis < 2ms.
   - OCS call < 100ms (cần OCS in-region, connection pool warm).
   - DB write < 20ms (SSD, synchronous commit nhưng **group commit** — gộp nhiều commit thành 1 write fsync).
   - Dự phòng: circuit breaker nếu OCS chậm → trả "pending" và xử lý async (báo user qua SMS/app).
8. **Mở rộng & lỗi**:
   - Scale horizontal topup-service theo autoscaler.
   - **Multi-region active-active** (nhiều vùng cùng phục vụ) cho HA (High Availability), routing theo vùng thuê bao.
   - DLQ cho các giao dịch fail, dashboard theo tỉ lệ mất/lệch.

**Câu xoáy đáp xoay**:
- *"Idempotency key hết hạn sau bao lâu? Tại sao?"* → 24h là điểm phổ biến — đủ dài cho client retry, đủ ngắn để Redis không phình. Nghiệp vụ critical (thanh toán) có thể 7 ngày. Sau khi hết hạn, request cùng key được xử lý lại từ đầu → chấp nhận rủi ro duplicate rất thấp (client thường không retry sau 24h).
- *"Nếu OCS charge OK mà ví trừ fail — rollback thế nào?"* → Đây là distributed transaction — không có 2PC real-time (quá chậm). Dùng **saga bù trừ**: detect lệch qua reconciliation → compensating action (trả quota về OCS, alert ops). User sẽ thấy "pending" → resolved trong vài phút. Nếu không thể bù → ghi vào bảng `manual_review` cho người xử lý.
- *"Reconciliation phát hiện lệch, tự bù hay chờ người?"* → Tùy loại lệch:
  - **OCS charge OK + DB thành công, nhưng Kafka mất event** → tự bù (re-emit).
  - **DB ghi success + OCS không charge** → tự bù (call OCS lại với idempotency).
  - **Số tiền lệch > ngưỡng** (ví dụ > 100k VNĐ) → manual review, không tự bù (tránh gây thiệt hại lớn hơn nếu fix sai).
- *"Nếu chạm 100k TPS (gấp 2)?"* → (1) Topic partition Kafka tăng (hash theo msisdn vẫn giữ ordering per-subscriber). (2) Scale topup-service và OCS instance (Kubernetes HPA — Horizontal Pod Autoscaler). (3) Bottleneck thường ở OCS — cần load test từ trước để biết cần pre-provision. (4) DB: partition bảng `topup` theo giờ (thay vì ngày) để ghi không bloat 1 partition.
- *"Multi-region active-active — làm sao đồng bộ ví?"* → 2 hướng: (1) **Per-region wallet** (thuê bao được pin vào region, không roaming write). Đơn giản nhưng không failover được. (2) **Replicated wallet với CRDT counter** — phức tạp hơn nhưng không mất write khi region chết. Viettel thường chọn (1) vì thuê bao gắn SIM vào region.
- *"Diameter protocol em chưa biết, sao vẫn dám design?"* → Em phân tách **logic** (charge reservation, authorize, finalize) khỏi **protocol**. Logic giống nhau dù Diameter, REST hay gRPC. Nếu làm thật, em sẽ học spec Diameter (3GPP TS 32.299) và code thư viện adapter — nhưng logic flow không đổi.

> **Clarify bắt buộc**: "Anh cho em clarify — 'không mất giao dịch' là không mất **request** (OK trả pending được) hay không được phép trừ tiền sai (chặt hơn)?"

---

## Câu 12. Thiết kế hệ thống gửi SMS OTP cho toàn bộ user Viettel: peak 200k SMS/phút, không gửi trùng, không mất.

**Ý chính**:
1. **Yêu cầu**:
   - 200k/phút ≈ 3.3k/s peak.
   - Idempotent: cùng `request_id` chỉ gửi 1 SMS.
   - Không mất: producer crash hoặc SMSC (SMS Center — trung tâm chuyển SMS của nhà mạng) slow đều không được làm mất.
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
6. **Observability**: metric tỷ lệ gửi thành công, delivery delay, số OTP verify fail (chỉ số UX).
7. **Mở rộng**:
   - Partition Kafka tăng khi throughput tăng.
   - Nhiều SMSC provider với routing (failover theo giá + success rate theo nhà mạng).
   - Geo-replication Kafka nếu cần DR (Disaster Recovery — khôi phục sau thảm họa).

**Kéo về thực tế**: Dự án em gửi OTP để ký hợp đồng. Em đã áp mô hình này ở quy mô nhỏ hơn — peak ~500 OTP/phút. Idempotency-Key là trụ cột (user nhấn ký 2 lần, SMS chỉ 1). Rate limit theo `phone` cực quan trọng — không có thì có user bị brute force OTP tới 100 lần/phút.

**Câu xoáy đáp xoay**:
- *"Partition Kafka theo phone — có gây hot partition không?"* → Có thể. Số điện thoại phân bố đều thì ít risk, nhưng nếu attacker spam 1 số → partition đó nóng. Mitigation: (1) rate limit trước khi vào Kafka (chặn spam ngay API), (2) **compound key** partition theo `hash(phone) mod N` thay vì raw phone.
- *"OTP giữ ở Redis — Redis mất, user không verify được, xử lý sao?"* → 2 lớp: (1) Redis Sentinel/Cluster để mất 1 node không mất data. (2) Persist OTP vào DB như fallback — khi Redis miss, thử DB. OTP TTL ngắn (3 phút) nên DB load không lớn.
- *"SMSC có rate limit từng nhà mạng — handle thế nào?"* → Consumer có **per-carrier rate limiter**. Khi SMSC trả error rate > threshold cho 1 nhà mạng → giảm throughput topic partition đó (consumer pause), route sang provider khác nếu có. Dashboard hiển thị "queue depth theo carrier".
- *"Nếu user không nhận được SMS, fallback gì?"* → (1) Retry gửi SMS lần 2 sau 30s. (2) **Voice call OTP** — gọi đọc OTP qua voice robot. (3) **Email OTP** backup (nếu có email verified). (4) UI cho user chọn "resend" — nhưng rate limit chặt.
- *"Không gửi trùng — nhưng user NGỜ mình chưa nhận được và retry API, mình gửi lại SMS thứ 2?"* → Đây là **user-intent retry** khác với **client-intent retry**. User-intent: hợp lệ nếu ngoài TTL và user chủ động. Em có rule: OTP còn TTL + chưa verify → trả OTP cũ, không gửi mới (chống spam chính user). OTP hết TTL hoặc đã verify fail → cho tạo mới, nhưng tính vào rate limit.
- *"200k/phút sustained vs peak — design khác thế nào?"* → Sustained → dimension infrastructure theo đó, consumer concurrency tính theo peak. Peak (burst) → Kafka buffer 15–30 phút, consumer giữ đều throughput, user chấp nhận delay vài giây. Business phải clarify — em luôn hỏi trước khi design.

> **Clarify**: "Anh cho em hỏi — 200k/phút là đỉnh tức thời (burst) hay sustained? Và có yêu cầu delivery confirmation từ SMSC không ạ?"

---

## Câu 13. Thiết kế hệ thống tính cước (charging) real-time cho cuộc gọi: độ chính xác 100%, báo cáo theo giờ.

> Domain Viettel thuần. Em trả lời theo kiến thức chung về charging telco.

**Ý chính**:
1. **Yêu cầu**:
   - Độ chính xác 100% — không được sai tiền khách, không được dư/thiếu cho nhà mạng.
   - Real-time: trừ tiền / cấp quota theo giây gọi.
   - Báo cáo theo giờ: tổng hợp dữ liệu sẵn sàng truy vấn sau mỗi giờ.
2. **Khái niệm cốt lõi**:
   - **CDR (Call Detail Record — bản ghi chi tiết cuộc gọi)** sinh ra từ switch / softswitch với `call_id, caller, callee, start, end, duration, cell, …`.
   - **OCS (Online Charging System — trừ quota/tiền online real-time)**: Gy interface trong 3GPP (chuẩn telco).
   - **OFCS (Offline Charging System — tạo CDR sau cuộc gọi, rating offline)**: Gz interface.
3. **Luồng** — gộp 2 nhánh:
   - Bắt đầu gọi: OCS reserve (đặt chỗ) quota (ví dụ 60s) cho thuê bao; nếu ví < 0 → reject.
   - Trong cuộc gọi: OCS re-authorize khi quota sắp hết (reserve tiếp hoặc cut-off).
   - Kết thúc: trả quota thừa về ví; sinh CDR đẩy vào Kafka topic `cdr.raw`.
   - **Rating engine** đọc CDR → tra **tariff plan / gói cước** → sinh CDR đã rating đẩy sang `cdr.rated`.
   - Billing aggregator đọc `cdr.rated` → aggregate theo giờ/ngày vào DB/data warehouse cho báo cáo.
4. **Đảm bảo 100% chính xác**:
   - **Exactly-once effectively** (về mặt hiệu quả): Kafka + idempotent consumer, mỗi CDR có `call_id` unique.
   - **Reconciliation**: đối soát `OCS reservation log` ↔ `CDR raw` ↔ `CDR rated` ↔ tổng trừ ví. Nếu lệch, chạy job bù.
   - Tariff plan lưu **versioned** (phiên bản theo thời gian) — rating dùng đúng version tại thời điểm gọi.
   - Audit log mọi thay đổi gói cước + mọi điều chỉnh tay.
5. **Mô hình dữ liệu**:
   - CDR lưu dạng **append-only** (chỉ thêm, không sửa) trên HDFS / Iceberg / ClickHouse — partition theo giờ.
   - OCS dùng **in-memory grid** (GridGain / Hazelcast — lưu data trong RAM phân tán) cho latency microsecond.
   - Billing aggregate lưu trong Postgres / ClickHouse cho BI (Business Intelligence — báo cáo).
6. **Báo cáo theo giờ**:
   - ClickHouse với **materialized view** (view tự động tính và lưu) rollup theo giờ → truy vấn tổng cước, top thuê bao, lỗi rating.
7. **HA & mở rộng**:
   - OCS active-active multi-node, replication đồng bộ ví để tránh âm.
   - CDR pipeline có thể xử lý chậm miễn không mất.
   - DR: Kafka multi-datacenter, tariff config đồng bộ.

**Câu xoáy đáp xoay**:
- *"Nếu tariff thay đổi GIỮA cuộc gọi — rating tính thế nào?"* → CDR phải ghi `tariff_version` tại thời điểm **bắt đầu cuộc gọi**. Rating dùng version đó cho toàn cuộc gọi. Hoặc chia cuộc gọi thành 2 segment nếu business yêu cầu (phổ biến ít, thường giữ tariff đầu cuộc).
- *"OCS reserve 60s — cuộc gọi 10 phút thì sao?"* → Trong lúc gọi, mỗi 60s OCS **re-authorize** (hỏi ví còn đủ không) → reserve tiếp 60s. Nếu ví hết → **cut-off** (ngắt cuộc gọi) + trả CDR với status "denied". Đây là flow **Credit Control** của Diameter Gy.
- *"Fraud detection real-time có tích hợp vào đây không?"* → Có, thường ở tầng OCS — detect pattern lạ (gọi quốc tế nhiều bất thường, gọi số premium liên tục, SIM swap — đổi SIM lừa đảo). Có thể block ngay tại reserve nếu pattern nghi vấn. Xây riêng 1 engine fraud song song với OCS, share data qua event stream.
- *"Gọi cùng mạng vs khác mạng — tariff khác, design có cần biết trước?"* → OCS cần **number lookup** — từ số callee xác định operator (thường có route database). Tariff apply dựa trên (origin operator, destination operator, call type, time-of-day). Lookup phải nhanh (< 10ms) → cache in-memory.
- *"Reconciliation phát hiện lệch — thường lệch ở đâu?"* → (1) CDR mất do switch crash ngay cuối cuộc gọi → reserve trừ nhưng không có CDR. Fix: scan OCS reservation log chưa kết thúc > 1h → generate CDR giả theo duration max. (2) Rating sai version → re-rate. (3) Kafka duplicate → dedupe theo `call_id`.
- *"Báo cáo theo giờ — 'sẵn sàng sau 1 giờ' nghĩa là real-time hay batch?"* → Thường **near-real-time**: streaming aggregation (Flink/Kafka Streams) giữ rolling window 1h → materialized view update liên tục. Query là instant. Không phải batch "chạy cron 1h/lần" — chậm quá cho business.

> **Clarify**: "Anh muốn em tập trung vào phần OCS real-time hay phần rating/aggregation báo cáo ạ?"

---

## Câu 14. Thiết kế notification service đa kênh (SMS/email/push): retry, ưu tiên, rate limit theo user.

**Ý chính**:
1. **Yêu cầu**: đa kênh, retry có kiểm soát, ưu tiên (OTP > marketing), rate limit theo user, audit đã gửi gì cho ai.
2. **API & data model**:
   - `POST /notify` body: `{user_id, channels: [sms,email,push], template_id, variables, priority, ttl, idempotency_key}`.
   - Bảng `notification`: `id, user_id, template_id, channel, status, priority, scheduled_at, sent_at, tries, last_error, request_id`.
   - Bảng `template` versioned; bảng `user_preferences` (user chọn kênh nào + mute giờ nào).
3. **Luồng**:
   - API nhận → validate + dedupe idempotency key → ghi DB `PENDING` → push vào Kafka topic theo priority (`notif.high`, `notif.normal`, `notif.low`).
   - Mỗi priority có consumer group riêng với concurrency khác nhau — high nhiều worker hơn.
   - Worker kiểm tra user preferences + rate limit → gọi provider tương ứng (SMS gateway, SMTP/SendGrid, FCM/APNs — Firebase Cloud Messaging / Apple Push Notification).
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
9. **Mở rộng**: nhiều provider mỗi kênh, routing theo cost + success rate; **WebHook** (provider call ngược lại mình khi có delivery report) ghi đè status.

**Kéo về thực tế**: Dự án em có cần gửi mail cho các bên sau mỗi hành động (mời ký, đã ký, đã hoàn tất, hết hạn). Em xây notification-service riêng với Kafka topic priority 3 mức, template versioned, user có thể tắt kênh. Bài học quan trọng nhất: **audit phải lưu đã gửi gì thực sự** (không chỉ "đã push event") — vì có khách hàng khiếu nại "tôi không nhận được email mời ký", mình cần bản log.

**Câu xoáy đáp xoay**:
- *"Priority queue — high vô hạn thì low bao giờ được xử lý?"* → Vấn đề **starvation** (bỏ đói). Giải pháp: (1) **Weighted fair**: reserve % worker cho low (ví dụ high=70%, normal=25%, low=5%), (2) **Aging**: message low quá lâu tự promote lên normal. Em thường dùng (1) — đơn giản, đủ.
- *"User preference cache TTL em set bao nhiêu?"* → 5 phút + jitter. User đổi preference → invalidate qua Kafka event. Cân bằng giữa latency (cache hit) và freshness (không gửi khi user đã tắt).
- *"Template thay đổi giữa lúc có message đang queue — dùng version mới hay cũ?"* → Dùng **version tại thời điểm enqueue**, không phải thời điểm send. Lý do: user đã expect nội dung cũ (request đến từ event X dùng template X). Ghi `template_version` vào message payload → worker render đúng version. Template mới áp dụng cho message enqueue sau.
- *"Delivery report từ FCM async — em match với notification record thế nào?"* → Lúc send sang FCM, lưu `provider_message_id` (FCM trả về) vào record. Webhook FCM callback → lookup bằng `provider_message_id` → update status. Phải có timeout — report không đến trong 24h thì mark `unknown_delivery`.
- *"Nếu user opt-out toàn bộ kênh, OTP có gửi không?"* → OTP là **transactional/security** — bypass opt-out marketing nhưng vẫn phải có kênh delivered (user không thể opt-out OTP). Phân loại message rõ: `transactional` (OTP, password reset) vs `marketing`. Opt-out chỉ áp cho marketing.
- *"Idempotency key retry — gửi API 2 lần, SMS có gửi 2 lần không?"* → **Không**. Dedupe ở API layer (Redis `SETNX` 24h) → request thứ 2 trả về record cũ, không enqueue mới. Nếu request đầu fail trước khi dedupe ghi → hai người cùng gửi → khả năng duplicate, vì vậy consumer cũng phải idempotent check `sent_at IS NULL` trước khi call provider.

> **Clarify**: "Anh có yêu cầu exactly-once delivery không, hay at-least-once chấp nhận được ạ?"

---

## Câu 15. Thiết kế log aggregation cho 500 microservices, 10TB log/ngày, query được trong 5 giây.

**Ý chính**:
1. **Yêu cầu**: 10 TB/ngày ≈ 116 MB/s sustained, peak có thể 3–5×. Query p95 < 5s với khoảng thời gian rộng.
2. **Kiến trúc 3 tầng**:
   - **Collect**: mỗi service log ra **stdout** (theo chuẩn **12-factor** — app ghi log ra stdout, không tự quản file) → sidecar / node agent (Fluent Bit, Vector, Filebeat) → forward.
   - **Transport**: **Kafka** làm buffer giữa collector và storage. Topic `logs.raw` partition theo service.
   - **Storage & index**: tách nóng/lạnh (**hot/warm/cold tiering**):
     - Hot (7 ngày): Elasticsearch / OpenSearch — index full-text, truy vấn nhanh.
     - Warm (30–90 ngày): ES index đã **forcemerge** (gộp segment để đọc nhanh) + ít replica.
     - Cold (>90 ngày): object storage (S3/MinIO) + Loki hoặc ClickHouse — rẻ, chậm hơn.
3. **Schema log**:
   - Bắt buộc **JSON structured** (cấu trúc): `timestamp, level, service, trace_id, span_id, user_id, message, fields...`.
   - `trace_id` từ **OpenTelemetry** để correlate (liên kết) cross-service.
4. **Đạt query < 5s**:
   - Index ES với mapping phù hợp: timestamp là `@timestamp`, service / level là `keyword` (không analyzed — không cần full-text search trên field này).
   - **ILM (Index Lifecycle Management — quản lý vòng đời index)**: rotate theo ngày, shard theo volume (~50 GB/shard).
   - Hot node dùng SSD NVMe + RAM lớn; query luôn có filter `@timestamp` + `service` để cắt shard.
   - Dashboard Kibana có saved query thường dùng → cache.
5. **Chi phí & kỷ luật**:
   - **Sampling** log `DEBUG` trong production (keep 10%).
   - **Không log PII** (Personally Identifiable Information — thông tin định danh cá nhân như CCCD, OTP, chữ ký, token) → mask ở pipeline trước khi vào ES.
   - Retention theo loại log: access log 7 ngày, audit log 10 năm (yêu cầu luật).
6. **HA & throughput**:
   - Kafka replication 3, retention 3 ngày (đệm trong lúc ES bảo trì).
   - ES nhiều node, shard allocation balance.
   - Backpressure (áp lực ngược): nếu ES chậm, Kafka consumer chậm, nhưng collector không drop (Kafka đệm).
7. **Observability cho chính hệ log**:
   - Metric lag của consumer ES, số docs rejected, disk ES, latency query.
   - Alert khi lag > 5 phút — thường là sign của **mapping explosion** (schema nổ, có field dynamic vô hạn) hoặc node chết.

**Kéo về thực tế**: Dự án em quy mô nhỏ hơn (~50 GB log/ngày) nhưng dùng đúng mô hình này: Filebeat → Kafka → Logstash → ES (hot 7 ngày) + S3 (cold 10 năm cho audit hợp đồng). Bài học: **bắt buộc log JSON + trace_id** ngay từ đầu — sau này mới thêm là tốn hàng tháng refactor. Em cũng đã bị **mapping explosion** — 1 service log field dynamic vào `user_id.xxx` — ES tạo 100k field, cluster gần sập. Fix bằng `dynamic: strict` cho index log.

**Câu xoáy đáp xoay**:
- *"Mapping explosion — em phát hiện thế nào?"* → 2 triệu chứng: (1) heap ES tăng đều không giảm, (2) `GET _cluster/stats` → `total_fields` > 1000. Em viết alert khi field count tăng > 10%/ngày. Fix: `dynamic: strict` (reject field mới), hoặc `dynamic: false` (ignore field mới nhưng vẫn lưu `_source`), hoặc refactor schema đưa dynamic field thành key-value pair.
- *"Nếu ES cluster down, log được buffered đâu?"* → Kafka giữ retention 3 ngày → collector tiếp tục ghi Kafka → khi ES sống dậy consumer catch-up. Nếu Kafka cũng down → sidecar Fluent Bit có **disk buffer** local → có dung lượng vài GB. Nếu cả Kafka + disk đầy → log bị drop, nhưng alert từ lâu rồi.
- *"Retention 10 năm trên S3 — query cold có feasible không?"* → Chấp nhận **chậm** cho cold (vài phút–giờ). Dùng **Athena** / **Presto** / ClickHouse để query S3 Parquet (column format, nén tốt). Format: JSON → convert sang Parquet khi move cold → giảm storage 5–10× và query nhanh hơn. Audit query thường là "tìm 1 record theo ID" → O(log n) với partition theo ngày + index.
- *"Sampling DEBUG theo nguyên tắc nào?"* → 3 strategy:
  - **Random**: đơn giản, 10% uniform — mất context cho 1 request (span bị đứt).
  - **Hash user_id**: cùng user luôn được sample → debug user-specific ok.
  - **Tail-based** (đầu cuối): chờ kết thúc trace, nếu error thì giữ 100%, success thì sample 10% — đắt hơn nhưng giữ được error.
  Em dùng **hybrid**: error 100%, success sample 10% theo hash.
- *"Log JSON có đắt hơn plain text không? Parse nặng không?"* → Đắt hơn ~20–30% CPU encode. Nhưng trade-off xứng: query nhanh hơn hẳn, structured query chính xác. Plain text phải regex — vừa chậm vừa dễ sai. Ở production không có quy mô nhỏ nào còn dùng plain text.
- *"Log 10 năm cho audit — Legal/Compliance yêu cầu cụ thể gì?"* → Với hợp đồng điện tử theo Luật Giao dịch điện tử 2023, audit trail giữ ≥ 10 năm, đảm bảo **tính toàn vẹn** (không bị sửa, thường dùng hash chain hoặc blockchain-like merkle tree), **truy xuất được** (có thể query theo contract_id), **tính đến thời điểm ký** (trace timestamp chính xác via TSA). Storage chỉ là 1 phần — compliance phải có process.
- *"Loki / ClickHouse làm primary log store — khác ES thế nào?"* → **Loki** chỉ index **labels** (service, level), nội dung không index → rẻ hơn nhiều, nhưng tìm theo nội dung chậm (phải scan raw). **ClickHouse** column store, SQL, tốt cho analytics nhưng full-text search yếu. **ES/OpenSearch** full-text mạnh, đắt nhất. Chọn: query nhiều theo nội dung → ES. Query theo filter labels + grep đôi khi → Loki. Log analytics đa chiều → ClickHouse.

> **Clarify**: "500 microservices có phải Viettel internal không anh? Em muốn hỏi về độ đồng đều log format."

---

> **Hết 15 câu Phần 1** (phiên bản enhanced — có bảng thuật ngữ, giải thích inline, và "câu xoáy đáp xoay" cho mỗi câu).

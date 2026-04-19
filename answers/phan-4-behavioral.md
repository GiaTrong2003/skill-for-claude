# Phần 4 — Behavioral & Câu hỏi ngược — Bộ câu trả lời

> **Format chung**: **Ý chính / nguyên tắc** → **Script mẫu** (có thể đọc gần thuộc lòng, luôn điều chỉnh theo thực tế bạn) → **Câu xoáy đáp xoay** → `> Clarify` / `> Cảnh báo` khi cần.
>
> **Khung STAR** cho câu tình huống: **S**ituation → **T**ask → **A**ction → **R**esult (+ **L**esson).
>
> **Quy ước**: thuật ngữ khó có giải thích bằng **ngôn ngữ đời thường** ngay sau.

---

## Bảng thuật ngữ tham chiếu nhanh — Phần 4

| Thuật ngữ | Giải thích đời thường |
|---|---|
| **STAR** | Khung kể tình huống: Situation (bối cảnh) → Task (nhiệm vụ) → Action (em làm gì) → Result (kết quả). |
| **Disagree and commit** | "Cãi nhau cho ra nhẽ, rồi ai thua thì cam kết thực thi 100%" — tranh luận thẳng, nhưng khi đã quyết thì không sabotage. |
| **ADR** | Architecture Decision Record — tài liệu ghi lại quyết định kiến trúc: bối cảnh, lựa chọn, lý do, hậu quả. Giúp team sau hiểu tại sao làm vậy. |
| **RFC** | Request for Comments — văn bản đề xuất giải pháp, gửi team review trước khi code — tránh làm rồi mới biết sai. |
| **p99** | 99% request nhanh hơn con số này — đo "đuôi dài" của latency. |
| **Idempotency key** | Khóa bất biến lặp lại — gọi API cùng key 1 hay 5 lần vẫn ra cùng kết quả, không tạo duplicate. |
| **IC track** | Individual Contributor track — con đường phát triển thiên kỹ thuật, không cần quản lý người. |
| **MTTD / MTTR** | Thời gian trung bình phát hiện sự cố / phục hồi sự cố. |
| **tech branding** | Xây dựng thương hiệu kỹ thuật — viết blog, nói chuyện tại sự kiện, đóng góp open source. |

---

## Câu 1. Giới thiệu bản thân (60 giây).

**Ý chính**:
1. **60 giây = 120–150 từ**, chuẩn bị sẵn và tập đến thuộc 80%.
2. Cấu trúc 4 phần — **hiện tại → quá khứ → thành tựu nổi bật → động lực / kết nối với vị trí**:
   - Tên + vai trò hiện tại (1 câu).
   - Bối cảnh hiện tại + domain (1–2 câu).
   - 1–2 thành tựu có số (2 câu).
   - Lý do muốn vị trí này (1 câu).
3. **Không** kể tiểu sử (sinh năm, quê quán, đại học nào) trừ khi hỏi cụ thể.

**Script mẫu**:
> *"Chào anh/chị, em là [Tên], hiện đang là [Software Engineer] tại [team/dự án] ở Viettel.
> Em đang làm chủ yếu ở mảng **Hợp đồng điện tử** — xây dựng nền tảng cho các doanh nghiệp và cá nhân ký kết hợp đồng số, tích hợp với Viettel-CA, USB token và SmartCA. Stack chính em dùng là Spring Boot, MariaDB, Kafka, Redis, và em cũng làm việc với cả front-end khi cần.
> Trong 2 năm qua, em có 2 điểm nhấn muốn chia sẻ: thứ nhất, em đã chủ trì refactor luồng ký async qua Kafka, giảm **p99** (99% request nhanh hơn con số này) từ 10 giây xuống 180ms. Thứ hai, em là on-call primary của team và đã handle vài incident Sev2, viết post-mortem mà team đang dùng làm template.
> Em tham gia phỏng vấn hôm nay vì em thực sự muốn gắn bó lâu dài với Viettel, lên chính thức và tiếp tục phát triển ở mảng domain em đang làm. Em rất mong được trao đổi thêm với anh/chị ạ."*

**Câu xoáy đáp xoay**:
- *"Em nói giảm p99 từ 10s xuống 180ms — làm thế nào, tóm tắt trong 30 giây?"* → Luồng ký ban đầu gọi Viettel-CA synchronous, CA đôi khi chậm 5–10s → user nghĩ hệ thống treo → ký lại → duplicate. Em tách sang Kafka async: response user ngay sau publish message, consumer riêng gọi CA, kết quả webhook về. Thêm **idempotency key** (khóa bất biến lặp lại) ngăn duplicate, **DLQ** (Dead Letter Queue — hàng đợi chứa message lỗi) cho CA fail.
- *"On-call primary — em handle bao nhiêu incident từ trước đến nay?"* → [Nêu số cụ thể, ví dụ: 3–4 incident Sev2, 1–2 lần Sev3]. Quan trọng hơn số lượng là **pattern** em đã rút ra: detect nhanh nhờ alert chủ động, mitigate trước root cause, escalate sớm thay vì cố gồng.
- *"Post-mortem template của em gồm gì?"* → 8 mục: Summary, Timeline, Impact (số liệu), Root cause, Contributing factors, What went well, What went poorly, Action items (có owner + deadline + priority). Điểm em đặc biệt thêm: **What went well** để tạo tinh thần team và nhận ra điểm cần giữ, không chỉ điểm cần sửa.

> **Clarify**: không cần clarify — đây là câu đầu tiên, nên trả lời mượt.
> **Cảnh báo**: **đừng lạm dụng buzzword** ("agile, scrum, microservices, devops...") mà không có context — interviewer thường vặn ngay.

---

## Câu 2. Điểm mạnh — điểm yếu (mỗi bên 2 điểm, có ví dụ).

**Ý chính**:
1. **2 điểm mạnh**: 1 technical + 1 non-technical. Luôn có **ví dụ đi kèm**.
2. **2 điểm yếu**: chọn yếu **thật**, không phải yếu "giả" kiểu "em quá cầu toàn". **Phải kèm cách em đang cải thiện**.
3. Tránh điểm yếu kill câu hỏi ("em không giỏi code"). Chọn yếu vừa đủ để tin được + có action cụ thể.

**Script mẫu — Điểm mạnh**:
> *"Điểm mạnh thứ nhất của em là **khả năng đào sâu troubleshoot** (tìm lỗi đến tận cùng). Ví dụ gần nhất em handle 1 incident p99 tăng đột biến, em trace từ symptom về root cause (transaction treo do catch thiếu rollback) trong 30 phút. Em không dừng ở triệu chứng mà luôn hỏi 'vì sao'.
> Thứ hai là **khả năng viết tài liệu và mentor**. Em đã viết template post-mortem và on-boarding guide cho team, mentor 1–2 junior. Em thấy giải thích cho người khác bắt em phải hiểu sâu hơn — đây là cách em học hiệu quả nhất."*

**Script mẫu — Điểm yếu**:
> *"Điểm yếu đầu tiên: em **ngại feedback xung đột trực tiếp**. Hồi đầu em thường tránh nói trái ý lead trong meeting, dù trong đầu không đồng ý. Em đang cải thiện bằng cách: trước mỗi design review em note sẵn 1–2 điểm không đồng ý + lý lẽ, để khi tới meeting em bắt buộc nói ra. Sau 6 tháng em đã chủ động challenge 2 quyết định kiến trúc — 1 lần em đúng, 1 lần em sai nhưng học được nhiều.
> Thứ hai: em **chưa mạnh về front-end**. Em chỉ sửa được vài chỗ React cơ bản. Em đang học thêm vào buổi tối (mỗi tuần 2–3 giờ), và em đã nhận 1 task front-end nhỏ mỗi sprint để thực hành."*

**Câu xoáy đáp xoay**:
- *"Troubleshoot 30 phút ra root cause — em dùng tool gì trong 30 phút đó?"* → Trình tự: (1) Grafana dashboard **RED** (Rate — số request, Error — lỗi, Duration — thời gian) để xác định symptom, (2) Jaeger/Tempo theo correlation ID (ID xuyên suốt request) để trace qua các service, (3) `SHOW FULL PROCESSLIST` + `information_schema.INNODB_TRX`/`INNODB_LOCKS` cho DB lock (MariaDB), (4) log grep theo correlation ID. 30 phút chỉ đạt được khi các tool này có sẵn và alert đủ context.
- *"Điểm yếu front-end — nếu vị trí này yêu cầu full-stack thì sao?"* → Em sẽ transparent ngay từ đầu về level FE hiện tại và thời gian cần để up-skill. Em đã chủ động học và nhận task FE nhỏ. Nếu vị trí yêu cầu FE strong từ ngày 1, cần clarify để 2 bên align kỳ vọng — em không muốn cam kết rồi thiếu.
- *"Ngại feedback xung đột — nếu bạn kia sai rõ ràng mà ảnh hưởng production thì sao?"* → Đây là trường hợp hoàn toàn khác — **safety/production risk override social comfort**. Em sẽ nói thẳng, cần thiết escalate ngay lên lead. Điểm yếu của em là với quyết định không ảnh hưởng ngay, không phải với tình huống khẩn cấp có rủi ro rõ ràng.

> **Clarify**: "Anh muốn em kể điểm mạnh thiên về kỹ thuật hay soft skill ạ?"
> **Cảnh báo**: KHÔNG nói "em cầu toàn quá" / "em làm việc quá chăm chỉ" — nghe giả. KHÔNG kể quá nhiều điểm yếu — chỉ 2 là đủ.

---

## Câu 3. Vì sao em muốn chính thức / gắn bó Viettel?

**Ý chính**:
1. **3 lý do** theo thứ tự ưu tiên — **trọng tâm là vì Viettel, không phải vì lương**.
2. Lý do nên gồm:
   - **Domain / bài toán lớn**: Viettel có hệ thống scale telco-grade mà ít công ty có.
   - **Sự phát triển cá nhân**: lộ trình rõ ràng, mentor tốt, học được nhiều.
   - **Văn hóa / team**: đã làm cùng, hợp, muốn tiếp tục.
3. Tránh nói "vì Viettel là tập đoàn lớn ổn định" — nghe như tìm vùng an toàn, không phải tìm cơ hội.

**Script mẫu**:
> *"Em có 3 lý do chính ạ.
> Thứ nhất, về **bài toán**: Viettel có những hệ thống scale rất lớn — millions user, đa dạng domain (telco, billing, hợp đồng, thanh toán). Em muốn tiếp tục được đối mặt với những bài toán khó mà không phải công ty nào cũng có. Hai năm em làm tại đây em đã học được rất nhiều từ incident thật, từ mô hình kiến trúc thật của hệ thống lớn.
> Thứ hai, về **người**: team em đang làm có lead và đồng nghiệp rất chất lượng, chia sẻ hào phóng, code review kỹ. Em đã học nghề từ họ. Em không muốn chuyển nơi khác để phải xây dựng lại mối quan hệ từ đầu.
> Thứ ba, về **lộ trình**: em thấy Viettel có con đường rõ ràng từ Senior → Tech Lead → Architect, và thực tế quan sát các anh/chị đi trước, lộ trình đó đi được. Em muốn lên chính thức để có stability đầu tư dài hạn, học sâu về domain và đóng góp vào các sáng kiến lớn hơn."*

**Câu xoáy đáp xoay**:
- *"Em đã làm 2 năm — tại sao bây giờ mới muốn chính thức?"* → Phụ thuộc context thực tế. Ví dụ: trước đây em là contractor/outsource, chưa có cơ hội thi tuyển chính thức; hoặc trước đây chờ đủ kinh nghiệm để tự tin ứng tuyển vị trí senior. Câu chuyện cần honest — interviewer có thể verify.
- *"Lead và đồng nghiệp tốt — nếu họ nghỉ hết, em vẫn gắn bó?"* → Đây là câu hay. Người tốt là lý do join, nhưng không phải lý do duy nhất để ở lại. Bài toán lớn và lộ trình rõ ràng quan trọng hơn trong dài hạn. Nếu team thay đổi, em vẫn có thể build mối quan hệ mới. Điều khó copy là domain knowledge, system scale, và institutional memory em đang tích lũy.
- *"Em nói 'lộ trình Senior → Tech Lead thấy đi được' — ai cụ thể trong team đã đi và mất bao lâu?"* → [Nêu tên/level cụ thể nếu biết, hoặc chuyển thành câu hỏi ngược:] "Anh/chị có thể chia sẻ thêm không — các Tech Lead hiện tại đã từ Senior mất khoảng bao lâu, để em có benchmark thực tế ạ?"

> **Clarify**: không cần.
> **Cảnh báo**: KHÔNG nhắc "vì ổn định, không bị layoff". Kể cả đúng ý — nói ra thành motivation chính là tự hạ giá.

---

## Câu 4. Kế hoạch 3 năm tới?

**Ý chính**:
1. Ghép **mục tiêu cá nhân** với **lộ trình Viettel**.
2. Chia 3 giai đoạn: **năm 1 (làm chắc) → năm 2 (mở rộng ảnh hưởng) → năm 3 (lead mảng)**.
3. Có mục tiêu đo được (mentor X người, own Y mảng, giảm Z % cost).

**Script mẫu**:
> *"Em chia 3 giai đoạn ạ.
> **Năm 1** em muốn làm chắc vai trò chính thức: own 1 module quan trọng trong hệ thống Hợp đồng điện tử, tham gia on-call rotation đầy đủ, lead 2–3 feature lớn end-to-end. Mục tiêu là chứng minh em xứng với level Senior chính thức.
> **Năm 2** em muốn mở rộng: làm design review cho team, mentor 1–2 junior lên mid, và đóng góp vào các quyết định kiến trúc cross-squad (ví dụ CA gateway chung, audit log chung).
> **Năm 3** em hướng đến Tech Lead một mảng nhỏ — dẫn dắt 3–5 người, vừa hands-on vừa định hướng kỹ thuật. Em cũng muốn đóng góp về tech branding (xây dựng thương hiệu kỹ thuật — viết blog, chia sẻ nội bộ, thậm chí public talk)."*

**Biến thể thiên quản lý** — xem Câu 9 Phần 3 đã có.

**Câu xoáy đáp xoay**:
- *"Tech Lead 'own 1 mảng' — em chọn mảng nào cụ thể và tại sao?"* → Em hướng đến mảng CA gateway và signing platform. Lý do: em đã có domain knowledge sâu nhất ở đây, và đây là core differentiator của Viettel Digital. Nếu có cơ hội phù hợp hơn ở mảng khác (ví dụ billing hay audit), em linh hoạt — quan trọng là được own bài toán có impact rõ.
- *"Năm 1 'chứng minh xứng Senior' — KPI cụ thể của em là gì?"* → Em muốn align với lead từ tuần đầu. Mục tiêu em tự đặt: (1) own 1 module end-to-end không cần hand-hold, (2) không có incident nào cause bởi code em trong 6 tháng, (3) junior dưới em có thể tự review PR cơ bản sau 3 tháng pair. Nếu lead có KPI khác, em sẽ điều chỉnh.
- *"Nếu sau 3 năm Viettel không có Tech Lead vị trí trống — em làm gì?"* → Trước tiên có cuộc nói chuyện thẳng với manager: lộ trình em đang tiến đến đâu, cần thêm gì, timeline thực tế. Nếu sau 6 tháng thảo luận vẫn không có hướng rõ, em sẽ re-evaluate. Nhưng "Tech Lead" là label — **impact và ownership thật sự** mới là tiêu chí. Nếu em có scope đó dù không có title, vẫn tốt.

> **Clarify**: "Em nên hình dung trong bối cảnh Viettel có nhiều thay đổi về tổ chức không ạ?" — tránh trả lời trong vacuum.

---

## Câu 5. Xung đột với đồng nghiệp — kể và cách xử lý.

**Ý chính**:
1. **Chọn xung đột có giải pháp tốt đẹp**, không chọn câu chuyện drama.
2. **STAR**:
   - Situation: bối cảnh ngắn, không đổ lỗi.
   - Task: vai trò của bạn.
   - Action: bạn chủ động làm gì (1-1, tìm data, bring lead vào nếu cần).
   - Result: kết quả + mối quan hệ sau đó.
3. Luôn **nói về bạn đã làm gì, không về người kia sai chỗ nào**.

**Script mẫu**:
> *"(S) Sprint cuối năm, em và 1 đồng nghiệp khác ý về cách implement luồng ký hợp đồng — em muốn tách async qua Kafka, bạn muốn giữ sync vì sợ phức tạp. Cả 2 đều có lý và đã tranh luận khá căng trong 1 meeting.
> (T) Em là người đưa ra đề xuất async nên cần bảo vệ, nhưng cũng cần giải quyết để sprint không chậm.
> (A) Em làm 3 việc:
>  (1) Hôm sau em chủ động hẹn 1-1 cafe, nói em hiểu lo của bạn về complexity, và em muốn nghe kỹ hơn.
>  (2) Em viết **RFC** (văn bản đề xuất gửi team review trước khi code) so sánh 2 phương án — latency, operational cost, rủi ro, effort — kèm POC nhỏ.
>  (3) Em đem document ra design review với lead + team để quyết chung, không để là quyết định 1-1 giữa em và bạn.
> (R) Cuối cùng team chọn async, nhưng có concede: em cam kết viết run-book + alert cho consumer group, giải quyết phần 'complexity' bạn lo. Sprint không chậm. Quan trọng hơn, em và bạn vẫn cộng tác tốt về sau — bạn thậm chí là reviewer chính cho PR của em, vì bạn hiểu luồng rõ nhất."*

**Câu xoáy đáp xoay**:
- *"1-1 cafe để hoà giải — nếu bạn kia từ chối meet?"* → Em gửi written summary quan điểm + 2 phương án qua message/email, đề xuất design review có lead tham gia. Không thể force meeting, nhưng vẫn cần giải quyết blockers. Lead review document là cách đưa quyết định ra khỏi conflict cá nhân.
- *"Em chọn async vì đúng — nhưng sau 6 tháng complexity thật sự là vấn đề như bạn lo?"* → Scenario này em đã lường trước khi cam kết viết run-book + alert. Nếu vẫn có vấn đề sau đó, em sẽ acknowledge bạn ấy đúng về risk, fix cùng team, và rút kinh nghiệm về risk assessment của mình. **Disagree and commit** (cãi cho ra nhẽ rồi ai thua cam kết thực thi) không có nghĩa là "em luôn đúng" — có nghĩa là quyết định dựa trên data tốt nhất lúc đó.
- *"Quyết định design có ghi lại không?"* → Có. Em dùng **ADR** (Architecture Decision Record — tài liệu ghi lại quyết định kiến trúc) format: context, options considered, decision made, consequences. Nếu bạn kia dissent, em ghi "alternative proposed by X, not chosen because Y" — có audit trail rõ ràng. Sau 6–12 tháng team retrospective có data cụ thể để đánh giá lại.

> **Clarify**: "Anh muốn em kể xung đột kỹ thuật hay xung đột về cách làm việc ạ?"
> **Cảnh báo**: KHÔNG kể câu chuyện mà mình đúng tuyệt đối / bạn kia sai tuyệt đối. Interviewer tìm người biết hợp tác, không tìm người luôn đúng.

---

## Câu 6. Lần gần nhất em thất bại / bị sai — học được gì?

**Ý chính**:
1. **Chọn thất bại đủ lớn để có bài học**, nhưng không phải thảm họa.
2. **STAR + L (Lesson)** — trọng tâm ở Lesson.
3. **Thừa nhận thẳng**, không đổ lỗi. Nhưng kết ở tone tích cực — cho thấy đã đổi hành vi sau đó.
4. Có thể dùng luôn câu chuyện **Câu 11 Phần 2 (cause sự cố)** nếu chưa kể.

**Script mẫu**:
> *"Em kể 1 lần em chọn sai kiến trúc đầu dự án. Lúc đó em thiết kế luồng ký hợp đồng là sync — API gọi Viettel-CA trực tiếp, chờ response rồi mới trả user. Em nghĩ sync đơn giản dễ debug.
> Vấn đề: khi peak, Viettel-CA đôi khi chậm 5–10 giây, **p99** (99% request nhanh hơn) API của em vọt lên 10 giây, user nghĩ hệ thống treo và ký lại → tạo duplicate. Cuối cùng em phải refactor sang async qua Kafka sau 3 tháng — tốn 3 sprint.
> Bài học em rút ra — và em đã áp dụng ở các tính năng sau:
> (1) Với luồng phụ thuộc third-party, **async là mặc định** trừ khi có lý do rõ để sync.
> (2) Em viết **RFC** 1–2 trang trước khi code cho mọi feature lớn, đưa team review — không quyết kiến trúc 1 mình nữa.
> (3) Em học về **idempotency key** (khóa bất biến lặp lại — cùng key gọi lại vẫn ra cùng kết quả) từ chính bug duplicate đó, giờ em dùng nó ở mọi API POST có retry."*

**Câu xoáy đáp xoay**:
- *"Idempotency key — implement thế nào cụ thể trong Spring Boot?"* → Client gửi header `X-Idempotency-Key: <UUID v4>`. Server lưu `(key, response)` vào Redis với TTL 24h. Nếu cùng key request lại trong 24h → trả lại stored response, không xử lý lại. Nếu request đang processing → trả 202 Accepted + `Retry-After: 5`. Dùng Redis `SETNX` (SET if Not eXists) làm mutex để tránh race condition khi 2 request cùng key đến đồng thời.
- *"RFC 1–2 trang — template của em gồm gì?"* → Problem statement (vấn đề là gì, tại sao cần giải), Proposed solution, Alternatives considered, Trade-offs (điểm đánh đổi của từng lựa chọn), Open questions, Affected components. Tổng không quá 2 trang. Mục tiêu: **align trước khi code** — không phải document sau khi đã làm.
- *"Async là mặc định — có trường hợp nào async sai không?"* → Có. (1) User cần kết quả ngay để quyết định tiếp (validate form, payment status realtime) — async làm UX tệ. (2) Infrastructure không có queue, complexity async vượt quá benefit cho scope nhỏ (team 2 người, 100 user). (3) Ordering nghiêm ngặt giữa events không thể guarantee qua async.

> **Clarify**: "Anh muốn em kể thất bại về kỹ thuật, hay về quy trình / phối hợp ạ?"
> **Cảnh báo**: KHÔNG "em chưa bao giờ thất bại". KHÔNG kể thất bại hoàn toàn không phải lỗi mình ("team không hỗ trợ") — nghe như đổ lỗi.

---

## Câu 7. Em làm việc độc lập hay team hơn?

**Ý chính**:
1. **Cả 2** — không chọn 1 phía tuyệt đối.
2. Cấu trúc: "tùy context, em làm tốt cả 2, và đây là khi nào em chọn từng kiểu".
3. Có **ví dụ cụ thể** cho mỗi kiểu.

**Script mẫu**:
> *"Em làm tốt cả 2 ạ, tùy context.
> Những lúc em làm **độc lập tốt nhất** là khi cần deep work — ví dụ khi em debug memory leak hoặc viết 1 bản RFC kiến trúc. Em thường block 2–3 giờ tập trung không meeting, tắt noti, chỉ mình và vấn đề. Em đã fix 1 memory leak Java mất 2 ngày đào sâu theo cách này.
> Những lúc em cần **team** là khi đưa ra quyết định kiến trúc, review PR, và on-call incident. Em không tin quyết định lớn làm 1 mình là tốt — phải có nhiều góc nhìn. Ví dụ incident gần nhất, team 3 người chia việc — 1 người check DB, 1 người check downstream, em lead war-room — **MTTR** (trung bình bao lâu phục hồi) chỉ 35 phút, solo em chắc mất gấp đôi.
> Em hợp với team có văn hóa cho deep work buổi sáng + collaboration buổi chiều."*

**Câu xoáy đáp xoay**:
- *"Deep work 2–3 giờ không meeting — ở Viettel thực tế có làm được không?"* → Em đang làm được bằng cách block calendar buổi sáng, thông báo team "morning = deep work, message sẽ trả buổi chiều." Văn hóa Viettel khá meeting-heavy — em muốn hỏi: team anh/chị thường meeting tập trung buổi nào để em plan phù hợp?
- *"Solo debug 2 ngày — team không biết em đang làm gì?"* → Em update daily brief qua async channel (Slack/Jira comment): "Đang debug memory leak PDF service, estimate xong cuối ngày mai, unblock: không cần gì từ team." Autonomous không có nghĩa invisible. Nếu block quá 4h, em sẽ reach out chủ động.
- *"Incident 3 người chia việc — ai assign việc cho ai? Có leader chính thức không?"* → Em tự lead war-room không cần chờ assign. Role assignment: người quen DB nhất check DB, người quen service check service log, em coord + communicate status ra ngoài. Không cần explicit assign nếu team đã quen nhau — tự organize theo expertise và availability là nhanh nhất.

> **Clarify**: không cần.
> **Cảnh báo**: KHÔNG nói "em thích làm 1 mình, không thích họp" — nghe antisocial. KHÔNG nói "em chỉ hợp làm team" — nghe không tự chủ.

---

## Câu 8. Áp lực cao nhất em từng chịu?

**Ý chính**:
1. Chọn câu chuyện có **áp lực thật** + **cách xử lý cụ thể** + **kết quả tốt**.
2. Có thể là: incident production nửa đêm / deadline sát / pressure từ stakeholder.
3. Không kể áp lực tâm lý chung chung ("em stress kéo dài"). Phải cụ thể thời điểm, việc, cách vượt qua.

**Script mẫu**:
> *"(S) Lần áp lực nhất là một đêm Sev2 — 11h đêm em bị paging, API ký hợp đồng 500 error 15%, đang là ngày cuối tháng, nhiều khách hàng gấp rút ký trước hạn.
> (T) Em là on-call primary, không có backup vì bạn kia đang đi công tác.
> (A) Em chia 30 phút đầu làm 3 việc song song:
>  (1) Mở war-room channel Slack, kéo lead + DBA vào sớm dù chưa rõ root cause — minh bạch hơn là ôm 1 mình.
>  (2) Mitigate ngay bằng rollback release gần nhất (chưa xác nhận đó là nguyên nhân, nhưng có hiện tượng đúng timeline) → error giảm.
>  (3) Song song, em trace và tìm root cause — 1 config Kafka topic mới chưa tạo ở prod.
> (R) MTTR 42 phút, khách hàng không mất hợp đồng nào. Sau incident em viết post-mortem, chia sẻ cho team. Lead khen cách em mở war-room sớm — không cố gồng.
> (L) Áp lực lớn không phải vấn đề — vấn đề là **cố ôm 1 mình**. Bây giờ em luôn kéo team vào sớm hơn, dù nghe có vẻ 'lớn chuyện'."*

**Câu xoáy đáp xoay**:
- *"Kéo lead vào sớm dù chưa rõ nguyên nhân — không sợ bị coi là không tự xử được?"* → Ngược lại. Kéo vào sớm khi có **user-facing impact** là professional responsibility, không phải dấu hiệu yếu. Sợ bị coi là yếu mà cố ôm 1 mình 30 phút mới là sai. Lead luôn muốn biết sớm để chuẩn bị communication với stakeholder và phân bổ support.
- *"Rollback ngay mà chưa xác nhận nguyên nhân — có rủi ro gì không?"* → Có — nếu nguyên nhân không phải release thì rollback vô ích + gây thêm downtime. Em dùng heuristic: nếu **timeline deploy khớp symptom onset trong ±1h** + error type phù hợp với thay đổi trong release → rollback là bet hợp lý trong 5 phút đầu. Sau khi ổn sẽ verify kỹ root cause thật.
- *"War-room Slack kéo vào — em có notify stakeholder non-tech (Product/Business) không?"* → Trong 5 phút đầu chưa — em focus mitigate. Sau khi có initial assessment (5–10 phút), em gửi message brief vào channel stakeholder: "Đang có incident X, ảnh hưởng Y, team đang xử lý, ETA Z phút." Không để họ nhận tin từ user trước khi biết từ team tech.

> **Clarify**: "Anh muốn em kể áp lực về kỹ thuật hay về tiến độ / stakeholder ạ?"

---

## Câu 9. Quyết định khó nhất em từng ra về kỹ thuật?

**Ý chính**:
1. Quyết định khó = **có trade-off rõ + không có đáp án đúng tuyệt đối**.
2. Khung: **Bối cảnh → 2 lựa chọn → tiêu chí so sánh → chọn + lý do → kết quả + retrospective**.
3. Thể hiện: mình cân nhắc trade-off, không quyết theo cảm tính, sẵn sàng admit nếu sau này thấy có thể tốt hơn.

**Script mẫu**:
> *"Quyết định khó nhất của em là chọn giữa **MariaDB partitioning** (`PARTITION BY RANGE (YEAR(created_at)*100 + MONTH(created_at))` — chia bảng theo tháng) vs **sharding** (phân mảnh dữ liệu ra nhiều node qua Spider engine hoặc application-level) cho bảng `signature` khi nó chạm 150M hàng.
> Sharding giải quyết scale tốt hơn nhưng effort gấp 5 (phải đổi code, saga cross-shard, ops rebalancing). Partitioning dễ hơn nhưng giới hạn single-node write.
> Em cân nhắc 4 tiêu chí: (1) tốc độ grow data — ước tính 2 năm đủ partitioning, (2) write throughput hiện tại — partitioning đủ đáp ứng 3× growth, (3) effort team 15 người sharding là quá nặng, (4) khả năng revert — partitioning rollback dễ hơn.
> Em chọn **partitioning theo tháng** + đặt checkpoint review sau 18 tháng. Em trình RFC với team, có 1 senior argue sharding sẽ 'đỡ đau sau này', em tôn trọng nhưng giữ quyết định vì tiêu chí 3 và 4.
> Kết quả: sau 14 tháng vẫn ổn, write latency không tăng. Nếu 6 tháng tới chạm giới hạn, em đã chuẩn bị RFC sharding song song rồi."*

**Câu xoáy đáp xoay**:
- *"Senior argue sharding — em giải quyết bất đồng này thế nào mà không gây conflict?"* → Em không bác bỏ — em ghi ý kiến của senior vào RFC là "alternative được xem xét, chưa chọn vì tiêu chí X, Y." Và đặt checkpoint rõ: "Nếu sau 18 tháng chạm giới hạn, em cam kết lead RFC sharding với senior đó là co-author." Kết quả: senior cảm thấy được nghe, audit trail rõ ràng, quyết định vẫn là của cả team.
- *"Partition theo tháng — hot partition của tháng hiện tại thế nào?"* → Tháng hiện tại luôn là hot partition (100% write). Em handle: (1) **MariaDB Event Scheduler** chạy job cuối mỗi tháng `ALTER TABLE signature REORGANIZE PARTITION pmax INTO (p202604 VALUES LESS THAN (...), pmax VALUES LESS THAN MAXVALUE)` để tạo partition tháng mới trước 3 tháng, không bao giờ hết, (2) index local trên partition hiện tại (MariaDB mỗi partition có index riêng), (3) monitor qua `information_schema.PARTITIONS` cho partition tháng hiện tại (`TABLE_ROWS`, `DATA_LENGTH`, `INDEX_LENGTH`), alert nếu write latency tăng bất thường.
- *"Checkpoint 18 tháng — em đo metric nào để biết 'đã chạm giới hạn'?"* → Write throughput > 80% IOPS capacity, write latency **p99** > SLO (cam kết chất lượng dịch vụ nội bộ), `SHOW SLAVE STATUS` → `Seconds_Behind_Master` > 1s, **InnoDB purge lag** (`SHOW ENGINE INNODB STATUS` section TRANSACTIONS — "History list length" > 10M → purge không kịp, undo log tăng). Alert ở **70%** để có thời gian chuẩn bị RFC sharding trước khi thực sự bị bottleneck.

> **Clarify**: "Anh muốn em kể quyết định về kiến trúc, về công nghệ, hay về quy trình ạ?"

---

## Câu 10. Nếu bất đồng với lead về giải pháp, em làm gì?

**Ý chính**:
1. Tinh thần: **disagree and commit** (cãi nhau cho ra nhẽ, rồi ai thua thì cam kết thực thi 100%, không sabotage).
2. 4 bước:
   - Lắng nghe kỹ, hỏi lại để hiểu lý do của lead.
   - Trình bày quan điểm mình có **data / tài liệu**, không cảm tính.
   - Nếu vẫn bất đồng — propose 1 cách thử nhỏ (POC, spike) để có dữ kiện.
   - Nếu lead vẫn quyết khác → cam kết thực thi tốt + note lại để sau review.
3. Thể hiện: tôn trọng hierarchy nhưng không "yes sir" mù.

**Script mẫu**:
> *"Em sẽ làm 4 bước.
> Một, em **lắng nghe** — hỏi lead lý do cụ thể cho quyết định. Nhiều khi họ đã nghĩ tới thứ mình chưa nghĩ (ràng buộc từ ops, tình hình team, context cao hơn).
> Hai, nếu em vẫn không đồng ý, em trình bày **quan điểm có data** — viết 1–2 trang so sánh trade-off (điểm đánh đổi), gửi lead trước khi cãi trong meeting. Cãi không data là cãi cảm tính.
> Ba, nếu còn bất đồng, em đề xuất **1 cách thử nhỏ** — POC 2 ngày, hoặc chạy cả 2 nhánh trên stage để có số đo cụ thể. Thường lead chấp nhận nếu effort nhỏ.
> Bốn, nếu cuối cùng lead vẫn quyết theo ý mình, em **commit 100% thực thi** tốt nhất có thể. Nhưng em sẽ log lại điểm em không đồng ý trong **ADR** (Architecture Decision Record — tài liệu ghi quyết định kiến trúc) — để sau 3–6 tháng retrospective, có data để nhìn lại.
> Em gọi tinh thần này là **disagree and commit**. Em đã áp dụng thực tế — 1 lần em argue về dùng RabbitMQ thay Kafka, lead chọn RabbitMQ, em commit. 6 tháng sau team review thấy đúng với scale hiện tại. Lần khác em argue về caching strategy, em đúng và lead sau đó đổi."*

**Câu xoáy đáp xoay**:
- *"Disagree and commit — nếu em commit nhưng biết chắc sẽ fail thì sao?"* → Nếu fail là predictable và hậu quả nghiêm trọng (data loss, Sev1 risk, vi phạm pháp lý) — đây không phải "bất đồng về trade-off" mà là **risk alert** — em có trách nhiệm escalate mạnh hơn, không chỉ commit. Disagree and commit áp dụng cho quyết định kỹ thuật có trade-off hợp lý, không phải cho rủi ro an toàn/dữ liệu rõ ràng.
- *"Lead chọn RabbitMQ, 6 tháng sau thấy đúng — em có 'rub it in' không?"* → Không. Mục tiêu retrospective không phải "ai đúng" mà "bài học gì để tốt hơn." Em present data thực tế (throughput, latency, ops cost), note trade-off quan sát được, đề xuất process cải tiến (ví dụ PoC nhỏ trước khi chọn tech lớn). Lead tốt sẽ acknowledge data — không cần ai phải "thắng."
- *"Log ADR 'không đồng ý' — lead có biết không, có gây friction không?"* → Em luôn mention trong meeting: "Em có note điểm này trong ADR là alternative." Không giấu. Mục tiêu không phải để chứng minh sau này — mà để team có **institutional memory** (ký ức tổ chức) về các trade-off đã cân nhắc. Khi team mới join sau này đọc ADR, họ hiểu tại sao quyết định đó được đưa ra, không phải làm lại từ đầu.

> **Clarify**: không cần.
> **Cảnh báo**: KHÔNG nói "em sẽ nghe lead vì lead là lead" — nghe thiếu chính kiến. KHÔNG nói "em sẽ đấu đến cùng" — nghe khó làm việc.

---

## Phụ lục — Câu hỏi NÊN hỏi lại interviewer

### Về vị trí
1. "Một ngày làm việc điển hình ở vị trí này trông như thế nào ạ?"
2. "Thách thức lớn nhất team đang gặp là gì ạ?"
3. "Anh kỳ vọng trong 90 ngày đầu em sẽ đạt được những gì ạ?"

### Về team & văn hóa
4. "Quy trình code review và deploy của team thế nào ạ?"
5. "Team xử lý production incident ra sao — có run-book, on-call rotation, post-mortem không ạ?"
6. "Team có hoạt động chia sẻ kỹ thuật nội bộ không ạ?"

### Về lộ trình
7. "Lộ trình thăng tiến từ vị trí này là gì, thời gian trung bình ạ?"
8. "Chu kỳ đánh giá performance và điều chỉnh lương như thế nào ạ?"
9. "Công ty hỗ trợ gì cho việc học tập — khóa học, hội nghị, chứng chỉ ạ?"

### Cuối buổi (mạnh nhất)
10. **"Có phần nào trong hồ sơ của em anh/chị còn băn khoăn không, để em giải thích thêm ạ?"** ← cho mình cơ hội vá lỗ hổng.
11. **"Bước tiếp theo của quy trình là gì, và khi nào em có thể nhận phản hồi ạ?"** ← kết chủ động.

## Phụ lục — Checklist nói behavioral

1. Dùng STAR, không kể lan man.
2. Có **số liệu** ở phần Result.
3. Câu chuyện **là của mình**, không phải của team mà kể như mình làm.
4. **Đừng đổ lỗi** — dù là nói về xung đột, thất bại, hay bất đồng.
5. **Im lặng 2–3 giây** trước khi trả lời câu khó — thể hiện suy nghĩ, không lắp bắp.
6. Câu trả lời **90–120 giây**, không quá 2 phút.

---

> **Hết 10 câu Phần 4.** 4 file đã xong — thư mục `answers/` đầy đủ.

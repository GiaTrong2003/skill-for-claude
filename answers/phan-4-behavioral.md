# Phần 4 — Behavioral & Câu hỏi ngược — Bộ câu trả lời

> **Format chung**: **Ý chính / nguyên tắc** → **Script mẫu** (có thể đọc gần thuộc lòng, luôn điều chỉnh theo thực tế bạn) → `> Clarify` / `> Cảnh báo` khi cần.
>
> **Khung STAR** cho câu tình huống: **S**ituation → **T**ask → **A**ction → **R**esult (+ **L**esson).

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
> Em đang làm chủ yếu ở mảng **Hợp đồng điện tử** — xây dựng nền tảng cho các doanh nghiệp và cá nhân ký kết hợp đồng số, tích hợp với Viettel-CA, USB token và SmartCA. Stack chính em dùng là Spring Boot, PostgreSQL, Kafka, Redis, và em cũng làm việc với cả front-end khi cần.
> Trong 2 năm qua, em có 2 điểm nhấn muốn chia sẻ: thứ nhất, em đã chủ trì refactor luồng ký async qua Kafka, giảm p99 từ 10 giây xuống 180ms. Thứ hai, em là on-call primary của team và đã handle vài incident Sev2, viết post-mortem mà team đang dùng làm template.
> Em tham gia phỏng vấn hôm nay vì em thực sự muốn gắn bó lâu dài với Viettel, lên chính thức và tiếp tục phát triển ở mảng domain em đang làm. Em rất mong được trao đổi thêm với anh/chị ạ."*

> **Clarify**: không cần clarify — đây là câu đầu tiên, nên trả lời mượt.
> **Cảnh báo**: **đừng lạm dụng buzzword** ("agile, scrum, microservices, devops...") mà không có context — interviewer thường vặn.

---

## Câu 2. Điểm mạnh — điểm yếu (mỗi bên 2 điểm, có ví dụ).

**Ý chính**:
1. **2 điểm mạnh**: 1 technical + 1 non-technical. Luôn có **ví dụ đi kèm**.
2. **2 điểm yếu**: chọn yếu **thật**, không phải yếu "giả" kiểu "em quá cầu toàn". **Phải kèm cách em đang cải thiện**.
3. Tránh điểm yếu kill câu hỏi ("em không giỏi code"). Chọn yếu vừa đủ để tin được + có action cụ thể.

**Script mẫu — Điểm mạnh**:
> *"Điểm mạnh thứ nhất của em là **khả năng đào sâu troubleshoot**. Ví dụ gần nhất em handle 1 incident p99 tăng đột biến, em trace từ symptom về root cause (transaction treo do catch thiếu rollback) trong 30 phút. Em không dừng ở triệu chứng mà luôn hỏi 'vì sao'.
> Thứ hai là **khả năng viết tài liệu và mentor**. Em đã viết template post-mortem và on-boarding guide cho team, mentor 1–2 junior. Em thấy giải thích cho người khác bắt em phải hiểu sâu hơn — đây là cách em học hiệu quả nhất."*

**Script mẫu — Điểm yếu**:
> *"Điểm yếu đầu tiên: em **ngại feedback xung đột trực tiếp**. Hồi đầu em thường tránh nói trái ý lead trong meeting, dù trong đầu không đồng ý. Em đang cải thiện bằng cách: trước mỗi design review em note sẵn 1–2 điểm không đồng ý + lý lẽ, để khi tới meeting em bắt buộc nói ra. Sau 6 tháng em đã chủ động challenge 2 quyết định kiến trúc — 1 lần em đúng, 1 lần em sai nhưng học được nhiều.
> Thứ hai: em **chưa mạnh về front-end**. Em chỉ sửa được vài chỗ React cơ bản. Em đang học thêm vào buổi tối (mỗi tuần 2–3 giờ), và em đã nhận 1 task front-end nhỏ mỗi sprint để thực hành."*

> **Clarify**: "Anh muốn em kể điểm mạnh thiên về kỹ thuật hay soft skill ạ?" — câu này cho mình thời gian + chọn phù hợp vị trí.
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
> **Năm 3** em hướng đến Tech Lead một mảng nhỏ — dẫn dắt 3–5 người, vừa hands-on vừa định hướng kỹ thuật. Em cũng muốn đóng góp về tech branding (chia sẻ nội bộ, thậm chí public talk)."*

**Biến thể thiên quản lý** — xem Câu 9 Phần 3 đã có.

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
>  (2) Em viết document so sánh 2 phương án — latency, operational cost, rủi ro, effort — kèm POC nhỏ.
>  (3) Em đem document ra design review với lead + team để quyết chung, không để là quyết định 1-1 giữa em và bạn.
> (R) Cuối cùng team chọn async, nhưng có concede: em cam kết viết run-book + alert cho consumer group, giải quyết phần 'complexity' bạn lo. Sprint không chậm. Quan trọng hơn, em và bạn vẫn cộng tác tốt về sau — bạn thậm chí là reviewer chính cho PR của em, vì bạn hiểu luồng rõ nhất."*

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
> Vấn đề: khi peak, Viettel-CA đôi khi chậm 5–10 giây, p99 API của em vọt lên 10 giây, user nghĩ hệ thống treo và ký lại → tạo duplicate. Cuối cùng em phải refactor sang async qua Kafka sau 3 tháng — tốn 3 sprint.
> Bài học em rút ra — và em đã áp dụng ở các tính năng sau:
> (1) Với luồng phụ thuộc third-party, **async là mặc định** trừ khi có lý do rõ để sync.
> (2) Em viết RFC 1–2 trang trước khi code cho mọi feature lớn, đưa team review — không quyết kiến trúc 1 mình nữa.
> (3) Em học về idempotency key từ chính bug duplicate đó, giờ em dùng nó ở mọi API POST có retry."*

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
> Những lúc em cần **team** là khi đưa ra quyết định kiến trúc, review PR, và on-call incident. Em không tin quyết định lớn làm 1 mình là tốt — phải có nhiều góc nhìn. Ví dụ incident gần nhất, team 3 người chia việc — 1 người check DB, 1 người check downstream, em lead war-room — MTTR chỉ 35 phút, solo em chắc mất gấp đôi.
> Em hợp với team có văn hóa cho deep work buổi sáng + collaboration buổi chiều. Em nghĩ Viettel cũng gần như vậy."*

> **Clarify**: không cần.
> **Cảnh báo**: KHÔNG nói "em thích làm 1 mình, không thích họp" — nghe antisocial. KHÔNG nói "em chỉ hợp làm team" — nghe không tự chủ được.

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

> **Clarify**: "Anh muốn em kể áp lực về kỹ thuật hay về tiến độ / stakeholder ạ?"

---

## Câu 9. Quyết định khó nhất em từng ra về kỹ thuật?

**Ý chính**:
1. Quyết định khó = **có trade-off rõ + không có đáp án đúng tuyệt đối**.
2. Khung: **Bối cảnh → 2 lựa chọn → tiêu chí so sánh → chọn + lý do → kết quả + retrospective**.
3. Thể hiện: mình cân nhắc trade-off, không quyết theo cảm tính, và sẵn sàng admit nếu sau này thấy có thể tốt hơn.

**Script mẫu**:
> *"Quyết định khó nhất của em là chọn giữa **Postgres partitioning** vs **sharding** cho bảng `signature` khi nó chạm 150M hàng.
> Sharding giải quyết scale tốt hơn nhưng effort gấp 5 (phải đổi code, saga cross-shard, ops rebalancing). Partitioning dễ hơn nhưng giới hạn single-node write.
> Em cân nhắc 4 tiêu chí: (1) tốc độ grow data — ước tính 2 năm đủ partitioning, (2) write throughput hiện tại — partitioning đủ đáp ứng 3× growth, (3) effort team 15 người sharding là quá nặng, (4) khả năng revert — partitioning rollback dễ hơn.
> Em chọn **partitioning theo tháng** + đặt checkpoint review sau 18 tháng. Em trình RFC với team, có 1 senior argue sharding sẽ 'đỡ đau sau này', em tôn trọng nhưng giữ quyết định vì tiêu chí 3 và 4.
> Kết quả: sau 14 tháng vẫn ổn, write latency không tăng. Nếu 6 tháng tới chạm giới hạn, em đã chuẩn bị RFC sharding song song rồi.
> Bài học: quyết định khó không phải 'đúng–sai', mà là 'chọn phương án phù hợp với team + thời điểm + có lối lui'. Em không hối tiếc quyết định đó — dù 2 năm nữa có thể vẫn phải sharding."*

> **Clarify**: "Anh muốn em kể quyết định về kiến trúc, về công nghệ, hay về quy trình ạ?"

---

## Câu 10. Nếu bất đồng với lead về giải pháp, em làm gì?

**Ý chính**:
1. Tinh thần: **disagree and commit** — tranh luận thẳng thắn, khi đã quyết thì cam kết thực thi, không sabotage.
2. 4 bước:
   - Lắng nghe kỹ, hỏi lại để hiểu lý do của lead.
   - Trình bày quan điểm mình có **data / tài liệu**, không cảm tính.
   - Nếu vẫn bất đồng — propose 1 cách thử nhỏ (POC, spike) để có dữ kiện.
   - Nếu lead vẫn quyết khác → cam kết thực thi tốt + note lại để sau review.
3. Thể hiện: tôn trọng hierarchy nhưng không "yes sir" mù.

**Script mẫu**:
> *"Em sẽ làm 4 bước.
> Một, em **lắng nghe** — hỏi lead lý do cụ thể cho quyết định. Nhiều khi họ đã nghĩ tới thứ mình chưa nghĩ (ràng buộc từ ops, tình hình team, context cao hơn).
> Hai, nếu em vẫn không đồng ý, em trình bày **quan điểm có data** — viết 1–2 trang so sánh trade-off, gửi lead trước khi cãi trong meeting. Cãi không data là cãi cảm tính.
> Ba, nếu còn bất đồng, em đề xuất **1 cách thử nhỏ** — POC 2 ngày, hoặc chạy cả 2 nhánh trên stage để có số đo cụ thể. Thường lead chấp nhận nếu effort nhỏ.
> Bốn, nếu cuối cùng lead vẫn quyết theo ý mình, em **commit 100% thực thi** tốt nhất có thể. Nhưng em sẽ log lại điểm em không đồng ý trong design doc — để sau 3–6 tháng team retrospective, có data để nhìn lại.
> Em gọi tinh thần này là **disagree and commit**. Em đã áp dụng thực tế — có 1 lần em argue về việc dùng RabbitMQ thay Kafka, lead chọn RabbitMQ, em commit. 6 tháng sau team review thấy đã đúng với scale hiện tại. Lần khác em argue về caching strategy, em đúng và lead sau đó đổi — vì có data."*

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

1. Dùng STAR / CPRRL, không kể lan man.
2. Có **số liệu** ở phần Result.
3. Câu chuyện **là của mình**, không phải của team mà kể như mình làm.
4. **Đừng đổ lỗi** — dù là nói về xung đột, thất bại, hay bất đồng.
5. **Im lặng 2–3 giây** trước khi trả lời câu khó — thể hiện suy nghĩ, không lắp bắp.
6. Câu trả lời **90–120 giây**, không quá 2 phút.

---

> **Hết 10 câu Phần 4.** 4 file đã xong — thư mục `answers/` đầy đủ.

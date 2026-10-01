# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 9:15–12:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 9:15–9:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (9:30–9:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Bổ sung lời chào, câu xã giao lịch sự hoặc thông tin tư vấn chung không chứa sai sót thực tế dù không nằm trực tiếp trong context. | Trả lời thông tin hư cấu/sai lệch nghiêm trọng về giá cả, thông số kỹ thuật, hoặc chính sách bảo hành của OrbitTech Store (hallucination). | Siết chặt system prompt ("chỉ trả lời dựa trên context"), giảm LLM temperature, thêm guardrail kiểm tra tính xác thực. |
| Answer Relevance | Khách hàng đặt câu hỏi quá mở/mơ hồ và trợ lý phải đưa ra các câu hỏi làm rõ (clarifying questions) thay vì trả lời trực tiếp ngay. | Trả lời hoàn toàn lạc đề (off-topic), cung cấp thông tin sản phẩm khác không liên quan đến thắc mắc của khách hàng. | Tối ưu prompt phát hiện ý định (intent detection), quy định cấu trúc phản hồi bám sát câu hỏi người dùng. |
| Context Recall | Câu hỏi ngoài phạm vi hỗ trợ (out-of-domain/unanswerable query) nơi kiến thức store không tồn tại và assistant từ chối hợp lệ. | Tài liệu kiến thức OrbitTech có đầy đủ câu trả lời nhưng bộ truy xuất (retriever) bỏ sót các chunk thông tin quan trọng. | Mở rộng và cải thiện chia nhỏ văn bản (chunking strategy), bổ sung từ khóa đồng nghĩa, nâng cấp lên hybrid search. |
| Context Precision | Top-k chunks retrieved chứa thông tin phụ trợ bổ ích và chunk quan trọng nhất vẫn nằm trong top-k dù không ở vị trí đầu tiên. | Các chunk đứng đầu (rank 1, 2) hoàn toàn là nhiễu/không liên quan, đẩy chunk chứa câu trả lời đúng xuống dưới hoặc ra ngoài k. | Bổ sung bước Reranking (Cross-Encoder reranker), tinh chỉnh thuật toán BM25 / Vector search score weight. |
| Completeness | Người dùng yêu cầu tóm tắt ngắn gọn ý chính và assistant bỏ qua các chi tiết phụ không bắt buộc. | Trả lời thiếu các bước hướng dẫn bắt buộc trong quy trình (ví dụ: đổi trả có 3 bước nhưng chỉ nêu 1 bước), làm khách hàng không thực hiện được. | Tăng max_tokens, điều chỉnh prompt yêu cầu liệt kê đầy đủ tất cả điều kiện và các bước theo quy chuẩn. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:*
> - **Condition 1 (Thứ tự gốc):** Đưa `Answer_A` ở vị trí `Option 1` và `Answer_B` ở vị trí `Option 2` vào prompt cho LLM Judge chấm pairwise.
> - **Condition 2 (Đảo thứ tự):** Đổi vị trí `Answer_B` lên `Option 1` và `Answer_A` xuống `Option 2`.
> - **Phân tích:** Nếu `Answer_A` thắng ở Condition 1 nhưng `Answer_B` lại thắng ở Condition 2 (luôn ưu tiên Option 1), hệ thống mắc Position Bias. Để khắc phục, chạy cả 2 lượt và lấy điểm trung bình hoặc áp dụng position swapping check.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:*
> - Đánh giá theo **mật độ thông tin chuẩn xác (information density)** thay vì độ dài.
> - Bổ sung tiêu chuẩn phạt (penalty criteria) cho các câu trả lời dài dòng, lặp từ, hoặc chứa nội dung rác (filler words).
> - Đưa vào vài ví dụ (few-shot examples) chứng minh câu trả lời ngắn gọn, đúng trọng tâm đạt điểm 5/5, trong đó câu trả lời rườm rà bị điểm thấp.
> - Yêu cầu LLM Judge trích xuất các ý chính (key points) trước khi chấm điểm thay vì đọc lướt văn bản.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:*
> - LLM Judge có thể mắc sai số hệ thống (systematic bias) hoặc đánh giá quá nương tay/quá khắt khe (leniency/severity bias).
> - Calibrate với nhãn của con người (domain experts) giúp đo lường mức độ tương quan (correlation coefficient như Cohen's Kappa, Spearman), phát hiện điểm mù của model và hiệu chỉnh prompt/weight để LLM Judge đạt độ tin cậy tương đương chuyên gia.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | `>= 0.85` | Trong tư vấn OrbitTech Store, câu trả lời sai sự thật gây thiệt hại uy tín và pháp lý nghiêm trọng nhất. Cần ngưỡng chặn khắt khe nhất. |
| Answer Relevance | `>= 0.75` | Đảm bảo trợ lý trả lời đúng trọng tâm thắc mắc của khách hàng, không trả lời vòng vo lạc đề gây lãng phí thời gian. |
| Completeness | `>= 0.70` | Đảm bảo cung cấp đủ thông tin hướng dẫn cốt lõi để khách hàng tự giải quyết được vấn đề mà không cần hỏi lại. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*
> - **Offline Evaluation (Pre-deployment / CI/CD):** Đánh giá tự động trên Golden Dataset trước khi deploy bản cập nhật mới nhằm phát hiện sớm regression và chặn các phiên bản không đạt quality gate.
> - **Online Evaluation (Production Monitoring):** Đánh giá liên tục trên dữ liệu thật của khách hàng qua telemetry, user feedback (thumbs up/down), tỉ lệ hỏi lại, và lấy mẫu log cho LLM Judge giám sát thời gian thực.
> - **Human Review (Expert Auditing & Calibration):** Đánh giá thủ công định kỳ bởi chuyên gia trên các ca điểm thấp (low-score alerts), ca lỗi (failure cases), để cập nhật Golden Dataset và calibrate LLM Judge.

---

## Part 2 — Core Coding (9:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.

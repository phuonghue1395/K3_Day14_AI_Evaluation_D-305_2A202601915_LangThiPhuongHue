# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 09:15–12:00

**Domain:** Northstar University Student Services

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 09:15–09:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (09:30–09:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Khi người dùng hỏi thăm xã giao (chit-chat), đặt câu hỏi mở không có trong tài liệu hoặc cố tình hỏi mẹo mà hệ thống cần từ chối khéo léo. | Khi câu hỏi của sinh viên liên quan đến chính sách cốt lõi (hạn nộp hồ sơ, học phí, điều kiện học bổng) nhưng hệ thống tự bịa ra thông tin không có trong tài liệu. | Điều chỉnh lại prompt của Generator để nhấn mạnh tính Grounding (chỉ trả lời dựa trên context), giảm Temperature, hoặc kiểm tra lại retriever xem có lấy sai context không. |
| Answer Relevance | Khi câu hỏi của sinh viên quá ngắn, mơ hồ hoặc có nhiều cách hiểu, khiến câu trả lời phải bao quát nhiều khía cạnh hoặc giải thích rộng hơn. | Khi sinh viên hỏi một đường (ví dụ: quy trình xin nghỉ học tạm thời) nhưng hệ thống trả lời một nẻo (giới thiệu về học bổng hoặc calendar). | Tối ưu hóa khâu xử lý câu hỏi đầu vào (Query Reformulation/Expansion), cải thiện Prompt để LLM tập trung trả lời đúng trọng tâm câu hỏi. |
| Context Recall | Khi câu hỏi đơn giản chỉ cần 1 thông tin nhỏ và hệ thống đã tìm thấy nó, dù expected answer dài hơn và chứa nhiều thông tin phụ không cần thiết. | Khi câu hỏi yêu cầu các điều kiện bắt buộc hoặc các ngoại lệ chính sách (ví dụ: điều kiện nhận học bổng) nhưng Retriever bỏ sót các tài liệu chứa điều kiện đó. | Nâng cao chất lượng Retrieval: tăng Top-K chunks, tối ưu hóa chunk size/overlap, sử dụng Hybrid Search (BM25 + Vector Search), hoặc cải thiện chất lượng embedding. |
| Context Precision | Khi hệ thống lấy về nhiều thông tin dư thừa (nhiễu) xếp ở cuối danh sách chunks nhưng Generator vẫn đủ thông minh để lọc và trả lời đúng. | Khi tài liệu chứa câu trả lời thực tế bị xếp ở cuối cùng của danh sách chunks và bị Generator bỏ qua hoặc không đọc tới (hiện tượng Lost in the Middle). | Tích hợp Reranker (như Cross-Encoder hoặc Lexical Overlap Reranker) để đẩy các chunks có độ liên quan cao nhất lên đầu tiên trước khi đưa vào LLM. |
| Completeness | Khi câu hỏi mang tính chất kham khảo sơ bộ và câu trả lời ngắn gọn là đủ để định hướng cho sinh viên, không cần liệt kê toàn bộ chi tiết vụn vặt. | Khi câu hỏi yêu cầu đầy đủ quy trình hoặc các mốc thời gian cụ thể nhưng câu trả lời lại thiếu mất các bước quan trọng hoặc các điều kiện đi kèm. | Cải thiện Prompt của Generator yêu cầu liệt kê đầy đủ chi tiết, điều kiện và các trường hợp ngoại lệ có trong context; sử dụng Chain-of-Thought để liệt kê các ý trước khi trả lời. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:*
> - **Chuẩn bị:** Chọn ra một tập dữ liệu gồm câu hỏi cùng hai câu trả lời ứng viên khác nhau (Candidate Answer A và Candidate Answer B).
> - **Condition 1 (Thứ tự A-B):** Đưa vào Prompt của LLM Judge với định dạng: `Response 1: [Answer A]`, `Response 2: [Answer B]`. Yêu cầu Judge đánh giá xem Response nào tốt hơn (hoặc chấm điểm cho từng Response).
> - **Condition 2 (Thứ tự B-A):** Đổi ngược vị trí trong Prompt: `Response 1: [Answer B]`, `Response 2: [Answer A]`. Giữ nguyên câu hỏi và tiêu chí đánh giá.
> - **Phân tích kết quả:** Nếu LLM Judge có xu hướng chọn câu trả lời ở vị trí `Response 1` (hoặc `Response 2`) nhiều hơn đáng kể bất kể nội dung là A hay B, thì hệ thống đang bị Position bias.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:*
> - **Thiết kế Rubric dạng Checklist/Fact-based:** Thay vì yêu cầu đánh giá chung chung bằng thang điểm 1-5, hãy chia nhỏ tiêu chí thành danh sách các điểm thông tin cụ thể (ví dụ: "+1 điểm nếu nêu đúng hạn chót ngày 15/9", "+1 điểm nếu nêu đúng mức phạt 10%").
> - **Đặt luật không thưởng cho độ dài:** Hướng dẫn rõ ràng trong prompt của Judge: "Không chấm điểm cao hơn cho các câu trả lời dài dòng hoặc chứa thông tin thừa. Một câu trả lời ngắn gọn, trực diện và đủ ý phải được điểm tối đa."
> - **Phạt thông tin dư thừa:** Đưa vào tiêu chí trừ điểm hoặc cảnh báo nếu câu trả lời chứa thông tin không liên quan hoặc lan man (Ví dụ: "Trừ 1 điểm nếu câu trả lời chứa thông tin ngoài phạm vi câu hỏi").

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:*
> - LLM Judge không hoàn hảo và thường mang các bias cố hữu (như tự ưu tiên câu trả lời của chính mình, thích câu trả lời dài, hoặc không hiểu sâu các sắc thái ngôn ngữ của một domain cụ thể).
> - Việc hiệu chuẩn (calibration) với nhãn do con người chấm (human labels) trên một tập validation nhỏ giúp:
>   1. Tính toán độ tương quan (Correlation - như Pearson hoặc Spearman) giữa điểm của LLM và con người.
>   2. Phát hiện và điều chỉnh các điểm lệch hệ thống (ví dụ: LLM luôn chấm rộng tay hơn con người 1 điểm).
>   3. Tinh chỉnh thiết kế prompt và rubric cho LLM Judge để tiệm cận nhất với tiêu chí đánh giá thực tế của chuyên gia.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.90 | Đối với chatbot dịch vụ sinh viên, thông tin chính sách phải tuyệt đối chính xác dựa trên tài liệu gốc để tránh các rủi ro pháp lý/tài chính hoặc khiếu nại từ sinh viên (ví dụ: nhầm lẫn hạn đóng học phí). |
| Answer Relevance | 0.85 | Đảm bảo chatbot thực sự hiểu và phản hồi trực tiếp câu hỏi của sinh viên, tránh tình trạng trả lời lạc đề gây ức chế cho người dùng. |
| Completeness | 0.80 | Đảm bảo sinh viên nhận được đầy đủ các điều kiện và ngoại lệ chính sách quan trọng. Mức này có thể thấp hơn Faithfulness một chút vì việc thiếu một chi tiết nhỏ ít nghiêm trọng hơn việc đưa ra thông tin sai lệch (hallucination). |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*
> - **Offline evaluation (Pre-deployment):** Chạy tự động trong luồng CI/CD mỗi khi có sự thay đổi về code, prompt của LLM, tham số retriever, hoặc cấu trúc dữ liệu. Sử dụng bộ dữ liệu chuẩn (Golden Dataset) để nhanh chóng phát hiện hồi quy lỗi (regression) trước khi deploy lên môi trường Production.
> - **Online evaluation (In production):** Chạy liên tục hoặc giám sát trên dữ liệu thực tế từ người dùng. Sử dụng các metric gián tiếp (phản hồi upvote/downvote, tỷ lệ thoát trang, fallback rate) hoặc các LLM Judge nhẹ để theo dõi hiệu năng hệ thống theo thời gian thực và phát hiện sự trôi lệch dữ liệu (data drift).
> - **Human review (Periodic / Sampled):** Thực hiện định kỳ (hàng tuần/hàng tháng) bởi các chuyên gia hoặc nhân viên hỗ trợ sinh viên trên một mẫu nhỏ dữ liệu thực tế (ví dụ: 1-5% số lượt hội thoại) hoặc các case mà hệ thống gắn cờ (flagged) nghi ngờ lỗi. Kết quả này dùng để cập nhật bộ Golden Dataset và hiệu chỉnh LLM Judge.

---

## Part 2 — Core Coding (09:45–10:40)

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

Thiết kế rubric domain-specific cho Student Services. Mỗi mức phải đủ cụ thể để
hai người chấm độc lập có thể hiểu giống nhau.

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

### Exercise 3.4 — Framework Comparison (Bonus +10)

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

# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Nguyễn Anh Đức
**Cohort:** _A20-K1_
**Ngày submit:** 06/05/2026

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

> Paste output của `python 00-setup/detect-hardware.py` vào đây, hoặc điền thủ công:

- **OS:** _Ubuntu 24.04 (WSL2 trên Windows 11)_
- **CPU:** _Intel(R) Core(TM) i5-10300H CPU @ 2.50GHz_
- **Cores:** _8 physical / 8 logical_
- **CPU extensions:** _AVX2_
- **RAM:** _7.7 GB_
- **Accelerator:** _NVIDIA GeForce GTX 1650, 4GB VRAM_
- **llama.cpp backend đã chọn:** _CUDA_
- **Recommended model tier:** _TinyLlama-1.1B_

**Setup story** (≤ 80 chữ): những gì cần thay đổi để lab chạy được trên máy bạn:

_Sử dụng WSL2 trên Windows. Cần cài đặt thêm `build-essential` và `python3-venv` để biên dịch thành công `llama-cpp-python` từ mã nguồn. Script setup ban đầu thiếu package `uvicorn` và `fastapi` cho server nên đã phải cài đặt thủ công bổ sung gói `[server]`._

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

> Paste bảng từ `benchmarks/01-quickstart-results.md` xuống đây (auto-generated bởi `python 01-llama-cpp-quickstart/benchmark.py`).

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| (Q4_K_M) | 4344 | 171 / 412 | 63.1 / 100.6 | 3830 / 6481 / 6778 | 15.9 |
| (Q2_K)   | 3070 | 318 / 1219 | 48.4 / 124.8 | 3053 / 9080 / 10590 | 20.7 |

**Một quan sát** (≤ 50 chữ): Q4_K_M vs Q2_K trên máy bạn — số liệu nói gì? Quality đáng đánh đổi không?

_Q2_K load và decode nhanh hơn (20.7 vs 15.9 tok/s), nhưng độ trễ TTFT và P99 cao, không ổn định. RAM 7.7GB hoàn toàn đủ chứa Q4_K_M, do đó chất lượng của Q4_K_M hoàn toàn xứng đáng đánh đổi một chút tốc độ decode._

---

## 3. Track 02 — llama-server load test

> Chạy 2 lần locust ở concurrency 10 và 50, paste tóm tắt bên dưới.

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.12 | 24000 | 46000 | 46000 | 0 |
| 50 | 0.10 | 24000 | 51000 | 51000 | 0 |

**KV-cache observation** (từ `record-metrics.py`): peak `llamacpp:kv_cache_usage_ratio` ở concurrency 50 = _[Bỏ qua bước này do dùng Python server]_, nghĩa là …

_Bỏ qua do bản llama-cpp-python không hỗ trợ endpoint /metrics của bản gốc C++._

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** _stub: localhost only_
- **N17 (Data pipeline):** _stub: in-memory dict_
- **N18 (Lakehouse):** _stub: SQLite_
- **N19 (Vector + Feature Store):** _stub: TOY_DOCS_

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: _0 ms_
- retrieve: _0.0 ms_
- llama-server: _27388.6 ms_

**Reflection** (≤ 60 chữ): bottleneck nằm ở đâu? Có khớp với kỳ vọng không?

_Bottleneck chính nằm ở phần llama-server sinh chữ (LLM latency) do pipeline RAG giả lập retrieval quá nhanh (trong bộ nhớ). Điều này hoàn toàn khớp với kỳ vọng vì inference luôn tốn nhiều tính toán nhất._

---

## 5. Bonus — The single change that mattered most

> **Most important section.** Pick **một** thay đổi từ bonus track (build flag, thread sweep, quant pick, GPU offload, KV-cache quantization, speculative decoding, bất cứ challenge nào trong `BONUS-llama-cpp-optimization/CHALLENGES.md`) đã tạo ra speedup lớn nhất trên máy bạn.

**Change:** _<vd: rebuild llama.cpp với `-DGGML_NATIVE=ON -DGGML_BLAS=ON`; vd: hạ `-t` từ 12 xuống 6; vd: bật Metal trên M2>_

**Before vs after** (paste 2-3 dòng từ sweep output):

```
before: <số liệu>
after:  <số liệu>
speedup: ~<X.Y>×
```

**Tại sao nó work** (1–2 đoạn ngắn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với một bạn cùng lớp đang ngồi cạnh. Tránh "vibes-based" reasoning — bám vào mô hình mental của hardware (memory bandwidth? compute? cache?). Nếu kết quả khác kỳ vọng từ deck, nói rõ — đó là phần grader thưởng điểm._

---

## 6. (Optional) Điều ngạc nhiên nhất

_(1–2 câu — không bắt buộc, nhưng người grader đọc tất cả)_

_Answer here._

---

## 7. Self-graded checklist

- [ ] `hardware.json` đã commit
- [ ] `models/active.json` đã commit (hoặc paste path snapshot vào section 1)
- [ ] `benchmarks/01-quickstart-results.md` đã commit
- [ ] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [ ] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [ ] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [ ] `make verify` exit 0 (chạy ngay trước khi push)
- [ ] Repo trên GitHub ở chế độ **public**
- [ ] Đã paste public repo URL vào VinUni LMS

---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.

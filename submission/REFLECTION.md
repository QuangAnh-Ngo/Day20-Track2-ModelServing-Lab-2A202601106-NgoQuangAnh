# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** _<Họ Tên>_
**Cohort:** _<A20-K1 / A20-K2 / ...>_
**Ngày submit:** _<YYYY-MM-DD>_

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

  Platform : Linux 6.6.87.2-microsoft-standard-WSL2 (x86_64)
  CPU      : 11th Gen Intel(R) Core(TM) i7-11850H @ 2.50GHz
             8 physical · 16 logical cores
             extensions: AVX-512, AVX2
  RAM      : 15.5 GB
  GPU      : nvidia_cuda
             - nvidia: NVIDIA RTX A2000 Laptop GPU, 4096 MiB
────────────────────────────────────────────────────────────────

  Model         : Gemma 4 E2B  [LAB_MODEL=gemma4-e2b]
                  unsloth/gemma-4-E2B-it-GGUF  (~5.2 GB)
                  primary  gemma-4-E2B-it-UD-Q4_K_XL.gguf  (2.97 GB)
                  compare  gemma-4-E2B-it-UD-Q2_K_XL.gguf  (2.24 GB)
                  chosen because: enough RAM for the default model
  Other option  : LAB_MODEL=qwen35-0.8b  ->  Qwen3.5 0.8B, ~0.9 GB, needs 4.0 GB RAM
  llama.cpp     : prebuilt release b10488  (llama-b10488-bin-win-cuda-12.4-x64.zip)
  GPU offload   : OFF -- an accelerator is installed but this llama.cpp build enumerates no devices -- offload would silently run on CPU
                  base track is unaffected (100 pts need no GPU).
                  to use the CUDA you have, build from source:
                  LLAMA_CMAKE_FLAGS=-DGGML_CUDA=ON make build-llama
  source build  : -DGGML_CUDA=ON  (bonus B1 -- not used by the base track)
  Tracks open   : 01-measure, 02-serve, 03-integrate, bonus/sweeps

**Chạy ở đâu:** _<laptop của tôi>_
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

_Answer here._
Không
---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 73020 | 377 / 434 | 63.9 / 65.8 | 4346 / 4559 / 4559 | 15.7 |
| UD-Q2_K_XL | 2.24 | 56122 | 625 / 740 | 61.4 / 68.7 | 4444 / 4810 / 4810 | 16.3 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

_Answer here._
Không đáng. Đã trả lời chi tiết trong quickstart
---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | | | | | | |
| 50 | | | | | | |

- **Offered load tăng 5×, throughput thực tăng:** _<X.XX>×_
- **P95 tăng:** _<X.XX>×_
- **Effective concurrency ở 50 users:** _<số>_ so với `--parallel` = _<số>_ slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): _<số>_ / _<slots>_ slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

_Answer here._

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | local run, không deploy cloud/IaC thật | stub |
| N17 Data pipeline | context nhỏ được chuẩn bị sẵn trong script | stub |
| N18 Lakehouse | không đọc từ lakehouse thật | stub |
| N19 Vector + features | keyword overlap fallback, không dùng vector DB/feature store thật | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.1 ms
- llm: 4159.3 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

Nghẽn cổ chai nằm ở stage LLM: 4159.3 ms trên tổng 4159.4 ms.Để giảm latency 2x => tối ưu LLM trước: giảm max_tokens, dùng LAB_N_THREADS=8, hoặc thử CUDA/GPU offload.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** hạ số thread từ `-t 16` xuống `-t 8`, tức dùng đúng 8 physical cores thay vì dùng hết 16 logical cores.


```
before:  4.5 tok/s  (-t 16)
after:   16.8 tok/s (-t 8)
speedup: 3.73×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

_Answer here._

Kết quả `make tune` cho thấy throughput tăng từ `4.2 tok/s` ở 1 thread lên `12.8 tok/s` ở 4 threads và đạt tốt nhất `16.8 tok/s` ở 8 threads. Nhưng khi tăng tiếp lên 16 threads, tốc độ rơi mạnh xuống chỉ còn `4.5 tok/s`. Vì vậy thay đổi quan trọng nhất là không dùng hết logical cores, mà giới hạn về số physical cores. 

Điều này hợp lý vì decode của LLM thường bị chặn bởi memory bandwidth hơn là compute thuần. Khi dùng 16 logical threads trên CPU có 8 physical cores, các hyperthreads tranh nhau cùng cache, memory bandwidth và execution resources. Thêm thread không tạo thêm băng thông bộ nhớ, mà còn tăng scheduling overhead và contention. Với `-t 8`, mỗi physical core làm việc hiệu quả hơn, nên throughput cao hơn rõ rệt.

(Có sử dụng AI để chau chuốt câu từ. ý hiểu ban đầu là từ 8 -> 16 đã giảm tok/s, lí giải do thiếu băng thông bộ nhớ)

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

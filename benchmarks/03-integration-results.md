# 03 - Integrate: RAG pipeline run

Host `Linux-x86_64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.1 | 6836.9 | 6837.0 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 2774.9 | 2775.1 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 2866.0 | 2866.1 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **4159.3** · total **4159.4**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real

- **N16 Cloud/IaC:** stub. Pipeline này chạy local, không deploy thật lên cloud/IaC.
- **N17 Data pipeline:** stub. Dữ liệu/context được chuẩn bị sẵn trong script, không có data pipeline sản xuất thật.
- **N18 Lakehouse:** stub. Không đọc từ lakehouse thật; retrieval dùng tập context nhỏ có sẵn.
- **N19 Vector + features:** stub. Backend retrieval là **keyword overlap**, không dùng vector database hay feature store thật.
- **N20 Serving:** real. Phần LLM gọi `llama-server` thật qua endpoint local.

Dominant stage là llm, chiếm gần như 100% tổng latency. Điều này đúng với kỳ vọng vì embed là 0.0 ms, retrieve chỉ khoảng 0.1 ms, còn LLM trung bình mất 4159.3 ms. Pipeline hiện tại không bị nghẽn ở retrieval, gần như toàn bộ thời gian nằm ở bước sinh câu trả lời.

Nếu phải giảm latency pipeline này 2x, tôi sẽ tấn công vào stage llm trước. Đây là nơi có nhiều thời gian nhất để tiết kiệm: giảm max_tokens, dùng thread count tốt nhất (LAB_N_THREADS=8), thử GPU offload/CUDA build, hoặc dùng model/quantization phù hợp hơn. Tối ưu embed hoặc retrieve sẽ gần như không tạo khác biệt vì hai stage đó chỉ chiếm khoảng 0.1 ms trên tổng hơn 4 giây.

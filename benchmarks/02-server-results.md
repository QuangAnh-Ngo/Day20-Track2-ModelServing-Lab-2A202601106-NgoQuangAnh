# 02 - Serve: load test + saturation reading

Host `Linux-x86_64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 18 | 0.35 | 23000 | 29000 | 29000 | 6.9 | 0.0% |
| 50 | 28 | 0.55 | 25000 | 44000 | 50000 | 14.2 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.58x** (32% of linear) |
| P95 latency | **1.52x** |
| Effective concurrency at 50 users | 14.2 vs `--parallel 4` slots (occupancy/slot ratio 3.55) |

**Saturated.** Throughput delivered only 1.58x for 5x the offered load, and effective concurrency (14.2) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

P95 grew no faster than throughput (1.52x vs 1.58x), so this server still has headroom at 50 users.

> **Small sample.** Only 18 requests completed in the
> shorter run, so these percentiles are indicative rather than solid. Note also that
> locust averages only *completed* requests: when the run ends with requests still
> queued, effective concurrency is an **under**-estimate. Trust the throughput-scaling
> row over the concurrency row here, and run longer (`-t 3m`) if you want firmer numbers.

## Your reading
What was the peak batch width, and does it match the effective concurrency in 02-server-results.md? If the two disagree, which do you trust and why?

Server bắt đầu saturate ở giữa 10 - 50 users. Bằng chứng là load tăng 5 lần nhưng throughput của nó chỉ tăng từ 0,35 RpS tới 0,55 RPS (1,58x). Tức là đa số người dùng phải chờ trong hàng đợi

Để có đầu vào tốt với latency SLO, tune --parallel và ước tính lại thay vì chạy quantization bởi vì 2 bit cải thiện quá ít. Ngoài ra, vấn đề là bottleneck là memory bandwidth như đã nói trước đây chứ không phải slot count.
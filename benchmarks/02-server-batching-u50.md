# 02 - Continuous batching under load (u50)

Host `Linux-x86_64` · `--parallel 4` · 28 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.89 of 4 slots (97%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 2725 |

Highest sampled value was **3.89 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation 

Peak batch width quan sát được là **3.89/4 slots**, tức khoảng **97%** số decode
slots đã được dùng. Điều này cho thấy continuous batching thật sự đang gom nhiều
request vào cùng các decode step, thay vì phục vụ từng request một.

Con số này không khớp trực tiếp với effective concurrency **14.2** trong
`02-server-results.md`, nhưng hai metric đo hai thứ khác nhau. `n_busy_slots_per_decode`
là số slot thật sự bận trong decode, nên bị chặn bởi `--parallel 4`. Effective
concurrency = RPS x average latency, nên nó tính cả các request đang chờ trong
queue; vì vậy nó có thể lớn hơn 4. Ở đây tôi tin gauge của llama.cpp hơn khi nói
về batch width/slot utilization, còn effective concurrency hữu ích hơn để chứng
minh có queueing. `requests_deferred` lên tới **46** xác nhận rằng server đã đầy
slot và request mới phải chờ, làm P95 tăng.

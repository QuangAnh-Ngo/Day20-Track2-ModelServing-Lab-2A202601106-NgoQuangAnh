# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Linux-x86_64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=0` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 73020 | 377 / 434 | 63.9 / 65.8 | 4346 / 4559 / 4559 | 15.7 |
| UD-Q2_K_XL | 2.24 | 56122 | 625 / 740 | 61.4 / 68.7 | 4444 / 4810 / 4810 | 16.3 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.04x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation (required -- replace this line)

_Is the smaller quantization worth it on your machine? Compare the numbers above,
then judge the answer quality yourself: run `make serve` on each and ask the same
question twice. Size and speed are measurable; usefulness is your call._

UD-Q2_K_XL nhỏ hơn UD-Q4_K_XL 0.73 GB và load nhanh hơn khoảng 16.9 giây. Về tốc độ decode, 2-bit đạt 16.3 tok/s so với 15.7 tok/s của 4-bit, tức chỉ nhanh hơn khoảng 1.04x. TPOT P50 cũng tốt hơn nhẹ, giảm từ 63.9 ms xuống 61.4 ms.

Tuy nhiên, TTFT của 2-bit kém hơn rõ rệt: P50 tăng từ 377 ms lên 625 ms, P95 tăng từ 434 ms lên 740 ms. E2E latency cũng cao hơn ở cả P50, P95 và P99. Điều này cho thấy 2-bit chỉ cải thiện nhẹ tốc độ sinh token, nhưng làm thời gian chờ token đầu tiên và tổng latency tệ hơn.

Vì vậy, trên máy này, 2-bit chỉ đáng dùng nếu ưu tiên tiết kiệm RAM/disk hoặc cần model load nhanh hơn. Nếu ưu tiên latency ổn định và trải nghiệm request tốt hơn, UD-Q4_K_XL là lựa chọn hợp lý hơn.

Câu trả lời bản 4 bit: hôm nay là ngày bao nhiêu, bao nhiêu ngày nữa thì tới tết

175 tokens
2.3s
75.05 tokens/s




Current time is 
2026-08-20T17:38:43+07:00
Tôi là một mô hình ngôn ngữ lớn và không có thông tin về ngày Tết cụ thể của bạn. Để biết hôm nay là ngày bao nhiêu và bao nhiêu ngày nữa thì tới Tết, bạn vui lòng cho tôi biết bạn đang ở quốc gia nào, vì ngày Tết sẽ khác nhau ở mỗi nơi.


gemma-4
E2B
UD-Q4_K_XL
it

Câu trả lời của bản 2 bit: hôm nay là ngày bao nhiêu, bao nhiêu ngày nữa thì tới tết

257 tokens
6.6s
39.09 tokens/s




Tôi đã cung cấp thời điểm hiện tại là ngày 20 tháng 8 năm 2026 theo múi giờ Asia/Saigon (UTC+7).

Tuy nhiên, để biết chính xác ngày Tết (Lễ Tết) và số ngày còn lại, bạn cần cung cấp thêm thông tin về:

Quốc gia/Vùng miền bạn đang đề cập đến (ví dụ: Tết Âm lịch ở Việt Nam, Tết Dương lịch, hay một ngày lễ khác).
Bạn đang muốn hỏi về Tết Âm lịch hay Tết Dương lịch?
Nếu bạn cho tôi biết bạn đang quan tâm đến Tết của quốc gia nào (ví dụ: Tết Nguyên Đán ở Việt Nam), tôi có thể giúp bạn tính toán chính xác hơn.


gemma-4
E2B
UD-Q2_K_XL
it


Kết luận: tôi đánh giá cao bản 4 bit hơn và sẽ chấp nhận đánh đổi tốc độ để lấy phản hồi tốt



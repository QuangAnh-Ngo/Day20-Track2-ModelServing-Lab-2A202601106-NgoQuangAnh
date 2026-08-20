# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Linux-x86_64` · llama.cpp `b10488`
CPU: **8 physical · 16 logical** cores · `ngl=0` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 4.2 | 25% |
| 4 | 12.8 | 76% |
| 8 | 16.8 | 100% |
| 16 | 4.5 | 27% |
| 32 | 1.3 | 8% |

**Best**: `-t 8` at 16.8 tok/s
**Slowest tested**: `-t 32` at 1.3 tok/s (12.61x spread)
**Against the physical-core default** (`-t 8`, 16.8 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=8 make bench
```

## Your explanation (required -- replace this line)

_Where is the knee, and why there? If the peak sits at your physical core count
and drops above it, say what the extra threads are competing for. If your curve
does something else -- flat, or still climbing at 2x logical cores -- say that
instead and reason about why. A result that contradicts the expected shape is
worth more than one that matches it, as long as you explain it._

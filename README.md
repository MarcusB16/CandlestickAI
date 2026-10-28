# CandlestickAI

An AI stock analyst that spots candlestick patterns, weighs competing interpretations using earnings and news, and gives each prediction a confidence score that is corrected over time by its own graded track record.

Personal tool and portfolio project. Not investment advice.

## How an analysis works

1. **Pick a stock.** Upload a chart screenshot (the AI reads the ticker and timeframe, you confirm) or type the ticker.
2. **Fetch real data.** About 6 months of daily candles plus a 2-year summary, the last 4 quarters of earnings with the next earnings date, and the last 14 days of news.
3. **Code detects, the AI interprets.**
   - Code detects candlestick patterns and computes context: trend, support and resistance, volume against average, ATR, earnings proximity.
   - The LLM receives that context, the earnings and news, and its own track-record table. It weighs competing interpretations and returns gradeable hypotheses.
4. **Learning mode** (optional) explains every term in plain English.

### Hypothesis shape

Every interpretation the analyst puts forward is falsifiable:

| Field | Example |
|---|---|
| Pattern | Hammer |
| Direction | Up |
| Trigger | Close above 184.10 |
| Invalidation | Close below 179.60 |
| Horizon | 5 days |
| Confidence | 62% |

### Pattern catalog

Hammer / hanging man, inverted hammer / shooting star, bullish / bearish engulfing, morning / evening star, doji (dragonfly, gravestone), piercing line / dark cloud cover, three white soldiers / three black crows, and **no signal** as a valid answer.

## How it gets smarter

- **Deterministic grader, no AI.** A hypothesis counts only if it triggers within 2 days. It wins at a 1 ATR move in the predicted direction, loses at invalidation, and expires after 5 days. Hypotheses that never trigger are excluded from hit rates. All thresholds are configurable.
- **Track record as context.** Hit rates per pattern, per prompt version and per model are fed back to the analyst as a stats table.
- **Replay testing.** Historical windows are disguised (ticker and dates hidden, prices rescaled) so the model can't recognize charts it saw in training. Replay and live results are scored separately, and a gap between them shows that leakage.
- **Calibration chart.** When the analyst says 70%, is it right about 70% of the time?

## Scope

- US stocks, daily candles. Timeframe is a parameter throughout so intraday can be added later.
- Replay universe: a fixed list of about 50 liquid large caps. These are all companies that exist today, so results are somewhat optimistic (survivorship bias).

## Architecture

- **Backend (this repo):** ASP.NET Core API, layered as `Application/`, `Domain/`, `Infrastructure/`. PostgreSQL in Docker via EF Core repositories and a unit of work.
- **Frontend (separate repo):** React + TypeScript (Vite) with TradingView Lightweight Charts.
- **LLM:** behind an interface. The model is named only in configuration or environment variables.
  - Development and replay: a local model (`qwen3:8b`) served by Ollama on a home server over its OpenAI-compatible API.
  - Evaluation: a strong cloud model run on the same replay set, for a local vs. cloud calibration comparison.
  - Every call logs its duration.
- **Market data:** daily candles, earnings and news providers behind client interfaces, cached in the database.

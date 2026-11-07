# STATE.md - geraked/metatrader5 safety review

Fork of https://github.com/geraked/metatrader5 (MIT). Review branch: `review-safe`. `master` is untouched.

Rules:
- Ek EA ek baar mein review hota hai, source `.mq5` padh kar.
- `Build/*.ex5` binaries install NAHI karne. Sirf source se MetaEditor mein compile karna.
- Sab EA pehle DEMO account par test hoga.
- "SAFE" ka matlab: meri manual code-reading mein koi hidden network/DLL/file-write nahi mila. NVIDIA SkillSpector score nahi hai (tool workspace mein available nahi tha).

## Shared code check (Include/EAUtils.mqh)
- Sirf `errordescription.mqh` include karta hai. `#import`, `WebRequest`, DLL: nahi.
- Orders sirf `OrderSend` / `OrderSendAsync` se jaate hain.
- News filter (default OFF) sirf MT5 ka built-in Calendar + local SQLite DB (common folder) padhta/likhta hai.

## EA status

| EA | Includes | Grid default | IgnoreSL default | Status |
|---|---|---|---|---|
| DHLAOS | EAUtils | ON | true | SAFE code (risky defaults, set Grid=false, IgnoreSL=false) |
| 2MAAOS | EAUtils | ON | false | PENDING |
| 2MACDSTO | EAUtils | ON | true | PENDING |
| 2MOMEMA | EAUtils | off | false | PENDING |
| 3MACD | EAUtils | ON | true | PENDING |
| 3MAF | EAUtils | ON | true | PENDING |
| AFAOSMD | EAUtils | ON | false | PENDING |
| BBRSI | EAUtils | ON | true | PENDING |
| CEZLSMA | EAUtils | ON | true | PENDING |
| COT1 | EAUtils, Cot.mqh, Sql.mqh | ON | false | PENDING (Cot.mqh file read/write, extra check needed) |
| LRCMACD | EAUtils | ON | false | PENDING |
| LRCUTB | EAUtils | ON | false | PENDING |
| NWERSIASF | EAUtils | ON | false | PENDING |

Templates: EATemplate, EATemplate2 (strategy nahi, review skip).

## DHLAOS review (v1.5)
Logic:
- Har nayi candle par check (M5, author test AUDUSD).
- Andean Oscillator (period 50, signal 9) + Daily High/Low.
- BUY: bull line signal ko upar cross kare, peeche (max 300 candles) ulta cross na mile, aur cross ke paas (50 candles) High < daily low ke saath bear > bull mila ho.
- SELL: ulta (daily high ke upar, bull > bear).
- SL: swing (7 candles) + 60 points. TP: 1.5 x SL distance. Risk 0.5%.

Risk flags:
- Grid ON, volume x1.1, max 50 levels (martingale jaisa).
- IgnoreSL/IgnoreTP true: SL/TP broker ko nahi jaate, EA ke andar chalte hain.
- AndeanOscillator.ex5 aur DailyHighLow.ex5 `#resource` se embed hote hain. Source Indicators/ mein hai, khud compile karna.

Verify checklist (demo):
1. Inputs: Grid=false, IgnoreSL=false, IgnoreTP=false.
2. Risk <= 0.5, MarginLimit 300, SpreadLimit set karna (gold ke liye).
3. Pair/TF: pehle AUDUSD M5 par backtest, phir gold par alag optimize.
4. Experts tab mein koi unknown error/Print nahi aana chahiye.
5. Trade khulte hi SL/TP broker side dikhna chahiye.

## Build: COT1 v1.2 (demo-safe)
- File: Experts/COT1 v1.2.mq5 (original COT1.mq5 se sirf defaults badle: Risk 0.5, Grid=false, IgnoreTP=false).
- Compile order: SuperTrend.mq5 pehle, phir COT1 v1.2.mq5. Compile test MetaEditor me user ko karna hai.

## Pending
- Baaki 12 EA ka ek-ek review.
- Gold (XAUUSD) ke liye tuned version, jab user bole.

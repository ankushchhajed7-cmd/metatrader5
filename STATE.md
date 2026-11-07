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
| COT1 | EAUtils, Cot.mqh, Sql.mqh, Shell.mqh | ON | false | CAUTION-OK (code clean, needs DLL import + internet to CFTC; no Strategy Tester) |
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

## COT1 review (v1.2)
Logic:
- Weekly CFTC COT data (publicreporting.cftc.gov) -> local SQLite DB (MT5 common folder).
- 8 currency futures (AUD, EUR, GBP, NZD, CAD, CHF, JPY, USD). Primary class Dealer, secondary Leveraged Funds.
- Top 2 "green" (net long change > 0, highest long %) x top 2 "red" (net short change < 0, highest short %) -> pair trades. Buy if green+red symbol exists, else sell reverse pair.
- Trades only Monday/Tuesday after OpenTime 03:00 (server), one trade per symbol per week. SuperTrend filter optional (off).
- SL on D1 (SL_AR, lookback 6, +30 points), TP 2x SL, trailing 50%.

Safety:
- Network: only WinINet to publicreporting.cftc.gov (official CFTC). No other host.
- Shell.mqh is included (kernel32/user32 imports, PowerShell/CMD helpers) but Cot.mqh/COT1 never call PShell/CShell/Open. Still needs "Allow DLL imports".
- Writes only CSV + SQLite in common folder (Cot data).

Risk flags:
- Risk default 3.5% (high), Grid ON x1.2 up to 20 levels, IgnoreTP true.
- OnTesterInit returns INIT_FAILED: Strategy Tester me chalta hi nahi. Sirf demo forward test.
- Gold/XAUUSD ke liye nahi bana (sirf currency pairs).

Verify checklist (demo):
1. Risk <= 0.5, Grid=false, IgnoreTP=false.
2. Tools > Options > Expert Advisors: Allow DLL imports ON.
3. Experts tab: "Downloading ... file" aur "successfully downloaded" dikhna chahiye.
4. Mon/Tue 03:00 ke baad hi signal; ek hi position (MultipleOpenPos=false).

## Pending
- Baaki 12 EA ka ek-ek review.
- Gold (XAUUSD) ke liye tuned version, jab user bole.

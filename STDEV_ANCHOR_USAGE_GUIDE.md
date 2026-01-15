# STDEV Anchor Distribution Model - Usage Guide

## Overview

The **STDEV Anchor Distribution Indicator** is a sophisticated Pine Script v5 tool designed for trading NASDAQ futures (NQ1!/MNQ1!) that:

1. **Automatically detects manipulation/displacement legs** during Late Asia → Early London sessions
2. **Projects Standard Deviation levels** based on the anchor leg (±1, ±1.5, ±2, ±2.5, ±3, ±4)
3. **Generates high-probability reversal signals** at extremes during London Open / NY AM sessions
4. **Applies strict confluence filters** including FVG, liquidity zones, and MSS/BOS triggers
5. **Non-repainting** - all signals are confirmed and reliable

---

## Quick Start

### 1. Installation
1. Copy the entire code from `STDEV_Anchor_Indicator.pine`
2. Open TradingView and navigate to Pine Editor
3. Paste the code and click "Add to Chart"
4. Apply to NQ1! or MNQ1! chart

### 2. Recommended Settings
- **Chart Timeframe**: 1-minute, 5-minute, or 15-minute
- **Instrument**: NQ1! (E-mini NASDAQ) or MNQ1! (Micro E-mini NASDAQ)
- **Initial Mode**: Enable Debug Mode for first few days to understand anchor detection

### 3. Timezone Setup
- Default timezone: `America/New_York`
- Adjust if needed: Set your exchange timezone in Settings → Session Settings → Timezone

---

## Understanding the Indicator

### Core Concept: The Anchor Leg

The indicator identifies a **manipulation/displacement leg** that typically occurs between Late Asia and Early London sessions (02:00-07:00 NY time). This leg consists of:

1. **Liquidity Sweep**: Price sweeps above a recent swing high (buy-side sweep) or below a recent swing low (sell-side sweep)
2. **Strong Displacement**: Immediate reversal with strong momentum in the opposite direction
3. **Range Establishment**: The start-to-end range (R) becomes the basis for STDEV projections

**Example**:
```
Buy-Side Sweep (expecting bearish displacement):
  High sweeps above recent pivot → Closes back below → Strong bearish candles follow

Sell-Side Sweep (expecting bullish displacement):
  Low sweeps below recent pivot → Closes back above → Strong bullish candles follow
```

### STDEV Level Projection

Once the anchor is found, the indicator projects levels from the **end price** of the displacement:

```
For Bearish Displacement (down):
  Mean (0)  = End Price
  -1.0 R    = End Price - 1.0 × Range
  -1.5 R    = End Price - 1.5 × Range
  -2.0 R    = End Price - 2.0 × Range (EXTREME - signals fire here)
  -2.5 R    = End Price - 2.5 × Range
  -3.0 R    = End Price - 3.0 × Range
  -4.0 R    = End Price - 4.0 × Range

For Bullish Displacement (up):
  Mean (0)  = End Price
  +1.0 R    = End Price + 1.0 × Range
  +1.5 R    = End Price + 1.5 × Range
  +2.0 R    = End Price + 2.0 × Range (EXTREME - signals fire here)
  +2.5 R    = End Price + 2.5 × Range
  +3.0 R    = End Price + 3.0 × Range
  +4.0 R    = End Price + 4.0 × Range
```

### SBZ (Sweet Buy Zone)
- Highlighted zone between ±1.0 and ±1.5
- Represents the first retracement zone (optional entry area)

---

## Signal Logic

### LONG Signal Requirements
Price must meet ALL of these conditions:

1. **Extreme Reached**: Close ≤ Lower Extreme Threshold (default: -2.0 R)
2. **Signal Window**: Inside London Open (02:00-05:00) OR NY AM (09:30-11:30)
3. **Liquidity Confluence** (if enabled): Near Asia Low, London Low, or Prev Day Low
4. **FVG Confluence** (if enabled): Near a bullish Fair Value Gap
5. **MSS/BOS Trigger** (if enabled): Confirmed Market Structure Shift upward
6. **Not Throttled**: Max signals per day not exceeded, cooldown period passed

### SHORT Signal Requirements
Same as LONG but inverted:

1. **Extreme Reached**: Close ≥ Upper Extreme Threshold (default: +2.0 R)
2. **Signal Window**: Inside London Open OR NY AM
3. **Liquidity Confluence**: Near Asia High, London High, or Prev Day High
4. **FVG Confluence**: Near a bearish Fair Value Gap
5. **MSS/BOS Trigger**: Confirmed Market Structure Shift downward
6. **Not Throttled**: Within daily limits

---

## Parameter Configuration

### Session Settings

| Parameter | Default | Description |
|-----------|---------|-------------|
| Asia Start | 18:00 | Start of Asia session tracking |
| Asia End | 00:00 | End of Asia session |
| London Anchor Window | 02:00-07:00 | Window to detect anchor leg |
| London Open Signals | 02:00-05:00 | Window to generate London signals |
| NY AM Signals | 09:30-11:30 | Window to generate NY signals |
| Timezone | America/New_York | Reference timezone |

**Note**: All times are in the specified timezone (default: NY time).

### Anchor Detection Parameters

| Parameter | Default | Recommended Range | Description |
|-----------|---------|-------------------|-------------|
| Pivot Lookback (Left/Right) | 5 | 3-10 | Bars for pivot high/low confirmation |
| Sweep Tolerance | 2.0 ticks | 1-4 ticks | Distance beyond pivot to confirm sweep |
| Displacement Body (ATR) | 0.8 | 0.5-1.5 | Minimum candle body size for displacement |
| Displacement Min Move | 20 pts | 10-40 pts | Minimum net move for valid displacement |
| Max Displacement Bars | 15 | 5-25 | Maximum bars allowed for displacement leg |
| Anchor Selection | Earliest | Earliest/Strongest | How to choose if multiple anchors found |

**Tuning Tips**:
- **Tight markets**: Lower sweep tolerance (1-2 ticks), lower displacement threshold (15 pts)
- **Volatile markets**: Higher sweep tolerance (3-4 ticks), higher displacement threshold (25-30 pts)
- **5m/15m charts**: Increase displacement min move (30-50 pts)

### STDEV Projection Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Show ±1.0 Levels | ✓ | First retracement zone |
| Show ±1.5 Levels | ✓ | SBZ boundary |
| Show ±2.0 Levels | ✓ | **Primary extreme zone** |
| Show ±2.5 Levels | ✓ | Extended extreme |
| Show ±3.0 Levels | ✓ | Deep extreme (SL placement) |
| Show ±4.0 Levels | ✓ | Maximum projection |
| Mean Calculation | End Price | End Price / Midpoint |
| Show SBZ Zone | ✓ | Highlight 1.0-1.5 zone |

**Recommendation**: Keep all levels visible initially, then hide ±4.0 if chart gets cluttered.

### Signal Generation Settings

| Parameter | Default | Recommended | Description |
|-----------|---------|-------------|-------------|
| Lower Extreme Threshold | -2.0 R | -2.0 to -2.5 R | Minimum level for LONG signals |
| Upper Extreme Threshold | +2.0 R | +2.0 to +2.5 R | Minimum level for SHORT signals |
| Enable London Signals | ✓ | ✓ | Allow signals during London Open |
| Enable NY Signals | ✓ | ✓ | Allow signals during NY AM |
| Max Signals Per Day | 2 | 1-3 | Daily signal limit |
| Min Bars Between Signals | 20 | 15-30 | Cooldown period |

**Conservative Setup**: Use ±2.5 R thresholds and max 1-2 signals/day
**Aggressive Setup**: Use ±2.0 R thresholds and max 2-3 signals/day

### Confluence Filters

| Filter | Default | When to Use |
|--------|---------|-------------|
| Liquidity Confluence | ON | Always (highly recommended) |
| Liquidity Proximity | 10 pts | Keep at 5-15 pts |
| FVG Confluence | ON | For cleaner entries |
| FVG Proximity | 15 pts | Adjust based on volatility |
| MSS/BOS Trigger | ON | **Critical for non-repainting** |
| MSS Lookback | 10 | 5-15 bars |

**Best Practice**: Start with ALL filters enabled, then disable FVG if signals are too rare.

### Risk Management (Visual Only)

| Parameter | Default | Description |
|-----------|---------|-------------|
| Stop Loss Method | Beyond Extreme | Beyond Extreme / Recent Pivot |
| SL Points Beyond Extreme | 10 pts | Buffer beyond ±3.0 level |
| TP1 Target | Mean (0) | First profit target |
| TP2 Target | ±1.5 | Second profit target |
| Show SL/TP Lines | ✓ | Display on chart |

**Typical Risk Profile**:
- Entry at ±2.0: ~40 points from mean
- SL at ±3.0 + buffer: ~50-60 points
- TP1 at Mean: ~40 points (1:1 RR)
- TP2 at opposite ±1.5: ~60-80 points (1.5-2:1 RR)

---

## Visual Elements

### What You'll See on Chart

1. **Orange Line**: Anchor leg (displacement) - shows the manipulation/displacement move
2. **Orange Horizontal Line**: Mean level (0)
3. **Blue Dashed Lines**: ±1.0 and ±1.5 levels (SBZ boundaries)
4. **Purple Solid Lines**: ±2.0 and ±3.0 levels (EXTREME zones)
5. **Purple Dotted Lines**: ±2.5 and ±4.0 levels (extended projections)
6. **Yellow Shaded Zones**: SBZ (Sweet Buy Zones) between 1.0 and 1.5
7. **Green Triangle Up**: BUY signal
8. **Red Triangle Down**: SELL signal
9. **Labels**: Entry price, SL points, TP points

### Debug Mode Elements

When Debug Mode is enabled, you'll also see:

- **Sweep markers**: Small labels showing "Buy Sweep" or "Sell Sweep"
- **Displacement markers**: "↑ Disp" or "↓ Disp" labels
- **Anchor found label**: Large label with details (sweep type, displacement strength, range)
- **FVG boxes**: Shaded boxes showing fair value gaps
- **MSS markers**: "MSS↑" or "MSS↓" labels
- **Failure messages**: "✗ Disp too weak" if anchor criteria not met

---

## Usage Workflow

### Daily Routine

1. **Morning Review (before London Open)**
   - Check if anchor was detected overnight
   - Verify anchor leg quality (check displacement strength in debug label)
   - Note the STDEV levels (especially ±2.0 and ±3.0)
   - Identify today's Asia High/Low and London High/Low

2. **London Open Session (02:00-05:00 NY time)**
   - Watch for price to reach ±2.0 or beyond
   - Wait for signal triangle and label to appear
   - Check confluence: Is price near liquidity zone? Is FVG present? Has MSS triggered?
   - If signal fires, note entry, SL, and TP from label

3. **NY AM Session (09:30-11:30 NY time)**
   - Same process as London Open
   - Often provides second chance entries if London didn't trigger

4. **Trade Management**
   - TP1: Mean level or ±1.0 (partial exit)
   - TP2: Opposite ±1.5 or ±2.0 (remaining position)
   - SL: Beyond ±3.0 level (or recent swing as per settings)

### Example Trade Scenario

**Setup**: Bearish displacement anchor detected at 03:00 NY time
- Anchor Range (R): 50 points
- End Price: 16,000
- Projected levels:
  - Mean: 16,000
  - -1.0: 15,950
  - -2.0: 15,900 ← **LONG signal threshold**
  - -3.0: 15,850

**Execution**:
1. Price drops during NY AM session
2. Price reaches 15,895 (below -2.0 threshold)
3. Price is near London Low (liquidity confluence ✓)
4. Bullish FVG present at 15,890-15,900 (FVG confluence ✓)
5. MSS Up triggers (close breaks above recent swing high) ✓
6. **LONG signal fires**: BUY at 15,895

**Trade Plan**:
- Entry: 15,895
- SL: 15,840 (below -3.0 + 10pt buffer) = 55 points risk
- TP1: 16,000 (mean) = 105 points profit
- TP2: 16,050 (+1.5) = 155 points profit
- Risk:Reward = 1:2 to 1:3

---

## Troubleshooting

### No Anchor Detected

**Possible Causes**:
1. Market was range-bound during London window (no clear sweep)
2. Displacement wasn't strong enough (check displacement thresholds)
3. Session times incorrect for your chart timezone

**Solutions**:
- Enable Debug Mode to see why anchor failed
- Lower displacement thresholds (try 0.6 ATR body, 15pts min move)
- Verify session times match your instrument's exchange

### No Signals Appearing

**Possible Causes**:
1. Price hasn't reached extreme levels (±2.0 or beyond)
2. Confluence filters too strict (all must pass)
3. Already exceeded max signals per day
4. Outside signal windows (London Open / NY AM)

**Solutions**:
- Check if price actually reached ±2.0 levels
- Temporarily disable FVG confluence filter
- Increase max signals per day to 3-4
- Verify signal window times are correct

### Signals Repainting

**This should NOT happen** if indicator is working correctly. All signals use:
- `barstate.isconfirmed` (only on closed bars)
- Confirmed pivots with right-side lookback
- No forward-looking calculations

**If you suspect repainting**:
1. Verify you're on the latest version of code
2. Check that pivotRightBars ≥ 3
3. Wait for bar to fully close before trusting signal
4. Enable bar replay to test historical accuracy

### Too Many Signals

**Solutions**:
- Increase extreme threshold from ±2.0 to ±2.5 or ±3.0
- Enable all confluence filters (Liquidity + FVG + MSS)
- Reduce max signals per day to 1-2
- Increase min bars between signals to 30-40

### Too Few Signals

**Solutions**:
- Lower extreme threshold from ±2.5 to ±2.0
- Disable FVG confluence (keep Liquidity + MSS)
- Lower displacement requirements (0.6 ATR, 15pts)
- Enable fallback/aggressive mode (modify code)

---

## Best Practices

### Risk Management
1. **Never risk more than 1-2% per trade**
2. **Use proper position sizing**: NQ = $20/point, MNQ = $2/point
3. **Respect the SL**: Don't move stops against you
4. **Take partials at TP1**: Lock in profit at mean level

### Trade Discipline
1. **Only take signals from the indicator** - don't anticipate
2. **Wait for bar close confirmation** - no FOMO entries
3. **Check all confluence factors** before entering
4. **Skip trade if uncertain** - there will be another tomorrow

### Backtesting
1. **Use TradingView's bar replay** to walk through historical days
2. **Track win rate, avg RR, and max drawdown**
3. **Test different extreme thresholds** (±2.0 vs ±2.5)
4. **Optimize confluence filters** for your risk tolerance

### Continuous Improvement
1. **Keep a trade journal**: Note what worked and what didn't
2. **Review missed opportunities**: Why didn't you take valid signals?
3. **Analyze false signals**: What confluence was missing?
4. **Adjust parameters gradually**: Don't over-optimize

---

## Advanced Tips

### Multiple Timeframe Analysis
- **Primary chart**: 5-minute (for signals)
- **Context chart**: 15-minute (for trend bias)
- **Execution chart**: 1-minute (for precise entries)

### Combining with Other Tools
- **Volume Profile**: Confirm liquidity zones at extremes
- **Order Flow**: Watch for absorption at ±2.0 levels
- **News Calendar**: Avoid signals during major news events

### Seasonal Adjustments
- **Low volatility periods** (summer): Tighten thresholds (±2.0)
- **High volatility periods** (earnings season): Widen thresholds (±2.5-3.0)
- **Holiday weeks**: Reduce max signals or skip trading

---

## Self-Test Checklist

Before going live, verify these on historical data:

- [ ] Anchor leg appears during London window (02:00-07:00)
- [ ] Anchor line is drawn correctly (orange line)
- [ ] STDEV levels project in correct direction from anchor end
- [ ] ±2.0 levels are clearly visible (purple lines)
- [ ] BUY signals only appear at lower extremes (≤ -2.0)
- [ ] SELL signals only appear at upper extremes (≥ +2.0)
- [ ] Signals only appear during London Open or NY AM windows
- [ ] Max 2 signals per day (or your configured max)
- [ ] No signals repaint after bar closes
- [ ] SL/TP lines appear correctly on signals

---

## Parameter Templates

### Conservative (Higher Win Rate, Fewer Trades)
```
Extreme Threshold: ±2.5 R
Max Signals Per Day: 2
All Confluence Filters: ENABLED
Displacement Body ATR: 1.0
Displacement Min Move: 25 pts
```

### Balanced (Recommended Starting Point)
```
Extreme Threshold: ±2.0 R
Max Signals Per Day: 2
Liquidity + MSS: ENABLED
FVG: ENABLED
Displacement Body ATR: 0.8
Displacement Min Move: 20 pts
```

### Aggressive (More Trades, Lower Win Rate)
```
Extreme Threshold: ±2.0 R
Max Signals Per Day: 3-4
Liquidity + MSS: ENABLED
FVG: DISABLED
Displacement Body ATR: 0.6
Displacement Min Move: 15 pts
```

---

## FAQ

**Q: Can I use this on other instruments besides NQ?**
A: The indicator is optimized for NQ/MNQ but can work on other futures (ES, YM, RTY) with parameter adjustments. Adjust displacement thresholds and sweep tolerance based on instrument's average range.

**Q: What timeframe works best?**
A: 5-minute is the sweet spot. 1-minute gives more signals but requires more screen time. 15-minute is more stable but fewer opportunities.

**Q: How many signals should I expect per day?**
A: With default settings, expect 0-2 signals per day. Some days have zero (no clear anchor or no extreme reached). Quality over quantity.

**Q: Can I automate this with alerts?**
A: Yes! Set up TradingView alerts on "Long Signal" and "Short Signal" conditions. You'll get notified when signals fire.

**Q: What if the anchor changes during the day?**
A: The indicator locks in the first valid anchor per day (or strongest, based on settings). Once anchor is found, it won't change until next trading day.

**Q: Should I trade against the anchor direction?**
A: Yes! The strategy is **contrarian**. Bearish anchor → expect reversal long. Bullish anchor → expect reversal short. We're fading the extremes.

---

## Support and Updates

For issues, suggestions, or questions:
1. Check this guide thoroughly first
2. Enable Debug Mode to diagnose issues
3. Review the code comments for detailed logic
4. Test on historical data using bar replay

**Version**: 1.0
**Last Updated**: 2025-01-15
**Pine Script Version**: v5
**Indicator Type**: Non-repainting, Overlay

---

## Disclaimer

This indicator is for **educational and informational purposes only**. It does not constitute financial advice. Trading futures involves substantial risk of loss. Always:
- Use a demo account first
- Never risk more than you can afford to lose
- Understand the logic before trading real money
- Comply with all applicable regulations

Past performance does not guarantee future results.

---

**Happy Trading! 📈**

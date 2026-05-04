# IMC Prosperity 4 Algorithmic Write-Up

## 1. Competition Context

IMC Prosperity 4 was a fictional space-themed trading competition where we traded planetary products and tried to earn as many **XIRECs** as possible.

Even though the assets were made up, the trading problems were realistic. We had to deal with market making, bid-ask spreads, fair value, options pricing, implied volatility, delta hedging, mean reversion, trend following, bundle relationships, position limits, sizing, risk control, and overfitting.

For each round, IMC gave us **three days of previous market data** to research and test on. They also gave us a backtester with only a small slice of the next day, around **10% of Day 4**, so we could sanity-check the bot before submission.

For the actual scoring, the trading algorithm was tested on hidden full evaluation data for that round. That final hidden test decided the real performance metrics.

For the algorithmic challenge, we submitted a Python bot that read market data and placed buy/sell orders.

- **Round 1:** trading groundwork on Intara.
- **Round 2:** same products, plus the Market Access Fee auction.
- **Round 3:** Velvet Fruit Extract options book.
- **Round 4:** trade data from market makers, retail traders, and informed traders.
- **Round 5:** 50 different products across 10 bundles.

The main lesson was simple: a good bot needs fair value, entries, exits, sizing, and risk control.

---

## 2. Tools Used

To build and improve the strategies, I used a few main tools.

### Monte Carlo Simulator

I used a Monte Carlo simulator to test strategy risk across different possible outcomes.  
It was **not** a random-walk simulator.  
It used the actual round data, then simulated realistic variations from that data.  
This helped test what could happen if the same market behaved slightly differently.  
It was mainly useful for stress-testing volatility, bad luck, and option risk.

### Backtesting Engine

The backtesting engine tested strategies on the three previous days of data given each round.  
It showed total PnL, product-level PnL, trades, and positions.  
I used it to compare different versions quickly.  
It helped show when a change improved the strategy or made it worse.  
It also helped catch overfitting before final submission.

### MultiCharts Visualizer

I converted the three days of data given each round and loaded it into **MultiCharts**.  
This let me view the products as visual candlestick charts.  
It helped me see trends, reversals, noisy products, and clean market-making products.  
This was especially useful in Round 5 with 50 products.  
The charts helped classify products and check whether the bot traded correctly.

### AI

I used AI as a research, coding, and writing assistant.  
It helped brainstorm strategies, debug code, and compare versions.  
It was useful for explaining option pricing and organizing ideas.  
It also helped turn the strategy work into a clear write-up.  
The final trading decisions still came from testing, logic, and results.

---

## 3. My General Algorithmic Approach

Across the rounds, my process was:

1. Work out what type of product I was trading.
2. Estimate fair value.
3. Find when the market price was wrong.
4. Trade only when the edge was big enough.
5. Exit when the edge disappeared.
6. Increase size when the signal was stronger.
7. Avoid overfitting to the backtest.

At first, I cared mostly about whether a strategy made money. Later, I cared more about **why** it made money.

That was important because a backtest can lie. A strategy can look good on old data and still fail on new data. So I tried to move toward simple ideas that had a real reason behind them.

---

## Round 1: Trading Groundwork on Intara

### What the Round Was

Round 1 took place on **Intara**, where we established our first Trade Outpost.

The algorithmic challenge was to trade two Intarian goods:

- **Ash-Coated Osmium** (`ASH_COATED_OSMIUM`)
- **Intarian Pepper Root** (`INTARIAN_PEPPER_ROOT`)

Both products had a position limit of **80**.

This round was about building the first real trading bot and proving that we could turn simple products into profit.

### Assets Traded

| Asset | Position Limit | Strategy |
|---|---:|---|
| Ash-Coated Osmium | 80 | Anchored moving average |
| Intarian Pepper Root | 80 | Buy-and-hold / extreme trend following |

### Ash-Coated Osmium Strategy

For **Ash-Coated Osmium**, I did not use one fixed fair value.

Instead, I used an **anchored moving average**. This smoothed the average price over time and gave the bot a moving reference point instead of a static number.

The idea was that Ash-Coated Osmium was volatile, but it still had structure. The moving average helped show when price was breaking away from its normal path.

```text
calculate anchored moving average

if price breaks above the moving average:
    buy / go long

if price breaks below the moving average:
    sell / go short
```

So the strategy was not classic fixed-fair-value trading. It was closer to a moving-average breakout / mean-reversion hybrid, using the anchored average to smooth noise and guide direction.

### Intarian Pepper Root Strategy

For **Intarian Pepper Root**, I used more of a buy-and-hold / extreme trend-following strategy.

The product behaved almost like a manufactured or artificial instrument. Its price did not move naturally like a noisy normal market. It mostly kept going up in a very steady and constant way.

Because of that, the best idea was not to overtrade it. The bot mostly wanted to stay with the upward trend and hold the position.

```text
price keeps trending upward
stay long
avoid overtrading
only exit if something abnormal happens
```

So Intarian Pepper Root was basically treated as a strong artificial trend product.

### Flash-Dump Insurance Clause

The risk with trend following is that a strong trend can suddenly break.

So I added a flash-dump insurance clause.

If Intarian Pepper Root suddenly moved in a way that looked unprecedented or clearly broke the trend, the bot would exit quickly instead of holding through a crash.

```text
if trend is normal:
    keep holding

if price breaks trend sharply:
    exit fast
```

This let the bot ride the trend while still having emergency downside protection.

### Results

| Metric | Result |
|---|---:|
| Round 1 algorithmic PnL | [FILL IN] |
| Rank after Round 1 | [FILL IN] |
| Ash-Coated Osmium PnL | [FILL IN] |
| Intarian Pepper Root PnL | [FILL IN] |
| Final submitted version | [FILL IN] |

### What I Learned

- Different products need different strategies.
- Osmium worked better as mean reversion.
- Pepper Root worked better as trend following.
- Strong trends still need emergency exits.

### What I Could Have Done Better

- Tune the mean-reversion bands better.
- Test the flash-dump rule more.
- Track when Pepper Root trend following should exit.

---

## Round 2: Growing the Outpost and Market Access Fee

### What the Round Was

Round 2 was still on **Intara**.

We traded the same two products as Round 1:

- **Ash-Coated Osmium** (`ASH_COATED_OSMIUM`)
- **Intarian Pepper Root** (`INTARIAN_PEPPER_ROOT`)

Both still had a position limit of **80**.

The big new feature was the **Market Access Fee**, or MAF.

This let us bid for **25% more quotes in the order book**. If our bid was accepted, we got extra market access. If it was rejected, we traded the normal book and paid nothing.

### Assets Traded

| Asset | Position Limit | Strategy |
|---|---:|---|
| Ash-Coated Osmium | 80 | Anchored moving-average strategy |
| Intarian Pepper Root | 80 | Buy-and-hold / extreme trend following |

### Strategy Refinement

The product strategies stayed similar to Round 1.

For **Ash-Coated Osmium**, I continued using the anchored moving-average idea. The bot smoothed prices, watched for breaks above or below the average, and traded those moves.

For **Intarian Pepper Root**, I kept the buy-and-hold / extreme trend-following idea. Since the product kept moving upward in a very artificial and steady way, the bot stayed with the trend unless the flash-dump rule triggered.

Round 2 was less about inventing a brand-new strategy and more about improving the Round 1 bot while deciding how much extra market access was worth.

### Market Access Fee Logic

The Market Access Fee was a blind auction.

The top 50% of bids got extra market access. If accepted, the bid was subtracted from final Round 2 profit.

```text
if bid accepted:
    final profit = trading profit - bid

if bid rejected:
    final profit = trading profit
```

So the bid had to be high enough to get accepted, but not so high that it destroyed the extra profit.

### How I Chose the Bid

In Round 1, the algorithm made around **98,000 XIRECs** in base PnL.

Extra market access gave around **25% more volume**, so I estimated the extra value as:

```text
25% of 98,000 ≈ 24,500 XIRECs
```

That meant the extra access could be worth around **24,500 XIRECs**.

But the important detail was that the fee was paid every **25,000 ticks**.

A full trading day had **100,000 ticks**, so the fee was paid **4 times**:

```text
100,000 ticks / 25,000 ticks = 4 payments
```

So if I bid around **6,000 XIRECs**, the full-day cost would be:

```text
6,000 × 4 = 24,000 XIRECs
```

That was very close to the expected extra profit from the 25% extra volume:

```text
expected extra profit ≈ 24,500 XIRECs
estimated access cost ≈ 24,000 XIRECs
```

So around **6,000 XIRECs per 25,000 ticks** was my break-even point.

I then placed the bid lower, around **4,000 XIRECs**, because I did not want to pay the full break-even value. I also compared the average PnL on the leaderboard to estimate what other teams might bid and what their break-even points might be.

```text
extra access could be worth about 24.5K total
6K per 25K ticks costs about 24K total
therefore 6K was near break-even
bid around 4K to keep more upside
```

### Results

| Metric | Result |
|---|---:|
| Round 2 algorithmic PnL before MAF | [FILL IN] |
| Market Access Fee bid | ~4,000 XIRECs per 25,000 ticks |
| Estimated raw value of extra access | ~24,500 XIRECs total |
| Estimated break-even bid | ~6,000 XIRECs per 25,000 ticks |
| Estimated full-day break-even cost | ~24,000 XIRECs total |
| Final Round 2 algorithmic PnL | [FILL IN] |
| Rank after Round 2 | [FILL IN] |
| Ash-Coated Osmium PnL | [FILL IN] |
| Intarian Pepper Root PnL | [FILL IN] |

### What I Learned

- Extra market access has value, but overpaying kills the edge.
- The bid was a game-theory problem, not just a trading problem.
- The best bid is not the highest bid. It is the cheapest bid that still gets accepted.
- Leaderboard PnL helped estimate what other teams might bid.

### What I Could Have Done Better

- Model the bidding game more formally.
- Test how much the extra 25% volume actually improved fills.
- Estimate other teams’ break-even points more carefully.

---

## Round 3: Velvet Fruit Extract Options and Hydrogel Packs

### What the Round Was

Round 3 had two underlying assets:

- **Velvet Fruit Extract**
- **Hydrogel Packs**

There was also a full options book on **Velvet Fruit Extract**. All options had the same expiry, but different strikes:

- 4,000
- 4,500
- 5,000
- 5,100
- 5,200
- 5,300
- 5,400
- 5,500
- 6,000
- 6,500

So Round 3 had two parts:

1. Market making on Velvet Fruit Extract and Hydrogel Packs.
2. IV trading on the Velvet Fruit Extract options.

### Assets Traded

| Asset | What It Was | How I Used It |
|---|---|---|
| Velvet Fruit Extract | Underlying asset | Market making / bid-ask spread capture |
| Hydrogel Packs | Underlying asset | Market making / bid-ask spread capture |
| VFE options chain | Options on Velvet Fruit Extract | IV comparison and fair-value trading |

### Underlying Strategy: Market Making

For **Velvet Fruit Extract** and **Hydrogel Packs**, I used a market-making strategy.

```text
buy near the bid
sell near the ask
capture the spread
manage inventory
```

I was not trying to predict massive price moves in these two products. I was trying to repeatedly earn small profits from the **bid-ask spread**.

This ended up being very important. My options model did not trade well in Round 3. In fact, the options side was negative. The market making on Velvet Fruit Extract and Hydrogel Packs is what pushed me back into profit overall.

### Options Strategy, Delta Hedging, and the Critical Error

For the Velvet Fruit Extract options, I tried to use IV scalping.

The goal was not just to bet on the option price going up or down. The goal was to trade **implied volatility**.

To do that properly, we had to **delta hedge** the options chain using Velvet Fruit Extract. Delta hedging helped remove the directional risk from the underlying. That way, the strategy was more focused on whether the option’s implied volatility was too high or too low, instead of just whether Velvet Fruit Extract moved up or down.

```text
option market price → implied volatility → compare to fair IV → trade rich/cheap options → delta hedge with Velvet Fruit Extract
```

The mistake was the fair-value equation.

I used a **modified Black-Scholes implied-volatility equation**, but it was not good enough for this market. It looked reasonable in theory, but it did not trade well.

The problem was that the Velvet Fruit Extract options did not behave like clean textbook Black-Scholes options. The equation was too simple. It did not capture the actual movement and uncertainty in the underlying well enough.

So even though the delta hedge helped isolate implied volatility, the IV model itself was still weak. The options side gave bad signals and ended up negative.

That meant Round 3 only ended around **+1,000 XIRECs** because the market making on Velvet Fruit Extract and Hydrogel Packs recovered the losses.

### Why 6,000 and 6,500 Mattered

The **6,000** and **6,500** options looked rich compared with the rest of the chain.

```text
short the rich options
hold while they stay rich
reduce when the edge disappears
```

But because the fair-value equation was weak, I could not fully trust the signal. The trade idea was sensible, but the model behind it was not strong enough yet.

### Results

| Metric | Result |
|---|---:|
| Round 3 algorithmic PnL | ~1,000 XIRECs |
| Rank after Round 3 | [FILL IN] |
| Final submitted version | [FILL IN] |
| Velvet Fruit Extract market-making PnL | [FILL IN] |
| Hydrogel Packs market-making PnL | [FILL IN] |
| Velvet Fruit Extract options PnL | Negative |
| Best product | [FILL IN] |
| Worst product | [FILL IN] |

### What I Learned

- Delta hedging was needed to trade IV instead of just option direction.
- My IV idea was right, but my equation was weak.
- The options model lost money.
- Market making the underlyings saved the round.
- Spread capture can be very valuable.

### What I Could Have Done Better

- Add variance earlier.
- Track options PnL separately from market-making PnL.
- Trust the Black-Scholes-style model less.
- Improve the fair-value equation before increasing size.

---

## Round 4: Same Products, Better Equation, Extra Trade Data

### What the Round Was

Round 4 used the **same products as Round 3**:

- **Velvet Fruit Extract**
- **Hydrogel Packs**
- the same Velvet Fruit Extract options chain

The options still had the same strikes:

- 4,000
- 4,500
- 5,000
- 5,100
- 5,200
- 5,300
- 5,400
- 5,500
- 6,000
- 6,500

The difference was that Round 4 gave us **trade data**.

That meant I could now look at who was trading, not just the order book.

Round 4 had three main improvements:

1. Keep market making Velvet Fruit Extract and Hydrogel Packs.
2. Improve the options fair-value equation by adding variance.
3. Use trade data to identify trader types.

### Assets Traded

| Asset | What It Was | How I Used It |
|---|---|---|
| Velvet Fruit Extract | Underlying asset | Market making / spread capture |
| Hydrogel Packs | Underlying asset | Market making / spread capture |
| VFE options chain | Options on Velvet Fruit Extract | IV fair value with variance |
| Trade data | Extra information | Trader classification and flow signals |

### Underlying Strategy: Market Making

For **Velvet Fruit Extract** and **Hydrogel Packs**, I kept the market-making strategy.

The bot quoted around fair value, bought when prices were attractive, sold when prices were attractive, and tried to capture the bid-ask spread.

This gave the strategy a stable base. Even if the options model had noise, the underlying market making helped smooth the PnL.

### Options Strategy: Why Variance Fixed the Model

This was the big improvement from Round 3.

In Round 3, I used a modified Black-Scholes-style IV equation. It sounded good, but it did not fit the market well. It treated the options too cleanly and ignored too much of the actual price movement.

In Round 4, I added **variance** into the fair-value calculation.

That was the real fix.

Options are all about movement. If the underlying moves around a lot, the option should be worth more. If the model does not account for that properly, it will misprice the whole chain.

We were still **delta hedging** the options with Velvet Fruit Extract. That was important because the goal was to trade implied volatility, not just take a directional bet on the underlying or on raw option prices.

Delta hedging helped isolate the IV edge. Then the variance-adjusted equation made that IV edge much more accurate.

Adding variance made the model much more realistic. It stopped treating the market like a perfect textbook example and started reacting to how Velvet Fruit Extract actually moved.

This made a huge difference. Round 3 made about **1,000 XIRECs**, and the options side was negative. Round 4 made about **10,000 XIRECs** because the model finally understood the options better.

The model changed from:

```text
modified Black-Scholes fair value
```

to:

```text
modified Black-Scholes fair value + variance adjustment
```

That gave a much better IV edge.

### Rich-IV Strategy

The rich-IV logic stayed similar:

```text
IV edge = market IV - improved fair IV
```

If the option was rich after the variance adjustment, the bot could short it.

Then the bot used Velvet Fruit Extract to delta hedge the option exposure. This mattered because I wanted the trade to profit from the option being mispriced in volatility terms, not just from the underlying moving in my favour.

If the edge closed, the bot reduced or exited.

The 6,000 and 6,500 options were still important, but the signal was now much more trustworthy.

### Trader Classification

The other new tool was trade data.

With trade data, I could classify traders by their habits.

#### Market makers

Market makers usually traded a lot and traded both sides.

Their habits:

- high trade count;
- buys and sells both sides;
- usually close to fair value;
- often managing inventory.

I did not want to blindly follow them. Their trades were not always directional signals.

#### Noise / retail traders

Noise or retail traders had weaker trade behaviour.

Their habits:

- worse timing;
- chasing prices;
- buying rich options;
- selling cheap options;
- less consistent profitability.

This flow could be useful to fade.

#### Informed traders

Informed traders were the most useful group.

Their habits:

- better timing;
- trades before price moves;
- more consistent direction;
- better predictive value.

If informed traders agreed with my model, I could trust the trade more. If they traded against my model, I had to be careful.

```text
IV edge tells me what is mispriced
trade data tells me whether the flow agrees
```

### Leverage and Size

Round 4 was also about taking more size when the evidence was stronger.

The best trades had:

- a strong IV edge;
- the variance-adjusted model agreeing;
- trade data not warning against the trade.

```text
stronger evidence = bigger position
weaker evidence = smaller position
```

This was much better than just using max size everywhere.

### Results

| Metric | Result |
|---|---:|
| Round 4 algorithmic PnL | ~10,000 XIRECs |
| Rank after Round 4 | [FILL IN] |
| Final submitted version | [FILL IN] |
| Velvet Fruit Extract market-making PnL | [FILL IN] |
| Hydrogel Packs market-making PnL | [FILL IN] |
| Velvet Fruit Extract options PnL | [FILL IN] |
| Best option strike | [FILL IN] |
| Worst option strike | [FILL IN] |

### What I Learned

- Delta hedging helped isolate the IV trade.
- Variance made the options model much better.
- Trade data helped show who to follow or fade.
- Market making stayed a useful base.
- Better model plus better sizing gave much better PnL.

### What I Could Have Done Better

- Use variance from the start.
- Build trader labels earlier.
- Track PnL by trader signal.
- Stress-test the larger positions.

---

## Round 5: Multi-Product Algorithm

### What the Round Was

Round 5 was the biggest algorithmic round.

There were **50 different products** to trade.

They were split into **10 bundles**, with **5 products in each bundle**.

The 10 bundles were:

1. Microchip
2. Sleep Pod
3. Panel
4. Pebbles
5. UV Visor
6. Translator
7. Robots
8. Snack Pack
9. Galaxy Sands
10. Oxygen Shake

This was no longer just one strategy. It was a portfolio problem.

Every product needed its own strategy choice. Some products were better for market making. Some were better for mean reversion. Some were better for trend following. Some were too noisy and needed to be traded lightly or ignored.

The bundles helped because products inside the same bundle often had related behaviour. By comparing the five products inside a bundle, I could get clues about which strategy made sense.

```text
if products in a bundle move together → possible basket/spread idea
if one product trends harder than the others → trend-following idea
if prices keep snapping back → mean-reversion idea
if spreads are stable → market-making idea
```

The goal was:

```text
classify all 50 products
choose the right strategy type for each product
compare products inside each bundle
trade stronger signals bigger
avoid weak/noisy products
manage total risk
```

### Full Product List

| Bundle | Products |
|---|---|
| Microchip | Circle, Oval, Rectangle, Square, Triangle |
| Sleep Pod | Cotton, Wool, Nylon, Polyester, Suede |
| Panel | 1x2, 1x4, 2x2, 2x4, 4x4 |
| Pebbles | Extra Small, Small, Medium, Large, Extra Large |
| UV Visor | Amber, Magenta, Orange, Red, Yellow |
| Translator | Astro Black, Eclipse Charcoal, Graphite Mist, Space Gray, Void Blue |
| Robots | Dishes, Ironing, Laundry, Mopping, Vacuuming |
| Snack Pack | Chocolate, Vanilla, Pistachio, Raspberry, Strawberry |
| Galaxy Sands | Black Holes, Dark Matter, Planetary Rings, Solar Flares, Solar Winds |
| Oxygen Shake | Chocolate, Evening Breath, Garlic, Mint, Morning Breath |

### Candlestick Chart Appendix

I created a PDF with candlestick charts for all 50 Round 5 products.

The charts show the products across **Day 2, Day 3, and Day 4**, with each candle built from **100 timestamps**.

This PDF should be attached as the chart appendix:

```text
round5_all_products_candlestick_charts (3).pdf
```

### Main Strategy Idea

Because there were 50 products, I could not use one strategy for everything.

The first job was to identify what each product was doing.

For each product, I had to ask:

```text
Is this a market-making product?
Is this a mean-reversion product?
Is this a trend-following product?
Is this a spread/bundle product?
Is this too noisy to trade heavily?
```

The bundle structure helped with this. Since each bundle had five related products, I could compare them against each other. If one product was moving differently from the rest of its bundle, that could be a signal. If all five moved together, that could suggest a bundle-level strategy.

So the strategy was not just “trade 50 products.” It was:

```text
study each bundle
compare the 5 products inside it
pick the best strategy for each product
size based on signal strength and risk
```

The final bot traded the stronger, cleaner signals and avoided weaker ones.

### Snack Pack

Snack Pack used a market-making / score-based strategy.

The bot calculated a signal and used that signal to decide size.

```python
if score_abs >= 3:
    size = 10
elif score_abs >= 2:
    size = 7
elif score_abs >= 1:
    size = 5
else:
    size = 3
```

Simple idea:

```text
stronger signal = bigger trade
weaker signal = smaller trade
```

### Oxygen Shake

Oxygen Shake followed a similar process to Snack Pack.

I tested it by itself first, then increased size when the signal looked good.

The aim was not random size. It was controlled aggression.

### Galaxy Sands

Galaxy Sands products were:

- Black Holes
- Dark Matter
- Planetary Rings
- Solar Flares
- Solar Winds

The strategy used the **v3 ring7 even-size** idea.

The goal was to trade the relationship across the Galaxy products while keeping exposure balanced.

### Panel

Panel products were:

- Panel 1x2
- Panel 1x4
- Panel 2x2
- Panel 2x4
- Panel 4x4

Panel 2x2 looked especially trend-following.

The final Panel idea used trend strength:

```text
normal trend  → target ±4
strong trend  → target ±7
extreme trend → target ±10
```

### Pebbles and Robots

Pebbles products were:

- Extra Small
- Small
- Medium
- Large
- Extra Large

Robot products were:

- Dishes
- Ironing
- Laundry
- Mopping
- Vacuuming

Pebbles v6 and Robot v9 were added as separate modules.

The main question was whether they improved the full bot, not just whether they looked good alone.

### Other Bundles

The other bundles were:

- Microchip
- Sleep Pod
- UV Visor
- Translator

These were part of the 50-product universe and were useful for checking trend, volatility, and bundle behaviour.

The candlestick charts helped compare them quickly.

### Dynamic Risk and Sizing

Round 5 made sizing very important.

The bot should not use the same size everywhere. Size should depend on:

- signal strength;
- volatility;
- spread;
- liquidity;
- current position;
- product risk.

```text
strong signal + low risk = bigger size
weak signal + high risk = smaller size or no trade
```

### Final Combined System

The final Round 5 bot combined:

- SnackPack v5;
- Oxygen v7;
- Galaxy v3 ring7 even size;
- Panel v3;
- Pebbles v6;
- Robot v9;
- market making;
- trend following;
- score-based sizing;
- dynamic risk sizing.

The goal was to trade many products at once without putting too much risk into one bundle.

### Results

| Metric | Result |
|---|---:|
| Round 5 algorithmic PnL | [FILL IN] |
| Final algorithmic rank | [FILL IN] |
| Final submitted script | [FILL IN] |
| Snack Pack PnL | [FILL IN] |
| Oxygen Shake PnL | [FILL IN] |
| Galaxy Sands PnL | [FILL IN] |
| Panel PnL | [FILL IN] |
| Pebbles PnL | [FILL IN] |
| Robots PnL | [FILL IN] |
| Best bundle | [FILL IN] |
| Worst bundle | [FILL IN] |

### What I Learned

- Round 5 was a portfolio problem.
- All 50 products needed strategy classification.
- Bundles helped compare related products.
- Sizing mattered as much as signal quality.
- Product-level PnL tracking was essential.

### What I Could Have Done Better

- Track PnL by bundle earlier.
- Make the code more modular.
- Track portfolio drawdown.
- Use the candlestick charts earlier.

---

## 6. Final Algorithmic Results Summary

| Round | Products | Main Strategy | Final Version | Algo PnL | Rank |
|---|---|---|---|---:|---:|
| Round 1 | Ash-Coated Osmium, Intarian Pepper Root | Anchored moving average + trend following | [FILL IN] | ~98,000 XIRECs | [FILL IN] |
| Round 2 | Same as Round 1, plus Market Access Fee | Refined Round 1 strategy + MAF bidding | [FILL IN] | [FILL IN] | [FILL IN] |
| Round 3 | Velvet Fruit Extract, Hydrogel Packs, VFE options chain | Market making + weak modified Black-Scholes IV model | [FILL IN] | ~1,000 XIRECs | [FILL IN] |
| Round 4 | Same products, plus trade data | Market making + variance-adjusted IV model + trader classification | [FILL IN] | ~10,000 XIRECs | [FILL IN] |
| Round 5 | 50 products across 10 bundles | Multi-product portfolio | [FILL IN] | [FILL IN] | [FILL IN] |

---

## 7. Final Reflection

Round 1 taught me that even simple products can need different strategies. Ash-Coated Osmium needed an anchored moving-average approach, while Intarian Pepper Root was better for trend following with emergency downside protection.

Round 2 taught me that trading performance was only part of the game. The Market Access Fee was a game-theory problem: bid enough to get extra flow, but not so much that the fee eats the profit.

Round 3 taught me that a model can sound right and still trade badly. The modified Black-Scholes IV equation was not strong enough, and the options side lost money. The round only ended positive because market making on Velvet Fruit Extract and Hydrogel Packs helped recover the losses.

Round 4 was the real improvement. Adding variance made the options equation much better. It helped the bot price the Velvet Fruit Extract options more realistically. Trade data also helped because I could identify market makers, noise traders, and informed traders by their trading habits.

Round 5 was a portfolio round. There were 50 products across 10 bundles, and each product needed a strategy choice: market making, mean reversion, trend following, spread trading, or avoiding it if it was too noisy. The bundle structure helped because comparing the five products inside each bundle gave clues about which strategy to use.

The biggest lesson overall was:

> A good algorithm is not just about finding a trade. It is about knowing why the trade works, how big it should be, and when to get out.

---

## 8. Fill-In Checklist

- [ ] Fill in Round 1 PnL and rank.
- [ ] Fill in Ash-Coated Osmium and Intarian Pepper Root PnL.
- [ ] Fill in Round 2 PnL and rank.
- [ ] Confirm whether the ~4,000 XIREC MAF bid was accepted.
- [ ] Confirm exact Round 3 rank.
- [ ] Confirm exact Round 4 rank.
- [ ] Fill in Round 5 PnL.
- [ ] Fill in final algorithmic rank.
- [ ] Add product-by-product PnL.
- [ ] Add final submitted script names.
- [ ] Attach `round5_all_products_candlestick_charts (3).pdf` as the Round 5 chart appendix.

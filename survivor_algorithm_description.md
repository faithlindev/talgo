# Survivor Options Trading Algorithm

This document provides a detailed explanation of the Survivor options trading algorithm, as implemented in `strategy/survivor.py`.

## Strategy Overview

The Survivor strategy is a systematic, automated approach to options trading based on the price movements of the NIFTY index. The core concept is to sell options (both Puts and Calls) to collect premium, while managing risk through a dynamic, gap-based execution system. The strategy is designed to capitalize on premium decay when the market moves beyond predefined thresholds.

## Core Logic

The algorithm's logic is built around three key concepts: dual-side trading, gap-based execution, and dynamic strike selection.

### 1. Dual-Side Trading

The strategy operates on both sides of the market, reacting to both upward and downward price movements:

*   **PE (Put) Trading:** The algorithm sells Put options when the NIFTY index price moves **up** by more than a predefined `pe_gap`. This is based on the principle that as the market rises, out-of-the-money Put options become less likely to be exercised, and their premium will decay.
*   **CE (Call) Trading:** The algorithm sells Call options when the NIFTY index price moves **down** by more than a predefined `ce_gap`. Similarly, as the market falls, out-of-the-money Call options are less likely to be exercised.

### 2. Gap-Based Execution

Trades are not executed on every price change. Instead, the strategy uses a "gap" system to trigger trades:

*   **Reference Points:** The algorithm maintains two key reference points: `nifty_pe_last_value` and `nifty_ce_last_value`. These are initialized at the start and are updated as trades are executed or when the reset mechanism is triggered.
*   **Trade Trigger:** A trade is triggered when the current NIFTY price moves beyond the `pe_gap` (for Puts) or `ce_gap` (for Calls) from its respective reference point.
*   **Position Sizing:** The size of the trade is determined by a `sell_multiplier`, which is calculated based on how far the price has moved past the gap. For example, if the `pe_gap` is 20 points and the NIFTY price moves up by 45 points, the `sell_multiplier` would be `floor(45 / 20) = 2`. The total quantity of options to sell would then be `2 * pe_quantity`.

### 3. Dynamic Strike Selection

The strategy dynamically selects which option strike to trade:

*   **Strike Gap:** The strike price is chosen based on the `pe_symbol_gap` and `ce_symbol_gap` parameters. For example, if the NIFTY is at 24500 and the `pe_symbol_gap` is 200, the algorithm will look for a Put option with a strike price around 24300.
*   **Premium Threshold:** Before placing a trade, the algorithm checks if the premium of the selected option is above the `min_price_to_sell` threshold. If the premium is too low, the strategy will look for a strike closer to the current price to ensure it's selling an option with sufficient value and liquidity.

## Reset Mechanism

To keep the strategy responsive to changing market conditions, a reset mechanism is in place:

*   When the market moves favorably after a trade has been placed (i.e., the price moves down after a PE trade or up after a CE trade), the corresponding reference point (`nifty_pe_last_value` or `nifty_ce_last_value`) is reset to be closer to the current market price.
*   This prevents the reference points from drifting too far from the current price, which would make it difficult to trigger new trades. The `pe_reset_gap` and `ce_reset_gap` parameters control when this reset occurs.

## Configuration Parameters

The behavior of the Survivor strategy is controlled by a set of parameters in the `strategy/configs/survivor.yml` file:

*   `index_symbol`: The underlying index to track (e.g., "NSE:NIFTY 50").
*   `symbol_initials`: The options series to trade (e.g., "NIFTY25807").
*   `pe_gap` / `ce_gap`: The price movement thresholds to trigger PE and CE trades.
*   `pe_symbol_gap` / `ce_symbol_gap`: The distance from the current price to select the option strike.
*   `pe_reset_gap` / `ce_reset_gap`: The thresholds for resetting the reference points.
*   `pe_quantity` / `ce_quantity`: The base quantities for each trade.
*   `min_price_to_sell`: The minimum option premium required to place a trade.
*   `sell_multiplier_threshold`: A risk parameter to limit the maximum size of a position.

## Risk Management

The Survivor strategy includes several features to manage risk:

*   **Premium Filtering:** By only selling options above a minimum premium (`min_price_to_sell`), the strategy avoids trading illiquid or low-value options.
*   **Position Scaling Limit:** The `sell_multiplier_threshold` prevents the strategy from taking on excessively large positions during periods of high volatility.
*   **Dynamic Strike Adjustment:** If the premium for a selected strike is too low, the algorithm adjusts the strike to find one with adequate premium, ensuring better trade quality.
*   **Reset Logic:** The reset mechanism prevents the strategy from accumulating an excessive number of positions in a runaway market.

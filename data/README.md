# WCC Synthetic Data v2 - README

## Overview
Synthetic data for the Wominjeka Coffee Co (WCC) vehicle routing project.
All parameter values are grounded in research (see References below).

Person A provides real cafe coordinates and distance matrix. Merge by matching
cafe_id to real locations at the CP1 meeting.

## Files

### products.csv
Product catalogue: 4 milk types + 3 bean types.
- `product_id`, `product_name`, `category` (milk/beans)
- `unit`: litres (milk) or kg (beans)
- `weight_per_unit_kg`: ~1.03 kg/L for milk, 1.0 kg/kg for beans
- `revenue_per_unit`, `cost_per_unit`, `margin_per_unit`
- `perishable`: True for milk, False for beans
- `shelf_life_hours`: milk = 8-10h, beans = effectively unlimited

### cafes.csv
35 cafes with placeholder IDs. Size distribution: 40% small, 40% medium, 20% large.

### demands.csv / demands_pivot.csv
Daily demand per cafe per product. Long format (245 rows) and pivot format.

### vans.csv
Fleet of 5 vans (use 2-5 in experiments). Capacity 500-700 kg, fuel $0.40-$0.50/km.

### perishability_params.csv
T_max (3h default, 2h/4h for sensitivity), spoilage cost, loading/service times.

### cafe_summary.csv
Pre-computed total daily weight (kg) and revenue ($) per cafe.

### depot.csv
Placeholder depot entry. Person A sets real coordinates.

## Key Assumptions and Research Basis

### Milk Consumption Per Cafe
- A cafe uses approximately 10.4 litres of milk per 100 cups of coffee [1].
- A small independent cafe sells 160-220 cups/day, using ~17-23L of milk [1].
- A medium cafe sells 220-370 cups/day, using ~23-38L [1].
- A large/busy cafe sells 370-800 cups/day, using ~38-83L [1].
- A busy Sydney cafe was observed using ~150L in a single day [2].
- Merlo Coffee uses ~1 million litres/year across 16 stores (~170L/store/day) [3].
- Our generated data: small avg 20L, medium avg 32L, large avg 68L. Consistent.

### Milk Type Split
- ~72% dairy milk, ~28% alternatives in the UK market [1].
- In Australia, 86% of coffees contain milk, 75% are >90% milk [3].
- Regular fat milk accounts for ~70% of Australian milk consumption [4].
- We use: ~75% dairy (full cream dominant), ~25% alternatives (oat > soy > almond).

### Coffee Bean Usage Per Cafe
- Small/cafe-restaurant: ~1 kg/day [5].
- CBD/coffee-centric: ~2-3 kg/day [5].
- Medium busy: ~3-5 kg/day [6].
- Large/high-volume: ~7-15 kg/day [6].
- Standard Australian espresso uses ~18g per double shot [7].
- Our generated data: small avg 2kg, medium avg 4kg, large avg 6kg. Consistent.

### Van Capacity
- Standard refrigerated delivery van: 500-800 kg payload.
- Fuel cost: $0.40-$0.50/km including fuel + wear for urban delivery.

### Perishability
- Fresh milk should be delivered within 2-4 hours to maintain quality.
- Default T_max = 3 hours, with 2h (tight) and 4h (relaxed) for sensitivity.
- Service time per cafe: 5 minutes (unloading). Loading at depot: 15 minutes.

## Summary Statistics (v2)
- Total cafes: 35 (17 small, 12 medium, 6 large)
- Total daily milk demand: 1,129 litres
- Total daily bean demand: 130 kg
- Total daily weight: ~1,291 kg
- Total daily revenue potential: ~$6,464
- 3-van utilisation: ~68%
- 5-van utilisation: ~42%

## References

[1] Start Right Retail, "How much milk does a cafe use?", 2024.
    https://www.startrightretail.com/post/how-much-milk-does-a-cafe-use

[2] CoffeeSnobs Forum, "How many coffees in a day for a busy cafe?", 2013.
    https://coffeesnobs.com.au/forum/coffeesnobs-discussions/general-coffee-related/33779-how-many-coffees-in-a-day-for-a-busy-cafe

[3] Dairy Australia / Canberra Times, "How Australia's coffee culture is helping drive milk consumption", 2019.
    https://www.canberratimes.com.au/story/6344459/how-coffee-is-helping-drive-milk-consumption/

[4] Australian Bureau of Statistics, "Share of milk and milk alternatives consumed on a daily basis in Australia FY 2023", via Statista, 2024.
    https://www.statista.com/statistics/1242204/australia-milk-mean-daily-share-grams-per-capita-by-fat-and-type/

[5] Quora, "How many pounds of whole bean coffee does a coffeehouse use in a typical day?", multiple cafe owners responding.
    https://www.quora.com/How-many-pounds-of-whole-bean-coffee-does-a-coffeehouse-use-in-a-typical-day

[6] Coffee Forums, "Generally how many Kg of coffee is required for a coffee shop?", 2013.
    https://www.coffeeforums.com/threads/generally-how-many-kg-of-coffee-is-required-for-a-coffee-shop.11247/

[7] BusinessDojo, "How to estimate coffee and milk costs for a coffee shop?", 2024.
    https://dojobusiness.com/blogs/news/coffee-shop-cost-estimation

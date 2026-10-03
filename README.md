# Live Recommendations for a Quick-Commerce App - SDA Final Project

## Milestone 1: load the data into MongoDB

### 1. One-time setup
Open a terminal inside this folder, then:

Windows (PowerShell):
    python -m venv venv
    venv\Scripts\Activate.ps1
    pip install -r requirements.txt

Mac / Linux:
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt

### 2. MongoDB Atlas
1. In Atlas, create a NEW database user for this project (Database Access > Add New Database User). Use a new password.
2. Atlas > Network Access: make sure your current IP address is allowed.
3. Copy `.env.example` to `.env` and put your connection string in it:
       MONGO_URI=mongodb+srv://<new_user>:<new_password>@<your-cluster>.mongodb.net/
       DB_NAME=qc_streaming
   `.env` is in `.gitignore`, so it will not be committed.
   (If the old password from Assignment 2 was ever pushed to GitHub, change it in Atlas too.)

### 3. Run
    python m1_load_data.py

### 4. Expected output (the SANITY CHECK block must match)
    products 268 | users 2500 | users with order history 2172
    order_history rows 5000
    products with >= 15 ratings 224 | of those, average < 3.0: 20

If any number differs, stop and send me the full output.

## Files
- config.py   every threshold, window and assumption (edit numbers here only)
- db.py       MongoDB connection
- m1_load_data.py   this milestone
- data/       the supplied CSV files


---

## Milestone 2: events flowing through Kafka into MongoDB

Needs 3 terminals (VS Code: Terminal > New Terminal; activate the venv in each).

0. Quick self-test, no Kafka needed (prints a few events and a summary):
       python m2_producer.py --dry-run --speed 100 --rate 1500 --duration 5
1. Install the new package and start Kafka (Docker Desktop must be running):
       python -m pip install -r requirements.txt
       docker compose up -d
   (If Kafka from Assignment 2 is already running on localhost:9092, skip the docker line.)
2. Create the topics:
       python m2_create_topics.py
3. Terminal A (start this FIRST):   python m2_store_consumer.py
   Terminal B:                      python m2_producer.py --rate 20
   Let both run 5 minutes, then press Ctrl+C in each.
4. Check the result:
       python m2_check.py

Expected from m2_check.py:
- events by type: mostly product_impression, then product_click, add_to_cart, app_open; a few order_placed, remove_from_cart, rating_submitted
- late (offline) events: about 5 to 10% (needs a few minutes of running)
- on-time lag median: well under 1 second
- events per Kafka partition: spread over partitions 0, 1 and 2

Always run the producer at --speed 1 (the default) when it feeds Kafka. Higher speeds are only for the --dry-run test.

Trend burst for the demo (later):  python m2_trend_burst.py --product "Ice Cream"


---

## Milestone 3: stream processor and the four rules

0. Offline rule tests (no Kafka or MongoDB needed):
       python m3_rule_tests.py
   Expect: 8 of 8 checks passed.

1. Make sure Kafka is running (Ubuntu window). Then open 3 VS Code terminals (venv active in each), in this order:
   Terminal A:   python m2_store_consumer.py
   Terminal B:   python m3_processor.py        (takes about 10 seconds to load, then prints "Processor running")
   Terminal C:   python m2_producer.py --rate 20
   Let them run 3 to 5 minutes. Terminal B prints events and recommendations every 5 seconds.

2. Stop with Ctrl+C: producer first, then the processor, then the store consumer.

3. Check the result:
       python m3_check.py

Demo settings (faster trending, used in the live demo):
       python m3_processor.py --trend-window 90 --trend-slide 2


---

## Milestone 4: the customer screen

New this time: fixes to the recommendation rules (no repeated product, offers only for a non-empty cart) and the screen itself.

1. Install the new packages:
       python -m pip install -r requirements.txt
2. Re-run the rule tests (now 11 checks):
       python m3_rule_tests.py
3. Start, in this order (Kafka running in Ubuntu first):
   Terminal A:   python m2_store_consumer.py
   Terminal B:   python m3_processor.py --trend-window 90 --trend-slide 2
   Terminal C:   python m2_producer.py --rate 20
   Terminal D:   python m4_app.py
4. Open in Chrome:
       Live recommendations (treatment):    http://127.0.0.1:8000/?group=treatment
       Stale recommendations (control):     http://127.0.0.1:8000/?group=control
   (open both side by side in two windows to compare)
5. Try it: add Bread or Milk to the cart and watch the recommendations change with their reasons;
   click Add on a recommendation; watch the free-delivery bar; place an order.

Demo customers are people who never appear in the simulated stream, so the screen shows only your own actions.


---

## Milestone 5: the business dashboard

New files: dashboard_data.py, m5_dashboard.py.  Changed: recommender.py, m4_app.py, web/index.html, m2_store_consumer.py, config.py, requirements.txt.

1. Install the new packages and re-run the rule tests (12 checks):
       python -m pip install -r requirements.txt
       python m3_rule_tests.py
2. Restart everything in this order (Kafka running in Ubuntu first). Use one terminal each, venv active:
   A   python m2_store_consumer.py
   B   python m3_processor.py --trend-window 90 --trend-slide 2
   C   python m2_producer.py --rate 20
   D   python m4_app.py            (customer screen, http://127.0.0.1:8000)
   E   python m5_dashboard.py      (dashboard,       http://127.0.0.1:8050)
3. Open the dashboard in Chrome and, in other tabs, the customer screens (?group=treatment and ?group=control).
4. Click around on the customer screens: the funnel, click-through and rule table move.
   Run  python m2_trend_burst.py --product "Ice Cream"  and watch events/sec and the trending recommendations.

Stop in this order: producer, processor, web app, dashboard, consumer.

### M5 fixes (after the first dashboard run)
Changed: dashboard_data.py, m5_dashboard.py.  New: m5_reset.py.
- Funnel is now nested (no stage above 100%).
- Latency chart uses a log axis, so a catch-up spike does not flatten it.
- Business impact shows 95% intervals, sample sizes and a noise verdict.
- Only the dashboard (terminal E) needs a restart.

Clean demo / recording: stop the producer, processor, web app, dashboard and consumer, then
       python m5_reset.py          (asks you to type RESET; deletes run data only, keeps the M1 data)
and start everything again.

---

## Milestone 6: the live-vs-stale experiment

New: interactive.py, m6_experiment.py.  Changed: recommender.py, simulator.py, m5_dashboard.py, dashboard_data.py.

    python m3_rule_tests.py                 (12 checks)
    python m6_experiment.py --quick         (about 30 seconds, rough numbers, to check it runs)
    python m6_experiment.py                 (full run, several minutes)

The full run prints the results table for slide 10 and saves it to results/ and to MongoDB
(collection experiment_results), where the dashboard picks it up (restart the dashboard).
The experiment runs offline (no Kafka needed).

### M6 fix: carts from an earlier visit
A new visit (app_open) now starts with an empty cart. Before, an abandoned cart stayed in the processor's memory, so the
customer screen could not recommend an item left in an old cart. This also changed the experiment results (slightly in
favour of live), so re-run it or load the supplied file:
    python m3_rule_tests.py            (15 checks)
    python m6_load_results.py          (loads results/experiment_*.json into MongoDB)
Restart the processor (terminal B) and the dashboard (terminal E) afterwards.
Demo burst (this Ice Cream has a reliable rating, so it is not swapped out):
    python m2_trend_burst.py --product-id 272170

### Dashboard tabs (four purposes)
The dashboard now has four tabs, matching the KPI document:
- Pipeline health (engineers and operators): events/sec, latency, recommendations/min, late events, streaming health, late events chart, alerts.
- Recommendation engagement (product team): shown, click-through, add-to-cart rate, recommendations by rule, rule performance.
- Business outcome (managers): sessions, conversion, order value, revenue per session, funnel, live vs stale, experiment.
- Quality guard rail (category managers): reliable ratings, low-rated items replaced, ratings by category, lowest-rated products.
Changed files: m5_dashboard.py, dashboard_data.py.  Restart the dashboard (terminal E) to see it.

### Realistic prices
The supplied prices are synthetic (onions at Rs 948). Realistic ones are now generated per product, inside a band per product name
(config.PRICE_BANDS, approximate and editable, NOT Blinkit's prices), plus an MRP (old price).
    python make_realistic_prices.py     (writes data/products_priced.csv; the same prices every time)
    python m1_load_data.py              (reloads MongoDB with the new prices; safe to re-run)
    python m6_load_results.py           (loads the re-run experiment results; the numbers changed, see results/)
Free delivery is now Rs 299 (a typical order is about Rs 280). Restart the processor, web app and dashboard afterwards.

### Presenting from another laptop
See PRESENT_FROM_ANOTHER_LAPTOP.md. In short: run the pipeline on your laptop, start the shop and the dashboard with --share
(python m4_app.py --share, python m5_dashboard.py --share) and open the addresses they print from the other laptop.

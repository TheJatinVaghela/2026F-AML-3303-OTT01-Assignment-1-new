# MapleFreight late-risk alerts (AML 3303 A1)

Student ID: c0968267

## What this is

MapleFreight runs roughly eight thousand shipments a week across Ontario, Quebec and the Prairies. On-time performance slipped from 94% to 88%. A lot of delays only show up after the promised window, when credits and customer calls are already in motion.

This repo trains a late-delivery classifier and sends high-risk loads to Slack channel `#dispatch-alerts` while dispatch can still re-route or warn the account.

## Data

File: `data/maplefreight_delivery_delay_dataset.csv` (6,035 rows, 23 columns).

| | |
|---|---|
| Target | `delivered_late` (1 = missed window) |
| Late share | ~20.6% |
| Leakage removed | `actual_transit_hours` (known only after delivery) |
| Cleaning | 32 dupes dropped; `service_level` casing fixed; missing fields imputed in the sklearn pipeline |

## Repo contents

```
data/maplefreight_delivery_delay_dataset.csv
screenshots/slack_alert.png
MapleFreight_Delivery_Delay_Prediction_c0968267.ipynb
requirements.txt
.env.example
.gitignore
README.md
```

## Run locally

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux
pip install -r requirements.txt
```

Slack: create `#dispatch-alerts`, add an Incoming Webhook, copy `.env.example` to `.env`, set `SLACK_WEBHOOK_URL`.

Open the notebook and run all cells from a clean kernel. The first code cell resolves the project folder if Jupyter starts somewhere else.

## Models and alert cut

| | |
|---|---|
| Baseline | `DummyClassifier(most_frequent)` ~79% accuracy, 0 late recall |
| Final | Logistic regression, `class_weight="balanced"` |
| Also tried | Random forest (balanced): better accuracy, worse late recall (~0.48) |
| Imbalance | class weights + probability threshold tuning |
| Alert cut | **0.45** |
| Test @ 0.45 | Late class: precision ~0.40, recall ~0.79, F1 ~0.54 |

I picked 0.45 because dispatch cares about missed late loads more than headline accuracy. Versus 0.50 on my holdout, recall moves from ~0.74 toward ~0.80 (false negatives drop from 65 to about 51). Precision drops, so the alert list is longer. For a pilot that trade-off made sense to me.

## What stood out in the data

- Contract Owner-Op late ~36%; MapleFreight Fleet ~13%.
- Storm ~46% late vs clear ~14%.
- Late trips skew toward later pickup, more stops, longer distance, heavier weight.
- Logistic coefficients pointed the same way: storm, owner-ops, distance, traffic, Economy up; clear weather and fleet down.

## AI Used

- For Read me and Comments and test and resolving error AI was used 

## Ops ideas (from the notebook)

1. Tighten Economy on storm/snow days when the window is already tight.
2. Watch Contract Owner-Op on Gold/Silver long hauls until their 30-day late count cools off.
3. Pause and re-check ETA if pickup is already 20+ minutes late.
4. Use fleet trucks more on long, high-traffic lanes when you can.
5. Run Slack at 0.45 for a few weeks, then nudge the cut up if the queue feels too noisy.


## Submission notes

Notebook filename includes c0968267 and is executed with outputs. Baseline and final model both report precision, recall, F1 and confusion matrices. Screenshot lives at `screenshots/slack_alert.png`. No `.env` or webhook URL in git. Dataset is under `data/`.

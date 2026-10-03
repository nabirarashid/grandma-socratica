# Grandma Must Win

<img width="2360" height="1640" alt="Grandma's Bakeria and The Bakery, side by side on a street" src="https://github.com/user-attachments/assets/d6cafe41-03f3-4732-87ef-84fd6876a84c" />

Grandma's bakery has a corporate chain opening right next door. Rather than
guess what she should do about it, this simulates the street.

A town of 400 customers is generated from five segments, each with their own
tastes, budget, allergies and daily routine. Every one of them walks past both
shops and either buys something or walks away. Market share comes out of
thousands of individual decisions instead of being assumed.

What makes it more than a toy is that the chain fights back. Change a price and
it answers, with the friction a real business has, and you can watch what your
clever idea is actually worth once they do.

## Try it

Two terminals. Backend first:

```bash
cd backend
uv sync
uv run python -m grandma_sim.api      # http://127.0.0.1:8000, API docs at /docs
```

Then the front end:

```bash
cd frontend
npm install
npm run dev                           # http://localhost:5173
```

Open [localhost:5173](http://localhost:5173), edit either menu, and press
Start to play the day out. No backend handy? `VITE_USE_MOCK=1 npm run dev`
replays a saved day instead.

## How it works

**The customers.** Five segments (commuters, the lunch crowd, students,
regulars, the after-work crowd) each get their own distributions for taste,
price sensitivity, portion preference, allergies, arrival time and loyalty. So
you get a crowd that disagrees with itself rather than 400 copies of an average
shopper.

**The choice.** Every item sits somewhere on three flavour meters (sweet to
savoury, bitterness, fruitiness) plus category tags, a price, a portion size,
allergens, and how much it appeals at each time of day. Customers carry the
same flavour profile as the thing they are looking for, so one distance
function scores any customer against any item. Each visit is a multinomial
logit over every allergen-safe item on both menus, plus the option to buy
nothing. Allergens exclude an item outright rather than just counting against
it.

**The competitor.** The Bakery works out which of its products rival which of
grandma's by taste, with no hardcoded pairs, then reprices on a weekly review
using the price it last saw four days ago, moving part of the way to its target
rather than jumping. It undercuts when it is losing, quietly raises prices when
it is comfortably ahead, and will not sell below what the item costs it to
make. So an ingredient shock pushes its prices up whether or not it is winning.

**The money.** Items are costed from real recipes against daily ingredient
prices, so a supplier shock flows all the way from the price of cream to what
grandma takes home. Every run is also simulated with the competitor's prices
frozen, which gives you the counterfactual: what a decision wins, against what
their response takes back.

## Things to try

```bash
cd backend

# Grandma cuts her signature parfait on day 2. Does it pay?
uv run python run_season.py --days 21 --cut fall_parfait=6.50@2

# A trade war triples the price of cream for everyone.
uv run python run_season.py --days 21 --spike cream=12@5

# She reformulates the parfait savoury, and the knock-off stops shadowing it.
uv run python run_season.py --days 21 --sweeten fall_parfait=-0.85@3
```

Each prints what the chain did and why, and what their response cost or saved
her, averaged over several random seeds. The averaging matters: a single
simulated run sits inside day-to-day noise and can come out with the wrong
sign.

## What's in here

```
backend/          Python simulation, managed with uv
  grandma_sim/
    core/         flavour meters, the day clock, shared vocabulary
    menu/         items, both bakeries' menus, recipes and costing
    customers/    population generation from segment distributions
    choice/       what a customer wants, and the logit that picks
    competition/  which items are rivals, and how the chain reprices
    simulation/   one day, a run of days, and the day's ledger
    api/          FastAPI service the front end talks to
  run_simulation.py   one day, as a readable report
  run_season.py       a run of days with the chain reacting
frontend/         React, TypeScript, Vite and Mantine; plays the day back
```

There is a longer write-up of the simulation in
[`backend/README.md`](backend/README.md), including the API routes and every
knob on the competitor's pricing policy.

## About

Built in a weekend at the Making Dough buildathon, as a team. This is my fork.

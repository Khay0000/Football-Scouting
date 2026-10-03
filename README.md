# Creative Midfielder Scouting Tool

An end-to-end data analytics project that scores attacking and creative midfielders in Europe's top 5 leagues and flags young players who may be undervalued by the market.

## Question
Which young midfielders in the top 5 European leagues perform like elite players but are likely undervalued by the market?

## Dashboard
![Final dashboard](images/final-dashboard.png)

The Power BI file is `football_scouting.pbix`. It includes filters for season, league, age and minutes, a score vs market value chart, a top 15 ranking, a skill profile for a selected player, and a best-value table.

## Key findings
- Among the highest-scoring players aged 24 or under in 2024, Ben Seghir (Monaco, age 20) scores 89.2 with a market value of about €28M, and Akliouche (Monaco, age 23) scores 91.2 at about €45M. Both offer high scores for a modest price.
- Fermín López (Barcelona, age 22) scores 96.7 at about €50M, close to Musiala (Bayern Munich, age 22), who scores 95.5 at about €140M, nearly three times the value.
- Score and market value are positively related but only loosely: Cole Palmer (Chelsea, age 23) is valued at about €120M with a score of 88.1, below Ben Seghir's 89.2 at roughly a quarter of the price.
## Data
- **Understat**: player stats per season (xG, xA, shots, key passes, xGChain, xGBuildup) for the Premier League, La Liga, Bundesliga, Serie A and Ligue 1, seasons 2022-2024.
- **Transfermarkt** (Kaggle dataset "Football Data from Transfermarkt"): date of birth and market value over time. These two files are large, so they are not in this repo. Download `players.csv` and `player_valuations.csv` from Kaggle and put them in `data/raw/`.

## Method
1. **Collect:** pulled player-season stats from Understat for 5 leagues and 3 seasons (8,357 player-seasons).
2. **Clean:** kept players with at least 900 minutes, converted totals to per-90 stats, combined players who changed clubs mid-season into one row, and cleaned names.
3. **Merge:** matched Understat to Transfermarkt by normalised name, then fuzzy matching (RapidFuzz). About 98% of players matched, and I checked the fuzzy matches by eye.
4. **Age and value:** age is calculated at the end of each season. Market value is the latest Transfermarkt valuation within a year before the season ended.
5. **Select:** midfielders are players Understat labels as M without F or D, giving a pool of attacking and creative midfielders.
6. **Score:** each stat is converted to a percentile within its season. Stats are grouped into creation (xA, key passes), build-up (xGChain, xGBuildup) and scoring (npxG, shots), because the stats inside each group were highly correlated (0.80 to 0.86). Group scores are averaged, then weighted 35% / 35% / 30%.
7. **Similar players:** nearest-neighbour search (scikit-learn) on the six percentile columns.
8. **Dashboard:** built in Power BI from `data/processed/dashboard_data.csv`.

## Limitations
- Understat has no tackles, interceptions or other defensive data, so the score measures attacking and creative contribution only. Defensive midfielders will score low.
- Understat's position labels include some wide players, so the pool includes a few wingers.
- Players at dominant teams get inflated numbers because their team has the ball more.
- Leagues differ in strength and the score does not adjust for that.
- The weights are my own judgement, not a validated model.
- Market values are Transfermarkt estimates, not transfer fees.
- Fuzzy name matching can occasionally miss a player or join the wrong one, although the matches I checked were correct.

## Project structure
- `01_data_pull.ipynb`: collect Understat data
- `02_cleaning.ipynb`: clean, filter and merge
- `03_analysis.ipynb`: scoring, similar players, undervalued list
- `04_dashboard_data.ipynb`: prepares the dashboard file
- `data/raw/`, `data/processed/`: data files
- `football_scouting.pbix`: Power BI dashboard
- `images/`: dashboard screenshot

## How to run
1. Create and activate a virtual environment: `python -m venv venv`, then `source venv/Scripts/activate` (Windows Git Bash).
2. Install packages: `pip install -r requirements.txt`
3. Add the two Transfermarkt files to `data/raw/`.
4. Run the notebooks in order, 01 to 04.

## Tools
Python, pandas, scikit-learn, RapidFuzz, matplotlib, seaborn, Power BI

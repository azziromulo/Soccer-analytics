# Soccer-analytics
An interactive, single-file BI application about the Brazilian football league (Campeonato Brasileiro Série A) in the round-robin era, 2003–2024
It is built to look and behave like a Power BI report, with report pages, a data view and a semantic-model view, but it runs entirely in the browser from one HTML file.

The project combines historical league analysis, a statistical lab, a match simulator, a 2024 player explorer and a card game, and it documents the equivalent Power BI model and DAX measures so the same analysis can be rebuilt in Power BI.How to run

No build step and no server are needed.
<img width="1783" height="787" alt="image" src="https://github.com/user-attachments/assets/90bd3384-1dd9-4174-83ca-68714017da7f" />

Download Brasileirão Analytics.html (you may rename it to index.html).
Open it in any modern browser (Chrome, Edge, Firefox, Safari).

The only external dependencies are loaded from CDNs: Chart.js 4.4.1 and Google Fonts (IBM Plex Sans, IBM Plex Mono, Saira Condensed). All data and club crests are embedded in the file.

Pages can be opened directly with URL hashes: #geral, #campeoes, #clubes, #estatistica, #simulador, #jogadores, #trunfo, #dados, #modelo.

To publish it, enable GitHub Pages on the repository and point it to the file renamed as index.html.

Data
Source	Content	Used in
brasileirao.csv	450 rows: final standings of every club in every Série A season 2003–2024 (position, points, games, W/D/L, goals for/against, goal difference)	All league pages
database.csv	Player-match records for the 2024 season (≈11.7k rows, 725 players): minutes, goals, assists, xG, xAG, shots, passes, tackles, dribbles, cards and more	Players and Trunfo pages
escudos-clubes-brasileiros.zip	Crests of the 44 clubs, converted to 128×144 PNG and embedded as data URIs	Every page
Data quality notes handled in the app
Variable format: 24 clubs / 46 games (2003–2004), 22 clubs / 42 games (2005), 20 clubs / 38 games (2006 onward). All comparisons use rates (points %, per-game figures), not raw points.
Court-adjusted points: rows where points ≠ 3×wins + draws are flagged and measured (e.g. São Caetano 2004).
Relegation rule: 2 clubs in 2003, 4 in every other season.
2024: two clubs appear with 37 games in the source data.

<img width="1879" height="894" alt="image" src="https://github.com/user-attachments/assets/2411c47e-d9b2-452d-a90e-29215188881e" />

<img width="1779" height="685" alt="image" src="https://github.com/user-attachments/assets/f4d9b5fc-e079-45d2-809c-ae695bd3e147" />

What was built
1. Report view (7 pages)

Overview – KPI cards (seasons, clubs, matches, goals per match, draw rate, champion's points %) with ▲ green / ▼ red comparisons against the previous season; champions strip with crests; titles per club; goals per match by season against the period average; champion vs runner-up points %; participations by state.

Champions analysis – a structured study with objectives, indicator definitions and answers:

3.1 Statistical profile of title-winning campaigns (mean, median, min, max, standard deviation) and how many champions also led each indicator.
3.2 Champions vs all other campaigns, with Cohen's d and Welch's t-test, plus a strip plot.
3.3 Goals scored, conceded and goal difference vs final position, with Pearson and Spearman correlations on a normalized position scale.
3.4 Historical evolution of champions with least-squares trend lines and significance.
3.5 Consistency: titles, top-4 presence and a season-by-season matrix, with longest streaks.
3.6 Exceptional campaigns: six leaderboards and highlights (best non-champion, weakest champion, best defense outside the top 4).

Clubs – profile for any of the 44 clubs: titles, points %, average and best position, top-4 and relegations, attack and defense strength, points vs expected; final position chart, actual vs expected points %, and a season table with deltas.

Statistics lab – region filter (via the state dimension); sortable ranking; goal difference per game vs points % scatter with regression; attack vs defense strength quadrant with crests; points above/below expectation; table cut lines (champion, 4th place, first club out of relegation).

Match simulator – pick two clubs and, optionally, a specific season for each side (cross-era matchups are allowed):

Attack/defense strengths relative to each season's league average, or a recency-weighted history with selectable half-life (1, 3, 6 years or equal weight) and a shrinkage prior.
Home advantage (×1.18 / ×0.85) or neutral venue.
Poisson model with Dixon-Coles correction (ρ = −0.08): win/draw/loss probabilities, expected goals, over 2.5, both teams to score, scoreline heatmap.
An animated 90-minute simulated match, 10,000 Monte Carlo simulations, and a head-to-head comparison.

Players · 2024 – filters by name, nationality, age, minutes, position and club; top 10 by a chosen metric; goals vs xG scatter; player profile with percentile bars against players of the same position group (900+ minutes) and a per-match goals/assists chart; sortable full table.

Trunfo (top-trumps card game) – a playable card game with every 2024 player with 900+ minutes. Cards have an Overall rating (45–99) built from per-90 percentiles weighted by position, plus eight attributes. Includes deck sizes (8/12/20), easy/hard computer opponent, a trump card that beats any non-forward, a round log and a searchable full deck.

2. Data view
The model tables (fClassificacao, dTime, dJogador, dTemporada, dEstado, dFaixa) with search and copy as TSV for pasting into Excel or Power BI "Enter data".
Crest replacement: upload one or many images (matched to clubs by file name, abbreviation or alias) or drag and drop; stored only in the browser's localStorage.
Data quality panel.
3. Model view
Star schema diagram: fact table fClassificacao, dimensions dTemporada, dTime, dFaixa, snowflaked dEstado, the players branch dJogador → fJogadorPartida, and disconnected tables for the simulator and card game. Hovering a table highlights its relationships.
Relationship table with cardinality, filter direction and purpose.
Measure catalog with DAX: syntax-highlighted code, copy button and a live value computed in the current context. Folders cover base measures, rates, achievements, statistics (pythagorean expectation, z-scores, correlation, attack/defense strength), comparisons (previous season, deltas with arrow labels and colors, moving average), champion analysis (including a Spearman measure and a LINESTX trend), players, Trunfo, simulation (Poisson grid with Dixon-Coles in DAX), calculated tables/columns and the Recorte calculation group (Champions / Others / Δ %).
Methods at a glance
Concept	Implementation
Points %	points ÷ (3 × games)
Pythagorean expectation	0.912 × GF^1.32 ÷ (GF^1.32 + GA^1.32), calibrated on this dataset (R² ≈ 0.87)
Attack / defense strength	goals for (against) per game ÷ league goals per team per game in that season
Relative position	(position − 1) ÷ (clubs − 1), so 20-, 22- and 24-club seasons share one scale
Group comparison	Cohen's d, Welch's t-test
Association	Pearson r, Spearman ρ (average ranks for ties)
Trend	ordinary least squares, reported per decade with p-value
Match model	Poisson goals × Dixon-Coles low-score correction, 0–10 goal grid
Technical notes
One self-contained HTML file: HTML, CSS and vanilla JavaScript, with Chart.js for charts.
Light and dark themes (follows the system setting), responsive layout down to phone width.
Accessibility: ARIA roles and labels, keyboard-focusable controls, live regions, and prefers-reduced-motion support.
No data leaves the browser; uploaded crests stay in local storage.
Repository structure
.
├── Brasileirão Analytics.html   # the full application (rename to index.html for GitHub Pages)
└── README.md
Disclaimer

Club names and crests belong to their respective owners and are used here for non-commercial, educational analysis. Statistics reflect the source files listed above.

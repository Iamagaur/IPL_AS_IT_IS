# The IPL GOAT Project

A contextual GOAT-ranking analysis of the Indian Premier League, built from 285,907 ball-by-ball deliveries across 1,202 matches (2008–2026).

**Live site:** push to GitHub Pages and you're done — see deployment below.

## What this is

Most "IPL all-time best" lists rank by raw runs, wickets, or career SR. Those numbers are biased by era (8.5 RPO in 2014 vs 9.7 RPO in 2024), tenure (Kohli has more matches than Russell), and the kind of cricket each player gets to play.

This project re-ranks every IPL career using a **contextual impact score** that adjusts for:

- **Era** — runs above what a league-average batter would score in that season
- **Phase** — powerplay, middle, and death overs are scored against their own baselines
- **Stage** — finals (3×), playoffs (2×), and league games (1×) weigh differently
- **Opposition strength** — performances vs strong teams are worth more

It then tests the famous IPL narratives ("Dhoni in chases," "Kohli in knockouts," "CSK at Chepauk," "Rohit groomed superstars") with raw ball-by-ball evidence and gives each one an honest verdict — *supported, partially supported, or rejected.*

## Methodology summary

### Batting impact
```
bat_impact = (runs_above_expected
            + sr_premium
            + win_bonus)
            × stage_weight
            × opposition_multiplier
```

### Bowling impact
```
bowl_impact = (runs_below_expected
             + (wickets × 20)
             + win_bonus)
             × stage_weight
             × opposition_multiplier
```

### Captain GOAT score
```
goat_score = titles × 100
           + knockout_wins × 15
           + (win_pct − 50) × matches × 0.4
           + stars_developed × 30
           + avg_teammate_lift × matches × 0.3
```

The "captain effect" is computed by taking each batter who played 15+ innings under a captain, comparing their per-innings impact under that captain to their full-career baseline, and averaging across all such teammates.

## Data source

All ball-by-ball data is from [Cricsheet](https://cricsheet.org), maintained by Stephen Rufus — the same public dataset used by professional analysts and academic research. 1,202 matches, 285,907 deliveries, 798 unique players. Captain assignments are 88% verified per match (the unmatched ~290 are mostly from 2007/08 where leadership was contested).

## Honest limitations

- **No fielding saves** — only catches, stumpings, and run-outs are recorded. Iconic boundary saves don't count.
- **No leadership intangibles** — dressing-room culture isn't in the JSON.
- **Captain assignment is human-curated** for accuracy — the script `derive_captains.py` contains the per-season list and can be audited or amended.
- **Knockout sample sizes are small** — a player with 6 knockout innings has noisy data. We flag this where relevant.

## Reproducing the pipeline

If you want to rebuild the data yourself:

```bash
# 1. Download Cricsheet IPL JSON archive from cricsheet.org
unzip ipl_male_json.zip -d data/
mkdir processed/

# 2. Run pipeline in order
python3 parse_matches.py        # raw → ball-by-ball parquets
python3 derive_captains.py      # per-match captain mapping
python3 build_aggregates.py     # innings & match aggregates
python3 build_impact.py         # contextual impact scoring
python3 captain_analysis.py     # captain GOAT scoring + grooming analysis
python3 team_and_narratives.py  # per-team GOATs + narrative tests
python3 build_dashboard_data.py # consolidate to data.json

# 3. Copy to site
cp processed/dashboard_data.json site/data.json
```

## License

MIT for the code. Cricsheet data is © Stephen Rufus and provided under [ODbL](https://opendatacommons.org/licenses/odbl/).

---

*Built April 2026.*

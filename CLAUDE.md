# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running Locally

This is a static site with no build step or package manager. It must be served over HTTP (not file://) because it fetches JSON data at runtime.

```bash
python3 -m http.server
# Then open http://localhost:8000
```

## Architecture

**CollegePredict** is a vanilla JavaScript single-page application that predicts college admissions eligibility for Indian engineering entrance exams (JEE Main, JEE Advanced, BITSAT, MHT-CET).

### Files

- `index.html` — Single HTML page with the form and results layout
- `script.js` — All application logic (data loading, form handling, rank prediction, results rendering)
- `style.css` — Full styling with CSS custom properties for theming, responsive breakpoint at 640px
- `data/cutoffs.json` — Static dataset of historical marks-to-rank mappings and college cutoff data

### Data Flow

1. On load, `script.js` fetches `data/cutoffs.json` and populates the exam dropdown
2. Selecting an exam reveals input fields and populates category/branch filters from that exam's college list
3. User enters marks or rank (togglable mode), selects category and branch
4. On "Find My Colleges": if marks mode, `getRankRange()` interpolates rank from historical data across multiple years (2022-2024); if rank mode, uses the rank directly
5. Colleges are filtered where `cutoffRank >= worstCaseRank`, then sorted ascending by cutoff rank
6. Results render as both a table (desktop) and cards (mobile), with a chance badge (Safe/Good/Reach) based on `cutoffRank / userRank` ratio

### Key Algorithms

- **Rank interpolation** (`interpolateRank`): Linear interpolation within marks ranges from historical year tables
- **Rank range** (`getRankRange`): Runs interpolation across all available years, returns best/worst case
- **Chance assessment** (`getChance`): ratio >= 2 = Safe, >= 1.2 = Good, else Reach
- **Fallback formula** (`formulaRank`): `max(1, round(1000000 * (1 - marks/totalMarks)))` when no historical data matches

### Data Schema (cutoffs.json)

```
exams.<ExamName>.totalMarks — max score for the exam
exams.<ExamName>.historicalMarksToRank.<Year>[] — array of {marksMin, marksMax, rankMin, rankMax}
exams.<ExamName>.colleges[] — array of {name, branch, cutoffRank, category}
```

# NBA Prediction Platform

A Python command-line project for exploring NBA player performance, estimating game statistics, and comparing predictions against user-entered over/under lines.

The main application combines a Ridge regression model with a player statistics lookup tool. Separate scripts provide Linear Regression, Lasso, and neural network experiments.

## Features

- Predict points, rebounds, assists, steals, blocks, and three-pointers made.
- Request predictions for multiple statistics in one session.
- Specify home/away status and an opponent.
- Compare predictions with entered lines and report held-out mean absolute error (MAE).
- View recent performance, opponent averages, and home/away averages.
- Look up season averages or averages against a specific opponent.

## Technology

Python, pandas, NumPy, scikit-learn, and Git LFS.

## Getting started

Install Python 3 and Git LFS, then clone the repository:

```bash
git lfs install
git clone https://github.com/jashp1880-hub/NBA-Prediction-Platform.git
cd NBA-Prediction-Platform
git lfs pull
python -m venv .venv
```

Activate the virtual environment:

```bash
# macOS / Linux
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Install the libraries used by the scripts:

```bash
python -m pip install pandas numpy scikit-learn
```

### Dataset requirements

The CSV files are tracked through **Git LFS**. Their small Git entries are pointers to the actual datasets; downloading the repository as a ZIP may leave those pointers instead of usable CSV data.

Run `git lfs pull` and confirm that `UpdatedPlayerStatistics (1).csv` contains CSV headers and game rows, rather than a first line beginning with `version https://git-lfs.github.com/spec/v1`.

The cleaned dataset is required by every prediction script. If the LFS objects are unavailable after import, the original dataset files must be restored to this repository's LFS storage or supplied locally under the same filenames.

## Run the application

Run commands from the repository directory:

```bash
python nba_over_under.py
```

### Mode 1: Predictions

1. Select `1` from the main menu.
2. Enter the player's full name exactly as it appears in the dataset.
3. Choose an opponent from the displayed team list.
4. Enter `h` for home or `a` for away.
5. Select statistic numbers separated by commas, such as `1,2,3`.
6. Enter an over/under line for each selected statistic.

| Choice | Statistic |
| --- | --- |
| 1 | Points |
| 2 | Rebounds |
| 3 | Assists |
| 4 | Steals |
| 5 | Blocks |
| 6 | Three-pointers made |

The output includes an estimated statistic, test-set MAE, recent averages, and an over/under comparison. When the prediction's distance from the line is smaller than MAE, the program reports “too close to call.”

### Mode 2: Statistics lookup

Select `2`, enter a player name, and choose:

- **Season averages:** enter the season's ending year, such as `2026` for 2025–26, or `current` for the latest season present in that player's data.
- **Opponent averages:** enter an opponent and choose the latest season in the dataset or all available career games against that opponent.

Type `quit` at the main menu to exit.

## How the main model works

The Ridge predictor trains a separate model for each requested player and statistic:

1. Sort game history chronologically and retain a four-year window ending at the player's latest recorded game.
2. Build lagged features using 3-, 5-, and 10-game rolling averages, recent trends, counting statistics, per-minute statistics, and home/away status.
3. Split usable games chronologically: 80% for training and 20% for testing.
4. Standardize features and fit Ridge regression with `alpha=50`. Training sample weights increase linearly from 1 to 3 so recent training games have more influence.
5. Calculate MAE on the test games and compare the prediction with the entered line.

Opponent history is shown as supporting context in the main application; the selected opponent is **not an input feature in its Ridge model**. Lasso and MLP experiments encode opponents as features.

## Repository guide

| File | Purpose |
| --- | --- |
| `nba_over_under.py` | Main Ridge predictor and statistics lookup interface |
| `prediction.py` | Linear Regression baseline for points, rebounds, and assists |
| `LassoRegression.py` | Lasso experiment using 20-game and 5-game averages with opponent encoding |
| `MLP.py` | MLP neural network experiment with two hidden layers and opponent encoding |
| `cleaning.py` | Filters raw game data and creates per-minute statistics |
| `PlayerStatistics (1).csv` | Raw dataset, stored with Git LFS |
| `UpdatedPlayerStatistics (1).csv` | Cleaned dataset, stored with Git LFS |
| `.gitattributes` | Git LFS tracking rules |

Run an alternative model with `python prediction.py`, `python LassoRegression.py`, or `python MLP.py`.

To regenerate the cleaned dataset from the raw CSV:

```bash
python cleaning.py
```

The cleaning script retains games from September 2003 onward, excludes preseason and All-Star games, removes zero-minute appearances, retains players with appearances since September 2023, and calculates per-minute statistics. It overwrites the local cleaned CSV.

## Current limitations

- This is a local command-line application; the repository does not include a web interface, live data feed, or saved trained models.
- Results depend on the supplied dataset's coverage and freshness. “Current season” means the latest season in the data.
- Prediction scripts reuse the last historical feature row instead of constructing a fully updated next-game row incorporating the latest game's result.
- The code does not automatically incorporate injuries, lineup changes, or live odds.
- MAE measures average test-set error; it is not a confidence interval or a win probability.
- Dependencies are not version-pinned, and no reproducible benchmark results are included.

## Project history

This personal repository was imported from `ezraant/data-project`, preserving the original project's files and commit history.

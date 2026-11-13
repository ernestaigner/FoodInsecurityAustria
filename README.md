# FoodInsecurityAustria – analysis of food insecurity in Austria (FIES survey 2023)

R Markdown analysis for the article *Food insecurity in high-income countries: a socio-ecological model of the Austrian case* (Global Food Security, first revision RV1). Only code and documentation are in this repository. **Data, renders, tables and figures stay local** (see below).

## Versions

| Tag | Content |
|---|---|
| `submission` | analysis as submitted (June 2026), with the original paths (does not run in this folder structure) |
| `submission-folders` | the same analysis, only the paths adapted to this folder structure (runs) |
| `RV1` | first revision: fixes and additional outputs requested by the reviewers (changelog at the top of the Rmd) |

`git diff submission-folders RV1` shows all changes of the revision, without the path changes.

## Folder structure (local)

The repository is one folder inside `FoodInsecurity/analysis/`; the other folders lie next to it and are not part of git:

```
FoodInsecurity/
├── analysis/
│   ├── FoodInsecurityAustria/   this repository
│   │   ├── FoodInsecurityAustria.Rmd   the analysis
│   │   └── README.md
│   ├── Revision/                manuscript RV1, technical appendix, revision strategy
│   ├── Reviews/  submission/  Infos/
├── data/                 survey data (never in git): data_expl.xlsx; imputation/ (stored imputations)
├── FIES/                 FIES calibration workbooks (template, FIES_<sample>.xlsx, calibrated FIES_<sample>c.xlsx)
├── distances/            walking distances by municipality (ortsliste.xls, walk_car.csv)
├── tables/               manuscript_tables.docx, lund_appendix_all.docx, variable_overview.docx
├── appendix/             additional tables and diagnostics for the technical appendix (rv1_appendix_additions.docx, plots)
├── figures/              figures of the article
├── html/                 renders
└── archive/              old versions and outputs (before the move to git)
```

All paths are set at the top of the setup chunk (`DIR_DATA`, `DIR_FIES`, ...) relative to this repository.

## Rendering (Terminal, in the folder `FoodInsecurityAustria`)

``` bash
# standard render: loads the stored imputations           -> ../../html
Rscript -e 'rmarkdown::render("FoodInsecurityAustria.Rmd", output_dir = "../../html")'

# run the multiple imputation again (3 samples x 40 imputations x 40 iterations)
Rscript -e 'rmarkdown::render("FoodInsecurityAustria.Rmd", params = list(reimpute = TRUE), output_dir = "../../html")'

# the same as a background run: keeps the Mac awake, continues when the terminal is closed
nohup caffeinate -i Rscript -e 'rmarkdown::render("FoodInsecurityAustria.Rmd", params = list(reimpute = TRUE), output_dir = "../../html")' > ../../html/render.log 2>&1 &
tail -f ../../html/render.log
```

Knitting in RStudio uses the defaults (`reimpute = FALSE`, `m_imp = 40`, `maxit_imp = 40`). To render with stored imputations made with other settings, pass them, e.g. `params = list(m_imp = 20, maxit_imp = 10)`.

## Imputation

Missing values in non-FIES items are imputed with `mice` (40 imputations with 40 iterations per sample: full, phone, online; `M_IMP`, `MAXIT_IMP`, chosen with the checks in the appendix: von Hippel 2020, R-hat) and the weights are raked within each imputed dataset. The result is stored in `../../data/imputation/imputations_rv1.rds` and loaded on later renders. Render with `reimpute = TRUE` whenever the data preparation, the imputation model (`imp_vars`), the imputation settings or the raking (`RAKE_VARS`, `RAKE_ON_MODE`) changes. The render stops if model, settings, raking or the number of missing values per imputed variable differ from the stored imputations; other changes in the data preparation are not detected.

## FIES calibration

The FIES calibration includes a manual step in Excel: the Rmd writes `../../FIES/FIES_<sample>.xlsx` from the template; the calibrated workbook `FIES_<sample>c.xlsx` is read back. Re-calibrate in Excel when the FIES data or weights change.

## Data protection

The `.gitignore` excludes all data and output files (`*.xlsx`, `*.xls`, `*.csv`, `*.sav`, `*.dta`, `*.rds`, `*.RData`, renders, tables, figures, knitr caches). Never add data to this repository; check `git status` before committing.

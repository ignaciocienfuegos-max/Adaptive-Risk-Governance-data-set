[README.md](https://github.com/user-attachments/files/32440481/README.md)
# Replication package — Plans Without Practice: Adaptive Risk Governance and the Normative–Organizational Gap in Metropolitan Santiago

Ignacio Cienfuegos, Facultad de Gobierno, Universidad de Chile (ORCID 0000-0003-0784-6132)
Correspondence: cienfuegos.ignacio@gmail.com

This package contains the raw source data, cleaned/merged analysis data, and
analysis code needed to reproduce every quantitative result reported in the
manuscript: the commune-level normative/organizational capacity indices for
the Región Metropolitana de Santiago (RM, n = 52 communes), the Firth
penalized-likelihood logistic regressions, the ordinary-logit separation
check, the fuzzy-set QCA (fsQCA), and the two figures.

Funding: Chile's National Agency for Research and Development (ANID),
FONDECYT INICIACIÓN Grant No. 11240400.

## Directory structure

```
data/
  raw/          six source spreadsheets + one .rds file, as obtained from
                their original providers (SINIM, ACHM, SERVEL, and the
                municipal DRM-plan status registry) — unmodified
  processed/    cleaned, merged, RM-only analysis files produced by
                code/01-02, ready to be read directly by code/03-06
code/           numbered scripts; run in order from inside code/
outputs/        firth_results.xlsx and the two manuscript figures, as
                produced by the scripts on this data
```

## How to reproduce

```
pip install -r requirements.txt
cd code
python3 01_build_normative_2023.py
python3 02_build_organizational_and_merge_2022.py
python3 02b_code_achm_gore_2024.py
python3 03_firth_logit.py
python3 04_logit_robustness.py
python3 05_fsqca.py
python3 06_make_figures.py
```

Steps 01–02b rebuild `data/processed/*.csv` from `data/raw/*`; steps 03–06
consume those processed files and rewrite `outputs/firth_results.xlsx` and
`outputs/figures/*.png`. The processed CSVs and outputs already shipped in
this package are the direct, unedited output of this exact pipeline — you do
not need to run 01–02b to reproduce the models and figures in 03–06; they are
included only so the full raw-to-results chain is auditable and independently
reproducible from the original spreadsheets.

Tested with Python 3.11; package versions are pinned as minimums in
`requirements.txt`, not exact versions — pin exact versions yourself if you
need bit-for-bit reproduction of floating-point output.

## Data provenance (data/raw/)

- `datos_municipales_2022.xlsx`, `datos_municipales_2023.xlsx` — municipal
  management-instrument indicators (participatory budget status, community
  council constitution, PLADECO/PLARECO status, transparency index, FCM
  dependency, per-capita municipal income), by commune and year.
- `datos_achm.xlsx` (sheet "Todas las Comunas") — Asociación Chilena de
  Municipalidades (ACHM) national survey on municipal disaster-risk-management
  organization: whether the municipality has a dedicated DRM unit, whether
  that unit has a named head/director, plus the national risk ranking and
  FIGEM hazard-typology group.
- `elec_munic_2021.xlsx` — SERVEL 2021 municipal-election results (registered
  voters and votes cast, by commune), used to compute voter turnout as a
  civic-engagement proxy (`trans_activa` in the processed files — labeled
  from the transparency-index column; turnout is `participacion`).
- `planes_comunales_emergencia.xlsx`, `planes_comunales_rdd.xlsx` — national
  registry of Municipal Emergency Plan and Disaster Risk Reduction (RDD) Plan
  formalization status, by commune and year; the analysis uses the 2024
  formalization status.
- `sinim_data.rds` — Sistema Nacional de Información Municipal (SINIM) panel:
  commune population and the CASEN-derived poverty rate, plus the count of
  professional (non-administrative) municipal staff, by commune and year.

`code/02b_code_achm_gore_2024.py` adds one further variable,
`achm_gore_2024`, that is *not* derived from any raw file in this package: it
is hand-coded from public ACHM press materials (April 2024) naming 19 RM
communes that received GORE Metropolitano-funded technical assistance to
produce formal RDD/emergency plans in 2024. It is used only as a
supplementary/robustness condition in the fsQCA (the ASSIST condition,
`code/05_fsqca.py`) and does not enter the paper's core Firth-logit models.
See the script's docstring for the full source note and the commune list.

## Codebook — data/processed/rm_merged_normative.csv and rm_merged_organizational.csv

These are the two analysis-ready files (normative outcomes measured 2023–24;
organizational outcomes measured 2023), each restricted to the RM's 52
communes and carrying the shared covariate and hazard-exposure set.

| Variable | Description |
|---|---|
| `comuna` | Commune name (ASCII, uppercase, accents stripped) |
| `pob_comunal` | Commune population (SINIM) |
| `log_pob_comunal` | log(pob_comunal) |
| `pobreza` | CASEN poverty rate (%, SINIM) |
| `profesionales` | Professional municipal staff (SINIM) |
| `plan_emergencia` | 1 = Municipal Emergency Plan formalized (2024) |
| `plan_rdd` | 1 = Disaster Risk Reduction (RDD) Plan formalized (2024) |
| `indice_planes` | `plan_emergencia + plan_rdd` (0–2 normative capacity index) |
| `unidad_grd` | 1 = municipality has a dedicated DRM unit (ACHM survey, 2023) |
| `director_grd` | 1 = that unit has a named head/director (ACHM survey, 2023) |
| `indice_unidad` | `unidad_grd + director_grd` (0–2 organizational capacity index) |
| `tiene_pladeco` | 1 = has a current PLADECO (communal development plan) |
| `tiene_plareco` | 1 = has a current PLARECO (communal land-use regulatory plan) — the PRC condition in the fsQCA |
| `cosoc_constituido` | 1 = municipality has a constituted civil-society council (COSOC) |
| `ord_pc` | 1 = has a participatory-budget ordinance |
| `dependencia_fcm` | Common Municipal Fund (FCM) dependency, % of municipal revenue |
| `ipp_per_capita` | Own permanent income (IPP) per capita |
| `pob_rural` | % of commune population classified rural |
| `comuna_rural` | 1 = `pob_rural` > 60% |
| `trans_activa` | Active-transparency compliance index |
| `participacion` | 2021 municipal-election voter turnout, % (votes / registered voters) — the CIVIC condition in the fsQCA |
| `risk_ranking`, `figem_group` | ACHM national disaster-risk ranking and FIGEM hazard-typology group |
| `san_ramon_fault_exposure` | 1 = commune is one of the six the ERD-RM 2024–2035 lists as exposed to the San Ramón fault (Vitacura, Las Condes, La Reina, Peñalolén, La Florida, Puente Alto) — the main hazard-exposure predictor |
| `piedemonte_flood_risk` | 1 = commune is in the ERD-RM's overlapping six-commune *piedemonte* flood list (Lo Barnechea in place of Vitacura) — used in the §9 sensitivity analysis |
| `poor_seismic_soil`, `volcanic_zone`, `prc_gap_named` | Additional ERD-RM-named hazard/planning-gap flags, used only in supplementary checks |
| `achm_gore_2024` | 1 = commune received 2024 ACHM/GORE Metropolitano technical assistance (see provenance note above) — the supplementary ASSIST condition |

`n_oocc` (org. chart code, unused) is carried over from the source files and
not used in any reported model.

## Code

- `01_build_normative_2023.py` — builds `rm_2023_normative.csv`: merges SINIM
  2023, the 2024 plan-formalization registry, `datos_municipales_2023.xlsx`,
  and 2021 turnout, restricted to the RM.
- `02_build_organizational_and_merge_2022.py` — builds
  `rm_2022_organizational.csv` from SINIM 2022 + the ACHM survey +
  `datos_municipales_2022.xlsx` + turnout; codes the five hazard/planning-gap
  dummies from the ERD-RM narrative; and produces the two merged files
  (`rm_merged_normative.csv`, `rm_merged_organizational.csv`) used by every
  downstream script.
- `02b_code_achm_gore_2024.py` — adds the `achm_gore_2024` supplementary
  variable (see provenance note above).
- `03_firth_logit.py` — Firth (1993) penalized-likelihood logistic
  regressions of `plan_emergencia`, `plan_rdd`, `unidad_grd`, and
  `director_grd` on `san_ramon_fault_exposure + tiene_plareco +
  log_pob_comunal`, with penalized likelihood-ratio tests and profile
  penalized-likelihood 95% CIs (Heinze & Schemper 2002). A self-contained
  implementation (no external Firth-logit package dependency) — see the
  module docstring for the estimating equations. Writes
  `outputs/firth_results.xlsx`.
- `04_logit_robustness.py` — the same four models fit by ordinary maximum
  likelihood (`statsmodels`), reported to document that
  `unidad_grd ~ san_ramon_fault_exposure + ...` fails to converge under
  ordinary ML (quasi-complete separation: no fault-exposed commune has a DRM
  unit), which is the paper's stated motivation for the Firth estimator.
- `05_fsqca.py` — fuzzy-set QCA: calibrates the PRC, POP (population), CIVIC
  (turnout), HAZARD, and (supplementary) ASSIST conditions, builds the truth
  tables, derives the complex/parsimonious solutions via Quine–McCluskey
  (`sympy.logic.boolalg.SOPform`), reports solution consistency/coverage, and
  places the six hazard-exposed communes in the core PRC × POP × CIVIC
  configurational space for both outcomes.
- `06_make_figures.py` — the two manuscript figures (grouped-bar decoupling
  chart; scatter/crosswalk of all 52 communes), written to
  `outputs/figures/`.

## License

Code: MIT License (see `LICENSE-CODE.txt`).
Data (data/ and outputs/): Creative Commons Attribution 4.0 International
(CC BY 4.0) (see `LICENSE-DATA.txt`), consistent with the source agencies'
own public-data licensing (SINIM, SERVEL, ACHM).

## Citation

If you use this data or code, please cite the manuscript this package
accompanies (full citation to be added on acceptance/publication) and, where
applicable, the original data providers: SINIM (Subsecretaría de Desarrollo
Regional y Administrativo, SUBDERE), SERVEL, and ACHM.

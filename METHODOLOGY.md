# Comparing Dry and Humid Heat Risk in U.S. Southwest and Southeast Cities
### Heatwaves, Hot Nights, Population Exposure, and Mortality
*Living methodology / paper draft. Updated as decisions are locked.*

## 1. Research question
Do dry-heat cities (Southwest) and humid-heat cities (Southeast) face different future
changes in extreme-heat exposure and health risk, when heat is measured against TODAY'S
extreme thresholds? Three risk channels: daytime dry heatwaves, humid-heat events, and
nighttime recovery risk (hot nights).

## 2. Study cities and matched model gridcells
| City | Region | Heat type | City lat/lon | Matched grid (lat/lon 0-360) | Land fraction |
|---|---|---|---|---|---|
| Phoenix, AZ | Southwest | Dry | 33.4484 / 247.926 | 33.19 / 247.50 | 100% |
| Tucson, AZ | Southwest | Dry | 32.2226 / 249.025 | 32.26 / 249.38 | 100% |
| Charlotte, NC | Southeast | Humid | 35.2271 / 279.157 | 35.06 / 279.38 | 100% |
| Miami, FL | Southeast | Humid | 25.7617 / 279.808 | 25.71 / 279.38 | 100% |

Extraction: nearest gridcell (xarray `.sel(method="nearest")`, equivalent to the `geo_idx`
helper in hwconus). Land fraction verified with the fixed `sftlf` field: all four cells are
100% land, so no ocean contamination. NOTE: Miami's matched cell sits ~0.4 deg inland
(hence 100% land); this avoids sea-surface bias but may slightly under-represent coastal
maritime moderation. State this in the methods.

## 3. Definitions - fixed historical thresholds
Thresholds are computed on warm-season (May-September) daily data over the baseline, then
applied UNCHANGED to both baseline and future. Rationale: the goal is public-health exposure
to today's-extreme heat, not anomalies relative to each future year's warmer climate. A
moving threshold would rise with warming and mask the exposure increase.

| Metric | Variable | Threshold | Event definition |
|---|---|---|---|
| Dry daytime heatwave | tasmax | TX95 = 95th pct of May-Sep daily max | tasmax > TX95 for >= 3 consecutive days |
| Hot nights | tasmin | TN95 = 95th pct of May-Sep daily min | tasmin > TN95; also 3+ consecutive events |
| Humid heatwave (SE) | heat index / apparent temp | HI95 = 95th pct May-Sep | HI > HI95 for >= 3 consecutive days |

Consecutive runs are detected WITHIN each warm season, so 30 Sep never links to the
following 1 May. Humid-heat metric to be computed from tasmax + hurs (metpy / xclim).

Secondary comparison: Comeau et al. use a non-stationary TX90 via quantile regression.
We report this only as a robustness contrast, to show how much the "exposure to today's
extremes" framing changes the signal versus an anomaly framing.

## 4. Model choice - MPI-ESM1-2-HR (locked)
Checked against ILAMB CMIP6 historical, DiurnalMaxTemperature vs CRU TS4.02.
Global-land Overall Score: GFDL-ESM4 0.901, MPI-ESM1-2-HR 0.900, CESM2 0.883,
IPSL-CM6A-LR 0.881, CanESM5 0.877, ACCESS-ESM1-5 0.874, BCC-CSM2-MR 0.866,
UKESM1-0-LL 0.865, MIROC-ESM2L 0.838, MeanCMIP6 0.838.

MPI-ESM1-2-HR is top-2 globally and was preferred after inspecting regional tasmax/tasmin
behaviour, where GFDL-ESM4 performed less well for our cities. Resolution ~100 km.
NOTE: ILAMB scores diurnal-max climatology, not extreme-percentile skill directly - treat
as supporting evidence, not proof of tail skill.
ESGF spells it MPI-ESM1-2-HR (dashes); ILAMB writes MPI-ESM1.2-HR. Same model.

## 5. Data source and exact paths (NERSC CMIP6 mirror)
Root: /global/cfs/cdirs/m3522/cmip6/CMIP6
DRS: CMIP6/activity/institution/source/experiment/member/table/variable/grid/version

IMPORTANT QUIRK: the historical run is published by MPI-M, but the SSP scenario runs are
published by DKRZ. Two different institution folders for the same model:
- historical : CMIP/MPI-M/MPI-ESM1-2-HR/historical/r1i1p1f1/day/<var>/gn/*/*.nc
- ssp370     : ScenarioMIP/DKRZ/MPI-ESM1-2-HR/ssp370/r1i1p1f1/day/<var>/gn/*/*.nc

Provenance verified by reading NetCDF global attributes (not just paths):
mip_era=CMIP6, activity_id=CMIP / ScenarioMIP, source_id=MPI-ESM1-2-HR,
variant_label=r1i1p1f1, frequency=day, grid_label=gn, nominal_resolution=100 km, units=K.

Availability on the mirror (r1i1p1f1):
- tasmax, tasmin: COMPLETE for historical (33 files) and ssp370 (18 files).
- hurs ssp370: COMPLETE (18 files).
- hurs historical: INCOMPLETE - only 1910-14, 1955-59, 1995-99. Gap 2000-2014.
  Checked all 10 members: none have a complete historical hurs record on the mirror.
  FIX: download the missing 2000-2014 historical hurs via intake-esgf (caches to ~/.esgf).
  This affects the humid-heat metric only; the dry-heat and hot-night metrics are unaffected.

Ensemble: all 10 members (r1-r10 i1p1f1) have COMPLETE daily tasmax and tasmin for both
historical and ssp370. Single-member (r1i1p1f1) results are reported first; the 10-member
ensemble should be run to establish robustness.

## 6. Periods - LOCKED
- Baseline 1996-2026, SPLICED because CMIP6 historical ends 2014-12-31:
  1996-2014 from historical, 2015-2026 from ssp370.
- Future 2027-2057, entirely ssp370.
- Scenario: SSP3-7.0 (ssp370).
Caveat to state: the 2015-2026 part of the baseline is the model's scenario projection,
not observations - standard for a model-consistent baseline, but note it.
Second caveat: the baseline is not internally stationary (it warms across 1996-2026), so
early baseline years show few exceedances and later years many. The 31-year mean is ~7.65
by construction. This is expected, not an error.

## 7. Verification performed
- Provenance read from file global attributes, confirming CMIP6 / correct model, experiment,
  member, frequency, units.
- Time axis continuity: all day-steps == 1 (no gaps, no overlap at the 2014/2015 splice).
- Coverage: 1996-2057, 62 distinct years, 22646 days, leap years present (Gregorian calendar).
- Land fraction via sftlf: 100% for all four cities.
- Threshold arithmetic: baseline hot_days = hot_nights = 7.7 per season for every city,
  i.e. 5% of the 153 May-Sep days, exactly as a 95th percentile requires.

## 8. Results so far (single member r1i1p1f1)
Baseline warm-season thresholds (degC):
| City | TX95 | TN95 | Diurnal range |
|---|---|---|---|
| Phoenix | 43.73 | 31.11 | 12.6 |
| Tucson | 40.73 | 27.47 | 13.3 |
| Charlotte | 31.90 | 23.66 | 8.2 |
| Miami | 34.89 | 28.61 | 6.3 |

The diurnal range reproduces the expected dry-vs-humid signature: dry desert air radiates
heat at night (12-13 degC swing) while humid air traps it (Miami only 6.3 degC).

Mean per warm season, baseline -> future:
| City | hot_days | hot_nights | hw_events | hw_days | hn_events | hn_days |
|---|---|---|---|---|---|---|
| Charlotte baseline | 7.7 | 7.7 | 1.1 | 4.4 | 0.8 | 2.7 |
| Charlotte future | 34.2 | 37.3 | 4.5 | 27.4 | 4.2 | 30.2 |
| Miami baseline | 7.7 | 7.7 | 1.0 | 4.1 | 0.6 | 2.9 |
| Miami future | 19.3 | 28.6 | 2.2 | 13.0 | 2.6 | 22.5 |
| Phoenix baseline | 7.7 | 7.7 | 1.3 | 5.3 | 0.9 | 3.7 |
| Phoenix future | 17.1 | 18.0 | 2.7 | 12.8 | 2.4 | 13.5 |
| Tucson baseline | 7.7 | 7.7 | 1.4 | 5.8 | 0.8 | 3.0 |
| Tucson future | 15.8 | 17.3 | 2.6 | 13.3 | 2.4 | 12.9 |

Findings:
1. The Southeast increases far more than the Southwest in RELATIVE terms. Hot days:
   Charlotte 4.4x, Miami 2.5x, Phoenix 2.2x, Tucson 2.1x. Likely mechanism: humid cities
   have a narrower temperature distribution, so the same warming shift pushes a much larger
   fraction of days past a fixed cutoff.
2. Hot nights outpace hot days everywhere, and most strongly in the humid Southeast
   (Miami 28.6 nights vs 19.3 days; Phoenix nearly symmetric at 18.0 vs 17.1). This is the
   nighttime-recovery-risk channel, concentrated where hypothesised.
3. Events lengthen as well as multiply. Mean heatwave duration rises from ~4.0 to ~6.1 days
   in Charlotte. Hot-night events go from ~4.8 to ~8.7 consecutive nights in Miami and 3.4
   to 7.2 in Charlotte. Multi-day stretches without night-time relief are the strongest
   driver of heat mortality, so this is likely the most health-relevant result.

CRITICAL CAVEAT for the write-up: these are exceedances of each city's OWN local threshold,
not absolute heat. Charlotte gets 34 days above 31.9 degC; Phoenix gets 17 days above
43.7 degC. Phoenix remains far hotter in absolute terms. The Southeast result means "many
more days above what that city currently considers extreme", which is the correct framing
for public health (populations acclimatise and build for local norms), but it must be
stated explicitly or readers will conclude Charlotte becomes hotter than Phoenix.

## 9. Later stages - population, demography, mortality
None of this is on NERSC; all are external downloads into data/.
- Current + historical demography and population change: US Census Bureau decennial
  (1990/2000/2010/2020) and ACS, at county and tract level, for Maricopa, Pima,
  Mecklenburg and Miami-Dade counties. Age structure (esp. 65+), race/ethnicity, income.
- Future population 2027-2057 consistent with the climate scenario: use SSP3 population
  projections to match SSP3-7.0. Hauer (2019, Scientific Data) US county-level by SSP;
  Gao (2020, NCAR/SEDAC) 1-km gridded SSP population for the US.
- Vulnerability: CDC/ATSDR Social Vulnerability Index; CDC Environmental Justice Index.
- Intra-urban heat equity: UESI (uesi.datadrivenlab.org).
- Heat mortality: CDC WONDER (ICD-10 X30) for the historical signal, plus published
  exposure-response functions (e.g. MCC network) to project attributable deaths onto
  future heat and future population.
Key references: Chakraborty et al. 2023 One Earth (residential segregation and urban moist
heat stress); Hsu et al. 2019 ERL 14 105003 (UHI and neighbourhood income, 25 cities).

## 10. Reference papers
- PLOS Climate 10.1371/journal.pclm.0000468
- PLOS Climate 10.1371/journal.pclm.0000610
- Science of the Total Environment S0048969725000312
- S2212094725000891

## 11. Open decisions
- [x] Model: MPI-ESM1-2-HR, member r1i1p1f1
- [x] Periods: baseline 1996-2026 (spliced), future 2027-2057
- [x] Scenario: SSP3-7.0
- [x] Gridcell selection: nearest, land verified via sftlf
- [ ] Run the 10-member ensemble for tasmax/tasmin robustness
- [ ] Humid-heat metric: NWS heat index vs apparent temperature (Steadman)
- [ ] Complete the historical hurs download (2000-2014) and compute HI95
- [ ] Sensitivity: 2x2 gridcell average vs single nearest cell

## 12. Reproducing this
1. conda env create -f environment.yml && conda activate heatwave
2. python -m ipykernel install --user --name heatwave
3. Run check_data.ipynb to verify mirror availability and provenance.
4. Run 02_heatwave_analysis.ipynb top to bottom; outputs land in data/.

# Breast Screening, Deprivation, Ethnicity and Access in Greater Manchester

This project looks at why breast screening rates differ between GP practices in Greater Manchester. It tests three things at the practice level:

1. Deprivation
2. The ethnic make-up of the local population
3. Distance to a fixed breast screening site

It also checks whether the results hold once you account for location, using spatial regression.

All the analysis is in one Jupyter notebook, `Cancer_GIS.ipynb`.

## Research question

Are deprivation, ethnic composition and distance to screening linked to breast screening uptake and coverage across GP practices in Greater Manchester? And do these links hold once spatial patterns are taken into account?

## Study area and data year

- **Area:** the 10 boroughs of Greater Manchester (Bolton, Bury, Manchester, Oldham, Rochdale, Salford, Stockport, Tameside, Trafford and Wigan).
- **Unit of analysis:** GP practices. There are 406 practices in the data.
- **Screening data:** 2024/25.
- **Deprivation data:** IMD 2025.

## Outcomes

The project uses two breast screening measures from Fingertips:

- **Uptake (indicator 91340):** women aged 50 to 70 who were screened after being invited.
- **Coverage (indicator 94063):** eligible women aged 53 to 70 who have been screened recently.

See the Fingertips indicator definitions for the exact time windows.

## Data sources

All data is publicly available. Each source is listed below with where it came from and its licence. The files are in the `data/` folder.

| File or folder in `data/` | What it is | Source | Licence |
|---|---|---|---|
| `fingertips_gp_screening_data/` | GP practice level cancer indicators. Used for breast screening uptake (91340), coverage (94063) and the practice deprivation score (94240, IMD 2025). | Office for Health Improvement and Disparities (OHID), [Fingertips Cancer Services profile](https://fingertips.phe.org.uk/profile/cancerservices) | Open Government Licence v3.0 |
| `E92000001.addresses.csv` | GP practice names, addresses and postcodes for England. | OHID Fingertips practice address download | Open Government Licence v3.0 |
| `gp-reg-pat-prac-lsoa-female.csv` | Number of female patients registered at each GP practice, by the LSOA they live in. | NHS England, [Patients Registered at a GP Practice](https://digital.nhs.uk/data-and-information/publications/statistical/patients-registered-at-a-gp-practice), October 2026 release | Open Government Licence v3.0 |
| `File_5_IoD2025_Scores_for_the_Indices_of_Deprivation.xlsx` | IMD 2025 scores for each LSOA. Used for the deprivation map. | Ministry of Housing, Communities and Local Government, [English Indices of Deprivation 2025](https://www.gov.uk/government/statistics/english-indices-of-deprivation-2025) | Open Government Licence v3.0 |
| `File_1_IoD2025_Index_of_Multiple_Deprivation.xlsx` | IMD 2025 ranks and deciles for each LSOA. Downloaded but not used in the current notebook. | Same as above | Open Government Licence v3.0 |
| `census2021-ts021-lsoa.csv` | Census 2021 ethnic group by LSOA (table TS021). | Office for National Statistics, [Census 2021 bulk data via Nomis](https://www.nomisweb.co.uk/sources/census_2021_bulk) | Open Government Licence v3.0 |
| `PostCodes/` | ONS Postcode Directory (ONSPD), February 2026. Gives a grid reference and an LSOA for every postcode. Only the BL, M, OL, SK, WA and WN files are used. | Office for National Statistics, [ONSPD February 2026](https://ckan.publishing.service.gov.uk/dataset/ons-postcode-directory-february-2026-for-the-uk) | Open Government Licence v3.0, plus the attribution statements below |
| `aac0602f-d4b1-40b2-b82f-fce8b447ba0c.gdb` | LSOA 2021 boundaries for England and Wales. | ONS [Open Geography Portal](https://geoportal.statistics.gov.uk/) | Open Government Licence v3.0 |
| `f0c9921b-27ba-4550-aba8-3200c91a62f9.gdb` | LSOA 2021 population weighted centroids. | ONS [Open Geography Portal](https://geoportal.statistics.gov.uk/) | Open Government Licence v3.0 |
| `gm_confirmed_fixed_sites.csv` | The 8 fixed breast screening sites used by the four services that cover Greater Manchester, with their postcodes. | Compiled by me from the screening services' websites. | n/a |

### Attribution

Contains public sector information licensed under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).

The ONS Postcode Directory also needs the following statements:

- Source: Office for National Statistics licensed under the Open Government Licence v.3.0
- Contains OS data © Crown copyright and database right 2026
- Contains Royal Mail data © Royal Mail copyright and database right 2026
- Contains GeoPlace data © Local Government Information House Limited copyright and database right 2026

### Screening service areas

Each GP practice was linked to a breast screening service using its sub-ICB code. These are the former CCG codes. The service areas come from the NHS Breast Screening Programme quality assurance reports for each service. The sub-ICB codes come from the [NHS SBS CCG to ICB list](https://www.sbs.nhs.uk/supplier-information/ccg-icb-list).

| Service | Areas covered | Sub-ICB codes |
|---|---|---|
| Greater Manchester Breast Screening Service (GMBSS) | Manchester, Oldham, Salford, Tameside, Trafford | 14L, 00Y, 01G, 01Y, 02A |
| Bolton Breast Screening Service | Bolton, Bury, Rochdale | 00T, 00V, 01D |
| East Cheshire and Stockport Breast Screening Service | Stockport (and East Cheshire) | 01W |
| South Lancashire Breast Screening Service | Wigan (and parts of Lancashire) | 02H |

## Method

### 1. Practice data

I downloaded the Fingertips indicators and kept the latest year for each one. I then reshaped the data so each GP practice has one row. Fifteen practices have no uptake value and one has no deprivation score. One practice (Y02625) has a coverage of about 25%. This is far below every other practice, so it is treated as an outlier and left out of the models.

### 2. Practice locations

Each practice postcode was matched to the ONS Postcode Directory to get its location. All 406 practices matched.

### 3. Distance to screening

I measured the straight line distance from the population weighted centroid of every Greater Manchester LSOA to the nearest fixed site of each screening service.

Women are invited for screening through their GP practice. So each practice was given the distance to the sites of its own screening service, not just the nearest site.

I then used the GP registration data to work out a patient weighted average distance for each practice. Each LSOA counts in proportion to how many of the practice's female patients live there. Patients living outside Greater Manchester (about 1% overall) were left out.

### 4. Ethnic composition

I calculated the percentage of residents who are not White British in each LSOA from Census 2021. I then turned this into a practice level value using the same patient weights as the distance measure.

### 5. Regression models

I fitted linear regression (OLS) models for uptake and coverage. I started with deprivation alone, then added distance and ethnicity. The models use robust standard errors (HC3) because the spread of the data is uneven.

### 6. Robustness checks

I tested whether the results change when:

- the outlier is put back in
- border practices with fewer than 80% of patients in Greater Manchester are removed
- small practices are removed
- Stockport practices are removed, because their distances are measured to a site outside Greater Manchester (Macclesfield)
- the screening service is added to the model (service fixed effects)

### 7. Spatial analysis

I checked whether the model errors cluster on the map using Moran's I, with the 8 nearest practices as neighbours. Practices that share a postcode were moved by up to 1 metre so the nearest neighbours could be worked out.

Lagrange Multiplier tests were used to choose between a spatial lag and a spatial error model. They pointed to a spatial error model. I then fitted that model for both outcomes.

## Key findings

1. **Deprivation.** Practices in more deprived areas have lower uptake and coverage. Deprivation on its own explains about 27% of the difference in uptake between practices and about 52% of the difference in coverage.
2. **Ethnicity.** Practices with a higher share of residents who are not White British also have lower uptake and coverage. This holds even after allowing for deprivation. Together, the two explain more than either one alone.
3. **Distance.** Distance only showed up as important once ethnicity was in the model. Practices with more ethnically diverse populations tend to be closer to screening sites, and this hid the effect of distance. Once that is accounted for, each extra kilometre is linked to about 0.3 percentage points lower uptake and coverage. This result held when Stockport was removed and when comparing practices within the same screening service.
4. **Location.** The model errors were clustered on the map, more strongly for coverage. A spatial error model removed this clustering and fitted the data better. All three effects stayed significant in this model.

Main results from the spatial error model (change in percentage points):

| Variable | Uptake (n = 391) | Coverage (n = 404) |
|---|---|---|
| Deprivation score (per point) | -0.31 | -0.40 |
| % not White British (per point) | -0.15 | -0.17 |
| Distance to screening (per km) | -0.31 | -0.31 |
| Spatial error parameter (lambda) | 0.26 | 0.48 |

## Limitations

- **Practice level data.** The results describe GP practices, not individual women. They should not be read as describing how any one person behaves.
- **Fixed sites only.** Many women are screened at mobile units. These are not included in the distance measure.
- **Straight line distance.** This does not reflect real travel time or public transport.
- **Population counts.** The GP registration and Census data cover all ages, not just women aged 50 to 70.
- **Associations, not causes.** The models show links between variables. They do not prove that one causes another.
- **Ethnicity as a marker.** Ethnic composition is likely standing in for other things, such as language, health literacy or how well services meet different needs.

## Future work

- Add English language proficiency to see how much of the ethnicity effect it explains.
- Add car availability to see if distance matters more where fewer households have a car.
- Use the separate IMD domains instead of the overall score.
- Add mobile screening unit locations.
- Use travel time instead of straight line distance.
- Test a Spatial Durbin Error Model, to see if neighbouring practices' characteristics matter.

## Repository structure

```
├── Cancer_GIS.ipynb     # the full analysis
├── data/                # all input data (see Data sources)
├── outputs/             # figures and processed tables made by the notebook
└── README.md
```

## How to run

1. Clone this repository.
2. Create a Python environment with these packages:
   `pandas`, `numpy`, `matplotlib`, `geopandas`, `statsmodels`, `openpyxl`, `libpysal`, `esda`, `spreg`
3. Open `Cancer_GIS.ipynb` in JupyterLab.
4. Run all cells from top to bottom. The notebook reads from `data/` and writes to `outputs/`.

## Author

Rahul Madhukumar
MSc Geographical Information Science, University of Manchester

- Portfolio: [rhm-1697.github.io/rahul-portfolio](https://rhm-1697.github.io/rahul-portfolio/)
- GitHub: [github.com/rhm-1697](https://github.com/rhm-1697)
- LinkedIn: [linkedin.com/in/rahul-madhukumar-389945190](https://www.linkedin.com/in/rahul-madhukumar-389945190)

# HAI Dashboard: US State, County, and Hospital Performance, 2025
An interactive Tableau dashboard exploring HAI performance across the US, with drill-down from state to county to individual hospital. Built from public CMS data with a reproducible R pipeline.
Live dashboard: https://public.tableau.com/app/profile/evan.heffelfinger/viz/HAIDashboard_17954062271410/HAIDashboard#2
# What it shows
For one infection at a time (CLABSI, CAUTI, C. diff, MRSA, SSI-colon surgery, SSI-abdominal hysterectomy), the dashboard classifies each state, county, and hospital as better than expected, no clear difference, or worse than expected.
Data

All data are public and were downloaded on 11/22/2026.

File	Source	Use
Healthcare Associated Infections - Hospital	CMS Provider Data Catalog (data.cms.gov/provider-data)	Observed infections, predicted infections, SIRs, and CMS comparison category per hospital. Reporting period: 01/01/2025 to 12/31/2025
Hospital General Information	CMS Provider Data Catalog	Hospital type, ownership, and overall rating
HAI data dictionary	CMS	Field definitions
2024 National and State HAI Progress Report (SIR data)	CDC	Used only as a rough rank-order cross-check; it covers a different year and likely a different baseline, so values are not directly comparable

Raw files are in data/raw/. [Note any files too large to include and how to download them.]

Methods
Standardized infection ratio (SIR)

SIR = observed infections / predicted infections, where predicted infections come from a national baseline model adjusted for facility characteristics. An SIR below 1 means fewer infections than predicted from that baseline, and above 1 means more.

Pooling to state and county

CMS reports SIRs at the hospital level only. State and county figures are built by summing, not averaging:

pooled SIR = sum(observed infections) / sum(predicted infections)

Averaging hospital SIRs would give a small hospital with a noisy ratio the same weight as a large one.

CMS does not report an SIR when a hospital's predicted infections are below 1.0, but observed and predicted counts are still in the file. This pipeline uses those counts, so low-volume hospitals contribute to pooled estimates even without a hospital-level SIR. The share of predicted infections coming from hospitals without a CMS SIR ranges from about 1% (C. difficile) to about 32% (abdominal hysterectomy SSI).

Confidence intervals and classification

Exact Poisson (Garwood) 95% confidence intervals are computed for each pooled SIR:

lower = qchisq(0.025, 2 * observed) / 2 / predicted        (0 if observed = 0)
upper = qchisq(0.975, 2 * (observed + 1)) / 2 / predicted

Each place is classified by its interval, not by the raw SIR:

Rule	Category
upper limit < 1	Better than expected
lower limit > 1	Worse than expected
otherwise	No clear difference

Why: many counties have fewer than one predicted infection, so a raw SIR can swing from 0 to 3 on a single case. Coloring by raw SIR would show tiny places as best or worst mostly by chance. Classifying by interval avoids claiming more than the data support.

Hospital-level classification uses CMS's own "Compared to National" category.

Other processing decisions
Pooled groups with zero predicted infections are dropped (SIR undefined).
U.S. territories (AS, GU, MP, PR, VI) are excluded from state and county pooling.
County keys are built from CMS county names plus state (for example, "Lehigh, PA"). Tableau geocodes county names, and some (independent cities, parishes, boroughs) may need manual correction. Check [list any known unresolved counties].
Hospital locations are mapped by [ZIP code centroid / geocoded street address (Census geocoder, X of 4,787 matched)].
All reporting hospital types (acute care, critical access, children's, VA, psychiatric, and others) are included. Hospital type is a column in the data.
Repository structure
├── data/
│   ├── raw/            CMS and CDC source files
│   └── clean/          hospital_level.csv, state_pooled.csv, county_pooled.csv, dashboard_data.csv
├── 01_clean_hai.R      cleaning, pooling, confidence intervals, classification
├── dashboard_screenshot.png
└── README.md
How to reproduce
Download the CMS files listed above into data/raw/ (update the file paths at the top of the script).
Install R packages: readr, dplyr, tidyr, stringr [and tidygeocoder if geocoding].
Run 01_clean_hai.R. It writes the four CSVs to data/clean/ and prints QA checks (row counts, share of predicted infections without a CMS SIR, national pooled SIRs).
Open Tableau and connect to dashboard_data.csv.
Observations (calendar year 2025, descriptive only)
Infection type differs more than geography. For CLABSI, CAUTI, C. difficile, and MRSA bacteremia, most states were better than expected; SSIs, especially after abdominal hysterectomy, were the exception. Because SIRs are measured against a fixed national baseline, this likely reflects national change since the baseline period, not state-level quality.
Regional differences are modest and a state that does well on one infection often does not on others.
Critical access hospitals had higher pooled SIRs than acute care hospitals for CLABSI, CAUTI, and both SSIs. This was not formally tested.
Most counties are classified "no clear difference," which reflects small expected counts, not equal performance.
Limitations
Single period. Calendar year 2025 only, so there is no trend analysis.
SIR comparisons are not strictly like-for-like. SIRs are indirectly standardized to a national baseline, so comparing places is informative but not exact.
Location is the hospital's, not the patient's. County and state reflect where the hospital is, not where patients live.
Interval-based classification favors large places. Large counties get flagged for small deviations, while small counties rarely do, so the map partly reflects sample size. With about 2,000 counties per infection and no multiple-comparison correction, some green and red counties will be chance findings.
"No clear difference" does not mean no difference. It means the data can't distinguish performance from the baseline.
Statistical significance is not clinical importance.
Footnote codes are carried through but not decoded. [Add decoded values if you download the CMS footnote crosswalk.]
Reporting and participation vary across hospitals, and critical access hospitals may differ in case mix and reporting.
Not for ranking or accountability decisions. This is an exploratory project, not an official performance measure.
Tools

R (readr, dplyr, tidyr, stringr), Tableau Public.

Author

Evan M. Heffelfinger

License and data use

Code: [MIT or other license]. Data are public CMS and CDC files and remain subject to their original terms.

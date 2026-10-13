# Next.co.uk - NeoLoad Performance Test Project

NeoLoad project for the Next.co.uk (UK retail) shopping-bag and sign-in journey.
The user path, correlation, parameterization, population and scenarios are already configured.

| | |
|---|---|
| NeoLoad version | 2026.2.1 (project format 8.11) |
| User path | `Next_Shopping` (Init / Actions / End) |
| Transactions | 12 |
| HTTP requests | 46 |
| Variable extractors (correlation) | 74 |
| File variables | `credentials`, `testdata` |
| Servers | www.next.co.uk, auth.next.co.uk, api.e.next.co.uk, paymentgateway.next.co.uk |
| Scenarios | `Smoke_1VU`, `Load_Test` |

## Repository structure

```
Next_Retail_NeoLoad/
├── Next_Retail/                         NeoLoad project folder
│   ├── Next_Retail.nlp                  project file (open this)
│   ├── config.zip                       user paths, servers, variables, populations, scenarios
│   ├── data/
│   │   ├── credentials.csv              File variable "credentials"
│   │   └── testdata.csv                 File variable "testdata"
│   └── lib/, scripts/, custom-resources/, recorded-*/   standard NeoLoad folders
└── download/
    └── Next_Retail_NeoLoad_Project.zip  same project as a single download
```

## How to open

**Option A - single zip**
1. Download `download/Next_Retail_NeoLoad_Project.zip`.
2. Unzip it.
3. In NeoLoad: **File > Open Project** and select `Next_Retail/Next_Retail.nlp`.

**Option B - clone or Download ZIP**
1. `git clone https://github.com/sagar9804644867/Next_Retail_NeoLoad.git` (or **Code > Download ZIP** and unzip).
2. In NeoLoad: **File > Open Project** and select `Next_Retail/Next_Retail.nlp`.

## Project design

### User path - `Next_Shopping`

| Container | Transactions |
|---|---|
| Init | T01_shoppingbag |
| Actions | T02_LoadStatelessSession, T03_c, T04_shoppingbag_osa, T05_myaccount, T06_Login, T07_shopping_bag, T08_c, T09_9fGRkU, T10_c, T11_bulk, T12_bulk |
| End | - |

Recorded think times are kept between transactions. Static resources (images, CSS, JS, fonts) and
third-party traffic (analytics, tag managers, ad and consent platforms) are excluded so that load is applied
only to Next.co.uk servers.

### Correlation
Dynamic server values are captured with NeoLoad variable extractors and replayed as `${variable}` in later requests:
- **JSONPath** extractors for JSON API responses
- **Regular-expression** extractors for HTML responses (including hidden form fields)
- **Header** extractors for `Location` / `Set-Cookie` values

Cookies are handled by the NeoLoad browser profile, not hard-coded.

### Parameterization
| File variable | File | Change policy | Distribution |
|---|---|---|---|
| `credentials` | data/credentials.csv | Each iteration | Global, sequential, cycle when out of values |
| `testdata` | data/testdata.csv | Each iteration | Global, sequential, cycle when out of values |

Add rows to the CSV files to drive more data. For real logged-in users, add one test account per row and switch
`credentials` to *Each virtual user / Unique* (needs at least one row per virtual user).

### Scenarios
| Scenario | Load | Duration | Purpose |
|---|---|---|---|
| `Smoke_1VU` | 1 virtual user, constant | 60 s | Validate the script before load |
| `Load_Test` | Ramp-up 1 to 5 users, +1 user every 30 s | 10 min | Baseline load test |

Population: `Population1` = 100% `Next_Shopping`, local load generator.

## Running
1. Select `Next_Shopping` and run **Check User Path** to validate the replay.
2. **Runtime** tab: select `Smoke_1VU` and run it.
3. When the smoke test is clean, run `Load_Test`. Scale users and duration in the scenario's population settings as needed.

A NeoLoad license (a NeoLoad Web lease or a local license) is required to run tests.

## Notes and known limitations
- Next.co.uk uses bot protection (sensor requests with browser-generated payloads). Those requests are
  JavaScript-generated and may be rejected on replay; they may need a JavaScript action, a whitelisted
  load generator IP, or exclusion in agreement with the application team.
- The payment gateway (`paymentgateway.next.co.uk`) should be stubbed or run against a test environment for load tests.
- Recordings (HAR/SAZ) are intentionally not stored in this repository because they contain session cookies.

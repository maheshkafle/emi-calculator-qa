# EMI Calculator – QA Assessment

Test plan, test cases, Playwright automation, bug reports, summary report and CI for the **Home Loan EMI Calculator** at <https://emicalculator.net/>.

| Deliverable | Location |
|---|---|
| 1. Test Plan | [`docs/Test_Plan.md`](docs/Test_Plan.md) |
| 2. Test Cases, grouped by type (56) | [`docs/Test_Cases.xlsx`](docs/Test_Cases.xlsx) |
| 2. Automation scripts | [`tests/`](tests), [`src/`](src) |
| 3. Bug Reports | [`docs/Bug_Reports.md`](docs/Bug_Reports.md) |
| 4. Test Summary Report | [`docs/Test_Summary_Report.md`](docs/Test_Summary_Report.md) |
| 5. Regression strategy (Q4) | [`docs/Regression_Strategy.md`](docs/Regression_Strategy.md) and summary below |
| 6. CI | [`.github/workflows/playwright.yml`](.github/workflows/playwright.yml) |

---

## Quick start

```bash
npm install
npx playwright install --with-deps chromium

npm test                  # everything (regression + E2E on Chromium)
npm run test:regression   # calculation-engine regression only – no browser, ~5 s
npm run test:e2e          # UI E2E on Chromium
npm run test:smoke        # @smoke subset
npm run test:headed       # watch the sliders move
npm run report            # open the HTML report
```

Options (environment variables):

| Variable | Default | Purpose |
|---|---|---|
| `BASE_URL` | `https://emicalculator.net/` | Point the suite at another environment |
| `BROWSERS=all` | off | Also run the Firefox and WebKit projects |
| `BLOCK_ADS=false` | ads blocked | Turn off network blocking of ad/analytics hosts |
| `EXCEL_DOWNLOAD_SELECTOR` | auto-detect | CSS selector of the "Download Excel" control, if it changes |

---

## Find and open project files

The repository root is the `emi-calculator-qa` folder. In VS Code, choose **File → Open Folder...**, select that folder, then browse the files in the Explorer. Use **Ctrl+P** and enter a relative path (for example, `docs/Bug_Reports.md`) to open a specific file.

From Windows Command Prompt:

```cmd
cd /d "C:\Users\mahes\Downloads\Project\emi-calculator-qa"
code .
```

`code .` opens the current folder in VS Code if its command-line launcher is installed. Replace the example path if the project is in a different location.

### File guide

| Location | What it contains |
|---|---|
| [`package.json`](package.json) | Project metadata, dependencies, and commands such as `npm test`, `npm run test:e2e`, and `npm run typecheck`. |
| [`package-lock.json`](package-lock.json) | Exact dependency versions installed by npm. |
| [`playwright.config.ts`](playwright.config.ts) | Playwright projects, browser settings, timeouts, reporters, and test artifact settings. |
| [`tsconfig.json`](tsconfig.json) | TypeScript compiler options and source/test file scope. |
| [`.gitignore`](.gitignore) | Files and generated folders excluded from version control. |
| [`.github/workflows/playwright.yml`](.github/workflows/playwright.yml) | GitHub Actions workflow for automated checks. |
| [`docs/Test_Plan.md`](docs/Test_Plan.md) | Scope, approach, environments, risks, and test entry/exit criteria. |
| [`docs/Test_Cases.xlsx`](docs/Test_Cases.xlsx) | Spreadsheet of the assessment's test cases, grouped by type. Open with Excel or another spreadsheet viewer. |
| [`docs/Bug_Reports.md`](docs/Bug_Reports.md) | Recorded defects, reproduction steps, expected/actual results, and analysis. |
| [`docs/Test_Summary_Report.md`](docs/Test_Summary_Report.md) | Execution summary, coverage, findings, and recommendations. |
| [`docs/Regression_Strategy.md`](docs/Regression_Strategy.md) | Regression tiers, what each tier covers, and maintenance rules. |
| [`src/data/reference.ts`](src/data/reference.ts) | Reference input and expected results used as the golden test case. |
| [`src/data/scenarios.ts`](src/data/scenarios.ts) | Reusable input scenarios for the UI tests. |
| [`src/fixtures/test.ts`](src/fixtures/test.ts) | Shared Playwright fixture that opens the calculator and provides its page object. |
| [`src/pages/EmiCalculatorPage.ts`](src/pages/EmiCalculatorPage.ts) | Page object for calculator inputs, summary, chart, schedule, and Excel download interactions. |
| [`src/pages/selectors.ts`](src/pages/selectors.ts) | Centralized selectors, slider ranges, and network host settings. |
| [`src/utils/currency.ts`](src/utils/currency.ts) | Helpers for parsing and formatting Indian currency amounts. |
| [`src/utils/emi.ts`](src/utils/emi.ts) | Independent EMI calculation and amortisation oracle used by tests. |
| [`src/utils/excel-schedule.ts`](src/utils/excel-schedule.ts) | Locates and aggregates schedule rows from downloaded workbook layouts. |
| [`src/utils/spreadsheet.ts`](src/utils/spreadsheet.ts) | Reads supported XLSX, CSV/TSV, and HTML-table spreadsheet formats. |
| [`tests/e2e/01-slider.spec.ts`](tests/e2e/01-slider.spec.ts) | Slider input, handle movement, and recalculation checks. |
| [`tests/e2e/02-emi-formula.spec.ts`](tests/e2e/02-emi-formula.spec.ts) | Compares displayed EMI and totals with the independent formula. |
| [`tests/e2e/03-chart-table.spec.ts`](tests/e2e/03-chart-table.spec.ts) | Reconciles chart, tooltips, schedule table, and oracle results. |
| [`tests/e2e/04-excel-download.spec.ts`](tests/e2e/04-excel-download.spec.ts) | Checks downloaded workbook contents against the UI schedule. |
| [`tests/e2e/05-input-validation.spec.ts`](tests/e2e/05-input-validation.spec.ts) | Checks invalid and boundary input behavior. |
| [`tests/regression/emi-engine.spec.ts`](tests/regression/emi-engine.spec.ts) | Fast, browser-free tests for calculation correctness and invariants. |
| [`tests/regression/spreadsheet-reader.spec.ts`](tests/regression/spreadsheet-reader.spec.ts) | Tests spreadsheet reading and schedule extraction. |
| [`tests/fixtures/sample-schedule.xlsx`](tests/fixtures/sample-schedule.xlsx) | Small sample workbook used by spreadsheet-reader tests. |

### Generated or installed folders

These are created locally and are not source files:

| Folder | How to access it |
|---|---|
| `node_modules/` | Installed dependencies. Run `npm install` to create or restore it; normally you do not need to browse or edit it. |
| `test-results/` | Playwright output such as JUnit/JSON results, downloads, screenshots, videos, and traces. Created by running tests. |
| `playwright-report/` | HTML test report. Run `npm run report` after a test run to open it. |

The assessment brief, `QE Assessment.pdf`, is stored separately at `C:\Users\mahes\Downloads\QE Assessment.pdf`; it is not inside this repository. Its reference case is represented in `src/data/reference.ts`.

---

## Assessment mapping (Q3)

| Requirement | Spec | How it's validated |
|---|---|---|
| **3a** Update Amount, Interest and Tenure **using the sliders** | `tests/e2e/01-slider.spec.ts` | Real mouse drag of each handle to its proportional position, then fine-tuning with the keyboard until the bound input shows the exact target. Asserts the input value, handle position on the track, recalculated EMI, range end-stops, two-way binding and keyboard access. |
| **3b** Capture EMI and **validate with a formula** in test code | `tests/e2e/02-emi-formula.spec.ts` | Six data-driven scenarios (including boundaries) set via sliders. The UI EMI, Total Interest, Total Payment and pie split are compared with `EMI = P·r·(1+r)^n / ((1+r)^n − 1)`, computed in `src/utils/emi.ts`, with a ±₹1 rounding tolerance. |
| **3c** **Chart vs table**, year-wise | `tests/e2e/03-chart-table.spec.ts` | Reads the bar chart's Highcharts data model (Principal, Interest, Balance series) and the table's year rows. Asserts the same years in the same order, and each value within ±₹1. Also hovers bars to compare tooltips, checks the table against the independent amortisation oracle, and checks internal reconciliation (P+I=Total, Σ principal = loan, closing balance 0). |
| **3d** **Download the Excel file** and check it | `tests/e2e/04-excel-download.spec.ts` | Captures the download, then checks file name/extension, size, that it parses, headers, the EMI value, year-by-year rows against the UI table, column totals, closing balance 0, and that a second download reflects changed inputs. |
| Extra: negative inputs | `tests/e2e/05-input-validation.spec.ts` | Alphabetic, negative, overflow, zero and fractional inputs, plus an XSS payload. Results must never show NaN/Infinity/negative values, and no script may run. |

---

## Architecture

```
emi-calculator-qa/
├── playwright.config.ts        # projects: regression | e2e-chromium | e2e-firefox | e2e-webkit
├── src/
│   ├── pages/
│   │   ├── selectors.ts        # ALL locators + slider ranges + blocked ad hosts (one place to update)
│   │   └── EmiCalculatorPage.ts# Page Object: setViaSlider, readSummary, readYearlyTable,
│   │                           #   readBarChart (Highcharts), hoverBarTooltip, downloadExcel …
│   ├── fixtures/test.ts        # custom `test` with ad-blocking + ready-to-use `emi` page object
│   ├── utils/
│   │   ├── emi.ts              # independent EMI / amortisation oracle (the "expected" side)
│   │   ├── currency.ts         # ₹ 1,07,96,711 ⇄ number
│   │   ├── spreadsheet.ts      # zero-dependency .xlsx / .csv / HTML-.xls reader
│   │   └── excel-schedule.ts   # finds the schedule in any workbook layout (yearly or monthly)
│   └── data/
│       ├── reference.ts        # golden master transcribed from the brief's screenshot
│       └── scenarios.ts        # data-driven slider scenarios with @tags
├── tests/
│   ├── regression/             # L1 – pure logic, no browser (178 checks)
│   ├── e2e/                    # L2 – UI specs 01–05 (24 tests)
│   └── fixtures/               # sample workbook for reader self-tests
├── docs/                       # plan, cases, bugs, summary, regression strategy
└── .github/workflows/          # CI
```

**Design principles**

* **Independent oracle.** Expected values are never taken from the app. `emi.ts` implements the maths and was proven against the brief's reference output: EMI, totals and all 21 rows match within ₹1.
* **Page Object Model + a single selector map.** Specs read like test cases, and a markup change touches only `selectors.ts`.
* **Deterministic slider control.** A coarse drag followed by keyboard fine-tuning reaches exact values regardless of step size.
* **No sleeps.** `waitForStableResults()` polls until the card and table stop changing.
* **Chart data from the source.** The Highcharts series are read directly, so every year is compared, not just the bars you can hover. Tooltip hovering adds a UI-level check on top.
* **Minimal dependencies.** Only Playwright and TypeScript. Excel parsing is built in, so there is no `xlsx`/`exceljs` supply-chain risk.
* **Rich failure evidence.** Trace, screenshot and video on failure. HTML, JUnit and JSON reporters.

---

## Regression strategy (Q4) – summary

Full write-up: [`docs/Regression_Strategy.md`](docs/Regression_Strategy.md).

1. **Tier 1 – engine regression** (`tests/regression`, < 10 s, blocking on every PR):
   * **Golden master:** the brief's 50 L / 9 % / 20 y output, frozen (EMI, totals, 21 year rows).
   * **Known-answer tests** from independently computed values.
   * **Invariant grid** (150 combinations): closing balance 0, Σ principal = loan, Σ interest = EMI·n − P, Total = P + I, EMI bounds, monotonically falling balance and interest.
   * **Metamorphic checks:** EMI is linear in principal, rises with rate and falls with tenure.
   * **Edge cases:** 0 % interest, 1-month tenure, zero principal, December → January rollover.
   * **Rounding policy as a test:** totals must use the unrounded EMI.
2. **Tier 2 – UI calculation regression** (every PR plus nightly): data-driven slider scenarios tagged `@smoke`, `@regression` and `@boundary`; card ↔ chart ↔ table ↔ Excel reconciliation.
3. **Tier 3 – cross-browser matrix** (nightly): Chromium, Firefox and WebKit.
4. **Rules:** every fixed defect adds a golden case. Baselines change only on approved policy changes. Flaky tests are quarantined, not retried forever.

---

## CI/CD (GitHub Actions)

`.github/workflows/playwright.yml`

* **Triggers:** push/PR to main, nightly at 02:30 IST, and manual dispatch (choose browsers and a `--grep` tag).
* **Job 1 – `regression`:** install, type-check, run the engine regression. It's fast and gates the next job.
* **Job 2 – `e2e`:** a browser matrix (Chromium on PRs; all three nightly or on demand). Installs only the needed browser and retries once in CI. Writes a pass/fail table to the job summary.
* **Artifacts:** the HTML report, traces, videos and JUnit/JSON results are uploaded for 14 days.

---

## Findings at a glance

The EMI engine is correct. Three low-severity rounding defects cause schedule rows and columns not to add up exactly to the summary card (₹1 per row, ₹3 on total interest). See [`docs/Bug_Reports.md`](docs/Bug_Reports.md).

## Notes / assumptions

* Slider ranges follow the on-page scales: amount 0–200 L, rate 5–20 %, tenure 0–30 years.
* The schedule's start month is read from the page ("starting from …") and used to build expectations, so tests don't go stale each month.
* Ad and analytics hosts are blocked during automation to keep it stable. Set `BLOCK_ADS=false` to test the page as users see it.
* The UI suite was built without access to the live site from the authoring environment. If the first live run shows a locator mismatch, fix it in `src/pages/selectors.ts` (or `EXCEL_DOWNLOAD_SELECTOR` for the download button).

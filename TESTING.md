# Testing evidence

Audit date: **October 13, 2026**. These are current, AI-assisted audit results, not a claim that the project owner personally executed every check or that these checks existed throughout development.

## Environment and method

Windows host; Microsoft JDK 21.0.12; Android SDK platform 36 and Build Tools 36.0.0. Tests ran on an isolated Android 15/API 35 x86_64 emulator with synthetic data. Both applications retain minimum API 23 and target API 36. This audit does not cover every supported Android version.

CalcWorx uses Java integration helpers plus reconstructed smali/resources, built with Apktool 3.0.3 and Android tooling. Titan1099 uses Java, Gradle 9.5.0 and Android Gradle Plugin 9.3.1. Development builds and test instrumentation used development signing, without production signing secrets.

Existing custom instrumentation exercises installed application code and views, including internal handlers through reflection. Selected CalcWorx cases were also exercised through the visible UI. Titan's main screen is custom drawn and exposes little useful UI automation text; its programmatic checks do not establish accessibility. Screenshots show actual app rendering, not mockups.

## Build and suite results

| Check | Result | Scope |
| --- | --- | --- |
| CalcWorx development APK | Passed | Rebuilt from the reviewed editable inputs using a separate development key |
| CalcWorx existing number-input suite | Passed: 26 checks | Parsing and invalid inputs |
| CalcWorx existing installed-app suite | Initial run passed: 310 assertions; later rerun failed | Conversions, formulas, invalid inputs, screens and settings |
| CalcWorx existing workflow suite | Initial run passed: 44 assertions; later rerun failed | Concrete/board-foot estimates, repeated saves, pricing and theme behavior |
| CalcWorx newly added number-input suite | Passed: 10 assertions | Exact fractions, whitespace, zero denominators and invalid forms |
| CalcWorx newly added tape suite | Passed: 6 assertions | Exact measurement, 1/64 rounding, zoom and invalid-input clearing |
| CalcWorx Gradle/lint | Not applicable | This maintained build is not a Gradle project |
| Titan1099 debug APK | Passed | Original project and a fresh sanitized source copy built |
| Titan1099 Gradle unit tests | No tests run | Task reported NO-SOURCE; not a passing unit-test suite |
| Titan1099 Android lint | **Failed: 25 errors, 212 warnings** | Includes location-permission and resource-type findings; not suppressed |
| Titan1099 existing runtime suite | Passed: 14 assertions | Mileage calculation, ledger totals and synthetic receipt OCR; run on debug, despite the suite's historical “optimized” output label |
| Titan1099 existing fuel-scanner suite | Passed: 5 assertions | Synthetic fuel OCR and scan cleanup; debug build |
| Titan1099 broad audit, initial run | **Failed** | Optional building tracking was off; old fixture expected a learned floor |
| Titan1099 broad audit, corrected fixture | Passed: 45 assertions | Enabled the existing optional feature explicitly; app behavior unchanged |
| Titan1099 newly added regression suite | Passed: 9 assertions | Mileage save/reopen, date validation, invalid input and CSV formatting |
| Titan1099 process restart | Passed | Saved mileage CSV hash unchanged after force-stop and relaunch; dashboard showed 25 saved miles |

Assertion totals describe these particular suites, not a coverage percentage.

## Representative inputs and independent expectations

| App and case | Expected result and basis | Observed result |
| --- | --- | --- |
| CalcWorx: 1/2 inch to millimetres | 12.7 mm, from 0.5 × 25.4 | 12.7 mm in the UI |
| CalcWorx: scaled squaring, 6 ft × 8 ft, measured diagonal 10 ft | 10 ft target because 6² + 8² = 10²; zero difference | Target 10 ft, difference 0, “Square” |
| CalcWorx: 1/3 inch on tape | Exact value remains 1/3; nearest 1/64 is 21/64 because 64/3 ≈ 21.333 | Exact value retained, rounded index 21; zoom preserved exact value |
| CalcWorx: 12 ft × 12 ft × 4 in slab, no waste | 48 cubic feet; at 0.6 cubic feet per 80 lb bag, 80 bags; at $7.25, $580 | 80 bags and $580; repeat save updated the item without duplicating it |
| CalcWorx: ten boards, 1 in × 12 in × 8 ft, $5 per board foot | 1 × 12 × 8 ÷ 12 × 10 = 80 board feet; $400 | 80 board feet and $400 |
| CalcWorx: 1/0 and empty tape input | No valid measurement; previous tape result must clear | Cleared |
| Titan1099: odometer 1000 → 1025 | 25 miles by subtraction | 25; saved CSV contains start, end and distance |
| Titan1099: second Save after fields clear | No new row without mileage input | Ledger unchanged |
| Titan1099: reversed/negative mileage and empty fields | Reject without changing saved ledger | Ledger unchanged |
| Titan1099: February 30 | Invalid calendar date | Rejected |
| Titan1099: CSV fields containing comma, quotes and newline | Quote affected fields; double embedded quotes | Exact expected CSV string matched |
| Titan1099: generated receipt image “TOTAL $42.50” | 42.50 | OCR populated 42.5/42.50 |

## Known limits and remaining checks

- Titan's lint findings need a separate engineering pass. A successful APK build does not resolve them.
- Correcting the optional-building test setup resolved that particular failure; it does not prove real-world GPS, floor detection, Bluetooth reception or walking accuracy.
- Camera tests used generated receipt images and programmatic result delivery. Physical camera capture, outdoor GPS, Android Auto hardware, and actual driving were **not tested**.
- Second-Save protection is narrower than universal duplicate detection. Re-entered identical records and all import duplicate cases are **not established** by the new suite.
- CalcWorx's narrow/tall screen layout, horizontal navigation and bottom controls need broader screen-size/accessibility testing. A 3D-startup fallback appeared after automated theme/lifecycle tests; a full graphics/motion audit was **not performed**.
- Release/Shipping builds and Play distribution were not changed or revalidated. Public Play availability was not verified, so no store or testing invitation link is advertised.
- Screenshots and test data are synthetic. No personal ledgers, receipts, addresses or location tracks were used for publication.

## A concrete diagnostic example from this audit

Titan1099's broad test originally stopped when it tried to read the first learned floor. Source inspection showed that current building tracking is optional and disabled by default, and the learning method returns immediately while it is disabled. The old fixture had not enabled it. The audit branch now enables the feature before testing it; the 45-assertion suite passed afterward. The initial failure remains recorded above. This is a current test-maintenance example, not an invented historical development story.

CalcWorx required a different reproducibility approach: its maintained editable inputs are reconstructed smali/resources plus Java helpers, and the old build script depended on an owner's local update key. A separate development build script now compiles the same inputs with a disposable development key. It rebuilt successfully and was exercised in the emulator; original signing files and the original project remained untouched.

## Repeatability finding

The packaged-runner recheck failed in CalcWorx's older theme tests: Theme gallery missing and Theme owns background mode 10. Source inspection and the visible startup warning show that the persisted 3D recovery state adds availability controls and prevents selecting a 3D theme. The older tests assume a fixed gallery and unrestricted theme selection. Initial counts above remain historical results from this audit, not a claim of repeatable green status. No app recovery checks were disabled. The new input and tape checks still passed. A follow-up should make the older theme tests state-aware and verify recovery explicitly.

Titan's broad CompleteAudit passed 45 assertions after its fixture correction, but a later isolated repeat did not finish within a 45-second limit (exit 124). The earlier packaged-runner attempt was stopped after it stalled. This repeatability issue remains unresolved; the broad suite is not consistently green. The runtime, scanner and new portfolio regression suites passed again during runner validation. The distributed runner bounds each instrumentation command to 90 seconds and reports missing PASS results as failures.

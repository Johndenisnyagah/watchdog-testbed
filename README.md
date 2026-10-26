# watchdog-testbed

> [!WARNING]
> **Deliberately vulnerable. Do not install or reuse anything here.**
> `requirements.txt` (Python) and `package-lock.json` (npm) pin packages to versions with known, published vulnerabilities on purpose. This repository has no code worth running and nothing here is ever installed; it exists only as a scan target.

This is the live demo for [sbom-watchdog](https://github.com/Johndenisnyagah/sbom-watchdog), a GitHub Action that scans a repository's dependencies every day and opens an issue only for vulnerabilities that were not there the day before. Everything you see here was produced by that action running on its own schedule.

## What to look at

- **[Issues](../../issues?q=is%3Aissue)**: filed and closed by `github-actions[bot]`, never by hand.
- **[Commit history](../../commits/main)**: one state commit per day, with a message saying what moved (`N new, M resolved`) or `no change (scan of YYYY-MM-DD)`.
- **[`.sbom-watchdog/findings.json`](.sbom-watchdog/findings.json)**: the committed record of every finding, with `first_seen`, `last_seen`, the issue it was filed under, and `resolved_on`.
- **[`SECURITY-INVENTORY.md`](SECURITY-INVENTORY.md)** and **[`sboms/`](sboms/)**: a plain-language summary and one dated CycloneDX SBOM per day, written daily since 26 October 2026.
- **[Workflow runs](../../actions/workflows/watchdog.yml)**: the daily 03:17 UTC scan.

## What has happened so far

| Date | What the action did | Why |
| --- | --- | --- |
| 24 Aug 2026 | Recorded 22 findings and filed issues #1–#9 for the High and Critical ones | The state was seeded empty on purpose, so the first scan treated everything as new and the issue path could be exercised |
| 24 Aug 2026 | Marked 7 findings resolved and closed #3, #7, #8 and #9 with a comment | `urllib3` was upgraded to 2.7.0, which fixed them |
| 25 Aug – 30 Sep | Filed nothing; committed `no change` daily | Nothing new was published against these versions |
| **1 Oct 2026** | **Filed #10 and #11** | **Two vulnerabilities in `urllib3` 2.7.0 were newly published. This is the case the tool exists for: a dependency that was clean yesterday is vulnerable today.** |
| 25 Oct 2026 | Recorded one new finding, filed no issue | It is rated Low, below the `High` threshold. Every finding is recorded; only High and above become issues |
| 26 Oct 2026 | Recorded 6 npm findings and filed #14–#17 for the Critical and High ones; wrote the first `SECURITY-INVENTORY.md` and `sboms/2026-10-26.json` | A second ecosystem, seeded on purpose like 24 August, to show npm handled end to end: a scoped package (`@babel/traverse`, #17), a vulnerability arriving through a dependency nobody listed (`minimist` via `mkdirp`, #14), and two findings on one package resolved by a single upgrade (`node-fetch`, #16). The inventory now names the ecosystem of each row, because it spans Python and npm. |

A normal adopter's first run is different from 24 August: with no state file, it records a baseline and files nothing, so installing the tool on a large project does not open hundreds of issues at once.

## How it runs

[`.github/workflows/watchdog.yml`](.github/workflows/watchdog.yml) calls the published action, pinned to the commit behind `v1.6`. Adopters can write `@v1` instead to pick up compatible releases. Until 25 October 2026 this repository ran a copied, older version of the scanner's code from a `src/` folder; it now runs only the published action, so what you see is what an adopter gets. Issues filed before that date use the older issue wording.

The workflow commits the state file, the inventory and the dated SBOM itself, as the action's README recommends. A run whose issue filing fails part way still commits the issue numbers it collected and then fails, so nothing is filed twice.

## The npm part

`package.json` lists four packages; `package-lock.json` is what Syft reads to build the inventory (a `package.json` on its own produces nothing). All four are under `dependencies` on purpose: Syft leaves `devDependencies` out by default.

| Package | Why it is here |
| --- | --- |
| `@babel/traverse` 7.23.0 | A scoped name with a Critical advisory |
| `mkdirp` 0.5.1 | Pulls in `minimist` 0.0.8, so a vulnerability arrives through a dependency nobody listed |
| `node-fetch` 2.6.0 | A High and a Low advisory on one package |
| `semver` 5.7.1 | A High advisory with a single, stable fix |

They are old, unmaintained release lines, chosen so the set of findings stays stable rather than growing every week.

## Things that are expected here

- **Dependabot alerts.** GitHub's own scanner sees the same vulnerable pins. Its automatic update pull requests are switched off on purpose, because they would "fix" the test target.
- **Mostly bot commits.** The daily state commit is the audit trail, so it is meant to be there.
- **Forks do not run the schedule.** GitHub disables scheduled workflows in forks. Enable Actions in your fork and run the workflow by hand to try it.

# Playwright TypeScript Test Portfolio

[![CI](https://github.com/JordyRdev/playwright-typescript-portfolio/actions/workflows/ci.yml/badge.svg)](https://github.com/JordyRdev/playwright-typescript-portfolio/actions/workflows/ci.yml)

Public sample of a small, trustworthy automation suite: UI journeys, API contracts, and a CI report a hiring manager can open.

This repository is the hands-on part of that work. It shows how I set up a suite a team can run, read, and trust.

The targets are public practice systems. [Sauce Demo](https://www.saucedemo.com) is the shop. [Restful Booker](https://restful-booker.herokuapp.com) is the API. Credentials in `.env.example` are the published demo accounts, not secrets.

## What this demonstrates

- **Risk-based selection.** Login, cart, checkout, auth, and booking CRUD. Each test protects a decision a user or a client system can get wrong.
- **Page Object Model and fixtures.** Screens stay in page objects. Tests ask for a `shopper` or a `bookingClient` instead of rebuilding setup.
- **UI and API in one runner.** Playwright Test drives the browser and the HTTP client, so reporting, retries, and CI stay in one place.
- **Contract checks.** Zod parses auth and booking payloads. A shape change fails the test with the field that drifted.
- **Cross-browser and mobile.** Chromium, Firefox, WebKit, and a Pixel 5 viewport run the same UI journeys.
- **Evidence on failure.** Trace, screenshot, and video are kept when a test fails. The HTML report is uploaded from CI and published to GitHub Pages from `main`.

## Tech stack

| Tool                | Role                                            |
| ------------------- | ----------------------------------------------- |
| Playwright Test     | UI, API, traces, HTML report                    |
| TypeScript (strict) | Page objects, fixtures, test data               |
| Zod                 | Response schemas                                |
| ESLint + Prettier   | Lint and formatting, including Playwright rules |
| GitHub Actions      | Lint, full suite, report artifact, Pages deploy |
| Node.js 22          | Runtime                                         |

## Folder structure

```text
tests/ui            login, sorting, cart, checkout
tests/api           auth, booking CRUD, contract checks
src/pages           page objects
src/fixtures        shopper and API fixtures
src/api             HTTP client and Zod schemas
src/data            users, catalog, customer, booking builder
src/support         checkout total rule
playwright.config.ts
.github/workflows/ci.yml
```

Test data lives under `src/data`. Tests describe behavior. Page objects know the selectors.

## Run locally

Requires Node.js 22.

```bash
npm ci
npx playwright install --with-deps
cp .env.example .env   # optional; the defaults already point at the public sites
npm test               # API once, UI on Chromium, Firefox, WebKit, and mobile Chrome
npm run test:ui
npm run test:api
npm run lint           # ESLint, Prettier, and tsc --noEmit
npm run report         # open the last HTML report
```

Narrow a UI run while debugging:

```bash
npx playwright test tests/ui --project=chromium
```

## Test strategy

I treat this as a regression suite for a product I do not own, so coverage follows risk rather than screens.

**What is in the suite**

- Login is the gate. Valid shopper, unknown credentials, locked account, and a missing username.
- Sorting checks name Z–A and price low–high, including the tie between the two $15.99 shirts.
- Cart checks price, badge, and removal.
- Checkout checks the required first name, then a full purchase. The taxed total is calculated the same way the shop does: 8% rounded to cents, then added to the item total.
- API auth checks the token and the failure payload.
- One CRUD test creates a booking with a unique name, reads it, replaces it, patches it, finds it, and deletes it.
- Contract tests cover the health check, the booking-id list, an unknown id, an empty body, and update or delete calls with no token.

**How the pyramid is applied**

API tests carry the data rules: status codes, auth, and payload shape. They do not open a browser. UI tests cover what only the browser can show: navigation, sorting, the cart badge, and the checkout copy. The same UI journey runs on four projects so a browser-specific break is visible. I do not re-test the booking matrix through the UI, because this shop has no booking API behind it.

**What stays out, on purpose**

Sauce Demo ships `problem_user`, `error_user`, and `performance_glitch_user` so people can see seeded defects and a five-second stall. Those accounts are outside this regression set. Pixel-level screenshot comparison is also out: it is noisy across three browser engines and would not tell a client whether checkout still takes money.

**Flake handling**

- Every UI test starts from a fresh browser context.
- API tests build a unique first name and delete the booking afterwards, including when an earlier step fails.
- Local runs do not retry. A failure stays a failure.
- CI retries twice. These targets are public and occasionally slow. A retry absorbs a network blip. A test that needs a retry every run gets fixed or removed.
- Nothing is skipped. `playwright/no-skipped-test` is an error, and CI sets `forbidOnly`.
- On failure, Playwright keeps the trace, screenshot, and video. Open `npm run report`, or download the `playwright-report` artifact from the GitHub Action.

**Practice-API behavior the tests lock in**

These are properties of Restful Booker, recorded so a change is obvious:

- A bad login returns HTTP 200 and `{ "reason": "Bad credentials" }`.
- A successful delete returns HTTP 201 and the body `Created`.
- An empty booking body returns HTTP 500.

## Continuous integration

On every push to `main` and on every pull request, GitHub Actions installs the browsers, runs lint, and runs the full suite. The HTML report is uploaded as the `playwright-report` artifact for 14 days.

Each push to `main` publishes that report on GitHub Pages:

<https://JordyRdev.github.io/playwright-typescript-portfolio/>

## License

[MIT](LICENSE)

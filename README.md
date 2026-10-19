name: IBES Engineering Verification
on: push: branches: ["main"] pull_request: branches: ["main"] workflow_dispatch:
permissions: contents: read
jobs: source-tests: name: Engineering Source Tests runs-on: ubuntu-latest steps: - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: 22

  - name: Run engineering tests
    run: npm run test:engineering
browser-acceptance: name: Real Browser Acceptance runs-on: ubuntu-latest timeout-minutes: 15 steps: - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: 22

  - name: Install Playwright
    run: npm install --no-save playwright

  - name: Install Chromium
    run: npx playwright install --with-deps chromium

  - name: Run circuit persistence journey
    env:
      PLAYWRIGHT_ARTIFACTS: playwright-artifacts
    run: node tests/electrical-circuit-e2e.playwright.cjs

  - name: Upload failure evidence
    if: failure()
    uses: actions/upload-artifact@v4
    with:
      name: ibes-browser-acceptance-failure
      path: playwright-artifacts/
      if-no-files-found: ignore
      retention-days: 7


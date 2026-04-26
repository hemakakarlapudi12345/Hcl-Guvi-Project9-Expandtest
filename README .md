# expandtest-framework

A Java-based Selenium + TestNG test automation framework targeting the [ExpandTesting practice site](https://practice.expandtesting.com). Built with the Page Object Model (POM) design pattern, it includes automatic browser setup, configurable settings, Extent HTML reporting, and screenshot capture on failure.

---

## Tech Stack

| Tool | Version | Purpose |
|---|---|---|
| Java | 11+ | Language |
| Selenium | 4.21.0 | Browser automation |
| TestNG | 7.10.2 | Test runner |
| WebDriverManager | 5.8.0 | Auto-manages ChromeDriver |
| ExtentReports | 5.1.1 | HTML test reports |
| Apache Commons IO | 2.11.0 | Screenshot file handling |
| Maven | 3.x | Build & dependency management |

---

## Project Structure

```
expandtest-framework/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   ├── base/
│   │   │   │   └── BasePage.java          # Reusable WebDriver helpers (click, type, getText, wait)
│   │   │   ├── config/
│   │   │   │   └── ConfigReader.java      # Reads config.properties (browser, baseUrl, timeout)
│   │   │   └── pages/
│   │   │       ├── AlertPage.java         # JS alert/confirm/prompt interactions
│   │   │       ├── FormPage.java          # Input, dropdown, checkbox, radio interactions
│   │   │       ├── LoginPage.java         # Login/logout actions and locators
│   │   │       └── NotesPage.java         # Notes CRUD stubs
│   │   └── resources/
│   │       └── config.properties          # Browser, base URL, timeout config
│   └── test/
│       └── java/
│           ├── tests/
│           │   ├── BaseTest.java          # @BeforeMethod/@AfterMethod — driver init & teardown
│           │   ├── AlertTest.java         # Tests for JS alerts
│           │   ├── FormTest.java          # Tests for form elements
│           │   ├── LoginTest.java         # Login/logout tests with DataProvider
│           │   ├── NotesTest.java         # Notes CRUD tests
│           │   └── SampleTest.java        # Scratch test
│           └── utils/
│               ├── ExtentManager.java     # Singleton ExtentReports setup
│               └── TestListener.java      # ITestListener — logs pass/fail + captures screenshots
├── reports/
│   └── ExtentReport.html                  # Generated after test run
├── screenshots/                           # Auto-saved on test failure
└── pom.xml
```

---

## Configuration

Edit `src/main/resources/config.properties` before running:

```properties
browser=chrome
baseUrl=https://practice.expandtesting.com
timeout=10
```

| Property | Description |
|---|---|
| `browser` | Browser to use (`chrome` supported) |
| `baseUrl` | Root URL of the application under test |
| `timeout` | Implicit wait and explicit wait duration in seconds |

---

## Prerequisites

- Java 11 or higher installed and on `PATH`
- Maven 3.x installed
- Google Chrome browser installed
- ChromeDriver is managed automatically by WebDriverManager — no manual download needed

---

## Running Tests

**Run all tests:**
```bash
mvn test
```

**Run a specific test class:**
```bash
mvn test -Dtest=LoginTest
```

**Run a specific test method:**
```bash
mvn test -Dtest=LoginTest#testLogout
```

---

## Test Coverage

### `LoginTest`
Uses a `@DataProvider` with three scenarios:

| Username | Password | Expected |
|---|---|---|
| `practice` | `SuperSecretPassword!` | Success message shown |
| `wrong` | `wrongpass` | Error message shown |
| _(empty)_ | _(empty)_ | Error message shown |

Also includes a `testLogout` test that logs in with valid credentials and verifies the user is redirected back to `/login` after logout.

### `AlertTest`
Navigates to `/alerts` and exercises all three JavaScript dialog types:
- JS Alert — accepted
- JS Confirm — dismissed
- JS Prompt — text entered and accepted

### `FormTest`
Covers multiple form pages in sequence:
- `/inputs` — types a value into the number input
- `/dropdown` — selects "Option 1"
- `/checkboxes` — toggles both checkboxes
- `/radio-buttons` — selects the first radio button

### `NotesTest`
Three tests covering note lifecycle (`testCreateNote`, `testEditNote`, `testDeleteNote`). These tests log in first, then navigate to `/notes`. The `NotesPage` methods are currently stubs that print to console — they can be wired to real UI locators.

---

## Reporting

After a test run, an HTML report is generated at:
```
reports/ExtentReport.html
```

Open it in any browser to view pass/fail status per test.

**On failure**, a screenshot is automatically captured and saved to:
```
screenshots/<TestName>.png
```
The screenshot path is also embedded into the Extent report for easy review.

Both of these are handled by `TestListener`, registered as a TestNG listener via `BaseTest`.

---

## Design Decisions

- **Page Object Model** — Each page has its own class under `pages/`. Locators and actions are encapsulated, keeping test classes clean.
- **BasePage** — All page classes extend `BasePage`, which provides shared utilities: explicit wait, fallback JS click, `type`, `getText`, and `isDisplayed`.
- **BaseTest** — Sets up and tears down the WebDriver around every test method. Chrome is launched with a fresh user profile (`/tmp/tempProfile`) to suppress password-save popups.
- **ConfigReader** — Centralises all environment values in one properties file; no hardcoded URLs or timeouts scattered across test classes.
- **ExtentManager singleton** — Ensures a single `ExtentReports` instance is shared across all listener callbacks in a test run.

---

## Extending the Framework

**Add a new page:**
1. Create `src/main/java/pages/MyPage.java` extending `BasePage`.
2. Define locators as `By` fields and write action methods.

**Add a new test:**
1. Create `src/test/java/tests/MyTest.java` extending `BaseTest`.
2. Annotate methods with `@Test`. The listener and driver lifecycle are inherited automatically.

**Add another browser:**
1. Add an `else if` block in `BaseTest.setUp()` for the new browser.
2. Update `config.properties` with `browser=firefox` (or whichever).

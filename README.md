# Csag.Automation.Selenium

Extensions for robust Selenium tests in .NET. Element selection waits for the element and retries, and the click and input helpers recover from stale or hidden elements, so UI tests fail less often on timing.

- **.NET 8.0 and .NET 10.0**
- **Selenium.WebDriver 3.141.0** and DotNetSeleniumExtras.WaitHelpers 3.11.0

## Installation

```bash
dotnet add package Csag.Automation.Selenium
```

Selenium 3 does not download browser drivers. `chromedriver` or `geckodriver`, matching the installed browser, has to be in the current directory or on the `PATH`.

## Usage

```csharp
using Csag.Automation.Selenium;
using Csag.Automation.Selenium.Extensions;
using Csag.Automation.Selenium.Screenshot;
using OpenQA.Selenium;

using var driver = WebDriverFactory.CreateChrome(new WebDriverOptions { Headless = true });
driver.Navigate().GoToUrl("https://example.com/search");

driver.SetInputValue(By.Id("query"), "selenium");
driver.ClickElement(By.CssSelector("button[type=submit]"));
var results = driver.SelectElement(By.Id("results"), WaitCondition.ElementVisible);

var screenshotPath = driver.TakeScreenshot("search results");
```

### Creating a driver

`WebDriverFactory.CreateChrome()` and `WebDriverFactory.CreateFirefox()` start a browser with the options in `WebDriverOptions`, or with its defaults when called without options:

| Option | Default | Effect |
| --- | --- | --- |
| `WindowSize` | 1280 × 720 | Window size of the browser |
| `Headless` | `false` | Chrome without a window (`--headless`, `--no-sandbox`, `--disable-gpu`) |
| `AcceptInsecureCertificates` | `false` | Accept invalid TLS certificates |
| `AllowInsecureLocalhost` | `false` | Chrome: `--allow-insecure-localhost` |
| `IgnoreCertificateErrors` | `false` | Chrome: `--ignore-certificate-errors` |
| `LanguageCode` | none | Chrome: browser language, for example `de` |
| `DisableSameOriginPolicy` | `false` | Chrome: `--disable-web-security` |
| `AllowAccessFromLocalFiles` | `false` | Chrome: `--allow-file-access-from-files` |
| `DisablePopupBlocking` | `false` | Chrome: allow popups |
| `DisableExtensions` | `true` | Chrome: no browser extensions |
| `DisablePdfViewer` | `false` | Chrome: download PDF files instead of showing them |
| `DefaultDownloadDirectory` | none | Chrome: directory for downloads |

Firefox uses `WindowSize` and `AcceptInsecureCertificates` only, so it always opens a window. On a build agent without a display, use Chrome with `Headless` set to `true`. The certificate options switch off TLS validation for every host, so use them only against test servers you control.

### Element helpers

The extension methods in `Csag.Automation.Selenium.Extensions` extend `IWebDriver`:

- `SelectElement(selector)` waits up to 10 seconds for the element to become clickable, then looks it up with up to 10 attempts, 500 ms apart. `SelectElement(selector, waitCondition)` waits for `ElementClickable`, `ElementExists` or `ElementVisible` instead. Both throw an `ElementNotFoundException` when the element cannot be found.
- `ClickElement(selector)` and `RightClickElement(selector)` click the element and look it up again when the reference got stale. `ClickElement` falls back to moving to the element and to a JavaScript click when the element is not interactable.
- `SendKeysToElement(selector, text)` types into the element. `SetInputValue(selector, text)` clears it first.
- `element.ClickWithJsExecutor(driver)` clicks an `IWebElement` through JavaScript.

When an action still fails after its retry, the helpers throw an `ElementActionException` from `Csag.Automation.Selenium.Exceptions`, with the original exception as the inner exception.

### Screenshots

`driver.TakeScreenshot()` and `driver.TakeScreenshot(name)` from `Csag.Automation.Selenium.Screenshot` save a PNG file to `__selenium_extensions/screenshots` in the temp directory and return its path. The file names are numbered in the order the screenshots are taken. Screenshots work with a `ChromeDriver`.

## Moving from Neolution.Automation.Selenium

`Csag.Automation.Selenium` 1.0.0 is the next version of `Neolution.Automation.Selenium` 0.1.0, with the same API and behavior:

1. Replace the `Neolution.Automation.Selenium` package reference with `Csag.Automation.Selenium`.
2. Change the namespaces: `using Neolution.Automation.Selenium;` becomes `using Csag.Automation.Selenium;`, and the same for `.Exceptions`, `.Extensions` and `.Screenshot`.

The package targets .NET 8 and .NET 10 instead of .NET Standard 2.0. Projects on .NET Framework or on .NET versions before 8 stay on `Neolution.Automation.Selenium` 0.1.0.

## Build

### Prerequisites

- The .NET SDK pinned in [global.json](global.json) (10.0.4xx, a newer 10.0 SDK rolls forward). It builds the `net10.0` and the `net8.0` targets.
- Node.js 24 with Corepack, for the changesets tooling: run `corepack enable` and then `pnpm install` once in the repository root.

### Build and pack

```bash
dotnet restore --locked-mode
dotnet build --configuration Release --no-restore
dotnet pack --configuration Release --no-build -o ./nupkgs
```

The project has a `packages.lock.json`, and CI restores in locked mode. After changing a package reference, run `dotnet restore` without `--locked-mode` and commit the updated lock file.

The repository has no automated tests. CI builds and packs the library.

## Preview packages

Each pull request to `main` or to a `release/**` branch publishes the package to a public preview feed on feedz.io. A draft PR, a PR from a fork and a PR from Dependabot publish no preview. The version is one patch above the current release, for example `1.0.1-pr.42.183` (PR 42, workflow run 183). The **"chore: version packages"** PR publishes a release candidate of the upcoming version instead, for example `1.1.0-pr.73.190`, so the changes of the next release can be tested before it ships. In prerelease mode the version in `package.json` is a prerelease such as `2.0.0-rc.0`, and the preview extends it, for example `2.0.0-rc.0.pr.42.183`. That version sorts after the prerelease and before the next one. A comment on the PR gives the version of its latest push. The feed removes previews after 30 days. Releases go to nuget.org only.

To use a preview in an application, add the feed to the `nuget.config` of the application. The feed needs no credentials:

```xml
<packageSources>
  <clear />
  <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  <add key="csag-buildautomation-selenium-previews" value="https://f.feedz.io/collana-solutions/csag-buildautomation-selenium/nuget/index.json" />
</packageSources>
<packageSourceMapping>
  <packageSource key="nuget.org">
    <package pattern="*" />
    <package pattern="Csag.Automation.Selenium" />
  </packageSource>
  <packageSource key="csag-buildautomation-selenium-previews">
    <package pattern="Csag.Automation.Selenium" />
  </packageSource>
</packageSourceMapping>
```

The `Csag.Automation.Selenium` pattern is also necessary under nuget.org. NuGet looks for a package only in the sources of its most specific pattern, so without it the released versions do not restore.

To test a preview:

1. Set `Csag.Automation.Selenium` to the preview version.
2. If the application restores in locked mode, update its `packages.lock.json` files.
3. Add the feed only on the branch that tests the preview. The main branch of the application then does not need feedz.io to restore.

## Release process

This repository follows the [collana-solutions release playbook](https://github.com/collana-solutions/release-playbook) (Changesets, NuGet variant):

1. Every PR that changes the package needs a changeset. Run `pnpm changeset`, choose the bump type and describe the change for consumers of the package. CI's **Changeset Check** fails a PR that changes the project folder, `Directory.Build.props`, `Directory.Packages.props` or `global.json` without a changeset. Add an empty one (`pnpm changeset --empty`) when such a change ships nothing. A PR that only touches CI or repository documentation needs none.
2. On merge to `main`, the Release workflow keeps a **"chore: version packages"** PR up to date. It bumps the version in `Csag.Automation.Selenium/package.json`, writes it into the `.csproj` with `scripts/sync-versions.mjs` and updates `CHANGELOG.md`.
3. Merging that PR creates the git tag and the GitHub Release, which start the **NuGet Publish** workflow. It packs the package and pushes it to nuget.org through trusted publishing.

Dependabot PRs get their changesets automatically. Do not edit `CHANGELOG.md` or the `<Version>` in the `.csproj` by hand.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

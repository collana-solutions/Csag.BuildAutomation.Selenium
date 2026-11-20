# Csag.Automation.Selenium

Extensions for robust Selenium tests. Element selection waits for the element and retries, and the click and input helpers recover from stale or hidden elements, so UI tests fail less often on timing.

## Installation

```bash
dotnet add package Csag.Automation.Selenium
```

The package targets .NET 8 and .NET 10 and builds on Selenium.WebDriver 3.141.0.

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

The [repository README](https://github.com/collana-solutions/Csag.BuildAutomation.Selenium#readme) describes all helpers and options.

---
"@collana-solutions/csag-automation-selenium": major
---

The package moves to a new ID and namespace, and to .NET 8 and .NET 10. The API and the behavior are the same as in 0.1.0.

**New package ID:**

| Up to 0.1.0 | From 1.0.0 |
| --- | --- |
| `Neolution.Automation.Selenium` | `Csag.Automation.Selenium` |

**To migrate:**

1. Replace the `Neolution.Automation.Selenium` package reference with `Csag.Automation.Selenium` 1.0.0 or later.
2. Change the namespaces: `using Neolution.Automation.Selenium;` becomes `using Csag.Automation.Selenium;`, and the same for `.Exceptions`, `.Extensions` and `.Screenshot`.
3. The package targets `net8.0` and `net10.0` instead of `netstandard2.0`. Projects on .NET Framework or on .NET versions before 8 stay on `Neolution.Automation.Selenium` 0.1.0.

**Dependencies:** Selenium.WebDriver stays at 3.141.0 and DotNetSeleniumExtras.WaitHelpers at 3.11.0. The package now also references Newtonsoft.Json 13.0.4, because Selenium.WebDriver 3.141.0 only asks for Newtonsoft.Json 10.0.3, which has a high severity vulnerability (GHSA-5crp-9r3c-p9vr).

The `Neolution.Automation.Selenium` ID gets deprecated on nuget.org, with `Csag.Automation.Selenium` as the alternate. Its existing version stays available, but it gets no new releases.

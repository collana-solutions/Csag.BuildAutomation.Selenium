---
"@collana-solutions/csag-automation-selenium": patch
---

`SelectElement` now picks the expected wait condition once and uses it for both waits. Its behavior is unchanged: it throws the same exceptions, with the same parameter names and messages. The package is now built with the code analyzers of `Csag.CodeAnalysis` 3.3.0, a build-only dependency.

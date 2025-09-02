<!-- (dl
(section-meta
    (title Archer Project Structure and Library Roles)
)
) -->

Everything is in exploration and apt to change.

| Project Name | Short Description | Unique Capability |
| ------------ | ----------------- | ----------------- |
| [Types](https://github.com/ArcherFSharpTesting/Types) | Foundational types and interfaces shared across all Archer libraries. | Test Definitions |
| [Reporting](https://github.com/ArcherFSharpTesting/Reporting) | Formats and outputs test results for Archer, supporting summaries and detailed reports. | Build Report strings for Core Types |
| [Validations](https://github.com/ArcherFSharpTesting/Validations) | Functional, composable assertion framework returning test results, not exceptions. | Assertion |
| [Runner](https://github.com/ArcherFSharpTesting/Runner) | Core engine for executing, filtering, and reporting on Archer tests. | Test Execution |
| [Core](https://github.com/ArcherFSharpTesting/Core) | Main F# testing language for defining, organizing, and running tests. | Test Creation |
| [VSAdapter](https://github.com/ArcherFSharpTesting/VSAdapter) | Integrates Archer with Visual Studio for test discovery and execution. | Test Discovery |

<!-- GENERATED DOCUMENT DO NOT EDIT! -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->

<!-- Compiled with doculisp https://www.npmjs.com/package/doculisp -->

# Archer F# Testing Framework: Documentation and Library Guide #

## Archer F# Testing Framework: Library Overview ##

This document provides a high-level overview of the core libraries that make up the Archer F# testing framework. Archer is designed for clarity, composability, and expressiveness, enabling idiomatic F# test authoring and execution.

---

### Archer.Core ###

Archer.Core is the heart of the Archer framework, providing the main language constructs for defining, organizing, and running tests. It enables:
- Descriptive, natural-language test names
- Composable features and sub-features
- Flexible setup/teardown logic
- Data-driven and tagged tests
- Easy test ignoring and filtering

**Example:**
```fsharp
let feature = FeatureFactory.NewFeature "Math Feature"
let ``Addition should work`` =
    feature.Test (fun _ ->
        let result = 2 + 2
        if result = 4 then TestSuccess else TestFailure "Addition failed"
    )
```

---

### Archer.Types ###

Archer.Types is the foundational types library, providing shared types for assertions, test results, runners, and more. It ensures consistency and type safety across the Archer ecosystem.
- Public types for test authors (e.g., `TestTag`, `TestResult`, `TestCase`)
- Internal types for test execution and lifecycle
- Runner types for managing execution and results
- Helper functions for common operations

---

### Archer.Reporting ###

Archer.Reporting handles formatting and outputting test results. It provides:
- Readable, structured test output
- Summaries and detailed reports
- Integration with the Archer test runner
- Helpers for string formatting and location reporting

---

### Archer.Validations ###

Archer.Validations (Fletcher) provides a functional, composable approach to test assertions. Instead of throwing exceptions, validations return `TestResult` values, supporting:
- Composability and functional pipelines
- Object, result, boolean, list, sequence, and array validations
- Approval (golden master) testing
- Extensible, idiomatic F# usage

**Example:**
```fsharp
let result = Should.BeTrue true
let result = [1;2;3] |> ListShould.Contain 2
```

---

### Archer.Runner ###

Archer.Runner is the core test execution engine. It supports:
- Serial and parallel test execution
- Filtering by tags or categories
- Lifecycle events for custom hooks
- Aggregated reporting of results
- Extensibility for custom runners

**Example:**
```fsharp
let runner = RunnerFactory().Runner()
runner.AddTests myTests
let report = runner.Run()
```

---

### Archer.VSAdapter ###

Archer.VSAdapter provides integration points for running Archer tests within Visual Studio and other compatible environments. (See the library for details.)

---

For more details, see the individual library READMEs and source files.

## Archer Project Structure and Library Roles ##

Everything is in exploration and apt to change.

| Project Name | Short Description | Unique Capability |
| ------------ | ----------------- | ----------------- |
| [Types](https://github.com/ArcherFSharpTesting/Types) | Foundational types and interfaces shared across all Archer libraries. | Test Definitions |
| [Reporting](https://github.com/ArcherFSharpTesting/Reporting) | Formats and outputs test results for Archer, supporting summaries and detailed reports. | Build Report strings for Core Types |
| [Validations](https://github.com/ArcherFSharpTesting/Validations) | Functional, composable assertion framework returning test results, not exceptions. | Assertion |
| [Runner](https://github.com/ArcherFSharpTesting/Runner) | Core engine for executing, filtering, and reporting on Archer tests. | Test Execution |
| [Core](https://github.com/ArcherFSharpTesting/Core) | Main F# testing language for defining, organizing, and running tests. | Test Creation |
| [VSAdapter](https://github.com/ArcherFSharpTesting/VSAdapter) | Integrates Archer with Visual Studio for test discovery and execution. | Test Discovery |

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->
<!-- GENERATED DOCUMENT DO NOT EDIT! -->
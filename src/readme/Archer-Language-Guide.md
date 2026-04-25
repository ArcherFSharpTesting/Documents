# Archer Test Language Guide

A comprehensive guide to the Archer F# test language syntax and patterns for writing tests.

## Table of Contents

1. [Module Structure](#module-structure)
2. [Feature Definition](#feature-definition)
3. [Test Definition](#test-definition)
4. [Test Context and Parameters](#test-context-and-parameters)
5. [Setup and Teardown](#setup-and-teardown)
6. [Data-Driven Tests](#data-driven-tests)
7. [Tags and Categorization](#tags-and-categorization)
8. [Assertions and Validations](#assertions-and-validations)
9. [Sub-Features](#sub-features)
10. [Ignoring Tests](#ignoring-tests)
11. [Test Results](#test-results)
12. [Complete Examples](#complete-examples)

---

## Module Structure

Every Archer test file is an F# module. Use double backticks for natural language module names:

```fsharp
module MyProject.Tests.``Calculator Should``

open Archer
open Archer.Core
```

For assertions, open the Validations namespace:

```fsharp
open Archer.Validations  // or just: open Archer
```

---

## Feature Definition

### Basic Feature

```fsharp
let feature = FeatureFactory.NewFeature ()
```

### Named Feature

```fsharp
let feature = FeatureFactory.NewFeature "Calculator Tests"
```

### Feature with Path and Name

```fsharp
let feature = FeatureFactory.NewFeature ("Math.Calculator", "Addition Tests")
```

### Feature with Tags

```fsharp
let feature = FeatureFactory.NewFeature (
    TestTags [
        Category "Unit"
        Category "Fast"
    ]
)
```

### Feature with Setup

```fsharp
let feature = FeatureFactory.NewFeature (
    Setup (fun () -> Ok calculatorInstance)
)
```

### Feature with Teardown

```fsharp
let feature = FeatureFactory.NewFeature (
    Teardown (fun () -> 
        // cleanup code
        Ok ()
    )
)
```

### Feature with Setup and Teardown

```fsharp
let feature = FeatureFactory.NewFeature (
    Setup (fun () -> Ok resource),
    Teardown (fun resource -> 
        resource.Dispose()
        Ok ()
    )
)
```

### Feature with Everything

```fsharp
let feature = FeatureFactory.NewFeature (
    "Math.Calculator",
    "Complex Tests",
    TestTags [ Category "Integration"; Serial ],
    Setup (fun () -> Ok setupData),
    Teardown (fun setupData -> Ok ())
)
```

---

## Test Definition

### Basic Test with Automatic Naming

The identifier name becomes the test name:

```fsharp
let ``should add two numbers correctly`` =
    feature.Test (fun _ ->
        let result = 2 + 2
        result |> Should.BeEqualTo 4
    )
```

### Test with Explicit Name

```fsharp
feature.Test (
    "Addition Test",
    fun _ ->
        let result = 2 + 2
        result |> Should.BeEqualTo 4
)
```

### Test Returning TestSuccess

```fsharp
let ``should succeed`` =
    feature.Test (fun _ ->
        // test logic
        TestSuccess
    )
```

### Test Returning TestFailure

```fsharp
let ``should fail with message`` =
    feature.Test (fun _ ->
        TestFailure "This test deliberately fails"
    )
```

---

## Test Context and Parameters

### No Setup - Ignore Context

```fsharp
let ``test without setup`` =
    feature.Test (fun _ ->
        // Underscore ignores the context parameter
        TestSuccess
    )
```

### With Setup - Use Context

```fsharp
let featureWithSetup = FeatureFactory.NewFeature (
    Setup (fun () -> Ok 42)
)

let ``test with setup`` =
    featureWithSetup.Test (fun ctx ->
        ctx.TestSetupValue |> Should.BeEqualTo 42
    )
```

### With Environment

```fsharp
let ``test with environment`` =
    feature.Test (fun ctx env ->
        // env contains RunnerEnvironment information
        env.RunEnvironment.RunnerName |> Should.NotBeNull
    )
```

---

## Setup and Teardown

### Setup Returns Result

Setup must return `Result<'T, SetupTeardownFailure>`:

```fsharp
Setup (fun () -> Ok setupValue)
Setup (fun () -> Error (GeneralSetupTeardownFailure ("Setup failed", location)))
```

### Teardown Receives Setup Result

```fsharp
Teardown (fun setupResult ->
    match setupResult with
    | Ok value -> 
        // cleanup with value
        Ok ()
    | Error _ -> 
        // handle setup failure
        Ok ()
)
```

### Teardown Receives Setup Result and Test Result

```fsharp
Teardown (fun setupResult testResult ->
    match testResult with
    | Some TestSuccess -> printfn "Test passed"
    | Some (TestFailure _) -> printfn "Test failed"
    | None -> printfn "Test not run"
    Ok ()
)
```

### Per-Test Setup/Teardown

```fsharp
let ``test with local setup`` =
    let localFeature = FeatureFactory.NewFeature (
        Setup (fun () -> Ok "local setup"),
        Teardown (fun _ -> Ok ())
    )
    
    localFeature.Test (fun ctx ->
        ctx.TestSetupValue |> Should.BeEqualTo "local setup"
    )
```

---

## Data-Driven Tests

### Basic Data Test

```fsharp
let ``test with data`` =
    let testData = [1; 2; 3; 4; 5]
    
    let dataFeature = FeatureFactory.NewFeature (
        Data testData
    )
    
    dataFeature.Test (fun ctx value ->
        value |> Should.BeGreaterThan 0
    )
```

### Data Test with Multiple Parameters

```fsharp
let ``addition tests`` =
    let testData = [
        (1, 1, 2)
        (2, 3, 5)
        (10, 5, 15)
    ]
    
    let dataFeature = FeatureFactory.NewFeature (
        "Addition",
        Data testData
    )
    
    dataFeature.Test (fun ctx input1 input2 expected ->
        let result = input1 + input2
        result |> Should.BeEqualTo expected
    )
```

### Data Test with Setup

```fsharp
let ``data test with setup`` =
    let testData = [1; 2; 3]
    
    let dataFeature = FeatureFactory.NewFeature (
        Setup (fun () -> Ok calculatorInstance),
        Data testData
    )
    
    dataFeature.Test (fun ctx value ->
        let result = ctx.TestSetupValue.Add(value, 1)
        result |> Should.BeEqualTo (value + 1)
    )
```

---

## Tags and Categorization

### Available Tag Types

```fsharp
TestTags [
    Category "Unit"        // Categorize tests
    Category "Integration" // Multiple categories allowed
    Serial                 // Run serially (not in parallel)
    Only                   // Run only tests marked with Only
]
```

### Tags on Feature

```fsharp
let feature = FeatureFactory.NewFeature (
    TestTags [ Category "Unit"; Category "Fast" ]
)
```

### Tags on Individual Test

```fsharp
let ``tagged test`` =
    feature.Test (
        TestTags [ Category "Slow" ],
        fun _ -> TestSuccess
    )
```

### Combining Feature and Test Tags

```fsharp
let feature = FeatureFactory.NewFeature (
    TestTags [ Category "Unit" ]
)

// This test will have both "Unit" and "Critical" tags
let ``critical test`` =
    feature.Test (
        TestTags [ Category "Critical" ],
        fun _ -> TestSuccess
    )
```

---

## Assertions and Validations

All assertions are in the `Archer` namespace (from `Archer.Validations`).

### Equality Assertions

```fsharp
actual |> Should.BeEqualTo expected
actual |> Should.NotBeEqualTo unexpected
actual |> Should.BeSameAs expected  // Reference equality
```

### Boolean Assertions

```fsharp
condition |> Should.BeTrue
condition |> Should.BeFalse
```

### Null/Default Assertions

```fsharp
value |> Should.BeNull
value |> Should.NotBeNull
value |> Should.BeDefaultOf<int>
value |> Should.NotBeDefaultOf<string>
```

### Type Assertions

```fsharp
value |> Should.BeOfType<string>
value |> Should.NotBeOfType<int>
```

### Option Assertions

```fsharp
someValue |> Should.BeSome
noneValue |> Should.BeNone
Some 42 |> Should.BeSome
```

### Result Assertions

```fsharp
okResult |> Should.BeOk value
errorResult |> Should.BeError errorValue
```

### Collection Assertions

```fsharp
list |> Should.Contain 3
list |> Should.NotContain 99
list |> Should.HaveLength 5
list |> Should.BeEmpty
list |> Should.NotBeEmpty
```

List-specific:

```fsharp
list |> ListShould.Contain 3
list |> ListShould.HaveLengthOf 5
list |> ListShould.HaveAllValuesPassTestOf (<@ fun x -> x > 0 @>)
```

Sequence-specific:

```fsharp
seq |> SeqShould.Contain 3
seq |> SeqShould.HaveLengthOf 5
```

Array-specific:

```fsharp
array |> ArrayShould.Contain 3
array |> ArrayShould.HaveLengthOf 5
```

### Chaining Assertions

```fsharp
result
|> Should.BeEqualTo 42
|> Should.BeOfType<int>
```

```fsharp
list
|> Should.HaveLength 5
|> Should.Contain 3
|> Should.NotBeEmpty
```

### Custom Messages

```fsharp
result 
|> Should.BeEqualTo 42 
|> withMessage "The calculation should return 42"
```

### Predicate Assertions

```fsharp
value |> Should.PassTestOf (<@ fun x -> x > 0 @>)
value |> Should.NotPassTestOf (<@ fun x -> x < 0 @>)
```

### Multiple Assertions

```fsharp
value |> Should.PassAllOf [
    Should.BeGreaterThan 0
    Should.BeLessThan 100
    Should.BeOfType<int>
]
```

---

## Sub-Features

Sub-features allow hierarchical organization of tests.

### Basic Sub-Feature

```fsharp
let parentFeature = FeatureFactory.NewFeature "Parent"

let subFeature = Sub.Feature "Child Feature"
```

### Sub-Feature with Setup

```fsharp
let parentFeature = FeatureFactory.NewFeature (
    Setup (fun () -> Ok parentValue)
)

let subFeature = Sub.Feature (
    "Child",
    Setup (fun parentValue -> Ok childValue)
)
```

### Sub-Feature with Tags

```fsharp
let subFeature = Sub.Feature (
    "Child",
    TestTags [ Category "SubTest" ],
    Setup (fun parent -> Ok child)
)
```

### Complete Sub-Feature Example

```fsharp
module MyTests.``Calculator Should``

let feature = FeatureFactory.NewFeature (
    Setup (fun () -> Ok (new Calculator()))
)

let ``Addition Tests`` = Sub.Feature (
    "Addition",
    TestTags [ Category "Math" ]
)

let ``should add positive numbers`` =
    ``Addition Tests``.Test (fun ctx ->
        let calc = ctx.TestSetupValue
        let result = calc.Add(2, 3)
        result |> Should.BeEqualTo 5
    )
```

---

## Ignoring Tests

### Ignore a Test

```fsharp
let ``ignored test`` =
    feature.Ignore (fun _ ->
        // This test will not run
        TestSuccess
    )
```

### Ignore a Feature

```fsharp
let ignoredFeature = FeatureFactory.Ignore ()
```

### Ignore with Message

```fsharp
let ``ignored with reason`` =
    feature.Test (fun _ ->
        TestIgnored (Some "Not implemented yet", location)
    )
```

### Conditional Ignore

```fsharp
let ``conditionally ignored`` =
    feature.Test (fun _ ->
        if not isImplemented then
            TestIgnored (Some "Feature not ready", location)
        else
            // run test
            TestSuccess
    )
```

---

## Test Results

### TestSuccess

Indicates the test passed:

```fsharp
TestSuccess
```

### TestFailure

Indicates the test failed. Usually returned by assertions:

```fsharp
TestFailure "Explicit failure message"
```

Or from assertions:

```fsharp
actual |> Should.BeEqualTo expected  // Returns TestFailure if not equal
```

### TestIgnored

Indicates the test was skipped:

```fsharp
TestIgnored (Some "Reason for ignoring", location)
TestIgnored (None, location)
```

### Combining Results

```fsharp
let result1 = value1 |> Should.BeTrue
let result2 = value2 |> Should.BeTrue

// Both must pass for combined result to pass
result1 + result2
```

### Result from Assertions

All `Should.*` assertions return `TestResult`:

```fsharp
let ``assertion returns result`` =
    feature.Test (fun _ ->
        42 |> Should.BeEqualTo 42  // Returns TestSuccess
    )
```

---

## Complete Examples

### Simple Unit Test

```fsharp
module MyApp.Tests.``String Extensions Should``

open Archer
open Archer.Core

let private feature = FeatureFactory.NewFeature (
    TestTags [ Category "Unit" ]
)

let ``reverse a string correctly`` =
    feature.Test (fun _ ->
        let result = StringExtensions.reverse "hello"
        result |> Should.BeEqualTo "olleh"
    )

let ``handle empty strings`` =
    feature.Test (fun _ ->
        let result = StringExtensions.reverse ""
        result |> Should.BeEmpty
    )

let ``Test Cases`` = feature.GetTests ()
```

### Test with Setup and Teardown

```fsharp
module MyApp.Tests.``Database Should``

open Archer
open Archer.Core

let private feature = FeatureFactory.NewFeature (
    TestTags [ Category "Integration" ],
    Setup (fun () -> 
        let db = Database.connect connectionString
        db.initialize()
        Ok db
    ),
    Teardown (fun dbResult ->
        match dbResult with
        | Ok db -> 
            db.cleanup()
            db.disconnect()
        | Error _ -> ()
        Ok ()
    )
)

let ``insert and retrieve record`` =
    feature.Test (fun ctx ->
        let db = ctx.TestSetupValue
        let record = { Id = 1; Name = "Test" }
        
        db.insert record
        let retrieved = db.get 1
        
        retrieved |> Should.BeEqualTo record
    )

let ``Test Cases`` = feature.GetTests ()
```

### Data-Driven Test

```fsharp
module MyApp.Tests.``Calculator Should``

open Archer
open Archer.Core

let ``add numbers correctly`` =
    let testData = [
        (0, 0, 0)
        (1, 1, 2)
        (5, 3, 8)
        (-1, 1, 0)
        (100, 200, 300)
    ]
    
    let dataFeature = FeatureFactory.NewFeature (
        "Addition Tests",
        TestTags [ Category "Unit" ],
        Data testData
    )
    
    dataFeature.Test (fun ctx a b expected ->
        let result = Calculator.add a b
        result 
        |> Should.BeEqualTo expected
        |> withMessage $"Adding {a} + {b} should equal {expected}"
    )

let ``Test Cases`` = [``add numbers correctly``]
```

### Hierarchical Tests with Sub-Features

```fsharp
module MyApp.Tests.``Calculator Should``

open Archer
open Archer.Core

let private feature = FeatureFactory.NewFeature (
    TestTags [ Category "Unit" ]
)

let ``Basic Operations`` = Sub.Feature (
    "Basic Operations",
    TestTags [ Category "Math" ]
)

let ``Addition`` = Sub.Feature (
    "Addition",
    TestTags [ Category "Addition" ]
)

let ``should add positive numbers`` =
    ``Addition``.Test (fun _ ->
        let result = Calculator.add 2 3
        result |> Should.BeEqualTo 5
    )

let ``should add negative numbers`` =
    ``Addition``.Test (fun _ ->
        let result = Calculator.add (-2) (-3)
        result |> Should.BeEqualTo -5
    )

let ``Subtraction`` = Sub.Feature (
    "Subtraction",
    TestTags [ Category "Subtraction" ]
)

let ``should subtract numbers`` =
    ``Subtraction``.Test (fun _ ->
        let result = Calculator.subtract 5 3
        result |> Should.BeEqualTo 2
    )

let ``Test Cases`` = 
    feature.GetTests () @ 
    ``Addition``.GetTests () @ 
    ``Subtraction``.GetTests ()
```

### Complex Test with Multiple Patterns

```fsharp
module MyApp.Tests.``User Service Should``

open Archer
open Archer.Core
open System

let private feature = FeatureFactory.NewFeature (
    TestTags [ Category "Integration"; Serial ],
    Setup (fun () ->
        let service = UserService.create()
        service.initialize()
        Ok service
    ),
    Teardown (fun serviceResult ->
        match serviceResult with
        | Ok svc -> svc.shutdown()
        | Error _ -> ()
        Ok ()
    )
)

let ``create new user successfully`` =
    feature.Test (fun ctx ->
        let service = ctx.TestSetupValue
        let user = { 
            Id = Guid.NewGuid()
            Name = "Test User"
            Email = "test@example.com"
        }
        
        let result = service.createUser user
        
        result 
        |> Should.BeOk 
        |> withMessage "User creation should succeed"
    )

let ``validate email format`` =
    let invalidEmails = [
        "not-an-email"
        "@example.com"
        "missing-domain@"
        ""
    ]
    
    let emailFeature = FeatureFactory.NewFeature (
        Setup (fun () -> Ok (UserService.create())),
        Data invalidEmails
    )
    
    emailFeature.Test (fun ctx email ->
        let service = ctx.TestSetupValue
        let user = { 
            Id = Guid.NewGuid()
            Name = "Test"
            Email = email
        }
        
        let result = service.createUser user
        
        result 
        |> Should.BeError
        |> withMessage $"Email '{email}' should be invalid"
    )

let ``handle concurrent requests`` =
    feature.Test (
        TestTags [ Category "Concurrency" ],
        fun ctx ->
            let service = ctx.TestSetupValue
            
            let tasks = [1..10] |> List.map (fun i ->
                async {
                    let user = { 
                        Id = Guid.NewGuid()
                        Name = $"User {i}"
                        Email = $"user{i}@example.com"
                    }
                    return service.createUser user
                }
            )
            
            let results = 
                tasks 
                |> Async.Parallel 
                |> Async.RunSynchronously
            
            results 
            |> Array.forall Result.isOk
            |> Should.BeTrue
    )

let ``Test Cases`` = feature.GetTests ()
```

---

## Summary

The Archer test language provides:

1. **Natural naming** using F# identifiers with backticks
2. **Flexible features** with setup, teardown, tags, and data
3. **Composable tests** that return results instead of throwing exceptions
4. **Rich assertions** through the `Should.*` API
5. **Hierarchical organization** with sub-features
6. **Data-driven testing** with the `Data` parameter
7. **Pipeline-friendly** assertions that chain together

This syntax enables writing clear, maintainable, and expressive tests that read like documentation.

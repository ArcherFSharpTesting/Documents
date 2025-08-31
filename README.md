# Archer

## Project Layout

Everything is in exploration and apt to change.

| Project Name | Short Description | Unique Capability | Uses |
| ------------ | ----------------- | ----------------- | ---- |
| [Types](https://github.com/ArcherFSharpTesting/Archer.CoreTypes) | This will hold interfaces that are shared between all the above. | Test Definitions | None |
| [Reporting](https://github.com/ArcherFSharpTesting/Archer.Logger) | A Logger used to report on Archer test framework | Build Report strings for Core Types | CoreTypes |
| [Validations](https://github.com/ArcherFSharpTesting/Archer.Fletching) | The Assertion Framework | Assertion | CoreTypes |
| [Runner](https://github.com/ArcherFSharpTesting/Archer.Bow) | Execution Engine | Test Execution | CoreTypes |
| [Core](https://github.com/ArcherFSharpTesting/Archer.Arrow) | The Testing Language | Test Creation | CoreTypes |
| [VSAdapter](https://github.com/ArcherFSharpTesting/Archer.Quiver) | Visual Studio Test Adapter | Test Discovery | Bow, CoreTypes |

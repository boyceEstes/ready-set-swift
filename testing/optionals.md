# Optionals

Optionals are a key componenet of the Swift language. How should you access optional properties in Swift during your unit tests?

## Force-Unwrapping
You can be much more leniant in the testing target with force-unwrapping than you *should* be in the main target of your app. 

The rule of thumb is to only force-unwrap something if it is not the System Under Test (sut). You can just assume that everything works appropriately. If it doesn't that should be covered in its own tests, or covered better.

The reason that you might want to avoid force-unwrapping is if you want the testing to continue to the other cases after this error, instead of crashing the program, it might be better to just get a single failed test.


## Sources
[QualityCoding - Unit testing optionals](https://qualitycoding.org/unit-test-optionals-swift/)

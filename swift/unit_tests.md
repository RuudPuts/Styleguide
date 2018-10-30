# Unit test guidelines

Wring unit tests can be done in a great many ways, but all of them have some default rules to adhere to. Further more we recommend using Quick & Nimble for unit tests in Swift, with writing mocks as described below.

## General guidelines

- Never use a shared instance of a class in any part of a unit test, i.e.
```swift
UserDefaults.sharedInstance
UIApplication.sharedApplication
```
- Never use any form of (async) waiting directly in the tests
```swift
sleep()
NSRunLoop.current
while (value != expected) { }
```

To wait for (async) code to finish use `XCTestExpectation` when writing a `XCTest` or `expectEventually` when writing a spec using `Quick & Nimble`

## Quick & Nimble guidelines

Quick & Nimble provide 5 important DSL keywords, below follow which should be used in different scenarios.

See [PriceFormatterSpec.swift](https://jmb.gitlab.schubergphilis.com/JumboApp/jumbo-ios/blob/a5d61a077169ea88f4439bae5e06becb82601975/Jumbo/Tests/JumboTests/Products/Resources/PriceFormatterSpec.swift) for a example following these guidelines.

### Describe

Use describe when `starting a spec function`. This is also the place where the `sut` is defined:
```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            ...
        }
    }
}

```

The other use of describe is to describe `a piece functionality within the class` under test. 

**Rule of thumb**: When adding a `beforeEach` statement in a `describe` it should only prepare for `context` calls to follow. Don't execute any code on the class under test. 

For example a `negate()` function within a `Bool` class;

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            describe("negating the value") {
                ...
            }
        }
    }
}

```


- Context

Use context to define the different flows of the a certain functionality within the `describe` closure, for example;

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            describe("negating the value") {
                context("negating a true value") {
                    ...
                }

                context("negating a false value") {
                    ...
                }
            }
        }
    }
}

```

- BeforeEach

BeforeEach should only be used within `describe` or `context` closures. Use beforeEach for;
1. Preparing mocks & test input data
2. Creating or modifying the `sut`
3. Execute the code under test (as defined in the `describe` closure)

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            describe("negating the value") {
                context("negating a true value") {
                    beforeEach {
                        sut = true
                        sut.negate()
                    }
                }

                context("negating a false value") {
                    beforeEach {
                        sut = false
                        sut.negate()
                    }
                }
            }
        }
    }
}

```

- It

It is where assertions are executed in the form of an `expect()` call.

**Rule of thumb**: Only execute one `expect` call per `it` closure

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            describe("negating the value") {
                context("negating a true value") {
                    beforeEach {
                        sut = true
                        sut.negate()
                    }

                    it("sets the value to false") {
                        expect(sut) == false
                    }
                }

                context("negating a false value") {
                    beforeEach {
                        sut = false
                        sut.negate()
                    }

                    it("sets the value to true") {
                        expect(sut) == true
                    }
                }
            }
        }
    }
}

```

- AfterEach

AfterEach should only be used within `describe` or `context` closures. Use afterEach for;
1. Nillify all variables instantiated in the current `describe` / `context`
2. Any other cleanup your test specificly need

**note**: Setting all variables to `nil` technically isn't nessecary, but it's a small effort to add them and have certainty all values are reset correctly.


```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            afterEach {
                sut = nil
            }

            describe("negating the value") {
                context("negating a true value") {
                    beforeEach {
                        sut = true
                        sut.negate()
                    }

                    it("sets the value to false") {
                        expect(sut) == false
                    }
                }

                context("negating a false value") {
                    beforeEach {
                        sut = false
                        sut.negate()
                    }

                    it("sets the value to true") {
                        expect(sut) == true
                    }
                }
            }
        }
    }
}

```
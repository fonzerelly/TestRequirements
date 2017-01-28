# Testing Requirements

## Preface

### Why do we need tests?
0. Proper Tests describe how your functions should work
0. Helps you to better reason about your code and how it should be structured
0. Allows us to release your code more quickly because it prooves itself that it works
0. Allows us more easily to refactore your code if necessary, because we see very fast if a refactoring broke your code.

Therefore we need you to follow the following rules. But these rules should
not be seen as unbreakable. If you have good reasons to not follow them,
please contact us and discuss your use case. Maybe you know a better Testing Framework
that we recommend or there are a few parts of your app that for specific reasons can
not be tested easily. This might be ok. But we need no know exactly about them to
estimate the risk in it.

### The Test Pyramid
There are roughly three kinds of tests that are relevant for us:
* UnitTests
* IntegrationTests
* System/GUITests

Each of these tests come with different Pros & Cons:

#### UnitTests
The goal of an UnitTest is, to test a small unit of your application,
* a class,
* a service,
* a directive

in separation to the rest of all its dependencies. This makes UnitTests
very easy in creation and maintenance.

#### IntegrationTests
Instead of the nature of the UnitTests, an IntegrationTest tries to test
the cooperation of several Units. This is helpful, when units get combined
by some kind of configuration. But since IntegrationTests are spread among
several Units it is much harder to setup an IntegrationTest.

#### System/ GUI-Tests
But what even the IntegrationTest can not do, is to proove that the system as
a whole works as expected. But the SystemTest comes with two major flaws:
0. The dependencies to the backend are so complex, that you can not setup
test data as you want it. This means it does not make sense to test on this level
that a specific value is where you expect it to be, because you do not know the
value and you can not identify it.
0. GUI-Tests are usually done with Selenium or better Protractor. Both have to
evaluate the tests asynchroniously to the GUI, which makes them quite flaky.
Every here and then it happens that a click happens too early or too late, so that
the expected output can not be reproduced validly, although the prodcution code works
fine. This circumstance is called FalsePositives and makes those test not as reliable
as you need it to be to have confidence in refactoring code.

This concept is called TestPyramid and was first mentioned in Mike Cohn's book "Succeed with Agile"
Its basic interpretation is, due to the nature of each Test we should build
our Software on a massive fundament of UnitTests that ensure that each Unit works for itself.
What we can not Test on that level should be better Tested as IntegrationTest, so that we only
have the setup overhead only where it is necessary. The GUI-Tests should only define the main
click throughs of the user, so that we at least can proofe those to work as expected but keep
the affort of mainaining them low. So this is our Testing strategy:

* Test as much as you can as UnitTest
* Use IntegrationTests where necessary
* Use GUI-Tests only for basic User Workflows


## What should be tested?
Everything should be tested. Be it a component or a service, you need to test it. For every
production file like "my-service.ts" there should be at least one .spec.ts file as "my-service.spec.ts".
If your test file seems to become too big or you feel the need write tests with different points
of view you can write several .spec.ts files: "my-service_prod.spec.ts", "my-service_local.spec.ts"
For angular you also need to test the markup of your Components. So similarily when you test the
method of a service, you check for the outcome, you have to test if the compiled output of a directive
looks as you exepct it. (See also https://docs.angularjs.org/guide/unit-testing)

## The Testing Strategy is a Pyramid
Each of these tests kinds of tests
UnitTests





##
* separation between "private functions" and interface
* Break up code for easier testing
* Parametrized Tests
* red - green - cycle
* evil review sample
* tests must not depend on each other!
* test strategy
* mockdata
* jasmine
 * when to use describe
 * beforeEach afterEach
 * Exclusive Tests (fit fdescribe), Turn out tests (xit xdescribe)
 * how to do mocking
 * testing asynchronious code



## How should be tested

```
var x = 2;
if (x == 2) {
    console.log("doof");
}
```


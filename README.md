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


# Requirements
## Everything should be tested
Everything should be tested. Be it a component or a service, you need to test it.
Let your tests be prepared, that we randomly modify your production code, to see
if the corresponding tests fail as expected.

## Naming Convention
For every production file like "my-service.ts" there must be at least one .spec.ts file as "my-service.spec.ts".
If your test file seems to become too big or you feel the need write tests with different points
of view you can write several .spec.ts files: "my-service_prod.spec.ts", "my-service_local.spec.ts"

## Structure of a test
As already mentioned, the tests have also a documentation function.
To do so, you have to keep a specific structure of the test.
Refering to jasmine, this means you should start with the name of the
module you want to test. Cascadingly you should start a describe-block,
that describes the specific method you try to test.
If you have to check the behaviour of a method under the circumstances
of different states of your class, service or directive, you should cascade
another directive-block inside the methods describe-block starting with "when ..."
to describe the specific circumstance you want to test.

Here is an example:
```
describe('myModule', () => {
    describe('methodOfModule', () => {
        describe('when state 1 happend', () => {
            it('should do this', () => {
            })
        })
        describe('when state 2 happened', () => {
            it('should to that', () => {
            })
        })
    })
})
```

## Interindependence of Tests
Be sure that tests do not build on each other. Sometimes you make this
error accidently by hardcoding stuff in the initialization of a service.
Therefor you have to make sure to randomize the order of you tests, so that
you see such accidents as soon as possible.

## Usage of realworld mockdata
When you are implementing against a REST-API, it might change. What you want
is to have tests that fail, if your REST-API changes, so that you can adapt
your code easily to it. Therefore you have to have some possibility to easily
* grap mockdata from your REST-API
* update your mockdata
* and use that exact same mockdata

This will also force you not write agains specific values in your tests, but
against a value location in your mock data:



# Recommendations

## Test driven development
To fullfill the above stated requirements, you should apply Test driven development
into your development process. This means that you first have to write a test for a
specific feature, see it fail and only then write the production code to fullfill the test.

### The Red-Green-Cycle
Let us do this in a small example. Let's say we want to write an increment method
in a MyMathService. How would we start off with that?
```
MyMath = require ('./my-math')
describe('MyMath', () => {

    describe('increment', () => {
        it('should return 4 for 3', () => {
            expect(MyMath.increment(3)).toBe(4)
        })
    })
})
```
If you run the test now, you will get the error,
that my-math.js can not be found or later that increment
is not part of that module. But this is not, what this test
aims for. This test aims for a result of an increment-call.
So** make sure that the test fails due to the reason
it needs to fail** , by setting up the my-math module with
its increment method:
```
function increment () {
}

module.exports = {
    increment: increment
}
```

If we run the test now, we can see an error which is all about
the result of the increment-call: "undefined is not 4"

When we implement the tested method:
```
function increment (n) {
    return 4
}
```
The test gets green. This gives us the opportunity to:
* refactor the production code now if we are not yet satisfied with it
* start writing a new test to triangulate the feature.

### Triangulation
As you might have recognized we realized a very poor implementation
to fullfill the test by simply returning 4. Using increment with another input
than 3 would not return the expecte value.

Having said that, for more complex algorithm, it is completly valid to start with
a poor implementation and improving it iteration for iteration.

So let us write another test to make sure,
```
...
    describe('increment', () => {
        it('should return 4 for 3', () => {
            expect(MyMath.increment(3)).toBe(4)
        })

        it('should return 5 for 4', () => {
            expect(MyMath.increment(4)).toBe(5)
        });
    })
...
```
Running this test will show, how poor our implementation really is. The new test will fail.
But this shows us a new direction to modify the algorithm:
```
function increment(n) {
    return n+1
}
```
As obvious this case is, let us stop here. Let me summarize the idea of Triangulation for you:
* start with simple tests and simple implementations
* nail the functionality of your algorithm down by writing more tests
* keep on doing it until your algorithm satisfies all requirements
* when there are more requirements to your method return to the tests and keep on doing so again

### Parametrized Tests
Writing down every single test for one method can be quite tedious. If you have methods with
some kind of mapping or a large value space, you might be better of with a so called parametrized test:
```
...
    describe('increment', () => {
        [
            {param1: 1, result: 2},
            {param1: 2, result: 3},
            {param1: 3, result: 4}
        ].forEach((testSpec) => {
            it(`should return $testSpec.result for $testSpec.param1`, () => {
                expect(MyMath.increment(testSpec.param1)).toBe(testSpec.result)
            });
        })
    })
...
```
This helps you to not write too much test code but yet you can still recognize for parameters
you methods fail.





https://martinfowler.com/bliki/TestPyramid.html
For angular you also need to test the markup of your Components. So similarily when you test the
method of a service, you check for the outcome, you have to test if the compiled output of a directive
looks as you exepct it. (See also https://docs.angularjs.org/guide/unit-testing)





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


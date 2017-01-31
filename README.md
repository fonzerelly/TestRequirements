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

This will also force you not to write agains specific values in your tests, but
against a value location in your mock data:
```
const MyService = require('./my-service')
describe('MyService', () => {
    let mockData
    beforeEach(inject($httpBackend) => {
        mockData = loadMockdata('http://my-rest-service/my-data')
	$httpBackend
            .when('GET', 'http://my-rest-service/my-data')
            .respond(mockData);
    })

    describe('myMethod', () => {
        it('should return name of data', () => {
            expect(MyService.myMethod()).toEqual(mockData.name);
	})
    })
})
```
Please see also section [MockData](#mockdata) in the Recommendations



**You must not copy mockdata manually into your tests!**



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

## When Tests get too complex
After knowing the foundation of writing automated tests now, we know the basics to do standard
craftsman work. What makes testing not only an art but also lifts your coding skills is the
necessity to decide which interface you want to test against with. Very often it is the best,
to simply test against the component you needed to implement anyhow. Still you are fee to divide
you code into private functions:
```
    // these functions can not be accessed fom outside directly,
    // they are private.
    function aFunction() {
        ...
    }

    function anotherFunction() {
        ...
    }

    module.exports = {
        interfaceToTestAgainst: function(params) {
            const value1 = aFunction()
            const value2 = anotherFunction(params)

            return value1 + value2
        }
    }
```

But when you feel that you tests get too complicated or that there are aspects you can not test,
then you should consider to split up your code into two separate units, which you can test separately:

```
// Split out methods to be tested for themselves

module.exports = {
    aFunction: () => {}
    anotherFunction: () => {}
}

// Original File
const Methods = require('./file1')

    module.exports = {
        interfaceToTestAgainst: function(params) {
            const value1 = Methods.aFunction()
            const value2 = Methods.anotherFunction(params)

            return value1 + value2
        }
    }
```
So use your intuition for interfaces wisely.

<a name="mockdata"></a>
## Mockdata
If you want your tests to fail if your REST-Services change you better use realworld mockdata that
can be easily updated to modified REST-Interfaces. This solves Mockingjay.js, which provides a
local webserver, that has two modes:
* a READONLY-Mode which ensures that mock data gets provided as you stored it
* an OVERWRITE-Mode which allows you to forward all requests against the Mockingjay-Server to your
  original REST-Service

This allows you to write Integrationtests or simply a set of REST-Calls that stores Mockdata for
you. With that you can simply update your Mockdata if necessary. In the meantime you can use that
server to run your tests reproducably.

## Additional Jasmine Tipps
###Exclusive Testing
If you have a huge Test Suite, what we hope you have, running your TestSuite can take sevaral
minutes. Nevertheless we want to encurage you, to keep your tests runnning in an endless loop
meanwhile you are developing, so that you get immediate feedback if your codechange worked or not.
To make this much less cumbersome, you can use "fdescribe" or "fit" to only run the tests that
reflect the state  of the service you are working on:
```
const MyService = require('./my-service')
describe('MyService', () => {
    it('this test will not be executed', () => {
        ...
    })

    fdescribe('myMethod', () => {
        it('this test will be executed', () => {
            ...
        })
        it('this test will also be executed', () => {
            ...
        })
    })

    describe('myOtherMethod', () => {
        it('this test will also not be executed', () => {
            ...
        })
        fit('but this test will be executed too', () => {
            ...
        })
    })
})
```
Similar to fdescribe and fit, you can turn of single tests or descriptions with "xdescribe" and
"xit". **Be careful that you do not commit fdescribe or fit to your repository. Use git hooks to prevent that**

### Testing asynchrone Code
When you need to test for example an http-request or other asynchrone code use the 'done'-callback
provided by jasmine:
```
    it('should call done in the future', (done) => {
        new Promise ((resolve, reject) => {
            resolve(42)
        }).then((value) => {
            expect(value).toEqual(42)
            done()
        })
    })
```
Any later test will only be executed after a timeout or you called done.


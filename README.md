# nose-random

`nose-random` is designed to facilitate [Monte-Carlo style](https://en.wikipedia.org/wiki/Monte_Carlo_method) unit testing. The idea is to improve testing by running your code against a large number of randomly generated input scenarios.

Even with random testing it's important that test success/failure is reproducible, otherwise it's hard to
* know if you've fixed a failing test
* know if an test fails only on some machines or configurations and not others
* debug a failing test

`nose-random` avoids this pitfall because it 
* uses a fixed seed so that each test run is identical
* tells you which scenario caused a test to fail
* lets you to run the test only on a specific scenario to facilitate debugging

## Installation

    pip install git+https://github.com/ZoomerAnalytics/nose-random.git
    
or clone the repo and run `python setup.py install`.
    
## Usage

The following [example](examples/tests.py) shows how to set up a randomized test.

```python
# tests.py
import unittest

from nose_random import randomize

class RandomTestCase(unittest.TestCase):
    
    def generate_scenario(self, rng):
        return rng.random(), 10 * rng.random()
        
    @randomize(1000, generate_scenario)
    def failing_test(self, scenario):
        x, y = scenario
        self.assertLess(x, y)

    @randomize(1000, generate_scenario)
    def passing_test(self, scenario):
        x, y = scenario
        self.assertLess(x, y + 1)
```

Running `nosetests` from the same folder will produce the following output:

    F.
    ======================================================================
    FAIL: failing_test (tests.RandomTestCase)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "c:\dev\nose-random\nose_random\__init__.py", line 62, in randomized_test
        test(self, scenario)
      File "C:\Dev\nose-random\examples\tests.py", line 13, in failing_test
        self.assertLess(x, y)
    AssertionError: 0.9903329105632304 not less than 0.0015605977653532221 with scenario 2FDO1B08J20A (27 of 1000)
    
    ----------------------------------------------------------------------
    Ran 2 tests in 0.067s
    
This means that the `failing_test` function succeeded with the first 26 scenarios but then failed with the 27th. To debug this scenario you can run the same command specifying the scenario id.

    nosetests --scenario=2FDO1B08J20A

This will cause the tests to only be run against the failing scenario (note '1 of 1' as opposed to '27 of 1000').

    AssertionError: 0.9903329105632304 not less than 0.0015605977653532221 with scenario 2FDO1B08J20A (1 of 1)
    
# PyCharm

Note that the module will also pick up the `--scenario=` string if passed as a parameter to a PyCharm unittest run/debug configuration.

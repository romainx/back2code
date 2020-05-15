---
title: 'Some pytest best practices'
date: '2020-05-15'
categories: ['dev']
tags: ['python', 'pytest']
---

Recently I had to use the R test framework called **[‌testthat][LK1]**. I noticed that this framework is very *human oriented*. Its name comes from the way to write tests that aims to make a readable sentence. 

<!--more-->

For example "Test that `str_length` is number of characters" becomes.

```R
test_that("str_length is number of characters", {
  expect_equal(str_length("a"), 1)
  expect_equal(str_length("ab"), 2)
  expect_equal(str_length("abc"), 3)
})
```

In the chapter [Testing][LK2] of the book *[R packages][LK3]* some guidelines are given on the *functional* way to write tests. For example:

> Tests are organised hierarchically: **expectations** are grouped into **tests** which are organised in **files**:
> - An **expectation** is the atom of testing. It describes the expected result of a computation: Does it have the right value and right class? Does it produce error messages when it should? An expectation automates visual checking of results in the console. Expectations are functions that start with `expect_`.
> - A **test** groups together multiple expectations to test the output from a simple function, a range of possibilities for a single parameter from a more complicated function, or tightly related functionality from across multiple functions. This is why they are sometimes called unit as they test one unit of functionality. A test is created with `test_that()`.
> - A **file** groups together multiple related tests. Files are given a human readable name with `context()`.

So I have thought of a way to use the same kind of guidelines / conventions in Python with `[pytest][LK4]`.

## Pytest best practices

It's not exhaustive at all nor a full--boring and long--list of guidelines. Just a couple of things I try to apply each time I have to write some tests in Python.

I'm starting by the example and I will summarize the main points below.

```Python
logger = logging.getLogger(__name__)

# 1. using arguments to be able to identify them easily
# ease parametrization in a second step
def test_eval_operation(operation="3 + 4", expected=6):
    # 2. instead of commenting logging a clear message saying what is tested
    test_that = f"{operation} = {expected}"
    logger.info(f"Test that {test_that}")
    actual = eval(operation)
    # 3. actual left, expected right
    # 4. proper error message with error left and what we got right
    assert actual == expected, f"{test_that} failed, got {actual}"
```

1. **Put inputs and output in the function parameters**: It permits to identify them easily--since they are clearly visible at the beginning of the method--and it will ease parametrization(running the same test with different inputs and outputs).
2. **Log a message instead of commenting the function**: I'm always try to log a clear message at the beginning of the test. It replaces the method comment and will be written in the test output. So it's easier to figure out what the test is actually trying to check.
3. **Write always assertion in the same way**: In assertions put always actual at the left side of the assertion and the expected result at the right side.
4. **Write assertion human readable error message**: Take the time to write clearly what is the error it will save debug time later.

Running this test produces the following output.

```bash
$ pytest_demo pytest

# [...]
# ---------- live log call ----------
# 2020-05-15 07:22:33 [    INFO] Test that 3 + 4 = 6 (test_sample.py:13)
# FAILED [100%]
# ========== FAILURES ==========
# /Users/romain/Git/pytest_demo/test_sample.py:17: AssertionError: 3 + 4 = 6 failed, got 7
# ========== short test summary info ==========
# FAILED test_sample.py::test_eval_operation - AssertionError: 3 + 4 = 6 failed, got 7
# ========== 1 failed in 0.04s ==========
```

## Parametrization

Once tests are written in this way it's **easy to reuse them through parametrization**. Parametrization permits to reuse the same test (code) with different conditions. Here is the same example with parametrization.

```python
@pytest.mark.parametrize(
    "operation,expected",
    [("2 + 4", 6), pytest.param("3 + 4", 6, marks=pytest.mark.xfail)],
)
def test_eval_operation_param(operation, expected):
    test_that = f"{operation} = {expected}"
    logger.info(f"Test that {test_that}")
    actual = eval(operation)
    assert actual == expected, f"{test_that} failed, got {actual}"
```

In the example above the same test has been reused only by extracting the parameters from the function to the `pytest.mark.parametrize` decorator. Nothing more to do. Note also the usage of the special `pytest.mark.xfail` to indicate that **we expect this test to fail**, see [`parametrize`][LK5] for the full list of features.

Running these tests produces the following output.

```bash
$ pytest_demo pytest

# [...]
# ---------- live log call ----------
# 2020-05-15 07:34:54 [    INFO] Test that 2 + 4 = 6 (test_sample.py:26)
# PASSED  [ 50%]
# test_sample.py::test_eval_operation_param[3 + 4-6] 
# ---------- live log call ----------
# 2020-05-15 07:34:54 [    INFO] Test that 3 + 4 = 6 (test_sample.py:26)
# XFAIL  [100%]
# ========== short test summary info ==========
# PASSED test_sample.py::test_eval_operation_param[2 + 4-6]
# XFAIL test_sample.py::test_eval_operation_param[3 + 4-6]
# ========== 1 passed, 1 xfailed in 0.05s ==========
```

## Pytest output tuning

To produce the outputs in this way I have used a customization of `pytest.ini`.

```toml
[pytest]
# Default options:
#   - `--tb=line` stand for shorter traceback format (only one line per failure)
#   - `-rfEPpxX` Select the short summary info to display
# See https://docs.pytest.org/en/latest/usage.html#modifying-python-traceback-printing
addopts = --tb=line -rfEPpxX
log_cli = 1
log_cli_level = INFO
log_cli_format = %(asctime)s [%(levelname)8s] %(message)s (%(filename)s:%(lineno)s)
log_cli_date_format=%Y-%m-%d %H:%M:%S
```

Note: I've put both files in a [gist][LK6].

[LK1]: https://testthat.r-lib.org/index.html
[LK2]: https://r-pkgs.org/tests.html
[LK3]: https://r-pkgs.org/
[LK4]: https://docs.pytest.org/en/latest/
[LK5]: https://docs.pytest.org/en/latest/parametrize.html
[LK6]: https://gist.github.com/romainx/6cfe532e177234fc9028b69206433650
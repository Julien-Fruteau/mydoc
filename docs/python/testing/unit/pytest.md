---
title: Pytest
tags:
  - python
  - test
  - unit test
  - pytest
  - mock
  - pytest-mock
---

## pytest-mock

Usage example of the ```pytest_mock``` module

### Mock object method

```python
from pytest_mock import MockerFixture

class A:
    def echo(self, text: str):
        fmt_text = text.upper()
        return fmt_text


def test_foo(mocker: MockerFixture):
    a = A()
    mocker.patch.object(a, 'echo', return_value="MOCKED", autospec=True)
    assert a.echo("hello") == "MOCKED"


def test_foo_no_mock():
    a = A()
    assert a.echo("hello") == "HELLO"
```

### Mock module function

- my_package.my_module file :

```python
def my_function_to_mock(text: str):
    return text.capitalize()

def function_using_function_to_mock(text: str):
    use: str = my_function_to_mock(text)
    return use + " Function"
```

- testing :

```python
from pytest_mock import MockerFixture
from my_package.my_module import function_using_function_to_mock

def test_function_using_function_to_mock(mocker: MockerFixture):
    mocker.patch("my_package.my_module.my_function_to_mock", return_value="MOCKED")
    actual = function_using_function_to_mock("hello")
    assert actual == "MOCKED Function"

def test_function_without_mocking(mocker: MockerFixture):
    actual = function_using_function_to_mock("hello")
    assert actual == "Hello Function"

```

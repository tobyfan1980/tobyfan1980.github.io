---
layout: post
title:  "Use decorator to control failures in unittest"
date:   2017-01-19 20:06:05
categories: python
tags: python unittest
---

Sometime, we want to test how our program reacts if a function raise some exceptions.
Thus in unittest we what to purposely inject some exception for the function call.
Furthermore, we also want to test the idempotency, i.e., if program works when calling the function after previous failure.
Therefore, we want to control when to fire exception for the function call. The following code shows how to fire exception on the xth call,
and fire exception for the first x calls. 

```python
import aspects
import itertools
from hosting_platform.testing.fixtures import default_fixture_name, get_default_fixture_name

def default_fixture_name(name):
    """decorator to give a fixture function a default fixture name"""
    def dec(func):
        """decorator function"""
        func.default_fixture_name = name
        return func
    return dec

@default_fixture_name('failure_wrapper')
def fail_on_xth_fixture(xth=1, exception=Exception, message="Error occurred.", **kwargs):
    assert xth > 0

    counter = itertools.count()

    def failure_wrapper():
        def failure(_self, *args, **kwargs):
            if counter.next() == (xth - 1):
                raise exception(message)
            else:
                yield aspects.proceed
                yield aspects.return_stop
        return failure

    return failure_wrapper()


@default_fixture_name('failure_wrapper')
def fail_x_times_fixture(count=1, exception=Exception, message="Error occurred.", **kwargs):
    counter = itertools.count()

    def failure_wrapper():
        def failure(_self, *args, **kwargs):
            if counter.next() < count:
                raise exception(message)
            else:
                yield aspects.proceed
                yield aspects.return_stop
        return failure

    return failure_wrapper()
```

To use these wrappers, we just need to add them before the unittest case fuction. 

```python
@fixture(fail_on_xth_fixture, xth=1, exception=TimeoutError)
def test_mytestcase(self, failure_wrapper, **kwargs):
    '''
    Testcase description
    '''
    # intentionally inject aws failure when provisioning vpn at the first time
    aspects.with_wrap(failure_wrapper, target_function)

    # verify the failure and status
    self.assertRaises(TimeoutError, function_to_invoke_the_target_function)
    
    # verify the 2nd call success
    aspects.without_wrap(failure_wrapper, target_function)
        
    function_to_invoke_the_target_function()
```


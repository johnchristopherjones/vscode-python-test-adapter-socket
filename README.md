# Minimal reproduction of test failure

You might want to restrict sockets to during tests to ensure network calls are
prevented. There's a plugin for [pytest](https://github.com/pytest-dev/pytest) to do
this, called [pytest_socket](https://github.com/miketheman/pytest-socket).

The microsoft/vscode-python extension for Visual Studio Code fails when sockets
are blocked by the test suite. The culprit appears to early, in "Test Results":

```
vscode_pytest.VSCodePytestError: Error attempting to connect to extension communication socket[vscode-pytest]: A test tried to use socket.socket.connect() with host "localhost" (allowed: "localhost").
```

As of vscode 1.83.1 and vscode-python v2023.19.13031019, tests will work if you disable
the `pythonTestAdapter` experiment in Settings:

```json
"python.experiments.optOutFrom": ["pythonTestAdapter"],
```

# Steps to reproduce

## Always fails

1. Setup — standard, whatever you like. E.g.:
    1. Install dependencies

        ```pdm install```

    2. "Select Interpreter…" and pick `./.venv/bin/activate`

2. Open `test_always_fail.py`

3. "Run Tests in Current File" (⌘; F)

4. View results in "Test Results" and Test Explorer

## With `--disable-socket`

1. Setup — standard, whatever you like. E.g.:

    1. Install dependencies

        ```pdm install```

    2. "Select Interpreter…" and pick `./.venv/bin/activate`

2. Open `pyproject.toml` and uncomment line 16 (`--disable-socket`) and optionally 17-18:

    ```toml
    # "--disable-socket",    # Disable network in test with pytest-socket
    # "--allow-unix-socket", # Allow unix sockets for async
    # "--allow-hosts=localhost",
    ```

3. Open `test_init_option_fail.py`

4. "Run Tests in Current File" (⌘; F)

5. View results in "Test Results" and Test Explorer

## Supposition: what's going on?

VSCode appears to run the test suite using `run_pytest_script.py`. E.g.,

```sh
$HOME/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/run_pytest_script.py
```

This script invokes `pytest` using `pytest.run()` in the same process and assumes it can
open ports in the same process as the test runner.

# Tests Results - Full Output

```
CLIENT: Server listening on port 50821...
Received JSON data in run script
============================= test session starts ==============================
platform darwin -- Python 3.11.6, pytest-7.4.3, pluggy-1.3.0
rootdir: /Users/jcj/src/vscode-python-test-adapter-socket
configfile: pyproject.toml
plugins: socket-0.6.0
collected 1 item

tests/test_always_fail.py Error attempting to connect to extension communication socket[vscode-pytest]: A test tried to use socket.socket.
If you are on a Windows machine, this error may be occurring if any of your tests clear environment variables as they are required to communicate with the extension. Please reference https://docs.pytest.org/en/stable/how-to/monkeypatch.html#monkeypatching-environment-variablesfor the correct way to clear environment variables during testing.


INTERNALERROR> Traceback (most recent call last):
INTERNALERROR>   File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 721, in send_post_request
INTERNALERROR>     __socket.connect()
INTERNALERROR>   File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/testing_tools/socket_manager.py", line 32, in connect
INTERNALERROR>     self.socket = socket.socket(
INTERNALERROR>                   ^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pytest_socket.py", line 80, in __new__
INTERNALERROR>     raise SocketBlockedError()
INTERNALERROR> pytest_socket.SocketBlockedError: A test tried to use socket.socket.
INTERNALERROR> 
INTERNALERROR> During handling of the above exception, another exception occurred:
INTERNALERROR> 
INTERNALERROR> Traceback (most recent call last):
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/main.py", line 271, in wrap_session
INTERNALERROR>     session.exitstatus = doit(config, session) or 0
INTERNALERROR>                          ^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/main.py", line 325, in _main
INTERNALERROR>     config.hook.pytest_runtestloop(session=session)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_hooks.py", line 493, in __call__
INTERNALERROR>     return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_manager.py", line 115, in _hookexec
INTERNALERROR>     return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 152, in _multicall
INTERNALERROR>     return outcome.get_result()
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_result.py", line 114, in get_result
INTERNALERROR>     raise exc.with_traceback(exc.__traceback__)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 77, in _multicall
INTERNALERROR>     res = hook_impl.function(*args)
INTERNALERROR>           ^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/main.py", line 350, in pytest_runtestloop
INTERNALERROR>     item.config.hook.pytest_runtest_protocol(item=item, nextitem=nextitem)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_hooks.py", line 493, in __call__
INTERNALERROR>     return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_manager.py", line 115, in _hookexec
INTERNALERROR>     return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 152, in _multicall
INTERNALERROR>     return outcome.get_result()
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_result.py", line 114, in get_result
INTERNALERROR>     raise exc.with_traceback(exc.__traceback__)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 77, in _multicall
INTERNALERROR>     res = hook_impl.function(*args)
INTERNALERROR>           ^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/runner.py", line 114, in pytest_runtest_protocol
INTERNALERROR>     runtestprotocol(item, nextitem=nextitem)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/runner.py", line 133, in runtestprotocol
INTERNALERROR>     reports.append(call_and_report(item, "call", log))
INTERNALERROR>                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/runner.py", line 226, in call_and_report
INTERNALERROR>     hook.pytest_runtest_logreport(report=report)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_hooks.py", line 493, in __call__
INTERNALERROR>     return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_manager.py", line 115, in _hookexec
INTERNALERROR>     return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 113, in _multicall
INTERNALERROR>     raise exception.with_traceback(exception.__traceback__)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 77, in _multicall
INTERNALERROR>     res = hook_impl.function(*args)
INTERNALERROR>           ^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/terminal.py", line 574, in pytest_runtest_logreport
INTERNALERROR>     *self.config.hook.pytest_report_teststatus(report=rep, config=self.config)
INTERNALERROR>      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_hooks.py", line 493, in __call__
INTERNALERROR>     return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_manager.py", line 115, in _hookexec
INTERNALERROR>     return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 152, in _multicall
INTERNALERROR>     return outcome.get_result()
INTERNALERROR>            ^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_result.py", line 114, in get_result
INTERNALERROR>     raise exc.with_traceback(exc.__traceback__)
INTERNALERROR>   File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 62, in _multicall
INTERNALERROR>     next(wrapper_gen)  # first yield
INTERNALERROR>     ^^^^^^^^^^^^^^^^^
INTERNALERROR>   File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 233, in pytest_report_teststatus
INTERNALERROR>     execution_post(
INTERNALERROR>   File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 660, in execution_post
INTERNALERROR>     send_post_request(payload)
INTERNALERROR>   File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 732, in send_post_request
INTERNALERROR>     raise VSCodePytestError(error_msg)
INTERNALERROR> vscode_pytest.VSCodePytestError: Error attempting to connect to extension communication socket[vscode-pytest]: A test tried to use socket.socket.
Error attempting to connect to extension communication socket[vscode-pytest]: A test tried to use socket.socket.
If you are on a Windows machine, this error may be occurring if any of your tests clear environment variables as they are required to communicate with the extension. Please reference https://docs.pytest.org/en/stable/how-to/monkeypatch.html#monkeypatching-environment-variablesfor the correct way to clear environment variables during testing.

Traceback (most recent call last):
  File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 721, in send_post_request
    __socket.connect()
  File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/testing_tools/socket_manager.py", line 32, in connect
    self.socket = socket.socket(
                  ^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pytest_socket.py", line 80, in __new__
    raise SocketBlockedError()
pytest_socket.SocketBlockedError: A test tried to use socket.socket.

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/run_pytest_script.py", line 67, in <module>
    pytest.main(arg_array)
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/config/__init__.py", line 169, in main
    ret: Union[ExitCode, int] = config.hook.pytest_cmdline_main(
                                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_hooks.py", line 493, in __call__
    return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_manager.py", line 115, in _hookexec
    return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 113, in _multicall
    raise exception.with_traceback(exception.__traceback__)
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 77, in _multicall
    res = hook_impl.function(*args)
          ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/main.py", line 318, in pytest_cmdline_main
    return wrap_session(config, _main)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/main.py", line 306, in wrap_session
    config.hook.pytest_sessionfinish(
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_hooks.py", line 493, in __call__
    return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_manager.py", line 115, in _hookexec
    return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 130, in _multicall
    teardown[0].send(outcome)
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/_pytest/terminal.py", line 857, in pytest_sessionfinish
    outcome.get_result()
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_result.py", line 114, in get_result
    raise exc.with_traceback(exc.__traceback__)
  File "/Users/jcj/src/vscode-python-test-adapter-socket/.venv/lib/python3.11/site-packages/pluggy/_callers.py", line 77, in _multicall
    res = hook_impl.function(*args)
          ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 367, in pytest_sessionfinish
    execution_post(
  File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 660, in execution_post
    send_post_request(payload)
  File "/Users/jcj/.vscode/extensions/ms-python.python-2023.19.13031019/pythonFiles/vscode_pytest/__init__.py", line 732, in send_post_request
    raise VSCodePytestError(error_msg)
vscode_pytest.VSCodePytestError: Error attempting to connect to extension communication socket[vscode-pytest]: A test tried to use socket.socket.
Finished running tests!

```
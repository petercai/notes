You can do this cleanly with a **custom marker** (annotation) and a small plugin in `conftest.py` that interprets the marker and runs the test at a fixed rate for a fixed duration.

Below is a complete, minimal setup that lets you write:

import pytest

  

@pytest.mark.rate(limit="1/s", duration="5m")

def test_my_endpoint(api_client__):_

    _# one invocation's logic – this will be executed once per second for 5 minutes_

    _resp = api_client.get("/health")

    assert resp.status_code == 200

…and Pytest will automatically run that test **once per second for 5 minutes**.

---

## 1) Declare the marker in `pytest.ini`

This avoids “unknown marker” warnings.

# pytest.ini

[pytest]

markers =

    rate(limit, duration): run a test repeatedly at a given rate for a given duration (e.g., limit="1/s", duration="5m")

---

## 2) Implement the runner in `conftest.py`

This plugin:

- Parses `limit` (e.g. `"1/s"`) and `duration` (e.g. `"5m"`).
- Executes the test body once per “tick”.
- Sleeps to maintain the requested cadence (1 Hz).
- Aggregates failures and stops early (optional).

# conftest.py

import math

import time

import pytest

  

def _parse_rate(limit_str__: str) -> float:_

    _"""_

    _Parse rate strings like "1/s", "2/sec", "0.5/s" into Hz (invocations per second)._

    _"""_

    _s = limit_str.strip().lower()

    if s.endswith("/s") or s.endswith("/sec") or s.endswith("/second"):

        value = s.split("/")[0]

        hz = float(value)

    else:

        raise ValueError(f"Unsupported limit format: {limit_str!r}. Use e.g. '1/s'."__)_

    _if hz <= 0:_

        _raise ValueError("Rate must be > 0")_

    _return hz_

_def_ parse_duration(duration_str: str) -> float:

    """

    Parse duration like "300s", "5m", "1.5m" into seconds.

    Supports s/sec/seconds and m/min/minutes.

    """

    s = duration_str__.strip().lower()_

    _if s.endswith(("s", "sec", "seconds")):_

        _num = s.rstrip("seconds").rstrip("sec").rstrip("s")_

        _seconds = float(num)_

    _elif s.endswith(("m", "min", "minute", "minutes")):_

        _num = s.rstrip("minutes").rstrip("minute").rstrip("min").rstrip("m")_

        _seconds = float(num) * 60.0_

    _else:_

        _raise ValueError(f"Unsupported duration format: {duration_str!r}. Use e.g. '300s' or '5m'.")

    if seconds <= 0:

        raise ValueError("Duration must be > 0")

    return seconds

  

def pytest_configure__(config):_

    _# Optional: register marker for --strict-markers environments_

    _config.addinivalue_line(

        "markers",

        "rate(limit, duration): run a test repeatedly at a given rate for a given duration"

    )

  

@pytest.hookimpl(tryfirst=True)

def pytest_runtest_call(item):

    """

    If a test function has @pytest.mark.rate(limit=…, duration=…),

    run it repeatedly according to the specified rate/duration.

    """

    marker = item.get_closest_marker("rate")

    if not marker:

        # No rate marker: let pytest run normally

        return

  

    limit = marker.kwargs.get("limit")

    duration = marker.kwargs.get("duration")

  

    if not isinstance(limit, str) or not isinstance(duration, str):

        raise pytest.UsageError("rate marker requires string kwargs: limit='1/s', duration='5m'")

  

    hz = _parse_rate(limit)

    total_seconds_ _=_ parse_duration(duration)_

    _period = 1.0 / hz_

    _# We'll repeatedly call the test body via the standard hook to ensure fixtures are applied_

    _start = time.perf_counter()

    next_tick_ _= start_

    _end = start + total_seconds

  

    failures = []

    iterations = 0

  

    # Loop until duration elapses

    while True:

        now = time.perf_counter__()_

        _if now >= end:_

            _break_

        _# Sleep until next tick to maintain cadence_

        _if now < next_tick:

            time.sleep(next_tick_ _- now)_

        _# Execute the test function one time (with fixtures)

        try:_

            _item.ihook.pytest_pyfunc_call(pyfuncitem=item)_

        _except Exception as e:_

            _failures.append(e)_

            _# Optional: stop early on first failure. Comment this out to continue collecting failures._

            _# break_

        _iterations += 1_

        _next_tick += period

  

    # If any failures occurred, raise an aggregated assertion error

    if failures:

        # Build a readable message

        msgs = []

        for i, exc in enumerate(failures, 1):

            msgs.append(f"[{i}] {type(exc).**name**}: {exc}")

        message = (

            f"{item.name} failed {len(failures)} "

            f"time(s) during rate-limited run ({limit} for {duration}).\n" +

            "\n".join(msgs)

        )

        raise AssertionError(message)

  

    # If no failures, we’re done; suppress default call by returning None

    return

### How it works

- `pytest_runtest_call` is the phase where Pytest normally calls a test function.
- If your test is marked `@pytest.mark.rate(...)`, we intercept that phase and call the test body multiple times ourselves using `item.ihook.pytest_pyfunc_call(pyfuncitem=item)`. That ensures fixtures are still applied correctly each iteration.
- We enforce the cadence with `time.sleep()` so you get ~1 invocation per second.

---

## 3) Write tests with the marker

# tests/test_service.py_

_import pytest_

_  

@pytest.mark.rate(limit="1/s", duration="5m")

_

_def test_healthcheck(api_client__):_

    _resp = api_client.get("/health")

    assert resp.status_code == 200

---

## Options & Tweaks

- **Failure strategy**\ In the example, I collect all failures and raise a summary at the end. If you’d prefer to **stop on first failure**, un-comment the `break` in the `except` block.
    
- **Capturing output / logs**\ Pytest’s capture will aggregate output across iterations. If you want per-iteration logging, include iteration counters in your test, or log to a file/ELK with a timestamp.
    
- **Measuring throughput / iterations**\ You can add counters or metrics (e.g., to `pytest.Item` via `item.user_properties`) if you want the exact count reported.
    
- **Accuracy**\ Sleep-based scheduling in Python is “good enough” for 1 Hz, but it’s not hard real-time. If the test body takes longer than 1 second, the next tick will drift. You can adjust to catch-up by recomputing `next_tick = max(next_tick + period, time.perf_counter())`.
    
- **Parallelism**\ If you use `pytest-xdist`, each worker will run its own loop. That means N workers × 1/s each unless you gate the rate globally (e.g., with a lock or a shared limiter).
    
- **Per-test configuration**\ You can have different rate/duration per test by changing the marker arguments.
    

---

## Quick sanity check

If you want to verify the cadence, you can include timestamps:

import pytest, time

  

@pytest.mark.rate(limit="1/s", duration="5m")

def test_tick():

    print(f"tick at {time.strftime('%H:%M:%S')}")

    assert True

You should see ~one line per second for ~5 minutes.

---

## Want a fixture-based alternative?

If you prefer the test to be **called once** and you loop inside it, you can expose a helper via a fixture:

# conftest.py

import time

import pytest

  

@pytest.fixture

def rate_runner__():_

    _def run(fn, *, limit_hz: float, duration_s__: float, stop_on_first_failure: bool = True):

        period = 1.0 / limit_hz_

        _start = time.perf_counter()

        next_tick_ _= start_

        _end = start + duration_s

        failures = []

        iterations = 0

        while True:

            now = time.perf_counter__()_

            _if now >= end:_

                _break_

            _if now < next_tick:

                time.sleep(next_tick_ _- now)_

            _try:_

                _fn()_

            _except Exception as e:_

                _failures.append(e)_

                _if stop_on_first_failure:

                    break

            iterations += 1

            next_tick += period

        if failures:

            raise AssertionError(f"Failed {len(failures)} iteration(s): {failures}")

        return iterations

    return run

Usage:

import pytest

  

@pytest.mark.rate(limit="1/s", duration="5m")

def test_something__(rate_runner):

    def one_call():

        # one invocation logic

        assert 1 + 1 == 2

    # Read marker and run

    m = pytest.mark.rate

…but since you asked for an **annotation-driven** approach with no extra boilerplate in the test, the plugin hook solution above is tidier.

---

If you want, I can package this as a small reusable plugin (e.g., `pytest-rate`) so you can `pip install` it internally and reuse across projects.



# pytest
## pytest_asyncio

No, `pytest` does **not** support `asyncio` by default. If you try to run an `async` test function directly with `pytest`, you'll get an error like:

```
RuntimeWarning: coroutine 'test_async_function' was never awaited
```

### 🔹 How to Enable `asyncio` Support in `pytest`
To properly run `async` tests, you need **pytest-asyncio**:

#### 1️⃣ Install `pytest-asyncio`
```sh
pip install pytest-asyncio
```

#### 2️⃣ Use `@pytest.mark.asyncio`
Mark your async test functions with `@pytest.mark.asyncio` so `pytest` runs them within an `asyncio` event loop.

```python
import pytest
import asyncio

async def async_function():
    await asyncio.sleep(1)
    return "Hello, Async!"

@pytest.mark.asyncio
async def test_async_function():
    result = await async_function()
    assert result == "Hello, Async!"
```

#### 3️⃣ Configure `pytest.ini` (Optional)
To avoid using `@pytest.mark.asyncio` for every test, set the mode globally in `pytest.ini`:
```ini
[pytest]
asyncio_mode = auto
```
This automatically runs all `async` tests in an event loop.

#### 4️⃣ Run Tests
```sh
pytest
```

### 🔹 Alternative: Use `pytest_asyncio.plugin` Fixture
Instead of marking each test with `@pytest.mark.asyncio`, you can use `pytest_asyncio.plugin`:
```python
@pytest.fixture
async def some_async_fixture():
    return await async_function()

async def test_async_with_fixture(some_async_fixture):
    assert some_async_fixture == "Hello, Async!"
```

### ✅ Summary
- `pytest` **does NOT** support `asyncio` by default.
- Use `pytest-asyncio` (`pip install pytest-asyncio`).
- Mark async tests with `@pytest.mark.asyncio` or configure `pytest.ini`.
 
## asyncio_default_fixture_loop_scope

To solve the `PytestDeprecationWarning` related to the `asyncio_default_fixture_loop_scope` configuration option, you need to explicitly set the default fixture loop scope in your `pytest` configuration file (e.g., `pytest.ini`, `tox.ini`, or `pyproject.toml`).

Here are the steps to resolve this warning:

1. **Locate or Create a Configuration File:**
   - If you don't already have a `pytest.ini`, `tox.ini`, or `pyproject.toml` file in your project root, you can create one.

2. **Set the `asyncio_default_fixture_loop_scope`:**
   - Add the following configuration to your `pytest.ini` file:
     ```ini
     [pytest]
     asyncio_default_fixture_loop_scope = function
     ```

   - If you are using `tox.ini`, add it under the `[pytest]` section:
     ```ini
     [pytest]
     asyncio_default_fixture_loop_scope = function
     ```

   - If you are using `pyproject.toml`, add it under the `[tool.pytest.ini_options]` section:
     ```toml
     [tool.pytest.ini_options]
     asyncio_default_fixture_loop_scope = "function"
     ```


3. **Choose an Appropriate Scope:**
   - The scope can be set to `"function"`, `"class"`, `"module"`, `"package"`, or `"session"`. The choice depends on your specific needs and the behavior you want to achieve with your asynchronous fixtures. `"function"` is a common choice as it provides a new event loop for each test function.

4. **Run Your Tests Again:**
   - After setting the configuration, run your tests again to ensure the warning is resolved.

By explicitly setting the `asyncio_default_fixture_loop_scope`, you avoid the deprecation warning and ensure that your tests behave as expected in future versions of `pytest-asyncio`.

## pytest code coverage

You can run pytest with code coverage using the `pytest-cov` plugin. Here’s how:

### 1. **Install `pytest-cov` (if not installed)**
```sh
pip install pytest-cov
```

### 2. **Run pytest with coverage**
```sh
pytest --cov=<your_package_or_module>
```
For example:
```sh
pytest --cov=my_project
```
This will measure test coverage for the `my_project` package.

### 3. **Generate a coverage report**
- **Show coverage summary in the terminal**:
  ```sh
  pytest --cov=my_project --cov-report=term
  ```
- **Generate an HTML report** (viewable in a browser):
  ```sh
  pytest --cov=my_project --cov-report=html
  ```
  This creates an `htmlcov` directory containing the report.
  
- **Generate an XML report** (useful for CI/CD tools):
  ```sh
  pytest --cov=my_project --cov-report=xml
  ```

### 4. **Exclude files from coverage**
You can exclude files by adding a `.coveragerc` file:
```ini
[run]
omit = 
    tests/*
    my_project/some_file.py
```

### 5. **Combine multiple test runs**
If you need to combine coverage from multiple test runs (e.g., different test environments):
```sh
pytest --cov=my_project --cov-append
```

### pip install -e .

The -e . (editable) mode is crucial in this context. It ensures that when you make changes to the code in either openbb or openbb_core, those changes are immediately available without having to reinstall the packages every time.

### pip show openbb

```bash
pip show openbb
```
You should see an output that confirms it is installed and the `Location` should be in your project.

## @pytest.mark.record_http
`@pytest.mark.record_http` is not a built-in marker in `pytest` or `pytest-httpx`, but it is likely a custom marker used in a project that records HTTP interactions during test runs. This is commonly done using tools like **vcrpy**, **pytest-vcr**, or **pytest-recording**.

### Possible Uses:
1. **Recording HTTP requests for later replay**
   - Some testing frameworks use this to capture API responses and save them in a cassette (YAML or JSON file).
   - When the test runs again, it replays the stored response instead of making a real API call.

2. **Integration with pytest-recording or pytest-vcr**
   - If the project uses [`pytest-recording`](https://pypi.org/project/pytest-recording/) or [`pytest-vcr`](https://pypi.org/project/pytest-vcr/), `@pytest.mark.record_http` might indicate that HTTP requests should be recorded for the test.

### Example with `pytest-recording`
```python
import pytest
import requests

@pytest.mark.record_http
def test_fetch_data():
    response = requests.get("https://api.example.com/data")
    assert response.status_code == 200
```
- The first time the test runs, it makes a real HTTP request.
- The response is recorded.
- Subsequent test runs replay the response, making the test faster and avoiding real API calls.

#### How to Check If `record_http` Is Defined?
Run:
```sh
pytest --markers
```
This will list all registered pytest markers, including `record_http`, if it's defined.

If your project uses this marker, check the `pytest.ini`, `conftest.py`, or dependencies to see if `pytest-recording` or another similar plugin is installed.

## register pytest marker
To configure `pytest-recording` for `pytest`, follow these steps:

### 1️⃣ Install `pytest-recording`
```sh
pip install pytest-recording
```

### 2️⃣ Enable `record_http` in `pytest.ini`
Add the following to your `pytest.ini` file to register the marker:
```ini
[pytest]
markers =
    record_http: Record HTTP requests and responses
```

### 3️⃣ Use `@pytest.mark.record_http` in Tests
Annotate your test functions that make HTTP requests:
```python
import pytest
import requests

@pytest.mark.record_http
def test_fetch_data():
    response = requests.get("https://api.example.com/data")
    assert response.status_code == 200
```
- The **first time** the test runs, it will make a real HTTP request and record the response.
- The **next time**, the response is replayed from the recorded data.

### 4️⃣ Configure Cassette Storage (Optional)
By default, recorded HTTP interactions are saved in the `.cache/vcr` directory. You can change this in `pytest.ini`:
```ini
[pytest]
recording_dir = tests/cassettes
```
This stores recorded responses in `tests/cassettes/`.

### 5️⃣ Run Tests with HTTP Recording
Run pytest normally:
```sh
pytest
```
or explicitly specify the recording mode:
```sh
pytest --record-mode=new_episodes  # Records new requests but replays existing ones
```

### Recording Modes:
- `once` (default): Only records requests if no cassette exists.
- `new_episodes`: Replays existing cassettes, but records new requests.
- `none`: Uses only recorded responses, fails if new requests occur.
- `all`: Always makes real HTTP requests and records them.

### 6️⃣ Deleting Old Recordings (if needed)
To clear all recorded interactions:
```sh
rm -rf .cache/vcr  # or tests/cassettes/
```

That's it! Your API tests will now be faster and deterministic while avoiding real API calls. 🚀


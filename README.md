# Poetry Caching Action

> Note: This repo is work in progress...

This action installs Poetry via [`snok/install-poetry`](https://github.com/snok/install-poetry), cache the poetry binary, install dependencies based on your `pyproject.toml` and `poetry.lock` and cache the dependencies.

The cache invalidation is triggered when: 

* **For poetry binary**<br/>The change of runner os, python version, and poetry version changed. It will create multiple cache if you are using matrix strategy.
* **For dependencies**<br/>The change of runner os and python version. If you change the content of `poetry.lock`, actions still download the cache folder but also run the `poetry install` and then save to the cache server. It will also create multiple cache if you are using matrix strategy.


## Improvements:

1. Compaarison with `snok/install-poetry` 
    * Usually take +- 10 seconds
    * Cache poetry binary and it will take around 2-4 seconds
1. Setup python + install poetry + install dependencies (list below) 
    * Usually take +- 34 seconds 
    * Cache dependencies only will take 13-14 seconds
    * Cache poetry and dependencies will take 3-4 seconds 

List of python dependencies for `#2`:

```toml
[tool.poetry.dependencies]
python = ">3.8.1,<3.10"
pydantic = {extras = ["dotenv"], version = "^1.10.2"}
requests = "^2.28.1"
PyMySQL = "^1.0.2"
pymysql-pool = "^0.3.7"
cryptography = "^38.0.1"
```

If you have more dependencies, github action will take more time to download from cache server, but it's usually significant faster than download and install it for every jobs and workflow.

## How to use

### Simple Use Case

We assume you already have `pyproject.toml`, `poetry.lock` and test use case for `pytest` to run this workflow.

Ps:

* Your first `push to main`, the workflow will download poetry and dependencies then save it to cache.
* Your second run *(whether it's on different job, re-run the job or different workflow [based on your first cached commit](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#restrictions-for-accessing-a-cache))* will use the cache and you can see the improvement.
* You can see list of your cache in `Your Repo` -> `Actions` tab -> `Caches` under `Managements` (left navbar, at the bottom). [Don't forget the limitation of cache](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy).

```yml
name: ci

on:
  # Triggers the workflow on push but only for the main branch
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      #----------------------------------------------
      #    Check-out repo and set-up python
      #----------------------------------------------
      - name: Check out repository
        uses: actions/checkout@v3
      - name: "Setup Python, Poetry and Dependencies"
        uses: packetcoders/poetry-caching-action@main
        with:
          python-version: 3.8
          poetry-version: 1.2.2

      #----------------------------------------------
      #    Run Your Actual Job
      #----------------------------------------------
      - name: Run tests
        run: |
          source .venv/bin/activate
          poetry run pytest
```

### Using [Matrix Strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

With matrix strategy, you will create several workflows run simultaneously with the combinations, for this case based on a combination of the python and poetry versions.

```yml
name: ci

on:
  # Triggers the workflow on push but only for the main branch
  push:
    branches: [ main ]

jobs:
  test:
    # Using matrix strategy
    strategy:
      matrix:
        python-version: [3.8, 3.9, 3.10]
        poetry-version: [1.2.2]
    runs-on: ubuntu-latest
    steps:
      #----------------------------------------------
      #    Check-out repo and set-up python
      #----------------------------------------------
      - name: Check out repository
        uses: actions/checkout@v3
      - name: "Setup Python, Poetry and Dependencies"
        uses: packetcoders/poetry-caching-action@main
        with:
          python-version: ${{matrix.python-version}}
          poetry-version: ${{matrix.poetry-version}}

      #----------------------------------------------
      #    Run Your Actual Job
      #----------------------------------------------
      - name: Run tests
        run: |
          source .venv/bin/activate
          poetry run pytest
```
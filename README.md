# GitHub Action - Setup and Cache Python Poetry


This action installs Poetry via [`snok/install-poetry`](https://github.com/snok/install-poetry), and also provides caching for the Poetry binary and your dependencies.

## Cache Creation
When is the cache populated?

* **Poetry binary**<br/>The change of runner OS, Python version, and Poetry version have changed. It will create multiple caches if you are using a matrix strategy.
* **Dependencies**<br/>The change of runner OS, Python version. If you change the content of `poetry.lock`, this Action will still download the cache folder, run the `poetry install`, and then save it to the cache server. It will also create multiple caches if you are using a matrix strategy.

## How to Use

### Simple Use Case

We assume you already have `pyproject.toml`, `poetry.lock` and a test use case for `pytest` to run this workflow example.

Note:

* For your first `push` to `main`, the workflow will download poetry and dependencies and then save it to the cache.
* Your second run (whether it's on a different job, re-run the job or different workflow [based on your first cached commit](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#restrictions-for-accessing-a-cache)) will use the cache and you can see the improvement.
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
      #-------------------------------------#
      #  Check out repo and set up Python   #
      #-------------------------------------#
      - name: Check out the repository
        uses: actions/checkout@v3
      - name: "Setup Python, Poetry and Dependencies"
        uses: packetcoders/action-setup-cache-python-poetry@main
        with:
          python-version: 3.8
          poetry-version: 1.2.2

      #------------------------#
      #  Run your actual job   #
      #------------------------#
      - name: Run tests
        run: |
          poetry run pytest
```

### Using [Matrix Strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

With a matrix strategy, you will create several workflows that run simultaneously with the combinations, for this case based on a combination of the Python and Poetry versions.

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
      #------------------------------------#
      #  Check out repo and set up Python  #
      #------------------------------------#
      - name: Check out the repository
        uses: actions/checkout@v3
      - name: "Setup Python, Poetry and Dependencies"
        uses: packetcoders/action-setup-cache-python-poetry@main
        with:
          python-version: ${{matrix.python-version}}
          poetry-version: ${{matrix.poetry-version}}

      #-----------------------#
      #  Run your actual job  #
      #-----------------------#
      - name: Run tests
        run: |
          poetry run pytest
```

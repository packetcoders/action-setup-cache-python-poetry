# GitHub Action - Setup and Cache Python Poetry

This action simplifies the setup and caching of Poetry, and provides the following functionality for GitHub Actions users:
* Python setup via [`actions/setup-python`](https://github.com/actions/setup-python).
* Poetry install via [`snok/install-poetry`](https://github.com/snok/install-poetry).
* Poetry binary caching via [`actions/cache`](https://github.com/actions/cache).
* Poetry dependency caching via [`actions/cache`](https://github.com/actions/cache).

## Basic Usage

**Note:** 
* We assume you already have `pyproject.toml`, `poetry.lock` and a test module created for `pytest` to run this workflow example.
* For your first `push` to `main`, the workflow will download Poetry and the required project dependencies, then save it to the cache.
* For your second run (whether it's on a different job, re-run the job or a different workflow [based on your first cached commit](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#restrictions-for-accessing-a-cache)) this Action will use the cache.
* You can see list of cache entries by going to: `Your Repo` -> `Actions` tab -> `Caches` under `Managements` (left navbar, at the bottom). 

[Don't forget the limitation of cache](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy).

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

## [Matrix Strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs) Usage

With a matrix strategy, several "workflows" are generated based on your matrix inputs. In this case, multiple caches for each of the matrix workflows will be generated.

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

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE).

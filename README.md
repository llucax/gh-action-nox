# Nox Action

This action runs a [nox](https://github.com/wntrblm/nox/) session.

> [!TIP]
> If you need to do some cross-arch `nox` testing using QEMU you can use the
> [`gh-action-nox-cross-arch`](https://github.com/frequenz-floss/gh-action-nox-cross-arch)
> action.

Here is an example demonstrating how to use it in a workflow with a matrix job:

```yaml
jobs:
  nox:
    name: Test with nox
    strategy:
      fail-fast: false
      matrix:
        os:
          - ubuntu-22.04
        python-version:
          - "3.11"
        nox-session:
          # To speed things up a bit we use the special ci_checks_max session
          # that uses the same venv to run multiple linting sessions
          - "ci_checks_max"
          - "pytest_min"
    runs-on: ${{ matrix.os }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run nox
        uses: frequenz-floss/gh-action-nox@v0.x.x
        with:
          python-version: ${{ matrix.python-version }}
          nox-session: ${{ matrix.nox-session }}
```

## Inputs

* `python-version`: The python version to use. Required.

  This is passed to the
  [`actions/gh-action-setup-python-with-deps`](https://github.com/frequenz-floss/gh-action-setup-python-with-deps/)
  action.

* `nox-session`: The nox session to run. Required.

* `nox-dependencies`: The dependencies to install using `pip` to run `nox`.
  Optional. Default: `".[dev-noxfile]"`.

  Projects not having any extra dependency to run nox can just use `"nox"` here.

## Recommended use with matrix jobs

When using a matrix, it is recommended to create a dummy job to *merge* all the
matrix jobs, specially if you want to require all matrix jobs to pass to allow
merging a pull request. If you do this, you only need to add the dummy job as
a requirement and you don't need to update your requirements each time you
update your matrix.

```yaml
  nox-all:
    # The job name should match the name of the `nox` job.
    name: Test with nox
    needs: ["nox"]
    runs-on: ubuntu-20.04
    steps:
      - name: Return true
        run: "true"
```

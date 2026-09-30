# PrefSampling

[![Build badge](https://github.com/bokveizen/prefsampling/actions/workflows/build.yml/badge.svg)](https://github.com/bokveizen/prefsampling/actions/workflows/build.yml)

This is a maintained fork of [COMSOC-Community/prefsampling](https://github.com/COMSOC-Community/prefsampling).
It retains the original authors' algorithms, attribution, GPL license, and
`prefsampling` import name. The fork's development version is `0.1.25.dev0`.
It includes repairs to seeded spatial sampling and other confirmed correctness
bugs. Install from this repository to use these repairs.

## Overview

PrefSampling is a lightweight Python library that provides preference samplers.
These are algorithms that generate random preferences based on precisely
defined statistical cultures. We consider different type of preferences:

- Ordinal: preferences are expressed as rankings of the candidates;
- Approval: preferences are expressed by indicating a set of approved candidates.

This package is part of the
[Guide to Numerical Experiments on Elections in Computational Social Choice](https://arxiv.org/abs/2402.11765).

## Installation

The maintained fork supports Python 3.10–3.14 and NumPy 1.21.6 or newer
(as compatible with your Python version). Install it directly from GitHub:

```shell
python -m pip install --upgrade "prefsampling @ git+https://github.com/bokveizen/prefsampling.git@main"
```

For reproducible experiments, replace `main` with the full commit hash you
validated. Record `prefsampling.__version__`, NumPy's version, the commit,
and all sampling parameters. Fixed seeds reproduce results within the tested
environment; results can change across versions.

The package on [PyPI](https://pypi.org/project/prefsampling/) is maintained
by upstream and does not include this fork's repairs.

## Documentation

The [upstream documentation](https://comsoc-community.github.io/prefsampling/)
provides the original API reference. Updated fork documentation is built as an
artifact by the [Documentation workflow](https://github.com/bokveizen/prefsampling/actions/workflows/docs.yml),
and can also be built locally.

## Citing our Work

If you are using this package we kindly ask you to cite the following reference to credit our work
[link](https://arxiv.org/abs/2402.11765).

```text

Boehmer N., Faliszewski P., Janeczko Ł., Kaczmarczyk A., Lisowski G., Pierczyński G., Rey S., Stolicki D., Szufa S., Wąs T. (2024).
Guide to Numerical Experiments on Elections in Computational Social Choice.
arXiv preprint arXiv:2402.11765.
```


## Development

### Setting up the development mode

We are more than happy to receive help with the development of the package.
If you want to contribute, here are some elements to take into account.

First, install the development dependencies by running the following command:
```shell
pip install -e ".[dev]"
```

### Conventions

We try to enforce uniformity within the package. Here are some general guidelines.

- All samplers have `num_voters` and `num_candidates` as their two first positional arguments
- All samplers accept a `seed` parameter to set the seed of the random number generator

Within the package, the samplers are organised in modules based on the ballot format they
generate. The `prefsampling.core` module is used for features used across samplers.
Within the submodule corresponding to the ballot format, there is a Python file 
for each family of samplers. All the samplers are imported and appear in the `__all__`
variable of the `__init__.py` file of the corresponding module (defined by the ballot
format).

### Tests

The tests are run with unittest. Simply run the following command to launch the tests:
```shell
python -m unittest
```

The structure of the test module follows that of the package. There is one submodule per
ballot format we sample. Within the submodule, there is one file per statistical culture.

At the submodule level, there is a file `test_all_ballotformat_samplers.py` that gathers the
test that are common to all samplers of the given ballot format.

In the file corresponding the statistical culture, there is a function that returns all 
the samplers (with their arguments set) that are used as test cases, together with
the tests that are specific to the sampler.

When a new sampler is added to the package, it needs to be added in several places within the test
module:

- A file `test/ballotformat/test_ballotformat_culturename.py` defining the tests specific to the sampler and the functions to use for the tests (called `random_ballotformat_culturename_samplers`).
- In `test_all_ballotformat_samplers.py`, add the functions for the sampler to the `random_ballotformat_samplers()` function.
- If it is a sampler for actual ballots (i.e., not points in space or trees), add the functions for the samplers to the `random_samplers()` in the file `test/test_all_samplers.py`.

### Validation

We aim at statistically validating the samplers we provide. All the code necessary to 
run the validation is gathered in the `validation` folder of the repository.

When a new sampler is added to the package, proceed as follows:
- Create the corresponding file in the `validation/ballotformat/` folder.
- In this file, define a class that inherit from the `validation.validator.Validator`. This requires you to define a set of methods used to compute the theoretical probabilities of the outcomes of the samplers.
- Add the validator in the `run.py` file.
- Run the `run.py` file (you may want to comment out some parts).
- Copy the generated graphs in the correct place of the `doc-source/source/validation_plots` folder.
- Update the `doc-source/source/validation.rst` file accordingly.

### Documentation

The doc is generated using sphinx. We use the [numpy style guide](https://numpydoc.readthedocs.io/en/latest/format.html).
The [napoleon](https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html) extension for Sphinx is used
and the HTML style is defined by the [Book Sphinx Theme](https://sphinx-book-theme.readthedocs.io/en/stable/).

To generate the doc, first move inside the `docs-source` folder and run the following:
```shell
make clean 
make html
```

This generates documentation locally in `docs-source/build`. The fork's CI
builds updated HTML artifacts on each push to `main`; generated HTML is not
committed back into the repository.

### Packaging and documentation checks

The fork's CI tests Python 3.10–3.14 and includes compatibility jobs for
NumPy 1.21.6, 1.26.4, and 2.2.6. Test, documentation, and statistical-validation
dependencies can be installed separately using `.[test]`, `.[docs]`, and
`.[validation]`; `.[dev]` installs all development tools.

The [distribution workflow](https://github.com/bokveizen/prefsampling/actions/workflows/publish.yml)
builds wheel and source archives and smoke-tests the installed wheel. The
[documentation workflow](https://github.com/bokveizen/prefsampling/actions/workflows/docs.yml)
builds HTML from source. Both upload artifacts for review. Publishing a
separately named distribution or deploying a documentation site is a future
maintenance decision.

### Repairs and maintenance review

The fork includes these corrections:

- Independent voter and candidate streams and a new seed for every Gaussian-ball
  proposal, including retries; caller-owned position dictionaries are preserved.
  This addresses [upstream issue #6](https://github.com/COMSOC-Community/prefsampling/issues/6),
  reported by @masiarek. The repair was submitted as
  [upstream PR #7](https://github.com/COMSOC-Community/prefsampling/pull/7).
- Ordinal Euclidean sampling uses L2 distance in every dimension. In 3D,
  a voter at `(0, 0, 0)` now prefers `(1.5, 0, 0)` over `(1, 1, 1)`.
  Flat one-dimensional position inputs are normalized to column arrays.
- Composition copies each sampler's arguments before setting population sizes.
  Reusing one dictionary for groups of two and three voters now yields five
  voters, rather than six. Ordinal truncation also preserves argument dictionaries
  and accepts generator inputs for per-voter approval sizes.
- Voter and candidate validation works under optimized Python (`python -O`).
- Invalid Gaussian truncation widths and scales are rejected before sampling;
  non-positive truncation widths can otherwise cause an endless rejection loop.

Seeded spatial coordinates intentionally differ from upstream 0.1.24. Ordinal
rankings and tie classes above two dimensions can also change with the corrected
metric. Python 3.7–3.9 are no longer advertised or tested by this fork.
Argument copies are shallow. The capped ball rejection sampler still warns and
uses the center point after too many retries; it is not an exact continuous
conditional sampler in that fallback case. Very narrow positive Gaussian bounds
can still make rejection sampling slow.

The upstream issue review on 30 September 2026 also found:

- [Issue #5](https://github.com/COMSOC-Community/prefsampling/issues/5) requests
  conversion from approval sets to a matrix. This is an API feature request,
  rather than a sampling defect, and remains a follow-up.
- Closed issues #2–#4 concern Euclidean-space configuration, duplicated validation
  documentation, and weak-order sampling. Weak orders are already implemented
  through `tie_radius` and `coin_flip_ties`.
- Mixture sampling documents that its outer seed controls mixture assignments
  only. Set seeds in each sampler's parameters to reproduce the component draws.

This is a focused correctness and maintenance review, not a mathematical
validation of every statistical culture. New changes should include a concrete
reproducer and run the relevant regressions plus the full unit-test suite.

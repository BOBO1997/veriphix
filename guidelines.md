# QAT coding guidelines

## Introduction
Code in science has many different roles. From simulating complex behaviors to generating figure, code is everywhere.
Our philosophy in the team is that it is important to confront our theories with experiment and software so as to implement feedback loops guiding the practical decisions while making the different protocols more realistic.
Even more, numerical explorations allow us to test new ideas and investigate them so it should be a playground for research.
In addition, publishing and sharing code allow us to make our scientific (and mostly theoretical) results more compelling.

Specifically we think that code can be used to illustrate theoretical results, investigate regimes that are unaccessible using analytical solutions and compare results to simulations and/or experimental results.

We highlight several properties that any code should fulfill.

### General principles

We emphasize three main characteristics that we believe our code should have:

- **Reproducibility**: given the same context, the software should consistently output the same result on anyone's machine (be it a correct result or a bug). What we call the context is not just the input of a given function but also the environment you use (including which packages are used, and in which versions).
Reproducibility is a fundamental requirement for scientific results and should be a goal for every published work. But it is not just something to aim for at the end of a project; it’s something we recommend putting in place from day one, as it greatly facilitates collaboration and debugging.
That is why we recommend using:
  - [**Version control**](#version-control) (git) to track both the versions of the code and the experiments you run with it. The commands you use to test your code, generate figures, or run benchmarks are code themselves and should be versioned.
  - [**Environment specification**](#virtual-environments) (e.g. [Conda](https://www.anaconda.com/docs/getting-started/miniconda/install), [Docker](https://www.docker.com/), [Guix](https://guix.gnu.org/)) to track dependencies and system configurations. These specifications are part of the codebase as well and should also be versioned.
  - [**Continuous Integration**](#continuous-integration) to systematically check your code in a standardized environment and across a broader range of platforms and software versions than you could reasonably test manually.
- **Reusability**: you or someone else should be able to reuse your software. This goes beyond reproducibility since you might want to use it again on different inputs or in a different context with or without making modifications. Code should:
  - be general and modular enough to be reused,
  - use already established libraries when possible: don't reinvent the wheel,
  - be well-structured and documented.
  This gives the possibility to infer the structure of the code and make modifications/generalisation where necessary. 
  A usual symptom is **copying-and-pasting**. If you copy-paste, it means there is a more general way of doing what you want. It is a quick-and-dirty solution that saves you time now but will be detrimental in the long run.

- **Robustness**: The software should be robust. For example slightly changing some parameters or modifying an implementation shouldn't result in a wildly different behaviour.
A way to insure this is through tests that allow you to define the expected behaviour of your code. For example there often are edge cases which have an analytical solution: check your code reproduces those! 
Another example is in the case of noisy simulations, check that if you set the noise to zero, you get the expected outcome!
Sometimes it is hard to find good tests but some are easier. We come back to this issue later. The key takeaway is that tests can be automated so they should! This guarantees that any modification doesn't break the previous behaviour.

The code should be as clear as possible, and ideally describe what is to be computed at the same level of abstraction we would use to explain it to ourselves. Rather than writing a formula in its final, pre-computed form, we prefer to write a program that performs the computation for us.

### Where we can help

- Project design and setup ("what do I want to do?" and "how should I do it?")
- Code testing and continuous integration ("what is my code supposed to do?")
- Performance (profiling, using the CLEPS cluster, algorithms) ("where are the performance bottlenecks?", "is it a structural issue or an algorithmic issue?")
- Refactor, how to generalize the code and reuse it for other projects in the team?

### Key takeaways

- Use anything you are confortable with in terms of language and IDEs. We focus on Python since this is what we have been doing so far but if you use another language, we'll be happy to learn with you. Just avoid stuff that is too [niche](https://en.wikipedia.org/wiki/Brainfuck), though.
- Use git (GitHub, GitLab): prefer GitHub for public repositories and gitlab.inria.fr for private repositories.
- Don't reinvent the wheel. Before embarking on writing a large piece of code from scratch, it's always advisable to investigate what already exists (in the community and in the team's knowledge base).
- Avoid notebooks in the development process. Owing to their intrinsic cell structure, notebooks (such as Jupyter or Mathematica) don't have a clear execution flow which hinders code development. Furthermore, notebook files don't get on well with version control systems (e.g., git). They can still be useful for demonstrations (drafts, teaching or tutorials).
- Prepare for Qode meetings:
  - Organise your thoughts.
  - Prepare a README to explain the broad scientific context and objective and detail the code structure (we can't be experts on everything).
  - Share your code **as soon as possible** and **regularly**.


Below we deep dive into some of the general principles we have highlighted here.

## Technical guidelines

### Project management

#### Virtual environments

We encourage you to rely on existing libraries as much as possible, and even compare and improve them if necessary, rather than reinventing the wheel. A consequence of this approach is that your project may quickly accumulate a large number of dependencies. A good practice is to use some form of container to host the dependencies your project requires. Here are the benefits of developing in a containerized environment:

- If you work on several projects at the same time, it helps you keep the environments separated.

- You can use different environments with different versions of dependencies, either to test compatibility or to work on multiple projects with different version requirements.

- It helps you keep track of the development context and document it, and, if things go wrong, to quickly rebuild it from scratch.

- Having the development context documented allows other people to reproduce it on their own machines.

Documenting the development context can be done at different levels, depending on what needs to be specified:

- Dependency requirements can be written in a `requirements.txt` file, which lists one `pip` package per line, possibly with version constraints. Variants such as `requirements-dev.txt` or `requirements-extra.txt` can be used to distinguish between mandatory and optional requirements. Note that `requirements.txt` only lists `pip` packages and cannot specify the Python version itself or system-level dependencies not managed by `pip` (e.g., `sage-math`). These requirements are installable via `pip install -r requirements.txt`.

- Dependencies can also be listed in a `pyproject.toml` file, which can include constraints on the Python version. This file declares your project as an installable package, which can be installed together with its dependencies. Within a `pyproject.toml`, you can even refer to a `requirements.txt` file. The project and its dependencies can then be installed via `pip install .` (where `.` is the current working directory, but it can also be replaced with a path to the project or even a repository URL).

- If you need to specify more than just `pip` dependencies, such as a specific Python version or other system-level dependencies, you can write a [conda](https://www.anaconda.com/docs/getting-started/miniconda/install) environment specification in an [`environment.yml`](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually) file. If you need to specify an entire container, including a particular Linux distribution with specific system libraries and tools, you can write a [`Dockerfile`](https://docs.docker.com/reference/dockerfile/) that explains how to build it. This is especially useful for continuous integration, as it allows precise control over the system that will run the tests.

The [Dependencies](#dependencies) section below provides more details about writing the `requirements.txt` and `pyproject.toml` files, and about using `pip`.

Python provides some lightweight ways to manage environments:

- The [`venv`](https://docs.python.org/3/library/venv.html) module, included in the Python standard library, creates isolated Python environments that allows you to install and manage dependencies separately from the system Python, reusing the existing Python interpreter. The dependencies are installed in a directory of your choice, typically a `.venv` subdirectory in your working directory. You can create the environment with `python -m venv .venv` and activate it with `.venv/bin/activate`. Subsequent calls to `pip` will install packages into the `.venv` directory. Note that `venv` only manages `pip` dependencies, not the Python version itself.

- The [conda environment manager](https://www.anaconda.com/docs/getting-started/miniconda/install) allows you to create distinct environments, each with a specific version of Python, specific `pip` packages, and even some system-level libraries, so long as they are available as conda packages.

By contrast, a [`Dockerfile`](https://docs.docker.com/reference/dockerfile/) allows you to write recipes to build environments from scratch on a chosen Linux distribution, using that distribution’s own package manager.

#### Dependencies

**Add a file `pyproject.toml` at the root of the repository.**

Such a file allows you to specify many things, but we’ll keep it
minimal for now to enable pip to install our package and its
dependencies. The goal is for anyone who wants to test, use, or
contribute to our code to be able to simply run the following command
and have the package installed along with all its requirements in
their Python environment:

```bash
pip install <repository>
```

Here, `<repository>` can either be a path on their local machine (for
example, `pip install .` to install the package in the current
directory), or the URL of a Git repository, or the name of a published
package, etc. For instance, to install the GitHub repository
`zac-vh/qopy`:

```bash
pip install git+https://github.com/zac-vh/qopy.git
```

The users will then be able to do the following in their own Python
code:

```python
import package_name
# or
from package_name import some_function
```

In particular, if you have some examples or tests located in another
directory, having your code installed as a package allows imports to
work independently of any specific IDE configuration.

Here is a skeleton for such a `pyproject.toml` file.
```toml
[project]
name = "<project name>"
authors = [{ name = "<your name>", email = "<your email address>" }]
version = "0.1"
dependencies = [
    "<package>",
]
```

You can find more details about writing the `pyproject.toml` file in [the Python Packaging User Guide](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/).

#### Installing packages

**Install your development package in-place to be able to use your modification directly.**

You can instruct `pip` to install your package by directly using the
files in your working directory (i.e., without copying them in the
environment's subtree), allowing you to edit the Python files and see
your updates directly.  This can be done with the `-e` option (for
`--editable`). Typically, you just have to run the following command
at the root of your repository:

```bash
pip install -e .
```

**Specify Package Versions with `pip` for third-party packages**

When installing a package using `pip install` or when specifying a
dependency, we can include a version constraint or refer to a Git
repository—and even to a specific branch within that repository.

To require a specific version of a package, we use the `==`
operator. For instance, the following command installs version
`0.11.0` of `ruff`:

```bash
pip install ruff==0.11.0
```

When specifying dependencies, it is good practice to include version
constraints for each one, such as `"numpy>=2.2.6,<3.0"`.

* The lower bound ensures that if another package requires an older
  version—one that our code hasn’t been tested with—`pip` will raise
  an error. Typically, we set the lower bound to the version currently
  installed, which you can check with:

```bash
pip show numpy | grep Version:
```

* The upper bound helps prevent compatibility issues that may arise if
  a future update installs a version of the dependency our package is
  not compatible with. We usually exclude future major releases (e.g.,
  `<3.0`) because, according to [semantic
  versioning](https://semver.org/), a major version bump often
  includes breaking changes.

If the package is hosted in a Git repository, we can use the `git+`
prefix followed by the repository’s HTTPS or SSH URL. The HTTPS form
is suitable for public repositories. For example:

```bash
pip install git+https://github.com/TeamGraphix/graphix.git
```

For private repositories, the SSH form allows us to use credentials
from our SSH agent. For example:

```bash
pip install git+ssh://git@gitlab.inria.fr/qat/stabilizer_gs.git
```

Note that we explicitly specify the user (`git@`) because GitLab
requires it for SSH access.

If we want to install a specific branch, tag, or commit from the
repository, we can append `@<ref>` to the URL, where `<ref>` is the
name of the branch, tag, or commit hash. For instance, to install the
`dev` branch of a repository:

```bash
pip install git+https://github.com/username/repo.git@dev
```

### Version Control

We use `git` for version control, and we host our repositories on GitHub for public projects and on gitlab.inria.fr for private ones.

#### Development Branches

We use dedicated branches for each development or feature, which we merge into the `main` branch when the development is mature enough, preferably after one or more rounds of code review. Code reviews use the pull request mechanism of GitHub or the similar merge request mechanism of GitLab.

You can create a new branch using the command `git checkout -b branch_name`, or preferably through the Git integration in your IDE.

Development branches can be hosted in the same repository on GitHub or GitLab, or in your own fork of the repository. Forking is very common, especially when working on projects maintained by other groups, where you may have limited rights on the official repository. Forking also allows multiple people to work on branches with the same name without interfering with each other, since each developer works in their own repository. Meanwhile, it remains possible to synchronize with other remotes when needed. We leverage Git’s ability to maintain multiple remotes (e.g., via the [`git remote`](https://git-scm.com/docs/git-remote) command) to collaborate easily across several forks of a repository.

#### Cleaning History

We should try to keep the commit history as clean as possible. A clean history makes it easier to explore changes and is especially helpful when using [`git bisect`](https://git-scm.com/docs/git-bisect) to track down when and how a bug was introduced.

Here are some guidelines for keeping the history clean:

- The `main` branch history should be linear (i.e., without merge commits), and we should never rewrite it. (The main branch is protected by default, and force-pushing is disabled in GitLab, we recommend keeping it that way.)

- [`git rebase`](https://git-scm.com/docs/git-rebase) is the preferred way to integrate changes from other branches into a development branch. We recommend setting `rebase` as the default `pull` behavior in your `~/.gitconfig`:

  ```
  [pull]
  rebase = true
  ```

- If rebasing becomes too tedious, a merge commit can temporarily be used in development branches. However, we recommend regularly rewriting and simplifying the history of development branches, or even squashing commits, before rebasing them into the `main` branch.

#### What should be versioned and what to ignore

All environment and dependency specifications should be included in the repository, along with formatter and linter configurations. These configurations are now typically done in the `pyproject.toml` file, which should be versioned. This ensures that all developers share the same set of coding conventions and that the dependency specification remains synchronized with the code.

What should **not** be versioned are generated files, caches, backup files, `venv` directories, and similar artifacts. If we want to share experimental results, the recipes of the experiment (i.e., the code that runs it and its parameters) should be versioned as part of the program, but the result files themselves should be shared separately. If these files are not too large, Git can be a reasonable tool to share them, but this should be done in a separate repository. Result files can easily outsize the codebase and frequently cause merge conflicts, which we want to avoid in a code repository.

We recommend using a versioned [`.gitignore`](https://git-scm.com/docs/gitignore) file to list files that should not be tracked, so that they do not appear as untracked when running [`git status`](https://git-scm.com/docs/git-status).

Typical entries include:

```
__pycache__/
*.py[cod]
.mypy_cache/
.ipynb_checkpoints/
```

You should also ignore files specific to your IDE or environment, such as:

- Emacs backup files (`*~`)

- VS Code settings (`.vscode/settings.json`)

- PyCharm project directory (`.idea/`)

- macOS metadata files (`.DS_Store`)

These files are tool-specific and project-independent, and other developers may not use the same tools. We recommend ignoring them globally by adding them to `~/.gitignore` (a `.gitignore` file in your home directory) and setting the following option in your `~/.gitconfig`:

```
[core]
excludesfile = ~/.gitignore
```

### File structure

Generally speaking, the top-level structure of a Python program includes the following:

#### At the beginning of the file:

* **A module-level docstring at the very top**
  ```python
  """This module provides utilities for vector operations."""
  ```
* **Imports**
* **Exports**, i.e., the definition of the global variable `__all__`
* **Logger definition**, e.g.:

  ```python
  logger = logging.getLogger(__name__)
  ```

As noted in [PEP 257](https://peps.python.org/pep-0257/#:~:text=All%20modules,docstrings.):

> All modules should normally have docstrings, and all functions and classes exported by a module should also have docstrings. 

The `ruff` rule `D100` ([`undocumented-public-module`](https://docs.astral.sh/ruff/rules/undocumented-public-module/)) checks that every public module has a docstring. See the section [Formatting the code](#formatting-the-code) for more details about the `ruff` tool.

The recommendation to place imports at the top of the file comes from [PEP 8](https://peps.python.org/pep-0008/#imports). This is enforced by the `ruff` rule `PLC0145` ([`import-outside-top-level`](https://docs.astral.sh/ruff/rules/import-outside-top-level/)).

#### In the body of the file:

* **Class definitions**
* **Function definitions**
* **Type aliases**:

  Type aliases are documented in [the specification for the Python type system](https://typing.python.org/en/latest/spec/aliases.html#type-aliases).

  * For Python < 3.12, use global variables with the `TypeAlias` annotation (introduced in [PEP 613](https://peps.python.org/pep-0613/)):

    ```python
    from typing import TypeAlias

    Vector: TypeAlias = list[float]
    ```
  * For Python ≥ 3.12, use the new syntax for type aliases (introduced in [PEP 695](https://peps.python.org/pep-0695/)):

    ```python
    type Vector = list[float]
    ```
* **Avoid other global variables**, as they create global state, which can make code harder to analyze and less suitable for parallelization.

* Constant-like values `ALL_CAPS` (e.g., `PI = 3.1415`) can be an exception to the “no globals” rule if they're truly immutable and universally relevant.

#### At the end of the file (if the script should be executable from the command line):

```python
if __name__ == "__main__":
    entrypoint()
```

where `entrypoint` is the main function of your code (or `app()` if you're using [Typer](https://typer.tiangolo.com)).

When code is parameterized, it is good practice to make this
parameterization explicit by writing the code within a function that
takes the parameter as an argument. This helps separate the parts of
the code that depend on the parameter from those that do not.

As a general rule, even for non-parameterized parts, it is good
practice to place most of the code inside functions rather than at the
top level. Functions allow us to clearly define—and limit—the scope of
variables.

### Documentation

Docstring begins with a verb in the imperative mood.
According to [PEP 257](https://peps.python.org/pep-0257/#:~:text=The%20docstring%20is,%22.):

> The docstring is a phrase ending in a period. It prescribes the function or method’s effect as a command (“Do this”, “Return that”), not as a description; e.g. don’t write “Returns the pathname …”.

The `ruff` rule `D401` ([`non-imperative-mood`](https://docs.astral.sh/ruff/rules/non-imperative-mood/)) checks that docstrings are in the imperative mood.
The `ruff` rule `D404` ([`docstring-starts-with-this`](https://docs.astral.sh/ruff/rules/docstring-starts-with-this/)) checks that docstrings don't start with `This`.
See the section [Formatting the code](#formatting-the-code) for more details about the `ruff` tool.

TODO: Generate documentation automatically?

### Testing

Testing your code during development is essential to ensure that the program does what _you want it to do_. Ideally, tests should be modular (i.e., there should be a set of tests for _every_ function of your code) and should be done programatically (i.e., verifying the function's output should _not_ be done by hand). Formalizing the expected behavior of a function in a test might feel like a slow down in the development process, but it will save you a lot of time and trouble in the long run.

#### Pytest

In Python, tests can be automatically run with [Pytest](https://docs.pytest.org/en/stable/). 

To install it, run
```shell
pip install pytest
```

Let's see a minimal example. Suppose that you have a module `average.py` where you define the following function:

```python
def get_average(values):
	return sum(values)/len(values)
```

To test this function, we would create an additional file `test_average.py`, containing, for instance, the following functions:

```python
from math import isclose
from average import get_average # This line assumes that you installed your development package in-place see §Installing packages

def test_get_average_1():
	vals = [-1, 1, 2.2, 3]
	assert isclose(get_average(vals), 1.3) # Avoid using ==  when comparing floats


def test_get_average_2():
	vals = []
	assert get_average(vals) == 0
```

- `pytest` recognizes a function as a test function if its name starts with the `test_` prefix.

- It is advisable to organize tests in a dedicated `tests/` folder, with one file per module. While you can run `pytest` on any Python file, if it is run without specifying a file or module, it will look for test functions in any file whose name starts with the `test_` prefix.

- Tests can be further organized into classes (see the  [docs](https://docs.pytest.org/en/stable/getting-started.html#get-started) for more details). `pytest` will run tests in classes whose names start with the `Test` prefix.

- You can run a specific test by specifying its full node ID, for example: `pytest test_average.py::test_get_average_1`.

To launch the test, we would run from terminal: 

```shell
pytest test_average.py
```

which would give the following output:

```console
================================================= test session starts =================================================
platform darwin -- Python 3.12.11, pytest-8.4.0, pluggy-1.6.0
rootdir: /Users/***
plugins: cov-6.2.1, mock-3.14.1
collected 2 items                                                                                                     

test_average.py .F                                                                                                      [100%]

====================================================== FAILURES =======================================================
_________________________________________________ test_get_average_2 __________________________________________________

    def test_get_average_2():
        vals = []
>       assert get_average(vals) == 0
            ^^^^^^^^^^^^^^^^^

test_average.py:10: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

values = []

    def get_average(values):
>       return sum(values)/len(values)
            ^^^^^^^^^^^^^^^^^^^^^^^
E    ZeroDivisionError: division by zero

average.py:2: ZeroDivisionError
=============================================== short test summary info ===============================================
FAILED test.py::test_get_average_2 - ZeroDivisionError: division by zero
============================================= 1 failed, 1 passed in 0.03s =============================================
```

Here, only the first test passed, which allows to quickly identify that our function is not robust against an empty list!

#### Writing good tests

- Tests should be simple but as comprehensive as possible, covering all the edge cases you can think of.

- Functions should be tested against reference values. Ideally, avoid using the function itself (or parts of its internal logic) to generate the expected result. 

  - In most cases, expected results should be hard-coded. However, when the implementation is trusted at a certain point, you can store its outputs and use them as reference values to detect regressions later.

  - If there are different ways to compute the same result (e.g., numerical vs. analytical), tests can compare the outputs of the different approaches.

  - Tests can also verify internal consistency of a function. For example, if results are expected to be invariant under certain parameter changes, tests should check that. However, note that checking internal consistency alone does not guarantee correctness, since a function can be consistently wrong. So it’s important to test against reference values as well.

- Tests should be as deterministically reproducible as possible. In particular, if the tested functions use randomness, they should allow passing in a random number generator or seed. Tests should then use a fixed seed to ensure reproducibility.

- More generally, functions should be designed to be testable. Designing an API that is easy to test often leads to an API that is easier to use.

  - Functions should be as independent as possible from each other.

  - Prefer explicit parameters over reliance on global state.

  - Access to system resources (e.g., files, network) should be separated from core logic, so that logic can be tested in isolation. If this separation is not feasible, consider using [mocking](https://docs.python.org/3/library/unittest.mock.html) to isolate tests.

### Formatting the code

We recommend using automatic tools to maintain consistent code formatting and to ensure that the project follows coding standards. Such tools provide the following benefits:

- They usually keep the code readable while saving you the time of handling formatting details yourself.

- They help maintain a consistent coding style across the project, especially when multiple people are working on the same files.

- They ensure the code remains well-formatted and reduce noise in commit diffs caused by purely formatting-related changes.

- They help avoid [bikeshedding](https://en.wikipedia.org/wiki/Law_of_triviality) debates on how to format the code.

- In addition to formatting, linters that enforce coding rules can detect programming errors or bad practices.

For Python, we recommend using [`ruff`](https://github.com/astral-sh/ruff). For formatting, the command is `ruff format`, and for checking coding rules, the command is `ruff check --fix` (the `--fix` option enables the tool to automatically fix most minor issues).

One common customization for `ruff format` is the line length, which is set to 88 by default. You can change it either via the command line with `ruff format --line-length=120`, or preferably in the `pyproject.toml` file:

```toml
[tool.ruff]
line-length = 120
```

Most [coding rules](https://docs.astral.sh/ruff/rules/) that `ruff` can check are disabled by default, meaning `ruff` does not enforce them unless explicitly configured to do so. You can control which rules are enabled in the `pyproject.toml` file. For instance, to enable all rules except a few that are incompatible with others:

```toml
[tool.ruff.lint]
preview = true
select = [
    "ALL",
]
extend-ignore = [
    "D203",
    "D212",
    "COM812",
    "ISC001",
]
```

The line `preview = true` enables even experimental rules. In the Graphix project, we follow an opt-out convention where all rules are enabled except for a fixed set that we explicitly disable. One could argue that an opt-in convention is preferable, as it prevents checks from breaking when a `ruff` update introduces a new rule.

The tool [`pre-commit`](https://pre-commit.com/) allows you to automate the execution of `ruff format` and `ruff check` before committing changes with Git, ensuring that only properly formatted and checked code is committed. Running `pre-commit install` sets up a Git hook in `.git/hooks/pre-commit` that runs `pre-commit` before each commit. The configuration for `pre-commit` is stored in a `.pre-commit-config.yaml` file at the root of the repository. For example, to run `ruff` before every commit:

```yaml
repos:
- repo: https://github.com/astral-sh/ruff-pre-commit
  # Ruff version.
  rev: v0.12.1
  hooks:
    # Run the linter.
    - id: ruff
    # Run the formatter.
    - id: ruff-format
      args: ["--check"]
```

Commits will be blocked if the code is not properly formatted or does not follow the coding rules. We intentionally disable automatic fixes here (by passing `--check` to `ruff format` and omitting `--fix` from ruff check), as it can be disruptive if `pre-commit` modifies the repository during a commit. This could result in unstaged changes appearing after `git commit`. We prefer to use `pre-commit` solely as a checker and let users apply fixes manually when needed.

### Typing

TODO: mypy

#### Use of abstract data structures

When specifying parameter types, we prefer more general (and abstract)
types like `Mapping[int, int]` over concrete types like `dict[int, int]`.

As noted in [PEP 484](https://peps.python.org/pep-0484/#:~:text=Note:%20Dict,AbstractSet.):

> "`Dict`, `DefaultDict`, `List`, `Set` and `FrozenSet` are mainly useful for annotating return values. For arguments, prefer the abstract collection types defined below, e.g. `Mapping`, `Sequence` or `AbstractSet`."

Using `Mapping[int, int]` ensures that any immutable or read-only
mapping will be accepted, not just mutable dict instances.

### Continuous Integration

Most software forges, such as GitHub and gitlab.inria.fr, provide mechanisms to automate tasks in response to various events occurring in the repository, such as pushes, merges or pull requests, and tagging. Continuous Integration (CI) refers to the use of such automation to verify that the code adheres to formatting and coding standards and that all tests pass. Automating these checks helps developers ensure they don’t overlook any steps and that tests succeed not only on their own machines but also in standardized environments (and possibly across a broader range of platforms and software versions than they can test themselves). In addition, automated tests handle routine aspects of the review process, allowing reviewers to focus on the substance of the contribution rather than superficial issues.

GitHub’s automation framework is called [GitHub Actions](GitHub Actions), and automated tasks are defined in [workflows](https://docs.github.com/en/actions/how-tos/writing-workflows). Each workflow is written in a `.yml` file stored in the `.github/workflows` directory of the repository. Here is an example of a workflow that can be saved as `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt
      - name: Check code formatting
        run: ruff format --check
      - name: Run linting
        run: ruff check --output-format=github
      - run: pytest
```

Here we use `ruff` for linting and formatting checks, and `pytest` to run unit tests. See the section [Formatting the code](#formatting-the-code) for more details about the `ruff` tool, and see the section [Pytest](#pytest) for more details about `pytest`.

The `on:` section specifies that this workflow runs on every pull request and can also be triggered manually (`workflow_dispatch`) by clicking **Run workflow** on the workflow page under the **Actions** tab in GitHub. The `concurrency:` section ensures that for a given branch, any newly launched workflow cancels the previous one, avoiding unnecessary computations if new commits supersede earlier ones. This workflow defines a single job, `test`, which checks out the repository, installs dependencies, runs `ruff` checks, and launches tests. The results of the CI are displayed in the pull request, and if any step fails, depending on user settings, notifications may be sent to the committer.

The equivalent of GitHub workflows in GitLab is called [pipelines](https://docs.gitlab.com/ci/pipelines/). Pipelines are configured in the `.gitlab-ci.yml` file at the root of the repository. Here is an example pipeline configuration:

```yaml
build:
  tags: [linux, small]
  image: conda/miniconda3
  script:
    - pip install -r requirements.txt
    - ruff format --check  # Check code formatting
    - ruff check --output-format=github  # Run linting
    - pytest
```

To use pipelines, the CI/CD feature must be enabled in your GitLab project:

- In the left sidebar of the GitLab interface, go to **Settings** → **General**.

- Expand the **Visibility, project features, permissions** section.

- Under **Repository**, enable **CI/CD**.

- Click **Save changes**.

You may consult the [GitLab CI gallery](https://gitlab.inria.fr/gitlabci_gallery/) for more examples of pipelines.

### Logging

TODO: https://docs.pytest.org/en/stable/how-to/logging.html

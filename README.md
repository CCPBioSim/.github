# CCPBioSim Shared Infrastructure

Shared defaults for CCPBioSim software projects: consistent dependency management, community-health files, and reusable CI, release and maintenance workflows.

**Centralising maintenance does not require centralising every decision.** Policy and repeated mechanics are shared. Scientific decisions stay in each package repository: dependency floors, supported Python versions, package paths, conda recipes and release exceptions.

```mermaid
flowchart LR
    A["CCPBioSim/.github<br/>shared defaults"] --> B["Renovate presets"]
    A --> C["Community-health defaults"]
    A --> D["Reusable workflows"]
    B --> R["Package repositories"]
    C --> R
    D --> R

    style A fill:#245b78,color:#ffffff,stroke:#245b78
```

## Status

| Area | Status |
|---|---|
| Renovate presets | Live |
| Pull request and issue templates | Live |
| `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md` defaults | Live |
| Reusable CI, release and maintenance workflows | Live |
| Workflow linting through pre-commit | Live |

## Using it

**Renovate.** Add a small `.github/renovate.json` to a repository:

```json
{
  "extends": [
    "github>CCPBioSim/.github//renovate/default",
    "github>CCPBioSim/.github//renovate/python"
  ],
  "assignees": ["<maintainer>"]
}
```

Changes here reach every repository on its next Renovate run. Runtime dependencies, dev tools and GitHub Actions are each grouped, and every pull request carries the `dependencies` label alongside `ci`, `python`, `production-deps` or `dev-dependencies`.

**Community-health files.** Nothing to do. GitHub inherits them for repositories without their own copy, and a local file always wins.

**Finding dependency updates across the organisation:**

```text
org:CCPBioSim is:pr is:open label:dependencies
```

## Workflows

Reusable workflows live in `.github/workflows/`. The folder is nested because this repository is itself called `.github`, so a caller writes the name twice:

```text
uses: CCPBioSim/.github/.github/workflows/<name>.yaml@<full commit SHA> # X.Y.Z
```

Callers pin a full commit SHA with the version in a trailing comment, so Renovate can propose updates. A called workflow cannot have more permissions than its caller, so a caller grants what the table lists. Triggers, schedules and concurrency stay in the caller.

| Workflow | Purpose | Caller provides |
|---|---|---|
| `ci-test-python` | Run pytest on a Python and OS matrix, report coverage to Coveralls in a separate `Coverage` job, and optionally re-run the tests at the lowest versions that `pyproject.toml` allows (`lowest-dependencies-check`) | `package`, and `checks` and `statuses` write |
| `ci-test-conda` | The same for conda-based setups such as AiiDA plugins | `package`, `conda-packages`, and `checks` and `statuses` write |
| `ci-docs` | Build Sphinx docs with warnings as errors | Nothing required |
| `ci-pre-commit` | Run all pre-commit hooks | Nothing required |
| `ci-conda-build` | Build a conda recipe without uploading it | Nothing required |
| `ci-coveralls-finish` | Close a parallel Coveralls build | Run it after the coverage jobs |
| `release-prepare` | Check the version, update version files through a merged pull request, tag and create the GitHub release | `version`, `contents` and `pull-requests` write, and `AUTO_PR_MERGE` when `version-files` is used. Optional `ZULIP_BOT_EMAIL`, `ZULIP_BOT_API_KEY` and `ZULIP_SITE` announce the release |
| `release-website` | Send the release metadata to the website repository | `version`, `tag` and `BUILD_TOKEN` |
| `release-pypi` | Build from the release tag and publish to PyPI | `tag` and `PYPI_API_TOKEN` |
| `release-conda` | Build from the release tag and upload to Anaconda.org | `tag` and `ANACONDA_TOKEN` |
| `maintenance-license-year` | Open a pull request that updates the license year | `contents` and `pull-requests` write |

Coverage is measured on one leg only, the first runner in `os` on `coverage-python-version`. That leg saves `coverage.xml` as an artifact, and the `Coverage` job uploads it to Coveralls, so the other legs run without the coverage overhead.

The `actions/cpp-coverage-upload` composite action turns gcov data into an lcov report and uploads it to Coveralls as a named parallel build, so C++ coverage combines with Python coverage. `ci-test-python` and `ci-test-conda` take `pre-test-command`, and `ci-docs` takes `pre-build-command`, for steps that must run first, such as downloading data or generating files. The release workflows also take `dry-run`, which runs every step except the ones that publish or change the repository.

Workflows in this repository are audited with `zizmor` through pre-commit, which runs in CI through `ci-pre-commit`.

A pull request workflow in a package repository looks like this:

```yaml
jobs:
  test:
    uses: CCPBioSim/.github/.github/workflows/ci-test-python.yaml@<sha> # X.Y.Z
    permissions:
      contents: read
      checks: write
      statuses: write
    with:
      package: mypackage
  docs:
    uses: CCPBioSim/.github/.github/workflows/ci-docs.yaml@<sha> # X.Y.Z
    permissions:
      contents: read
  pre-commit:
    uses: CCPBioSim/.github/.github/workflows/ci-pre-commit.yaml@<sha> # X.Y.Z
    permissions:
      contents: read
```

## Defaults and policy

The defaults in the shared workflows are the organisation baseline. Checks that validate compatibility run the full supported Python set by default. Checks that validate presentation run one leg by default. A repository widens or narrows these explicitly in its own workflow, and each exception is its own choice to document.

| Workflow | Default Python | Default OS |
|---|---|---|
| `ci-test-python` | Full supported set | Linux |
| `ci-test-conda` | Full supported set | Linux |
| `ci-conda-build` | Full supported set | Linux |
| `ci-docs` | Latest only | Linux |
| `ci-pre-commit` | Latest only | Linux |

**Python versions.** The default set is the three newest stable CPython releases. A package's own `pyproject.toml` is the source of truth, so its matrix should match its `requires-python` and classifiers. A package with a narrower range overrides the default in its workflow. Adding a Python version waits until the package's dependencies support it, and dropping one is a deliberate, reviewed change.

**Operating systems.** The default is Linux only, on `ubuntu-26.04`. Add macOS and Windows where a package has platform-specific behaviour, such as native code, numerical libraries or documented platform support. A scheduled workflow can run the wider matrix so pull requests stay quick.

## Where versions are controlled

| What | Where it lives | How it stays current |
|---|---|---|
| Shared-workflow pins in a package repository | the `uses:` lines of its workflows | the Renovate presets in `renovate/` |
| Python versions and the runner | defaults in the shared workflows | edit them here and release |
| Python dependencies, including AiiDA | the package's own `pyproject.toml` | tested at the newest version in range, and at the lowest with `lowest-dependencies-check` |
| Programs pip cannot install (GROMACS, PLUMED, AmberTools) | `conda-packages` in the package's `ci.yaml` | the repository |

The runner label is written as a literal `runs-on` in some workflows, which Renovate updates, and in the `os` defaults, which it cannot see. When Renovate proposes a runner update, change the `os` defaults in the same pull request. `grep -rn "ubuntu-" .github` lists every place.

A package repository keeps one `ci.yaml` that runs on pull requests, on pushes to `main` and weekly. Python versions come from the shared defaults, so the file lists no versions. A conda-based package adds `conda-packages` for the programs pip cannot install, and a package that needs an extra matrix axis puts a normal `strategy.matrix` on the calling job:

```yaml
jobs:
  unit:
    uses: CCPBioSim/.github/.github/workflows/ci-test-conda.yaml@<sha> # X.Y.Z
    permissions:
      contents: read
      checks: write
      statuses: write
    with:
      package: mypackage
      conda-packages: someprogram=1.2
      coverage: ${{ github.event_name == 'pull_request' || github.event_name == 'push' }}
  all-checks:
    needs: [unit]
    if: always()
    runs-on: ubuntu-26.04
    steps:
      - env:
          RESULTS: ${{ join(needs.*.result, ' ') }}
        run: '[[ "${RESULTS}" != *failure* && "${RESULTS}" != *cancelled* ]]'
```

The `all-checks` job is the one check to require in branch protection, so a matrix change never breaks it.

**Dependency floors.** Set `lowest-dependencies-check: true` on the test job. After the normal tests, each leg reinstalls the lowest versions `pyproject.toml` allows for its direct dependencies and runs the tests again, so the floors are checked on every OS and Python version without extra jobs. Coverage is measured only in the normal run. The weekly run of `ci.yaml` includes it too, so no separate floors workflow is needed.

---

<p align="center">
  <sub>Maintained by the CCPBioSim community.</sub>
</p>

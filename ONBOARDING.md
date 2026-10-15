# Onboarding a new package

A checklist for adding a package to CCPBioSim. Steps marked *(outside GitHub)* cannot be done from this repository and must be done by hand.

## Repository

- [ ] Create the repository under the `CCPBioSim` organisation, from `Python-Template` where suitable.
- [ ] Add `LICENSE`, `README.md` and `CITATION.cff`.
- [ ] Use `pyproject.toml` (PEP 621) for package metadata.
- [ ] Do not add a local `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, issue templates or PR template unless the project needs different content. The organisation defaults apply otherwise.

## Dependency updates

- [ ] Install the Renovate app on the repository.
- [ ] Add `.github/renovate.json` extending the shared presets, with local assignees or reviewers:

  ```json
  {
    "extends": [
      "github>CCPBioSim/.github//renovate/default",
      "github>CCPBioSim/.github//renovate/python"
    ],
    "assignees": ["<maintainer>"]
  }
  ```

  Non-Python repositories extend `renovate/default` only. Add custom managers locally where needed.
- [ ] Check the dependency dashboard issue appears after the first run.

## CI

- [ ] Add a pull request workflow: tests, docs build and pre-commit.
- [ ] Pin third-party actions to commit SHAs with a version comment, and set `permissions:` explicitly.

## Documentation and citation

- [ ] Configure Read the Docs *(outside GitHub)*.
- [ ] Enable the Zenodo integration and add the DOI badge *(outside GitHub)*.

## Releases

- [ ] Configure PyPI publishing for the package *(outside GitHub)*.
- [ ] Add the Anaconda channel token as a repository secret if the package is published to conda *(outside GitHub)*.
- [ ] Add the release notification secrets if the repository announces releases.
- [ ] Record any package that depends on another CCPBioSim package, so its release order is known.

## Repository settings

- [ ] Protect the default branch and set the required checks.
- [ ] Enable private vulnerability reporting.
- [ ] Check the labels used by the templates and Renovate exist: `bug`, `enhancement`, `documentation`, `general`, `dependencies`, `ci`, `python`, `production-deps`, `dev-dependencies`.

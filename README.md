# Helm Chart hosting on GitHub

This repository contains templates which can be used to host your own [Helm](https://helm.sh/)
chart repository on GitHub.


Hosting your own charts on GitHub has become very easy as there are many GitHub
Actions which can be used to set everything up.

This repository contains ready to use Github Workflows to

- [Lint all files](.github/workflows/linter.yml) for pull requests via [Super-Linter](https://github.com/github/super-linter)
- [Lint and Test](.github/workflows/lint-test.yaml) Helm charts for pull request via [chart-testing](https://github.com/helm/chart-testing-action) and [Kubernetes IN Docker](https://github.com/helm/kind-action)
- [Release Charts](.github/workflows/release.yaml) using GitHub Releases and GitHub Pages via [chart-releaser](https://github.com/helm/chart-releaser-action)
- [Sync Repository README](.github/workflows/sync-readme.yaml) to GitHub pages
    so that any updates there also visible to GitHub pages

The only assumption is that your default branch is called `main`. If you are
using something else then you should search for occurences of `main` and
replace it with the you default branch name. 

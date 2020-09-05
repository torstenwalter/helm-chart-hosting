# Helm Chart hosting on GitHub

This repository contains templates which can be used to host your own [Helm](https://helm.sh/) chart repository on GitHub.


Hosting your own charts on GitHub has become very easy as there are many GitHub Actions which can be used to set everything up.

This repository contains ready to use Github Workflows to

- [Lint all files](.github/workflows/linter.yml) for pull requests via [Super-Linter](https://github.com/github/super-linter)
- [Lint and Test](.github/workflows/lint-test.yaml) Helm charts for pull request via [chart-testing](https://github.com/helm/chart-testing-action) and [Kubernetes IN Docker](https://github.com/helm/kind-action)
- [Release Charts](.github/workflows/release.yaml) using GitHub Releases and GitHub Pages via [chart-releaser](https://github.com/helm/chart-releaser-action)
- [Sync Repository README](.github/workflows/sync-readme.yaml) to GitHub pages
    so that any updates there also visible to GitHub pages

The only assumption is that your default branch is called `main`.
If you are using something else then you should search for occurrences of `main` and replace it with the you default branch name.

## Chart Migration

Do you want to migrate an existing chart from one git repository to a new
one, but want to get rid of all files which do not belong to the chart and at
the same time preserver the chart history?
Then this section is for you!

Typical use cases:

- migration of a chart from stable or incubator repository to a new location as
    those repositories have been deprecated
- migration from one chart from a repository which contained next to the chart
    also application source code to a reppository, which should only host the
    chart

This is basically a summary of helm chart migrations which already took place:

1. `stable/prometheus-*` to <https://github.com/prometheus-community/helm-charts>
   Thanks to [Scott Rigby](https://github.com/scottrigby) who was the primary of driver of that migration,
   which included 17 prometheus related charts.
   The whole migration was tracked in <https://github.com/prometheus-community/helm-charts/projects/1> and <https://github.com/prometheus-community/helm-charts/projects/2>.
   That's where many of the code snippets, templates and inspirations came from!
1. `stable/jenkins` to <https://github.com/jenkinsci/helm-charts>
1. `stable/grafana` to <https://github.com/grafana/helm-charts>
   That migration was done in about a day.
   https://github.com/grafana/helm-charts/issues/3 was used to document the executed steps.
   That's helped to write this.

### Prerequisites

- [git](https://git-scm.com/) is installed on you machine

  ```shell
  brew install git
  ```

- [git-filter-repo](https://github.com/newren/git-filter-repo) is installed

  ```shell
  brew install git-filter-repo
  ```
- an empty repository in GitHub where the chart should be migrated to

### Migration Steps

1. Clone the original repository

   ```shell
   git clone <SOURCE_REPO_URL> temp-repo
   ```

1. Filter the history
   The command below filters the history and also renames the directory so that the chart is afterwards in a directory called `charts/`

   ```shell
   cd temp-repo
   git filter-repo --path-glob '<CHART-DIRECTORY>/*' --path-rename
   <CHART-PARENT-DIRECTORY>/:charts/
   ```

   This was used to filter grafana chart's history from stable repository
  
   ```shell
   git filter-repo --path-glob 'stable/grafana/*' --path-rename stable/:charts/
   ```

1. Push the repository to it's new location 
  
   ```shell
   git checkout -b main
   git remote add origin <NEW_REPO_URL>
   git push origin main
   ```

1. Create an empty GitHub Pages branch
   This branch will be used to publish the `index.yaml` which is the index of the chart repository.

   ```shell
   git checkout --orphan gh-pages
   git rm -rf .
   git commit --allow-empty -m "root commit"
   git push origin gh-pages
   ```

1. Configure branch protection rules
   We have two important branches in our repository `main` and `gh-pages`.
   It's worth to protect both from accidental force pushes etc.
   These settings have proven useful:
   - `main` brach
     - Require pull request reviews before merging
     - Dismiss stale pull request approvals when new commits are pushed
     - Require review from Code Owners
       This is not strictly required, but quite useful if you host multiple
       charts in the same repository as it allows you to define who has to
       approve changes in which chart.
     - Require status checks to pass before merging
       Once the first PR pipeline was run you should come back here and
       activate:
       - DCO
       - Lint Code Base
       - lint-test
   - `gh-pages` branch
     The whole configuration can be empty.
     It's only purpose is to prevent force push and prevent delections.
     PR approvals are not useful here as the release pipeline want's to push here directly.

1. Copy template files to the new repository
   You should copy the following files to your new repository.
   Don't worry about the content for now we will get to that in a moment.

   ```text
   LICENSE
   ct.yaml
   CODE_OF_CONDUCT.md
   .github/CODEOWNERS
   .github/workflows/lint-test.yaml
   .github/workflows/linter.yml
   .github/workflows/release.yaml
   .github/workflows/sync-readme.yaml
   CHART-README.md
   CONTRIBUTING.md
   REPO-README.md
   ```

1. Decide which License to use
   The template contains Apache 2.0 License.
   If you want something different then replace that file.

1. Update Contributing Guidelines
   CONTRIBUTING.md is a standard contribution template.
   That should serve as a starting point.
   If you want to enforce DCO check then [DCO](https://github.com/apps/dco) GitHub App could be used for that.
   If you don't want DCO then also remove it from contributing guidelines. 

1. Code of Conduct
   The template points to the CNCF Code of Conduct.

1. Repository README
   You can rename `REPO-README.md` to `README.md`.
   The file contains some placeholders, which need to be replaced.

1. GitHub Workflows
   These are the files in `.github/workflows/`.
   In case you chart depends on other charts you need to add this to `release.yaml`workflow directly after installation of helm

   ```yaml
         - name: Add dependency chart repos
           run: |
             helm repo add stable https://kubernetes-charts.storage.googleapis.com/
             helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
   ```

1. Run linter on all files
   The workflow for super linter just checks modified files.
   To provide a good contributor experience you should check all files for
   errors now and fix them.
   That way you don't put that burdon on the first one who changes a file with
   errors.

   ```shell
   docker run -e RUN_LOCAL=true -e VALIDATE_YAML=false -v $(pwd):/tmp/lint github/super-linter
   ```

   Note: VALIDATE_YAML is also disabled in the workflow as helm templates are golang templates and not valid yaml files.

1. Chart README
   `CHART-README.md`, which was copied from <https://gist.github.com/scottrigby/8117c4a4a10a33e597ca98f6aca95e8a> is a great template for a Helm Chart README.
   You can use that structure to adapt the existing README if you like.
   In any case you should update the installation instructions as at least the chart repository was changed.

1. Release first version of the Chart
   Together with the README changes you should increase the Chart version in `Chart.yaml`.
   I suggest to increase at the minor version.

1. Commit all changes and create a pull request for it
   Now you are ready to go to create your first pull request for your new repository.
   You should see the Github checks running (and hopefully passing).
   That's a good time to revisit you branch protection settings and activate the status checks,
   which should be required to pass before a pull request can be merged.
   If everything it ok, you can go ahead and merge your PR.

   Congratulations!
   The first version of the chart should be released in your new repository!

1. Test the new repository
   Follow your updated installation instructions of you chart and check if everything is working.

1. Deprecate the old chart
   Now that everything is migrated you should go ahead and deprecate your old chart.
   For this you need to
   - prefix the description with "DEPRECATED - ",
   - add `deprecated: true`
   - add a comment above the deprecation flag where the chart was moved to
   - remove all maintainers as deprecated charts should not have maintainers
   - release the chart

1. List your new chart in <https://artifacthub.io/>

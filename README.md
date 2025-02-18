# Pages Preview

## Table of contents

- [What does it do?](#what-does-it-do)
- [Setup](#setup)
- [Inputs](#inputs)

## What does it do?

A lot of third-party services allow you create preview deployments of branches and pull requests, so that you can use them to review and test your changes. This action allows you to do the same thing, but directly with GitHub Pages.  

In particular, this action deploys your website to a different repo, which will contain the previews of all the repos you choose to use this on. 

If you're interested in the logic behind this action, you can check out the [flow diagram](docs/flow_diagram.md).

## Setup

### Preview repo

#### Using my template

1. Go to [this template](https://github.com/EndBug/preview-template) and generate your repo from there: click "Use this template", fill in name and description, check that "Include all branches" is ticked, and create the repo.  
  It doesn't matter whether you create it as public or private, but remember that the Pages website will always be public.

2. Go into your repo settings, in the Pages tab (Repo settings > Pages) and set "GitHub Actions" as the source.

#### Manually

1. Create a new repo that will host your previews.  
  This repo will be used for the previews from all your repositories, so you'll need to set this up only once.

2. Make sure that this repo has two branches: `main` and `gh-pages` (you can also choose different names).
    - `main` should be your default branch, and it will only hold a workflow (and any additional files you want to add, liKE a README, a license, etc.). 
    - `gh-pages` will be the branch that will contain the actual previews, and it should be empty.

3. Create a new file in the `main` branch, and name it `.github/workflows/preview.yml`. Then copy the contents of [`dependents/preview-repo.yml`](dependents/preview_repo.yml) into it.  
  You shouldn't need to change anything in this file, the config options will all be in the source repo workflow.  
  This file might need to be updated if you update the action to a different major version.

4. Go into your repo settings, in the Pages tab (Repo settings > Pages) and set "GitHub Actions" as the source.

### Personal Access Token (PAT)

In order for the action to be able to trigger the deployment in the preview repo from the source repo, you'll need to create a Personal Access Token (PAT).  

There are currently two types of PATs: fine-grained, which are more secure but still in beta, and classic. I'd suggest to use fine-grained PATs, but if you can't, you can also use classic PATs.  

#### Fine-grained PAT

1. Go to [Account settings > Developer settings > Fine-grained tokens](https://github.com/settings/tokens?type=beta).
2. Click on "Generate new token".
3. Give it a recognizable name and set an appropriate expiration date.
4. Make sure that the "Resource owner" is the same user/org that owns the preview repo.
5. Set the "Repository access" to "Only selected repositories" and then select the preview repo.
6. In the "Repository permissions" sections, set "Actions" and "Content" to "Read and write". "Metadata" will also be granted as "Read-only", as it is required for the other two.
7. Click on "Generate token", copy the token and save it somewhere for later.

#### Classic PAT

1. Go to [Account settings > Developer settings > Tokens (classic)](https://github.com/settings/tokens).
2. Click on "Generate new token" > "Generate new token (classic)"
3. Give it a recognizable name and set an appropriate expiration date.
4. Select the `repo` scope.
5. Click on "Generate token", copy the token and save it somewhere for later.

### Source repo

This steps need to be repeated for each repo you want to use this action on.

1. Go to the repo that contains the source code of your website.
2. Go to Repo settings > Secrets and variables > Actions.
3. Create a new repository secret called `PREVIEW_TOKEN` and paste the PAT you created in the previous step.
4. Go back to the repo contents and add the deployment workflow: you can either create a new one or add the same steps to your existing one. Use the [`dependents/source-repo.yml`](dependents/source_repo.yml) file as a template/example.  
  Make sure to change the `PREVIEW_REPO` and `PAGES_BASE` env variable, along with the commands needed to build your website.  
  Also, make sure to change `EndBug/pages-preview`'s inputs to match your needs: more info on that in the ["Inputs"](#inputs) section of this file.

All done! You're now ready to use the action 🎉

## Inputs

```yaml
- uses: EndBug/pages-preview@v1
  with:
    # The directory in which the website has been built, in the a/b/c format
    build_dir: build

    # The GitHub Pages base URL of the preview repo
    preview_base_url: https://octocat.github.io/preview

    # The repository to push previews to, in the Owner/Name format
    preview_repo: octocat/preview

    # The token to access the preview repo, that you created during setup
    preview_token: ${{ secrets.PREVIEW_TOKEN }}

    # --- OPTIONAL ---
    # The name of the environment to use for the deployment
    # Default: 'preview'
    deployment_env: 'development'

    # Whether to use the deployments API
    # Default: 'true'
    deployments: false

    # The name of the author of the resulting commit
    # Default: the GitHub Actor
    git_author_name: Mona

    # The email of the author of the resulting commit
    # Default: the GitHub Actor's
    git_committer_name: mona@users.noreply.github.com

    # The committer of the resulting commit
    # Default: copies git_author_name
    git_committer_name: GitHub Actions

    # The email of the committer of the resulting commit
    # Default: copies git_author_email
    git_committer_email: 41898282+github-actions[bot]@users.noreply.github.com

    # Whether to comment on PRs
    # Default: 'true'
    pr_comment: 'false'

    # The name of the branch that hosts the previews
    # Default: gh-pages
    preview_branch: custom-pages-branch

    # The name of the workflow file that contains the comment workflow in the preview repo
    # Default: preview.yml
    preview_workflow_file_name: custom_workflow.yml
```

# pre-commit-ci
A drop-in pre-commit pipeline for any repository, designed to be fast.

pre-commit.ci is a great hosted service, but you might want a self-hosted alternative - for private repos, GitHub Enterprise, or to avoid relying on a third party. This workflow gives you the same auto-fix-and-push experience as a reusable GitHub Actions workflow you can call from any repo.

## Features
* Drop-in reusable workflow - call it from any repo with a few lines of config
* Configurable Python version - defaults to 3.14, override per-project
* Fast cold runs with uv - uses Astral's uv for dependency installation
* Cached hook environments - ~/.cache/pre-commit is keyed on OS, Python version, and config hash for near-instant warm runs
* Optional auto-fix - automatically commits formatter and linter fixes back to the PR branch when enabled
* CI-retriggering pushes via GitHub App - supply a bot's client ID and private key as secrets, and auto-fix commits will trigger your other workflows
* Fail-fast mode - disable autofix to make the workflow fail loudly on hook violations instead of patching them


## Pipeline Template
1. Create the file inside your repo.
    ```yaml
    # .github/workflows/pre-commit.yml
    name: pre-commit-ci

    on:
        pull_request:

    jobs:
        run-pre-commit:
            uses: Zaur-Labs-ApS/pre-commit-ci/.github/workflows/pre-commit.yml@v0.0.1
            with:
                python_version: '3.13'
            permissions:
                contents: write
                pull-requests: write
    ```

2. Push a commit and pre-commit will run.

## Create a Github bot (Recommended)
By default, commits made by GITHUB_TOKEN don't trigger other workflows on the PR (this is GitHub's built-in protection against infinite loops). A GitHub App token bypasses this, so your tests and other CI re-run on the auto-fix commit.

1. Create a GitHub app by going to https://github.com/settings/apps. You can also get there by going to GitHub, click on your avatar, choose **Settings**, and **Developer Settings**.

2. Click the **New Github App** button, on the top of the page.

3. Give the bot a name, and a homepage url (Could be your repo or profile url).

4. Inside permissions, give the bot **Read and write** access to **Contents**

5. Click **Create Github App**

6. Copy the **Client ID** for later.

7. Click **Generate a private key** and save for later.

8.  From the App's settings page, click Install App in the left sidebar.

9. Choose the org/account and select the repos where you want the App to operate. "Only select repositories" is recommended.

10. Go into your repository **Secrets and variables** by clicking on the **Settings** tab inside the repository.

11. Click on **Actions**

12. Click **New repository secret**

13. Give the secret a name. E.g: PRE_COMMIT_CLIENT_ID

14. Fill the client id you saved earlier.

15. Click **Add secret**

16. Do the same for the private key you saved earlier.

17. Go into your Github workflow, and add the credentials like this:
    ```yaml
    # .github/workflows/pre-commit.yml
    name: pre-commit-ci

    on:
        pull_request:

    jobs:
        run-pre-commit:
            uses: Zaur-Labs-ApS/pre-commit-ci/.github/workflows/pre-commit.yml@v0.0.1
            with:
                python_version: '3.13'
                bot_username: 'bot-name[bot]'
                bot_email: 'bot-name[bot]@users.noreply.github.com'
            secrets:
                bot_client_id: ${{ secrets.PRE_COMMIT_CLIENT_ID }}
                bot_private_key: ${{ secrets.PRE_COMMIT_PRIVATE_KEY }}
            permissions:
                contents: write
                pull-requests: write
    ```

18. Now your workflow can commit and trigger pipelines.

## Inputs
| Type   | Name                | Description                                                                                                      | Default                                               |
|--------|---------------------|------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------|
| Input  | python_version      | Used as the Python version.                                                                                      | 3.14                                                  |
| Input  | autofix             | If true, auto-commits hook fixes back to the PR branch. If false, the workflow fails when hooks find issues.     | true                                                  |
| Input  | organization_domain | Used if used on the GitHub enterprise platform.                                                                  | github.com                                            |
| Input  | bot_username        | The username of the GitHub bot.                                                                                  | github-actions[bot]                                   |
| Input  | bot_email           | The email of the GitHub bot.                                                                                     | 41898282+github-actions[bot]@users.noreply.github.com |
| Secret | bot_client_id       | The client id of the GitHub bot.                                                                                 |                                                       |
| Secret | bot_private_key     | The private key of the GitHub bot.                                                                               |                                                       |

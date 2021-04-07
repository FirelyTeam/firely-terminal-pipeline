# firely-terminal-pipeline
Run Firely Terminal, and optionally the Java Validator too, as GitHub Actions on your FHIR GitHub repository.

Example setup can be found on [this project](https://github.com/FirelyTeam/ACMEGitHubExample):
* Copy the contents of or download [this example `main.yml` setup file](https://github.com/FirelyTeam/ACMEGitHubExample/blob/main/.github/workflows/main.yml).
* Add the contents to a `main.yml` file in your GitHub repository in the `.github/workflows` folder.
* You should now see the pipeline on your Actions tab and it should run on every push or pull request to the `main` or `master` branch. This can be configured in the yml file.

You can add a badge to your repository with the current pipeline status:
* Go to the Actions tab on your repository
* Select the Firely Validation pipeline
* Select the three dots and click 'Create status badge'
* Paste this code in the beginning of your `README.md` file.

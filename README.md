# firely-terminal-pipeline
Run [Firely Terminal](https://fire.ly/products/firely-terminal/), and optionally the Java Validator too, as GitHub Actions on your FHIR GitHub repository. Validate conformance resource and examples against the FHIR specification and [your custom business rules](https://fire.ly/2021/03/04/quality-control-how-to-validate-full-fhir-specifications-in-one-click/) on every commit and pull request.
<img align="right" width="40%" src="illustration_firely_terminal.png">

Example setup can be found on [this project](https://github.com/FirelyTeam/ACMEGitHubExample):
* Copy the contents of or download [this example `main.yml` setup file](https://github.com/FirelyTeam/ACMEGitHubExample/blob/main/.github/workflows/main.yml).
* Add the contents to a `main.yml` file in your GitHub repository in the `.github/workflows` folder.
* You should now see the pipeline on your Actions tab and it should run on every push or pull request to the `main` or `master` branch. This can be configured in the yml file.

You can add a badge to your repository with the current pipeline status:
* Go to the Actions tab on your repository
* Select the Firely Validation pipeline
* Select the three dots and click 'Create status badge'
* Paste this code in the beginning of your `README.md` file.

## Options

You can specify the following options using the ["with" syntax](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepswith) in your GitHub Actions yml configuration:

* PATH_TO_CONFORMANCE_RESOURCES:
    - description: 'Relative paths of the folder(s) containing FHIR Conformance resources (StructureDefinition, ValueSet, CodeSystem)'
    - required: true
* PATH_TO_EXAMPLES:
    - description: 'Relative paths of the folder(s) containing examples for the FHIR Conformance resources defined by the project'
    - required: false
* DOTNET_VALIDATION_ENABLED:
    - description: 'Boolean flag to run the .NET validator to validate conformance resources and examples'
    - required: false
    - default: true
* JAVA_VALIDATION_ENABLED:
    - description: 'Boolean flag to run the offical HL7 Java validator to validate conformance resources and examples'
    - required: false
    - default: false
* EXPECTED_FAILS:
    -  description: Specify which steps in your validation workflow are expected to fail due to possible bugs in the validator(s). Allowed values: 'VALIDATION_CONFORMANCE_JAVA', 'VALIDATION_CONFORMANCE_DOTNET', 'VALIDATION_EXAMPLES_JAVA', 'VALIDATION_EXAMPLES_DOTNET'
    -  required: false
    

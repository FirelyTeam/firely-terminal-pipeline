# firely-terminal-pipeline
Run [Firely Terminal](https://fire.ly/products/firely-terminal/), and optionally the Java Validator too, as GitHub Actions on your FHIR GitHub repository. Validate conformance resource and examples against the FHIR specification and [your custom business rules](https://fire.ly/2021/03/04/quality-control-how-to-validate-full-fhir-specifications-in-one-click/) on every commit and pull request.
<img align="right" width="40%" src="illustration_firely_terminal.png">

Example setup can be found on [this project](https://github.com/FirelyTeam/fhir-specification-template-repository):
* Copy the contents of or download [this example `main.yml` setup file](https://github.com/FirelyTeam/fhir-specification-template-repository/blob/main/.github/workflows/main.yml).
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
* PATH_TO_QUALITY_CONTROL_RULES:
    - description: 'Relative path pointing to Quality Control rules. Path MUST not include the .rules.yaml part of the file. Runs minimal.rules.yaml by default.'
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
* OUTPUT_FORMAT:
    - description: Specify the format of the validation output: Allowed values: 'RAW', 'SUMMARY' (produces a markdown compatible overview of all validation issues)
    -  required: false
 * JAVA_VALIDATION_OPTIONS:
   - description: 'Custom options passed to the Java validator. See https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator'
   - required: false
 * SIMPLIFIER_USERNAME:
   - description: 'Simplifier email address (not username), needed for running Quality Control checks. Please use GitHub Secrets for this variable.'
   - required: true
 * SIMPLIFIER_PASSWORD:
   - description: 'Simplifier password, needed for running Quality Control checks. Please use GitHub Secrets for this variable.'
   - required: true

## Changelog

### v0.2.1 - 2021-06-26
- Feature: Upgrade Java validator to version 5.4.6

### v0.2.0  - 2021-06-23
- Feature: Enable Quality Control if DOTNET_VALIDATION_ENABLED is enabled. A Simplifier Login is required to use this feature. See SIMPLIFIER_USERNAME and SIMPLIFIER_PASSWORD option.
- Feature: Upgrade Java validator to version 5.4.5

### v0.1.0-beta2 - 2021-04-22
- Feature: Upgrade Java validator to version 5.3.11

### v0.1.0-beta1 - 2021-04-14
- Feature: Upgrade Java validator to version 5.3.9

### Intital Release - 2021-04-08
    
## Known Issues
- PATH_TO_EXAMPLES is currently not supported in combination with DOTNET_VALIDATION_ENABLED. The .NET part will currently ignore examples.
- All folders in PATH_TO_CONFORMANCE_RESOURCES are currently validated individually in the Java validator. Each folders therefore is validated in it's own context with no access to the other folders

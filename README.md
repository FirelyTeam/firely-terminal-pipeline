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

## Input (FHIR Conformance resources)

The Firely Terminal Pipeline allows to validate all FHIR conformnce resources and corresponding examples.
The pipeline expects the following content to be present in the repository:

* package.json containing all dependencies of the project (required, root-level of the project)
* project.yaml containing [canonical claims](https://docs.fire.ly/projects/Simplifier/simplifierCanonicalClaims.html) (optional, root-level of the project)
* A folder containing all terminology resources (optional, path can be configured. See PATH_TO_CONFORMANCE_RESOURCES)
* A folder containing all StructureDefinitions (optional, path can be configured. See PATH_TO_CONFORMANCE_RESOURCES)
* A folder containing all examples (optional, path can be configured. See PATH_TO_EXAMPLES)

## Options

You can specify the following options using the ["with" syntax](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepswith) in your GitHub Actions yml configuration:

* PATH_TO_CONFORMANCE_RESOURCES:
    - description: 'Relative paths of the folder(s) containing FHIR Conformance resources (StructureDefinition, ValueSet, CodeSystem).'
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
    -  description: Specify which steps in your validation workflow are expected to fail due to possible bugs in the validator(s). Allowed values: 'VALIDATION_CONFORMANCE_DOTNET', 'VALIDATION_CONFORMANCE_JAVA', 'VALIDATION_EXAMPLES_JAVA'
    -  required: false
 * JAVA_VALIDATION_OPTIONS:
   - description: 'Custom options passed to the Java validator. See https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator'
   - default: '-output-style compact'
   - required: false
 * SIMPLIFIER_USERNAME:
   - description: 'Simplifier email address (not username), needed for running Quality Control checks. Please use GitHub Secrets for this variable.'
   - required: true
 * SIMPLIFIER_PASSWORD:
   - description: 'Simplifier password, needed for running Quality Control checks. Please use GitHub Secrets for this variable.'
   - required: true
 * SUSHI_ENABLED:
   - description: 'Boolean flag to run fsh-sushi on the current repository to generated conformance resources based on FHIR shorthand'
   - required: false
   - default: false
 * SUSHI_OPTIONS:
   - description: 'Custom options passed to SUSHI. See https://www.npmjs.com/package/fsh-sushi'
   - required: false
 * FIRELY_TERMINAL_VERSION:
   - description: 'Version of Firely Terminal used for .NET-based validation'
   - default: '3.3.2'
   - required: true
 * FIRELY_TERMINAL_VALIDATOR_ENGINE:
   - description: 'Firely Terminal validator engine to use for .NET validation. Available options: Regular, Advanced'
   - default: 'Advanced'
   - required: false
 * JAVA_VALIDATOR_VERSION:
   - description: 'Version of org.hl7.fhir.core library used for Java-based validation'
   - default: '6.5.2'
   - required: true
* JAVA_VALIDATOR_DOWNLOAD_LOCATION:
    description: 'URL from which to download the Java validator JAR'
    default: 'https://github.com/hapifhir/org.hl7.fhir.core/releases/download/$JAVA_VALIDATOR_VERSION/validator_cli.jar'
 * SUSHI_VERSION:
   - description: 'Version of SUSHI used for compiling the FSH files'
   - default: '3.13.1'
   - required: true

## Changelog

### v0.6.0 - 2025-01-01

- Feature: Upgrade SUSHI to v3.14.0

### v0.5.0 - 2024-12-26

This version is beside the fix mentioned below identical to v0.4.7. Due to the previously removed and changed options, the last release should have been released as v0.5.0 already. Sorry!

- Fix: Adjusted color for "All OK" output

### v0.4.7 - 2024-12-26

- Feature: Upgrade SUSHI to v3.13.1
- Feature: Upgrade Java validator to v6.5.2
- Feature: Added JAVA_VALIDATOR_DOWNLOAD_LOCATION as an input variable to specify from where to download the Java validator .jar
- Feature: Use '-output-style compact' as the default for JAVA_VALIDATION_OPTIONS. This enhances the readability of the log. It's recommended to include this in case JAVA_VALIDATION_OPTIONS get overridden by a local config
- Feature: Use colors to highlight "Error", "Warning", "Information" messages in the output of the Java validator
- Changed: Remove OUTPUT_FORMAT variable, use '-output-style compact' instead

### v0.4.6 - 2024-12-12

- Feature: Upgrade Java validator to v6.5.0

### v0.4.5 - 2024-11-25

Hotfix for v0.4.4 which accidentally increased the SUSHI version to v3.12.2 which doesn't exist (yet)

### v0.4.4 - 2024-11-25

- Feature: Upgrade Firely Terminal to v3.3.2
- Feature: Upgrade Java validator to v6.4.4
- Feature: Upgrade SUSHI to v3.12.2

### v0.4.3 - 2024-10-18

- Feature: Upgrade Firely Terminal to v3.2.0
- Feature: Upgrade Java validator to v6.3.32
- Feature: Upgrade SUSHI to v3.12.0

### v0.4.2 - 2024-06-05

- Feature: Upgrade Java validator to v6.3.14
- Feature: Upgrade SUSHI to v3.11.0

### v0.4.1 - 2024-03-18

- Feature: Upgrade Firely Terminal to v3.1.0
- Feature: Upgrade Java validator to v6.3.1
- Feature: Upgrade SUSHI to v3.8.0

### v0.4.0 - 2023-02-20

- Feature: Upgrade Java validator to v5.6.98
- Feature: Remove 1 by 1 validation with .NET validator in favor for Simplifier Quality Control

### v0.3.5 - 2022-09-22

- Fix: Always execute fhir restore if SUSHI is enabled
- Fix: Run SUSHI before validation
- Fix: Only install Firely Terminal if it's not already installed
- Feature: Add FIRELY_TERMINAL_VERSION, JAVA_VALIDATOR_VERSION, SUSHI_VERSION as options
- Feature: Upgrade Firely Terminal to v3.0.0
- Feature: Upgrade Java validator to v5.6.65
- Feature: Upgrade SUSHI to v2.7.0

### v0.3.4 - 2022-06-01

- Feature: Upgrade SUSHI to v2.5.0

### v0.3.3 - 2022-03-30

- Fix: Only load XML and JSON files in Java validator
- Fix: Check fhirVersion in package.json and execute fhir spec
- Feature: Upgrade Java validator to v5.6.39

### v0.3.2 - 2022-01-26

- Feature: Upgrade Firely.Terminal to v2.5.0-beta-7
- Feature: Upgrade Java validator to v5.6.27
- Feature: Upgrade SUSHI to v2.2.6

### v0.3.1 - 2021-12-08

- Feature: Upgrade Firely.Terminal to v2.5.0-beta-4

### v0.3.0 - 2021-11-25

- Feature: Add SUSHI_ENABLED and SUSHI_OPTIONS options
- Feature: Add option to push resource without subfolders
- Feature: Update to Firely.Terminal v2.5.0-beta-2
- Feature: Update Java validator to v5.6.0

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

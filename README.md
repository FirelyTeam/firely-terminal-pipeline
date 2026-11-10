# firely-terminal-pipeline
Run [Firely Terminal](https://fire.ly/products/firely-terminal/), and optionally the Java Validator too, as GitHub Actions on your FHIR GitHub repository. Validate conformance resources and examples against the FHIR specification and [your custom business rules](https://fire.ly/2021/03/04/quality-control-how-to-validate-full-fhir-specifications-in-one-click/) on every commit and pull request.
<img align="right" width="40%" src="illustration_firely_terminal.png">

Example setup can be found on [this project](https://github.com/FirelyTeam/fhir-specification-template-repository):
* Copy the contents of or download [this example `main.yml` setup file](https://github.com/FirelyTeam/fhir-specification-template-repository/blob/main/.github/workflows/main.yml).
* Add the contents to a `main.yml` file in your GitHub repository in the `.github/workflows` folder.
* Make sure the `dotnet-version` in the `actions/setup-dotnet` step matches the Firely Terminal version you use (see [Prerequisites](#prerequisites)).
* You should now see the pipeline on your Actions tab and it should run on every push or pull request to the `main` or `master` branch. This can be configured in the yml file.

You can add a badge to your repository with the current pipeline status:
* Go to the Actions tab on your repository
* Select the Firely Validation pipeline
* Select the three dots and click 'Create status badge'
* Paste this code in the beginning of your `README.md` file.

## Prerequisites

The action runs on Linux x64 runners, such as `ubuntu-latest`. It does not install .NET, Java or Node.js itself: set up the tools you need in earlier steps of your workflow.

| Tool | Needed when | Setup action |
|------|-------------|--------------|
| .NET SDK | `DOTNET_VALIDATION_ENABLED` or `TERMINOLOGY_SERVICE_BFARM_ENABLED` is `true` | [actions/setup-dotnet](https://github.com/actions/setup-dotnet) |
| Java | `JAVA_VALIDATION_ENABLED` is `true` | [actions/setup-java](https://github.com/actions/setup-java) |
| Node.js / npm | `SUSHI_ENABLED` is `true` | [actions/setup-node](https://github.com/actions/setup-node) |

Quality Control checks with the .NET validator also need a [Simplifier](https://simplifier.net) account, see `SIMPLIFIER_USERNAME` and `SIMPLIFIER_PASSWORD`.

### .NET version

Firely Terminal 4.x (the default `FIRELY_TERMINAL_VERSION`) requires the **.NET 10 SDK**:

```yaml
- name: Setup .NET SDK
  uses: actions/setup-dotnet@v6
  with:
    dotnet-version: 10.0.x
```

If your workflow still sets up .NET 8 (`dotnet-version: 8.0.x`), installing Firely Terminal 4.x will fail. Either switch to `10.0.x`, or pin `FIRELY_TERMINAL_VERSION: '3.5.0'`, which is the last version that runs on .NET 8.

## Input (FHIR Conformance resources)

The Firely Terminal Pipeline validates the FHIR conformance resources and corresponding examples in your repository.
It expects the following content to be present:

* `package.json` at the root of the repository, containing the FHIR version (`fhirVersions`) and all dependencies of the project. If there is no `package.json` but there is a `sushi-config.yaml`, the action generates a `package.json` from it (see `SUSHI_USE_CONFIG_DEPENDENCIES`).
* `project.yaml` containing [canonical claims](https://docs.fire.ly/projects/Simplifier/simplifierCanonicalClaims.html) (optional, root-level of the project)
* One or more folders containing all conformance resources, such as StructureDefinitions, ValueSets and CodeSystems (see `PATH_TO_CONFORMANCE_RESOURCES`)
* One or more folders containing all examples (optional, see `PATH_TO_EXAMPLES`)

Supported FHIR versions are 3.0.2 (STU3), 4.0.1 (R4), 4.3.0 (R4B), 5.0.0 (R5), and the ballot versions 4.1.0 (R4B) and 4.6.0 (R5). For other versions the action logs a warning, and .NET validation stops with an error.

## Options

You can specify the following options using the ["with" syntax](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepswith) in your GitHub Actions yml configuration. Where an option takes multiple paths or values, separate them with spaces.

### Paths

* `PATH_TO_CONFORMANCE_RESOURCES` (required)
    - Relative path(s) of the folder(s) containing FHIR conformance resources (StructureDefinition, ValueSet, CodeSystem).
* `PATH_TO_EXAMPLES` (optional)
    - Relative path(s) of the folder(s) containing examples for the FHIR conformance resources defined by the project. Examples are only validated by the Java validator, see [Known Issues](#known-issues).
* `PATH_TO_QUALITY_CONTROL_RULES` (optional)
    - Relative path pointing to Quality Control rules. The path MUST NOT include the `.rules.yaml` part of the file name. Runs `minimal.rules.yaml` by default.

### Validation

* `DOTNET_VALIDATION_ENABLED` (optional, default: `true`)
    - Run Firely Terminal Quality Control checks, including validation with the .NET validator. Requires the .NET SDK, `SIMPLIFIER_USERNAME` and `SIMPLIFIER_PASSWORD`.
* `JAVA_VALIDATION_ENABLED` (optional, default: `false`)
    - Validate conformance resources and examples with the official HL7 Java validator. Requires Java.
* `EXPECTED_FAILS` (optional)
    - Steps in your validation workflow that are expected to fail, for example because of bugs in the validator(s). Errors in these steps are reported but don't fail the workflow. Allowed values: `VALIDATION_CONFORMANCE_DOTNET`, `VALIDATION_CONFORMANCE_JAVA`, `VALIDATION_EXAMPLES_JAVA`.
* `CLOSE_SLICING_FOR_VALIDATION` (optional, default: `false`)
    - Constrains all defined element slicings to closed, ensuring that instance elements must conform to one of the declared slices. This strengthens validation by preventing acceptance of elements that do not match the defined slicing discriminators.
    - Only JSON StructureDefinitions that are not used as the base of another profile in the project are changed. The files are modified in place before validation; if `SUSHI_ENABLED` is `true`, SUSHI runs again afterwards to restore them.

### Simplifier

* `SIMPLIFIER_USERNAME` (required when `DOTNET_VALIDATION_ENABLED` is `true`)
    - Simplifier email address (not username), needed for running Quality Control checks. Please use GitHub Secrets for this variable.
* `SIMPLIFIER_PASSWORD` (required when `DOTNET_VALIDATION_ENABLED` is `true`)
    - Simplifier password, needed for running Quality Control checks. Please use GitHub Secrets for this variable.

### Firely Terminal (.NET)

* `FIRELY_TERMINAL_VERSION` (default: `'4.0.0'`)
    - Version of Firely Terminal used for .NET-based validation. Firely Terminal 4.x requires the .NET 10 SDK; use `3.5.0` if you need to stay on .NET 8. See [.NET version](#net-version).
    - If a `fhir` command is already available on the runner, that installation is used and this option is ignored.
* `FIRELY_TERMINAL_VALIDATOR_ENGINE` (optional)
    - Validator engine to use for .NET validation. Available options: `firely`, `netsdk` (lowercase). From Firely Terminal 3.5.0 onwards, the default validator engine changed; use `netsdk` to use the old validator engine from pre-3.5.0 versions. If not specified, Firely Terminal uses its built-in default.
* `TERMINOLOGY_SERVICE_BFARM_ENABLED` (optional, default: `false`)
    - Download terminology packages from the central terminology server for Germany ([ZTS](https://terminologien.bfarm.de/fhirpackages.html)). Packages listed under `devDependencies` in `package.json` that exist in the ZTS catalog are downloaded into the Firely Terminal package cache. Requires the .NET SDK.

### Java validator

* `JAVA_VALIDATOR_VERSION` (default: `'7.0.3'`)
    - Version of the org.hl7.fhir.core library used for Java-based validation.
* `JAVA_VALIDATOR_DOWNLOAD_LOCATION` (default: `'https://github.com/hapifhir/org.hl7.fhir.core/releases/download/$JAVA_VALIDATOR_VERSION/validator_cli.jar'`)
    - URL from which to download the Java validator JAR. `$JAVA_VALIDATOR_VERSION` in the URL is replaced with the value of `JAVA_VALIDATOR_VERSION`.
* `JAVA_VALIDATION_OPTIONS` (optional, default: `'-output-style compact'`)
    - Custom options passed to the Java validator. See [Using the FHIR Validator](https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator). If you override this option, consider keeping `-output-style compact` for a readable log.
* `JAVA_SNAPSHOT_ENABLED` (optional, default: `false`)
    - Let the Java validator create the snapshots for the package dependencies. Snapshots created by Firely Terminal are disregarded: the Firely Terminal package cache is removed before Java validation, so Firely Terminal must be installed (`DOTNET_VALIDATION_ENABLED` is `true`). Does not yet work together with `TERMINOLOGY_SERVICE_BFARM_ENABLED`.

### SUSHI

* `SUSHI_ENABLED` (optional, default: `false`)
    - Run [SUSHI](https://www.npmjs.com/package/fsh-sushi) on the repository to generate conformance resources from FHIR Shorthand (FSH) before validation. Requires Node.js / npm.
* `SUSHI_VERSION` (default: `'3.21.0'`)
    - Version of SUSHI used for compiling the FSH files.
* `SUSHI_OPTIONS` (optional)
    - Custom options passed to SUSHI. See https://www.npmjs.com/package/fsh-sushi
* `SUSHI_USE_CONFIG_DEPENDENCIES` (optional, default: `false`)
    - Controls whether `sushi-config.yaml` or `package.json` is the source of the FHIR version and dependencies:
        - `true`: a `package.json` is generated from `sushi-config.yaml`, overwriting an existing `package.json`. The action fails if there is no `sushi-config.yaml`.
        - `false`: an existing `package.json` is used. If there is no `package.json`, one is generated from `sushi-config.yaml`.

## Caching

The action caches the FHIR package cache (`~/.fhir/packages`) and the Java validator JAR with [GitHub Actions caches](https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching), so later runs and other jobs in the same repository don't download them again.

- The FHIR package cache is keyed on `package.json`, `FIRELY_TERMINAL_VERSION` and `TERMINOLOGY_SERVICE_BFARM_ENABLED`. It is only used when the .NET validator is enabled. It is not used when a dependency has no fixed version (`current`, `dev`, `latest` or a version range), because those change over time.
- The Java validator JAR is keyed on `JAVA_VALIDATOR_VERSION` and `JAVA_VALIDATOR_DOWNLOAD_LOCATION`.
- To force a fresh download, delete the cache under **Actions > Caches** in your repository.

## Known Issues
- Examples in `PATH_TO_EXAMPLES` are not validated by the .NET validator, because Firely Terminal Quality Control can't distinguish examples from conformance resources. Enable `JAVA_VALIDATION_ENABLED` to validate examples.
- The Java validator validates each folder in `PATH_TO_CONFORMANCE_RESOURCES` and `PATH_TO_EXAMPLES` separately (with all conformance resource folders loaded as context). Only `.json` and `.xml` files directly in these folders are validated; subfolders are not included.

## Changelog

Release notes for v0.6.1 and later are published on the [Releases page](https://github.com/FirelyTeam/firely-terminal-pipeline/releases). Notable changes are listed below.

### Next release

- Cache the FHIR packages and the Java validator JAR between runs. See [Caching](#caching).
- Breaking: Upgrade Firely Terminal to v4.0.0, which requires the .NET 10 SDK. Update `dotnet-version` in your `actions/setup-dotnet` step to `10.0.x`, or pin `FIRELY_TERMINAL_VERSION: '3.5.0'` to stay on .NET 8. See [.NET version](#net-version).
- Fix: Support FHIR 4.3.0 (R4B release). Previously only the R4B ballot version 4.1.0 was recognized, so .NET validation of R4B projects failed.

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

### Initial Release - 2021-04-08

# Compute Version GitHub Action

"Compute Version" is a GitHub Action tailored for projects employing the GitFlow branching strategy and adhering to Semantic Versioning (SemVer). It automates version computation across CI/CD pipelines, utilizing branch names and git tags to dynamically generate version numbers at various development stages.

## Integrating GitFlow and SemVer

This action is particularly designed for teams following the GitFlow branching model, which involves maintaining separate branches for features, releases, and hotfixes. Combined with SemVer, it ensures that every merge into the mainline (main or master) and development (develop) branches results in a predictable and meaningful version increment.

## Features

- **Git Tag Analysis**: Identifies the latest version based on git tags, aligning with SemVer tagging practices.
- **Branch-Specific Versioning**: Generates versions according to the GitFlow branch type:
  - Feature branches result in pre-release versions.
  - Release branches trigger minor or patch version increments.
  - Hotfix branches may result in patch version increments.
- **Initial Versioning for New Projects**: Defaults to `0.1.0` for repositories without previous tags, facilitating a standardized starting point.
- **JIRA Card ID Integration**: Enhances version strings with JIRA card IDs from branch names, providing traceability.
- **Version Compliance**: Validates generated versions against SemVer 2.0, Docker tag, and OCI tag standards.

## Inputs

| Input              | Description                                        | Required | Default |
|--------------------|----------------------------------------------------|----------|---------|
| `test_version_tag` | Test version tag for testing or simulation.        | No       | ''      |

## Outputs

| Output    | Description                             |
|-----------|-----------------------------------------|
| `version` | The computed SemVer-compatible string.  |

## Usage with GitFlow and SemVer

### Pre-requisites

- Initialize your project with GitFlow branching strategy.
- Ensure the repository is checked out using `actions/checkout@v3` before this action.

### Example Workflow

This workflow demonstrates using the Compute Version action within a GitFlow and SemVer context:

```yaml
name: Compute and Use Version

on: [push, pull_request]

jobs:
  versioning:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        name: Checkout repository
      
      - name: Compute Version
        id: compute_version
        uses: ActionsCI/ComputeVersion@1.0.0
        with:
          test_version_tag: '' # Optional: Specify for testing purposes
        
      - name: Use Computed Version
        run: echo "Computed version: ${{ steps.compute_version.outputs.version }}"
Handling Different Branch Types in GitFlow
Feature Branches: Generate pre-release versions to indicate ongoing development.
Release Branches: Increment the version to the next minor or patch level, preparing for a stable release.
Hotfix Branches: Similar to release branches, hotfixes result in an immediate patch version increment to address urgent issues.
Contributing
Your contributions and suggestions are welcome to improve the integration and functionality of "Compute Version" within GitFlow and SemVer practices.

Support and Issues
For support or to report issues, please open an issue in the GitHub repository's issue tracker.

License
This action is released under the MIT License. See the LICENSE file in the repository for more details.





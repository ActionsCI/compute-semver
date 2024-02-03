# Compute Version GitHub Action

This GitHub Action, "Compute Version", is designed to automate the process of version computation based on branch names and git tags within your CI/CD pipelines. It's especially useful for projects that adhere to Semantic Versioning and require dynamic version calculation at various stages of development and deployment.

## What This Action Does

- **Fetches Git Tags**: Determines the most recent version tag from the git history.
- **Branch Name Analysis**: Computes a new version based on the branch name (supports feature branches and release branches).
- **Fallback Versioning**: Defaults to `0.1.0` if no previous tags are found, ensuring a smooth operation for new projects.
- **JIRA Card ID Handling**: Optionally incorporates JIRA card IDs from branch names into the version, with a fallback for branches without JIRA IDs.
- **Version Validation**: Ensures the computed version adheres to Semantic Versioning 2.0, Docker tag, and OCI tag standards.

## Inputs

| Input             | Description                                       | Required | Default |
|-------------------|---------------------------------------------------|----------|---------|
| `test_version_tag`| Optional test version tag for testing purposes.   | No       | ''      |

## Outputs

| Output    | Description                             |
|-----------|-----------------------------------------|
| `version` | The computed version string.            |

## Usage

### Pre-requisites

Ensure your workflow includes a step to check out the repository using `actions/checkout@v3` before using this action.

### Example Workflow

This example demonstrates how to include the Compute Version action in a workflow to compute and use a version string based on the current branch and git tags.

```yaml
name: Compute and Use Version

on: [push]

jobs:
  versioning:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
      
    - name: Compute Version
      id: compute_version
      uses: ActionsCI/ComputeVersion@1.0.0
      with:
        test_version_tag: '' # Optional: Specify this for testing purposes
        
    - name: Use Computed Version
      run: echo "The computed version is ${{ steps.compute_version.outputs.version }}"
Handling Different Branch Types
Release Branches
For release branches (e.g., release/1.2.3), the action will increment the patch version by default.

Feature Branches
For feature branches, especially those not following a strict naming convention, the action will generate a version string incorporating the short SHA commit hash to ensure uniqueness.

Contributing
We welcome contributions and suggestions to improve this action. Please follow the contributing guidelines in the repository.

Support and Issues
If you encounter any issues or require assistance, please file an issue on the GitHub repository issue tracker.

License
This action is released under the MIT License. See the LICENSE file for more details.

markdown
Copy code

### Notes for Customization:
- **Repository URL and Action Name**: Replace `ActionsCI/ComputeVersion@main` with the actual GitHub path to your action if it differs.
- **Contribution Guidelines**: If you have specific contribution guidelines, mention them or link to the file in the repository.
- **License Information**: Ensure the license link (`[LICENSE](LICENSE)`) points to the correct license file in your repository.

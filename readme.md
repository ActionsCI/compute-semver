# Compute Version GitHub Action

"Compute Version" is a GitHub Action designed to facilitate version computation in CI/CD pipelines, leveraging branch names and git tags. This action is optimized for projects utilizing trunk-based development practices and adhering to Semantic Versioning (SemVer), dynamically generating version numbers to streamline deployments and releases.

## Trunk-Based Development and SemVer

This action supports trunk-based development, a strategy that involves frequent merges to a single branch (often `main` or `master`). Combined with SemVer, it ensures consistent and meaningful version increments, aligning version management with continuous integration principles.

## Features

- **Git Tag Analysis**: Determines the most recent version tag, ensuring alignment with SemVer practices.
- **Dynamic Version Generation**: Automates version number creation based on the commit context, supporting the linear history characteristic of trunk-based development.
- **Initial Project Versioning**: Provides a default starting version (`0.1.0`) for new projects, establishing a clear versioning baseline.
- **JIRA Card ID Integration**: Optionally incorporates JIRA card IDs from commit messages or branch names into the version, enhancing traceability.
- **Version Validation**: Confirms that the generated version complies with Semantic Versioning 2.0, Docker tag, and OCI tag standards.

## Inputs

| Input              | Description                                  | Required | Default |
|--------------------|----------------------------------------------|----------|---------|
| `test_version_tag` | Test version tag for simulations.            | No       | ''      |
| `test_branch_name` | Test branch name for simulations.            | No       | ''      |

## Outputs

| Output    | Description                               |
|-----------|-------------------------------------------|
| `version` | The SemVer-compatible version string.     |

## Usage in Trunk-Based Development

### Pre-requisites

- Adopt a trunk-based development workflow.
- Use `actions/checkout@v3` to check out the repository before running this action.

### Example Workflow

This example showcases how to compute and utilize a version string in a trunk-based development context:

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
          test_version_tag: '' # Optional
        
      - name: Use Computed Version
        run: echo "Version: ${{ steps.compute_version.outputs.version }}"
## Contributing
Contributions to enhance "Compute Version" are welcome, especially those that further refine its support for trunk-based and SemVer practices.

## Support and Issues
Encounter an issue or need assistance? Please file an issue in the GitHub repository's issue tracker.

## License
This action is available under the MIT License. See LICENSE in the repository for more details.

## Support This Project
If "Compute Version" adds value to your workflow, please consider supporting its development:

PayPal: Donate via PayPal.me @ paypal.me/burninmedia
Cryptocurrency: Ethereum Address - 0xbB767477D28560672BE7b2E2270c70F80F9341eE
Your support is greatly appreciated and helps sustain ongoing development.

# Compute Version Action

This action computes a version string based on the branch name and existing git tags, suitable for use in continuous integration and deployment workflows.

## Description

The `compute-version` action is designed to automatically generate a version identifier for your software builds. This action is particularly useful in a CI/CD pipeline, ensuring each build can be uniquely identified and traced back to a specific state in your repository.

## How It Works

- For release branches (named `release/...`), the action increments the patch version based on the latest semantic version tag.
- For other branches (e.g., feature branches), it constructs a version string combining the latest version, a Jira card ID extracted from the branch name, and a short SHA.
- The generated version is validated against Semantic Versioning 2.0, Docker tag, and OCI tag standards using regular expressions.

## Usage

To use this action in your workflow, add the following step:

```yaml
steps:
  - name: Compute and Validate Version
    id: compute-version
    uses: path/to/compute-version
```

You can then access the computed version in subsequent steps of your workflow:

```yaml
steps:
  - name: Use Computed Version
    run: echo "The computed version is ${{ steps.compute-version.outputs.version }}"
```

## Inputs

This action does not require any inputs.

## Outputs

### `version`

The computed version string.

## Additional Notes

- Ensure that your repository's tags follow the semantic versioning format (e.g., `v1.2.3`).
- The action's version computation logic may need to be adjusted based on your project's specific versioning needs and branch naming conventions.

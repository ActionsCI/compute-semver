## Usage

To use this action in your workflows, ensure you first check out your repository using the `actions/checkout@v2` action. Then, add this action to compute the version based on branch names and git tags.

### Workflow Example

Below is a simple example of how to incorporate the Compute Version action into your workflow:

```yml
jobs:
  versioning:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: ActionsCI/ComputeVersion@main
      id: compute_version
    - run: echo "Computed Version: ${{ steps.compute_version.outputs.version }}"
```

### Inputs

| Input             | Description                                       | Required | Default |
|-------------------|---------------------------------------------------|----------|---------|
| `test_version_tag`| Optional test version tag for testing purposes.   | No       | `''`    |

### Outputs

| Output    | Description                             |
|-----------|-----------------------------------------|
| `version` | The computed version string.            |
```

### Revised Example Workflows

#### Basic Semantic Versioning

```yml
# Example of a release workflow
name: Release Workflow
on:
  push:
    branches:
      - 'release/*'

jobs:
  compute-version:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: ActionsCI/ComputeVersion@main
      id: compute_version
    - run: echo "Release version: ${{ steps.compute_version.outputs.version }}"
```

#### Feature Branch Versioning

```yml
# Example for feature branches
name: Feature Branch Workflow
on:
  push:
    branches:
      - 'feature/**'

jobs:
  compute-version:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: ActionsCI/ComputeVersion@main
      id: compute_version
    - run: echo "Feature version: ${{ steps.compute_version.outputs.version }}"
```

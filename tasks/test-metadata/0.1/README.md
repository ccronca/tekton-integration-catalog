# Test Metadata

Version: 0.1

The `test-metadata` task is designed to extract and log metadata from a JSON snapshot or PipelineRun object during a Tekton pipeline execution. This task helps in identifying and documenting critical information such as the event type (push or pull request), Git URL, Git revision, container image, GitHub organization, repository, pull request number, and pull request author.

## Parameters

The task accepts the following parameters:

- **SNAPSHOT**: A JSON string representing the Snapshot under test. This JSON contains detailed information about the components and their sources.
- **test-name**: A string representing the name of the test being executed.
- **oras-container**: A string representing the ORAS container used to store all test artifacts. A new container will be generated with a test tag from this container.

### Example of SNAPSHOT

```json
{
  "components": [
    {
      "name": "component-1",
      "source": {
        "git": {
          "url": "https://github.com/example/repo",
          "revision": "commit-sha"
        }
      },
      "containerImage": "quay.io/example/component-1:latest"
    }
  ]
```

## Results

The task produces the following results:

1. **test-event-type**: Indicates if the job is triggered by a Pull Request or a Push event.
    - **Description**: This result helps to understand the context of the event that triggered the job. Knowing whether the job was triggered by a push (direct commit to the repository) or a pull request (a proposed change from a fork or a branch) can influence how the results are interpreted and acted upon.

2. **pull-request-number**: The pull request number if the job is triggered by a pull request event.
    - **Description**: This result provides the specific pull request number, which is crucial for tracking changes and associating test results with the correct pull request in a CI/CD system.

3. **git-url**: The Git URL from which the test pipeline is originating. This can be from a fork or the original repository.
    - **Description**: This result identifies the source repository URL, helping to trace back the origin of the code being tested. It is useful for verifying the source of the changes and ensuring they come from a trusted repository.

4. **git-revision**: The Git revision (commit SHA) from which the test pipeline is originating.
    - **Description**: This result specifies the exact commit SHA being tested, which is vital for reproducibility and debugging. It allows developers to know the precise state of the codebase at the time of testing.

5. **container-image**: The container image built from the specified Git revision.
    - **Description**: This result provides the reference to the container image that was built during the pipeline execution. It helps in verifying which image was produced from the specific code revision and is essential for deployment and further testing.

6. **git-org**: The GitHub organization from which the test is originating.
    - **Description**: This result indicates the organization or user namespace on GitHub where the repository resides. It helps in organizing and identifying the source of the code.

7. **git-repo**: The repository from which the test is originating.
    - **Description**: This result provides the name of the repository containing the code under test. It is useful for documentation, tracking, and context about the source of the changes.

8. **oras-container**: The ORAS container used to store all test artifacts.
    - **Description**: This result specifies the ORAS (OCI Registry as Storage) container where test artifacts are stored. It is crucial for retrieving and managing artifacts generated during the test, such as logs, reports, and other outputs.

9. **pull-request-author**: The GitHub author of the pull request event.
    - **Description**: This result identifies the author of the pull request, providing context about who made the changes.

10. **target-repo-branch**: The Git branch which the test pipeline is targeting. This can be from an original repository the Pull Request or Push event is targeting.
    - **Description**: This result identifies the target repository branch.

11. **job-spec**: The konflux CI job spec metadata generated.
    - **Description**: This result contains a JSON object with comprehensive details about the job specification, including container image, component name, git information, and event type. It encapsulates all the relevant metadata for the CI job in a structured format, aiding in documentation and analysis.

## Usage

To effectively utilize the `test-metadata` task in a Tekton Pipeline, you can follow the example provided below. This example demonstrates how to define a Tekton pipeline that includes the `test-metadata` task, ensuring that all necessary parameters are specified correctly.

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: test-metadata-pipeline
spec:
  tasks:
    - name: extract-metadata
      taskRef:
        resolver: git
        params:
          - name: url
            value: https://github.com/konflux-ci/konflux-qe-definitions
          - name: revision
            value: main
          - name: pathInRepo
            value: common/tasks/test-metadata/test-metadata.yaml
      params:
        - name: SNAPSHOT
          value: "<your-snapshot-json>"
        - name: test-name
          value: "<your-test-name>"
        - name: oras-container
          value: "<your-oras-container-without-tag>"
```

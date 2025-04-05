# Chocolatine GitHub Actions Workflow

## Overview

**Chocolatine** is an automated CI/CD workflow built with GitHub Actions, designed to enhance the development lifecycle by ensuring that your code:
- Follows coding style guidelines.
- Compiles correctly.
- Passes tests.
- Does not use banned functions.
- Pushes updates to a mirror repository.

This workflow will automatically trigger on push or pull request events, but it will **ignore branches** that start with `ga-ignore`.

## Features

The Chocolatine workflow performs the following tasks:

1. **Coding Style Checking**: It checks your code against the Epitech coding style using a dedicated Docker container. Errors are flagged as annotations in GitHub.
2. **Compilation Check**: It ensures that your project compiles by running `make`. It checks if the expected executables are compiled and executable.
3. **Test Execution**: It runs the `make tests_run` command to execute your project’s tests.
4. **Banned Function Check**: It checks if the code uses any banned functions like GLIBC functions, which can be restricted in certain environments.
5. **Push to Mirror Repository**: If all checks pass, it pushes the code to a specified mirror repository using SSH keys.

## Bonus !
1. **Banned Function Check**.
2. **Coding Style Checker**: Coding style issues are marked as warnings, not errors, so the job does not fail.
3. **Compilation Checker**: A message is displayed in all cases: if the binary doesn't exist, if it is found but is not a Linux binary.
4. **Pushing to Mirror**: The Pixadev GitHub action is not used, but I implemented my own.

### Workflow Structure

The workflow consists of the following jobs:

1. **check_coding_style**: Checks if the code adheres to the required coding style.
2. **check_program_compilation**: Runs `make` to verify that the code compiles correctly and checks for the existence and executability of expected binaries.
3. **run_tests**: Runs the unit tests using `make tests_run` to ensure that everything is functioning as expected.
4. **check_banned_function**: Ensures that banned functions (such as GLIBC functions) are not used in the project.
5. **push_to_mirror**: Pushes the final code to a mirror repository if the push event is triggered and all previous jobs pass successfully.

## Prerequisites

Before using this workflow, make sure to configure the following:

### GitHub Secrets:
1. **`MIRROR_URL`**: The URL of the mirror repository (e.g., `git@github.com:username/mirror-repo.git`).
2. **`EXECUTABLES`**: A comma-separated list of paths to the executables (e.g., `my_program,tests/my_test`).
3. **`GIT_SSH_PRIVATE_KEY`**: The SSH private key used to authenticate with the mirror repository.

### Allowed Function List (optional):
- If your project has an `allowed_function.note` file, it will be used to specify the list of allowed functions. If this file is missing, the workflow will display a warning.

## Workflow Configuration

### Trigger Events

This workflow is triggered by the following GitHub events:
- **Push**: Triggered when code is pushed to any branch except branches starting with `ga-ignore`.
- **Pull Request**: Triggered when a pull request is created, excluding branches starting with `ga-ignore`.

### Environment Variables

The workflow uses the following environment variables:
- `MIRROR_URL`: The URL of the mirror repository (set in GitHub Secrets).
- `EXECUTABLES`: A comma-separated list of paths to executables (set in GitHub Secrets).

### Jobs

#### 1. `check_coding_style`
- **Runs in**: `ghcr.io/epitech/coding-style-checker:latest` container.
- **Description**: Checks the code for adherence to coding style. It reads the `coding-style-reports.log` and flags any issues as errors or warnings.

#### 2. `check_program_compilation`
- **Runs in**: `epitechcontent/epitest-docker` container.
- **Description**: Runs `make` to verify that the code compiles and that the executables specified in the `EXECUTABLES` environment variable are present and executable.

#### 3. `run_tests`
- **Runs in**: `epitechcontent/epitest-docker` container.
- **Description**: Runs `make tests_run` to execute the project’s tests and ensures everything is working.

#### 4. `check_banned_function`
- **Runs in**: Ubuntu runner.
- **Description**: Checks for the presence of banned functions (such as GLIBC functions). It compares the functions used in your executable against a list of allowed functions.

#### 5. `push_to_mirror`
- **Runs only on `push` events**.
- **Description**: Pushes the code to a mirror repository using an SSH key. It removes `.github` from the mirror to avoid redundant workflows and repositories.

### Conditions for Skipping Jobs

The workflow will **skip execution** for branches that start with `ga-ignore-` (e.g., `ga-ignore-feature-x`). This is achieved through the `branches-ignore` filter for both `push` and `pull_request` events.

## Usage

### Setting Up Secrets

Make sure to configure the following secrets in your GitHub repository:

1. **`MIRROR_URL`**: The URL of the mirror repository. Example:
   ```
   git@github.com:username/mirror-repo.git
   ```
2. **`EXECUTABLES`**: A comma-separated list of paths to the executables. Example:
   ```
   my_program,tests/my_test
   ```
3. **`GIT_SSH_PRIVATE_KEY`**: Your private SSH key for pushing to the mirror repository. Ensure this key has the necessary permissions for the mirror repository.

### Adding the Workflow

1. Clone your repository locally.
2. Create a new file in `.github/workflows/chocolatine.yml`.
3. Copy the workflow YAML from this file into the `chocolatine.yml` file.
4. Commit and push the file to GitHub.

### Triggering the Workflow

The workflow will be triggered automatically when you push changes or create a pull request (except for branches starting with `ga-ignore-`).

### Monitoring the Workflow

- You can view the progress and results of the workflow in the **Actions** tab of your GitHub repository.
- The jobs will report success or failure, and if there are errors, they will be displayed in the **checks** section of the pull request or commit.

## Troubleshooting

1. **SSH Key Authentication Issues**:
   - Ensure the `GIT_SSH_PRIVATE_KEY` secret is correctly configured and has access to the mirror repository.
   - Double-check that the public key is added to the SSH configuration of your GitHub mirror repository.

2. **Coding Style Errors**:
   - Review the error annotations produced by the `check_coding_style` job to identify the file and line causing the issue.
   - Ensure that your code adheres to the Epitech coding style guidelines.

3. **Compilation Errors**:
   - Check the logs for any missing files or compilation errors in the `check_program_compilation` job.

4. **Banned Function Detection**:
   - If the `check_banned_function` job reports a banned function, ensure you’re not using any prohibited functions in your code. Modify the `allowed_function.note` if necessary.

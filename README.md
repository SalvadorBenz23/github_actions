# GitHub Actions: Memo on Workflow Automation

![image](https://github.com/user-attachments/assets/7c63cc36-69c9-4ba4-a108-0b6d85358970)


This document serves as a detailed reference for working with GitHub Actions. It includes an overview of key concepts, practical steps to create workflows, and explanations of essential commands and configurations. The examples and exercises provide guidance for setting up and managing workflows effectively.

## Table of Contents
1. [Introduction to GitHub Actions](#introduction-to-github-actions)
2. [Key Concepts](#key-concepts)
   - [Supported Shells](#supported-shells)
   - [Default Environment Variables](#default-environment-variables)
3. [Lab: Setting Up Your First Workflow](#lab-setting-up-your-first-workflow)
   - [Creating a Workflow](#creating-a-workflow)
   - [Testing and Debugging](#testing-and-debugging)
   - [Working with Multiple Shells](#working-with-multiple-shells)
4. [Visual Guide](#visual-guide)
5. [Conclusion](#conclusion)

---

## Introduction to GitHub Actions

GitHub Actions is a robust tool for automating software development workflows. It allows developers to define workflows triggered by specific events, such as pushing code, opening pull requests, or merging branches. GitHub Actions seamlessly integrates with CI/CD pipelines, simplifying the process of building, testing, and deploying applications.

---

## Key Concepts

### Supported Shells

GitHub Actions supports a variety of shells across platforms. Selecting the right shell depends on the operating system and the task requirements.

| **Supported Platform** | **Shell Parameter** | **Description**                                                                                      | **Command Run Internally**                                      |
|-------------------------|---------------------|------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| All                     | `bash`             | Default shell for non-Windows platforms. On Windows, it uses the bash shell included with Git for Windows. | `bash --noprofile --norc -eo pipefail {0}`                     |
| All                     | `pwsh`             | PowerShell Core. Appends `.ps1` to your script name.                                                | `pwsh -command ". '{0}'"`                                      |
| All                     | `python`           | Executes Python commands.                                                                            | `python {0}`                                                   |
| Linux / macOS           | `sh`               | Fallback shell for non-Windows platforms if `bash` is unavailable.                                  | `sh -e {0}`                                                    |
| Windows                 | `cmd`              | Command Prompt. Appends `.cmd` to script names.                                                     | `%ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}""`                 |
| Windows                 | `powershell`       | Windows PowerShell Desktop.                                                                         | `powershell -command ". '{0}'"`                                |

---

### Default Environment Variables

GitHub Actions provides built-in environment variables that enhance workflow context and debugging. Here are the most commonly used ones:

| **Variable**           | **Description**                                                                                  |
|-------------------------|--------------------------------------------------------------------------------------------------|
| `HOME`                 | Home directory in the virtual machine. Cannot be changed.                                       |
| `GITHUB_WORKFLOW`      | Name of the currently running workflow.                                                         |
| `GITHUB_ACTION`        | Unique identifier for the current step.                                                         |
| `GITHUB_ACTIONS`       | Always `true` when running on GitHub Actions.                                                   |
| `GITHUB_ACTOR`         | Username of the GitHub account that triggered the workflow.                                     |
| `GITHUB_REPOSITORY`    | Name of the repository (`username/repository`).                                                 |
| `GITHUB_EVENT_NAME`    | Event that triggered the workflow (e.g., `push`, `pull_request`).                               |
| `GITHUB_WORKSPACE`     | Directory path containing repository files in the virtual machine.                              |
| `GITHUB_SHA`           | Commit ID (SHA) that triggered the workflow.                                                    |

---

## Lab: Setting Up Your First Workflow

### Creating a Workflow
1. **Create a Repository**:
   - Name your repository `github_actions` on GitHub.
   - Clone it locally:
     ```bash
     git clone https://github.com/your-username/github_actions.git
     ```

2. **Set Up the Workflow Directory**:
   ```bash
   mkdir -p .github/workflows
   cd .github/workflows
   ```
   
3. **Write Your Workflow File**:  
   Create a file named `my_first_workflow.yml` and include the following content:

   ```yaml
   name: Shell commands
   on: [push]
   jobs:
     run-shell-command:
       runs-on: ${{ matrix.os }}
       strategy:
         matrix:
           os: [ubuntu-latest, ubuntu-20.04]
       steps:
         - name: echo a string
           run: echo "Hello world"
         - name: multiline script
           run: |
             python3 -V
             touch my_file_with_workflow.py
             ls -l
   ```
4. **Commit and Push Your Workflow**:
   Add and push the workflow to GitHub:
   ```bash
   git add .
   git commit -m "my first action"
   git push
   ```
---

## Testing and Debugging

### Enable Debugging:
1. Go to **Settings > Secrets** in your GitHub repository.
2. Add two secrets:
   - `ACTIONS_RUNNER_DEBUG` with a value of `true`
   - `ACTIONS_STEP_DEBUG` with a value of `true`

### Induce an Error:
Modify the workflow file to introduce an error. Replace:
```yaml
run: echo "Hello world"
```
with:
```yaml
run: echoo "Hello world"
```

Push the changes and observe the error in the Actions tab on GitHub. Analyze the logs to debug the issue.

---
## Working with Multiple Shells

### 1. Create a New Workflow:
Write a file named `multiple_shells.yml` with the following content:

```yaml
name: Multiple shells commands
on:
  push:
    branches:
      - main
  pull_request:
    types: [closed, assigned, opened, reopened]
jobs:
  run-shell-command:
    runs-on: ubuntu-latest
    steps:
      - name: python commands
        run: |
          import platform
          print(platform.processor())
        shell: python
  run-windows-command:
    runs-on: windows-latest
    needs: ["run-shell-command"]
    steps:
      - name: current directory with powershell
        run: Get-Location
      - name: print environment variables
        run: |
          echo "HOME: ${HOME}"
          echo "GITHUB_WORKFLOW: ${GITHUB_WORKFLOW}"
          echo "GITHUB_ACTIONS: ${GITHUB_ACTIONS}"
        shell: bash
```

#### This workflow demonstrates:

- Running Python commands on Ubuntu.
- Running PowerShell and Bash commands on Windows.
- Using the needs key to ensure sequential job execution.

### 2. Push the Workflow:

```bash
git add .
git commit -m "multiple shells workflow"
git push
```

### 3. Observe Dependencies:

- View the workflow in the Actions tab on GitHub. The second job will execute only after the first job completes successfully.

---
## Visual Guide

### Navigating the GitHub Console
- **Actions Tab**:
  ![image](https://github.com/user-attachments/assets/19ded7c8-64c1-4f61-8bd5-9dbf700c1095)
  
- **Workflow List**:
  ![image](https://github.com/user-attachments/assets/0f9d4e47-2e41-4469-a9b4-c04ca1f42342)

- **Workflow Details**:
  ![image](https://github.com/user-attachments/assets/106571f8-545b-44e4-a4f8-427d1c6bebed)


### Debugging
- **Secrets Configuration**:
  ![image](https://github.com/user-attachments/assets/ed3f2503-3bde-4233-aa7e-1562a9048c87)
 
- **Debug Logs**:
  ![image](https://github.com/user-attachments/assets/21e6ff0d-a4f1-4f11-8ce8-34be3e6cb5fa)


---

### Conclusion

This document highlights:
1. Setting up GitHub Actions workflows to automate CI/CD processes.
2. Understanding shell compatibility, job sequencing, and environment variables.
3. Debugging workflows using built-in GitHub tools.

Mastering GitHub Actions enhances development workflows, ensures consistency, and simplifies the deployment process for scalable applications.

---

# GitHub Events & Actions Step by Step

![Events & Actions](https://github.com/user-attachments/assets/d21fae99-bea5-4b4f-b9ad-8394d2015842)


In this section, we expand upon previous workflows by building an action that:

- Runs unit tests with Pytest.
- Checks code quality using Flake8.
- Explains the use of environment variables for better flexibility.

![issues](https://github.com/user-attachments/assets/cedea55a-462b-4b50-a1c2-9f1d73687fce)
(we could imagine that some issues will trigger specific Events/Action according to labels)


---

### 1. Pytest and Flake8 Setup

**Pytest** helps test Python scripts effectively.  
**Flake8** ensures your code complies with PEP8, identifies unused imports, and tracks cyclomatic complexity.

Here’s how to set up a test script for Pytest in your repository:

Create `tests/file_test.py`:

```python
def test_calc_addition():
    output = 2 + 4
    assert output == 6

def test_calc_subtraction():
    output = 2 - 4
    assert output == -2

def test_calc_multiply():
    output = 2 * 4
    assert output == 8
```


### 2. Create a New Workflow

Add a workflow file named `control_tests.yaml` to the `.github/workflows/` directory:

```yaml
name: Pytest & Flake8 Control
on: push
jobs:
  qa:
    name: Run Tests and Code Quality Checks
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v3
        with:
          python-version: "3.9"

      - name: Install Dependencies
        run: |
          pip install pytest flake8

      - name: Run Tests
        run: pytest tests/

      - name: Run Flake8
        uses: py-actions/flake8@v2
```


### 3. Commit and Push the Workflow

Save your changes, commit, and push the workflow to your repository:

```bash
git add .
git commit -m "Added Pytest & Flake8 workflow"
git push
```

### 4. Debugging the Workflow

Go to the **Actions** tab in your repository to monitor the workflow execution. If any test fails or the code does not adhere to PEP8, debug the errors based on the log outputs.

Example of a successful test run:
![Successful Workflow Run](https://github.com/user-attachments/assets/a688643a-3a3d-46a0-9f91-94ad3244abb5)

### 5. Environment Variables

#### a. Defining Environment Variables

Environment variables in GitHub Actions can be set at different levels: workflow-wide, job-specific, or step-specific.

Example:

```yaml
name: Greeting on a Variable Day

on: workflow_dispatch

env:
  DAY_OF_WEEK: Monday ## Workflow-wide variable

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    env:
      GREETING: Hello ## Job-specific variable
    steps:
      - name: Print Greeting
        run: echo "$GREETING, today is $DAY_OF_WEEK!"
        env:
          NAME: GitHub Actions ## Step-specific variable
```

#### b. Using Secrets for Sensitive Information

Secrets allow storing sensitive data securely (e.g., API keys, tokens). To create a secret:
1. Navigate to **Settings > Secrets** in your repository.
2. Add a new secret (e.g., `API_KEY`).
3. Reference it in your workflow:
   ```yaml
   env:
     API_KEY: ${{ secrets.API_KEY }}
```

### 6. Pushing Files to the Repository

Automate file commits and pushes within a workflow:

```yaml
jobs:
  commit_changes:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Git
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"

      - name: Commit Changes
        run: |
          echo "Generated by GitHub Actions" >> generated_file.txt
          git add .
          git commit -m "Generated new file"
          git push
```

### Conclusion

In this section, you learned to:
1. Integrate Pytest and Flake8 into GitHub Actions workflows.
2. Use environment variables and GitHub secrets for flexibility and security.
3. Automate file commits and pushes directly from workflows.

This knowledge provides a foundation for automating CI/CD pipelines and improving code quality and testing standards.




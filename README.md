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

## Conclusion

This document highlights:
1. Setting up GitHub Actions workflows to automate CI/CD processes.
2. Understanding shell compatibility, job sequencing, and environment variables.
3. Debugging workflows using built-in GitHub tools.

Mastering GitHub Actions enhances development workflows, ensures consistency, and simplifies the deployment process for scalable applications.



# Remote Gradle Project Builder

English | [简体中文](README.zh-CN.md)

Fetch a Git repository, run a Gradle build, and upload the resulting artifacts with GitHub Actions. Supports Linux, Windows, macOS, multi-module projects, projects in subdirectories, Git submodules, and Git LFS.

## Usage

1. Commit `.github/workflows/build-gradle.yml` to your GitHub repository's default branch.
2. Open **Actions → Build external Gradle project → Run workflow**.
3. Enter the target repository's HTTPS Git URL, Git ref, and build arguments, then start the workflow.
4. Download the build outputs from **Artifacts** on the workflow run page. The logs and run summary record the commit used for the build.

| Input | Description | Default |
| --- | --- | --- |
| `repository_url` | Target repository's HTTPS Git URL, e.g. `https://github.com/owner/project.git` | Required |
| `git_ref` | Branch, tag, or full commit SHA. Use `refs/heads/name` or `refs/tags/name` if a branch and tag share the same name. | `HEAD` (default branch) |
| `project_dir` | Project directory containing the Wrapper, relative to the target repository root | `.` |
| `java_version` | Eclipse Temurin JDK version; choose a version compatible with the project | `21` |
| `gradle_version` | Leave blank to use the project's Gradle Wrapper, or specify a version for projects without a Wrapper | Empty |
| `gradle_arguments` | Tasks and arguments, e.g. `clean build` or `:server:build -x test` | `clean build` |
| `runner` | Operating system for the build | `ubuntu-latest` |
| `artifact_paths` | Artifact globs relative to the **target repository root**, separated by semicolons or newlines; supports `!` exclusion patterns | JARs, distributions, APKs, AABs |

Arguments use shell-style quoting, e.g. `build -Pmessage="hello world"`. The workflow runs Gradle only; shell commands cannot be appended to the arguments. Use `/` for Windows paths.

For a project in `backend/`, set `project_dir=backend` and use an artifact pattern such as `backend/build/libs/*.jar`. The default `**/build/libs/*.jar` also matches subdirectories and multi-module outputs. Leave `artifact_paths` blank to disable uploads. Missing artifacts produce a warning. If the build fails, the workflow still attempts to upload artifacts already generated.

## Private GitHub Repositories

In the repository containing this workflow, open **Settings → Secrets and variables → Actions** and add a repository secret:

- Name: `TARGET_GIT_TOKEN`
- Value: A GitHub personal access token with read access to the target repository. For a fine-grained PAT, grant **Contents: Read** for the target repository. Private submodules also require read access to their repositories.

Public repositories do not require this secret. Credentials apply only to `https://github.com/` and are passed to Git through the fetch step's environment, without being stored in the target repository's Git configuration. Public HTTPS repositories on other Git services are also supported. For private repositories on other services, extend the fetch step with the authentication required by that service.

## Build Environment

By default, the workflow uses the project's committed Gradle Wrapper and runs `clean build`. The target project's source code does not need to be stored in this repository. The workflow installs the JDK and uses `setup-gradle` to manage the Gradle cache.

You can change the Git repository, ref, project directory, and Gradle tasks. Projects that require an Android SDK, NDK, database, signing keys, private Maven repository credentials, or other tools need additional setup steps or secrets before building. Choose compatible JDK and Gradle versions for the target project.

Each build has a 60-minute timeout, and artifacts are retained for 7 days. Both settings can be changed in the workflow.

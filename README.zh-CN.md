# 远程 Gradle 项目构建

[English](README.md) | 简体中文

通过 GitHub Actions 拉取指定 Git 仓库，执行 Gradle 构建并上传产物。支持 Linux、Windows、macOS，以及多模块项目、子目录项目、Git 子模块和 Git LFS。

## 使用

1. 将 `.github/workflows/build-gradle.yml` 提交到 GitHub 仓库的默认分支。
2. 打开仓库 **Actions → Build external Gradle project → Run workflow**。
3. 填写目标项目的 HTTPS Git 地址、Git ref 和构建参数后运行。
4. 在本次运行的 **Artifacts** 中下载构建产物；日志与运行摘要会记录实际构建的提交。

| 参数 | 说明 | 默认值 |
| --- | --- | --- |
| `repository_url` | 目标 Git 仓库的 HTTPS 地址，如 `https://github.com/owner/project.git` | 必填 |
| `git_ref` | 分支、标签或完整提交 SHA；有同名分支和标签时使用 `refs/heads/name` 或 `refs/tags/name` | `HEAD`（默认分支） |
| `project_dir` | 包含 Wrapper 的项目目录，相对于目标仓库根目录 | `.` |
| `java_version` | Eclipse Temurin JDK 版本，按项目要求选择 | `21` |
| `gradle_version` | 留空使用项目自带 Wrapper；填写后使用指定版本，适用于没有 Wrapper 的项目 | 空 |
| `gradle_arguments` | 任务及参数，例如 `clean build`、`:server:build -x test` | `clean build` |
| `runner` | 构建运行环境 | `ubuntu-latest` |
| `artifact_paths` | 相对于**目标仓库根目录**的产物 glob，以分号或换行分隔；支持 `!` 排除模式 | JAR、发行包、APK、AAB |

参数中的引号按 shell 风格解析，例如 `build -Pmessage="hello world"`。这里只执行 Gradle，不支持在参数中拼接 shell 命令；Windows 路径建议使用 `/`。

子目录示例：项目位于 `backend/`，则设置 `project_dir=backend`，产物可以填写 `backend/build/libs/*.jar`。默认的 `**/build/libs/*.jar` 也能匹配子目录和多模块产物。留空 `artifact_paths` 可关闭上传；没有匹配文件时只提示警告。构建失败时仍尝试上传已生成的产物。

## 私有 GitHub 仓库

在存放 workflow 的仓库中打开 **Settings → Secrets and variables → Actions**，新增 repository secret：

- 名称：`TARGET_GIT_TOKEN`
- 值：有目标仓库读取权限的 GitHub PAT。Fine-grained PAT 应授权目标仓库的 **Contents: Read**；私有子模块也需要对应仓库的读取权限。

公开仓库无需配置该 secret。凭据仅对 `https://github.com/` 生效，通过步骤环境传给 Git，不写入目标仓库的 Git 配置。其他 Git 服务的公开 HTTPS 仓库也可构建；其他服务的私有认证需要按该服务扩展拉取步骤。

## 构建环境

默认使用项目提交的 Gradle Wrapper 和 `clean build`，不需要把目标项目代码放入此仓库。workflow 自动安装 JDK，并使用 `setup-gradle` 管理 Gradle 缓存。

“任意项目”指可以更换 Git 仓库、版本、目录和 Gradle 任务；项目额外需要的 Android SDK、NDK、数据库、签名密钥、私有 Maven 仓库凭据或其他工具，仍需在构建前添加相应步骤或 secrets。JDK 与 Gradle 的兼容版本也应根据目标项目选择。

单次构建超时为 60 分钟，产物保留 7 天，可以在 workflow 中调整。

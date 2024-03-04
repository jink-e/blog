---
date: 2020-10-20 10:25:38
category:
  - dev
tag:
  - vscode
  - docker
---

# vscode docker 开发环境搭建

最近的一个 Release， `Visual Studio Code Remote - Containers` 插件，可以方便用户使用 docker 容器作为开发环境，在项目目录下创建一个`devcontainer.json`文件，告诉 vscode 如何进入/创建`开发容器`，并自动设置 vscode 工具，本地`workspace`的文件可以 mount/copy/clone 到容器里

能想到的好处：

- 团队统一开发环境，可以减少新入职员工在开发配置的时间开销

- 可以将 docker 环境配置到云端，远程开发也挺香？

- 切换开发环境只需要切换 container 即可

<!--more-->

## 安装依赖

### Local

- Windows: Docker Desktop 2.0+ on Windows 10 Pro/Enterprise. Windows 10 Home (2004+) requires Docker Desktop 2.3+ and the WSL 2 back-end. (Docker Toolbox is not supported. Windows container images are not supported.)
- macOS: Docker Desktop 2.0+.
- Linux: Docker CE/EE 18.06+ and Docker Compose 1.21+. (The Ubuntu snap package is not supported.)

### Containers

- x86_64 / ARMv7l (AArch32) / ARMv8l (AArch64) Debian 9+, Ubuntu 16.04+, CentOS / RHEL 7+
- x86_64 Alpine Linux 3.9+

## 安装

1. 安装 [Docker](https://www.docker.com/get-started)
   1. Windows/macOS
      1. 安装 [Docker Desktop for Windows/Mac](https://www.docker.com/products/docker-desktop)
      2. 右键点击任务菜单里的 docker 图标，选择 `Settings/Preferences` 并设置 `Resources > File Sharing` 允许访问的目录
      3. 如果使用的是 windows 的 WSL2, 打开 `Windows WSL 2 back-end`: 右键任务菜单 docker 图标, 选择`Settings`, 勾选`Use the WSL 2 based engine` 并且验证 'Resources > WSL Integration'
2. 安装 [VScode](https://code.visualstudio.com/)
3. 安装 [Remote Development extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)

## 设置开发环境

1. vscode 打开本地的一个项目
2. `F1` 打开 vscode 命令行, `Remote-Containers: Open Folder in Container`
3. 选择一个基础镜像，选择完后 vscode 会自动在项目根目录生成 `.devcontainer/devcontainer.json` 配置文件
4. 等待 vscode 安装相关依赖

## 官方参考

https://code.visualstudio.com/docs/remote/containers

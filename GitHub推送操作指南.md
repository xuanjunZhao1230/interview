# GitHub 推送操作指南

本项目对应的远程仓库：

```text
https://github.com/xuanjunZhao1230/interview.git
```

## 首次上传时执行过的命令

在 Git Bash 中先进入项目目录：

```bash
cd "/c/Users/zhaoxuanjun/Desktop/测开面试模拟"
```

### 1. 初始化本地仓库

```bash
git init
```

作用：在当前文件夹创建本地 Git 仓库。一个项目通常只需执行一次。

### 2. 添加要上传的文件

```bash
git add -- "Day 1.md" "Day 2.md"
```

作用：将指定文件加入待提交列表。首次只选择了这两个 Markdown 文件。

### 3. 创建本地提交

```bash
git commit -m "Upload Markdown interview notes"
```

作用：把待提交列表中的文件创建为一个本地版本记录。`-m` 后的文字是本次改动说明。

### 4. 设置默认分支

```bash
git branch -M main
```

作用：将当前分支命名为 `main`。通常只需执行一次。

### 5. 关联 GitHub 仓库

```bash
git remote add origin https://github.com/xuanjunZhao1230/interview.git
```

作用：将本地仓库关联到 GitHub 远程仓库。`origin` 是这个远程地址的简称。通常只需执行一次。

### 6. 配置本项目的网络代理

```bash
git config http.proxy http://127.0.0.1:7897
git config https.proxy http://127.0.0.1:7897
```

作用：让此项目的 Git 连接通过本机代理访问 GitHub。该设置只影响本项目，不影响其他 Git 仓库。

> 如果代理程序未运行、端口变更，或不再需要代理，推送可能会失败。

### 7. 首次推送到 GitHub

```bash
git push -u origin main
```

作用：把本地 `main` 分支上传至 GitHub，并建立本地与远程分支的关联。首次需要 `-u`，以后可直接使用 `git push`。

---

## 以后更新并推送

每次开始前，先进入项目目录：

```bash
cd "/c/Users/zhaoxuanjun/Desktop/测开面试模拟"
```

### 查看当前有哪些改动

```bash
git status
```

作用：查看哪些文件被修改、新增或删除，以及它们是否已加入待提交列表。

### 推送某一个修改或新建的文件

```bash
git add "文件名.md"
git commit -m "说明这次改了什么"
git push
```

例如，修改 `Day 1.md` 后：

```bash
git add "Day 1.md"
git commit -m "完善 Day 1 面试笔记"
git push
```

### 推送所有新增、修改和删除的文件

```bash
git add -A
git commit -m "更新面试资料"
git push
```

注意：`git add -A` 会将当前项目内的所有改动加入提交，包含非 Markdown 文件。执行前建议先运行 `git status` 确认清单。

---

## 常见问题

### 推送时提示网络连接被重置

确认代理软件正在运行，并且代理端口仍是 `7897`。可检查当前项目的代理设置：

```bash
git config --get http.proxy
git config --get https.proxy
```

若端口已变化，以新端口重新设置：

```bash
git config http.proxy http://127.0.0.1:新端口
git config https.proxy http://127.0.0.1:新端口
```

### 不再使用代理

```bash
git config --unset http.proxy
git config --unset https.proxy
```

### 提交前的推荐检查顺序

```bash
git status
git add "要上传的文件名"
git status
git commit -m "本次改动说明"
git push
```


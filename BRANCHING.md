# 分支与仓库管理

gfast 是我们自己的仓库。WeKnora 是腾讯官方仓库上的魔改，推送必须进我们自己的 Gitee，官方更新从 GitHub 拉。两边都用「个人长期分支 + 集成分支」，用 `git merge` 合并。同一行两边都改过时，Git 会停下来等人处理，没有一种分支节奏能跳过这一步。

不要每次提交都做「master 合进个人分支，再立刻合回 master」。那样只会多出一串空合并。平时提交只发生在自己的 `feature/<名字>` 上。

| 时机 | 方向 |
| --- | --- |
| 开工，把别人已经进集成分支的提交拿过来 | 集成分支 → `feature/<名字>` |
| 这段工作要给别人用 | `feature/<名字>` → 集成分支 |

个人分支一旦推送过，就继续用 merge。不要 rebase，也不要 force push。rebase 会改写别人可能已经基于它拉过的历史。

## 两个仓库

### gfast

- 目录：`jzin-bk/`
- 远程 `origin`：`https://gitee.com/jzins/v2all-jzin-bk.git`
- `master`：团队集成结果。仓库允许直接推，日常仍只通过下面的发布步骤更新它
- `feature/<名字>`：每人一条长期分支，例如 `feature/jzin`。所有提交都在这里

`feature/<名字>` 是个人工作区，不是每人一份独立产品。大家共用的是 `master`。

### WeKnora

- 目录：与 gfast 平级的 `WeKnora/`
- `origin`：`https://gitee.com/jzins/WeKnora.git`，推送只去这里
- `upstream`：`https://github.com/Tencent/WeKnora.git`，只 fetch。本机 push 地址设为 `DISABLED`，避免误推到腾讯仓库
- `main`：官方镜像。禁止在这上面提交魔改。它跟踪 `upstream/main`
- `master`：我们的魔改集成分支，对应 gfast 的 `master`
- `feature/<名字>`：从 `master` 拉出，日常规则与 gfast 相同

Gitee 上这个仓库的默认分支是 `main`。克隆下来先看到的是官方镜像，没有 gfast 嵌入。要魔改代码请切到 `master` 或自己的 `feature/<名字>`。

魔改尽量加新文件，在官方文件里只留几行挂载。改官方原文件的 diff 越小，下次合入官方更新越容易。

## 当前落地状态

2026-11-02：

| 仓库 | 分支 | 提交 | 跟踪 |
| --- | --- | --- | --- |
| gfast | `master`、`feature/jzin` | `a302e94` | 各自的 `origin` |
| WeKnora | `main` | `80e834590` | `upstream/main` |
| WeKnora | `master`、`feature/jzin` | `9b670237a` | 各自的 `origin` |

`9b670237a` 是 gfast 门户嵌入：乾坤入口、会话换票路由和相关前端挂载。

WeKnora 工作区里这些改动故意没有入库，各自机器自己留着：

- `config/config.yaml` 里的本机端口（当前是 `8202`）
- `go.mod`、`go.sum`：本机用 Go 1.27.2 整理依赖的结果。官方 `go.mod` 仍是 Go 1.26.0。这两份文件最容易和官方更新撞车
- `run_bk.bat`、`run_web.bat`、`scripts/mingw_snprintf_shim.c`：本机 Windows / MinGW 启动脚本

## 访问 GitHub

Gitee 一般直连即可。`git fetch upstream` 超时或连接被中断时，给这一次命令带上本机代理，不要写入 `git config`：

```powershell
$env:HTTP_PROXY = "http://127.0.0.1:7897"
$env:HTTPS_PROXY = "http://127.0.0.1:7897"
git fetch upstream
```

端口按自己的代理软件实际端口改。下面的 WeKnora 官方更新步骤里，如果 `git fetch upstream` 失败，先执行这两行再重试。

## gfast 日常

以下命令在 gfast 仓库根目录执行。把 `feature/jzin` 换成自己的分支名。

第一次从 `master` 拉出个人分支：

```powershell
git checkout master
git pull origin master
git checkout -b feature/你的名字
git push -u origin feature/你的名字
```

开工：

```powershell
git checkout feature/你的名字
git pull origin feature/你的名字
git fetch origin
git merge origin/master
```

开发过程中只在 `feature/你的名字` 上提交并推送：

```powershell
git push origin feature/你的名字
```

这段要给别人用：

```powershell
git push origin feature/你的名字
git checkout master
git pull origin master
git merge feature/你的名字
git push origin master
git checkout feature/你的名字
git merge origin/master
```

最后一步让个人分支和刚更新的 `master` 对齐。`git push origin master` 被拒绝，说明别人刚推过：停在 `master` 上再执行 `git pull origin master`，处理完冲突后再推。

## WeKnora 日常

个人分支和 `master` 的开工、发布命令与上一节相同，远程都是我们的 Gitee `origin`。区别只在官方更新，以及不要把魔改提交到 `main`。

已经有一份指向腾讯仓库的本地克隆时，先改远程，再拉取我们的分支：

```powershell
git remote rename origin upstream
git remote add origin https://gitee.com/jzins/WeKnora.git
git remote set-url --push upstream DISABLED
git fetch origin
git checkout -B master origin/master
git checkout -b feature/你的名字
git push -u origin feature/你的名字
```

`git remote rename` 只有在当前 `origin` 仍是 `https://github.com/Tencent/WeKnora.git` 时执行一次。已经指向 Gitee 的克隆不要再改名。

新克隆直接从我们的仓库开始：

```powershell
git clone https://gitee.com/jzins/WeKnora.git
cd WeKnora
git remote add upstream https://github.com/Tencent/WeKnora.git
git remote set-url --push upstream DISABLED
git fetch upstream
git branch --set-upstream-to=upstream/main main
git checkout master
git checkout -b feature/你的名字
git push -u origin feature/你的名字
```

确认远程：

```powershell
git remote -v
```

应看到：

```text
origin    https://gitee.com/jzins/WeKnora.git (fetch)
origin    https://gitee.com/jzins/WeKnora.git (push)
upstream  https://github.com/Tencent/WeKnora.git (fetch)
upstream  DISABLED (push)
```

`git branch -vv` 里，`main` 应跟踪 `upstream/main`，`master` 和 `feature/<名字>` 应跟踪 `origin`。

## 把 WeKnora 官方更新合进来

按版本或每周做一次，不要每次提交都做。冲突集中在「`main` 合入 `master`」这一次。

```powershell
git fetch upstream
git checkout main
git merge --ff-only upstream/main
git push origin main
git checkout master
git merge main
git push origin master
git checkout feature/你的名字
git merge master
git push origin feature/你的名字
```

`git merge --ff-only` 失败，说明本地 `main` 上有魔改提交。把那些提交改放到 `master`，让 `main` 回到和 `upstream/main` 相同的提交，再继续。不要在 `main` 上解决功能冲突。

`main` 的 push 不要加 `-u`。加了会把 `main` 的跟踪从 `upstream/main` 改成 `origin/main`，以后 `git pull` 就不再默认拿官方更新。

官方更新先合进 `master`。其他人本机再按开工步骤把 `master` 合进自己的 `feature/<名字>`。

## 冲突

出现冲突时 Git 会列出文件。打开文件，保留应该留下的两边内容，删掉冲突标记，然后：

```powershell
git add 冲突文件
git merge --continue
```

想放弃这一次合并：

```powershell
git merge --abort
```

同一处冲突下次还会遇到时，可以在本机打开记忆（只影响自己的 Git，不进仓库）：

```powershell
git config --global rerere.enabled true
```

两人同时改同一文件时，后合并的人在自己的 `git merge` 处处理一次。勤合 `master`，冲突会小一些。

---
title: Git Hooks 阻止合并特定分支
date: 2021-08-01 16:06:14
tags:
  - git
  - git hooks
categories:
  - tools
  - git
---

## 引言

项目开发时，有开发分支，测试分支，主干分支等。一般不能把**测试分支**合并到其他分支里，然而可能一不小心（手抖）合并了，甚至在不知情的情况下还加了新的东西，后面上线时才发现（或者没发现，直接把测试分支的代码带到了线上），后果可大可小，回滚时也麻烦。

那能不能在合并阶段直接禁止合并非法分支呢？答案是可以的。只要解决了下面问题即可。

<!-- more -->

- 是否在合并中？
- 当前分支名叫啥？
- 要合并进来的分支名又叫啥？
- 当前分支 和 要合并进来的分支 2 者是否满足条件【这里是 要合并进来的分支 不能是 测试分支】

## 前置知识

### git hooks

[git hooks](https://git-scm.com/docs/githooks)，简单来说就是在执行 git 命令的过程中会触发的钩子函数（脚本程序）。只要知道特定 git 命令会触发什么 hooks，就可以做对应处理，比如可以用来检查提交信息是否符合规范（如 [commitlint](https://github.com/conventional-changelog/commitlint)，以及本文即将要了解的阻止合并特定分支。

### git 合并

命令: `git merge <branch>`

合并可能有 3 种情况

- **fast-forward merge**: 合并时，当前分支和要合并进来的分支，分支历史没有分叉【简单理解就是 要合并的分支是基于当前分支前进的，且当前分支从要合并的分支新建后 再也没发生过变更】。可以通过 `git merge --no-ff <branch>`变为第 2 种合并情况

![fast-forward merge | atlassian](https://wac-cdn.atlassian.com/dam/jcr:d90f2536-7951-4e5e-ab79-f45a502fb4c8/03-04%20Fast%20forward%20merge.svg?cdnVersion=1735)

- **no fast-forward merge**: 新增 1 个历史节点，其直接父节点指向为要合并的 2 个分支

![no fast-forward merge | atlassian](https://wac-cdn.atlassian.com/dam/jcr:91aece4a-8fa0-4fc3-bae9-69d51932f104/05-06%20Fast%20forward%20merge.svg?cdnVersion=1735)

- **merge conflict**: 合并冲突了，此时需要解决冲突，然后重新 add & commit

> 图片来自：https://www.atlassian.com/git/tutorials/using-branches/git-merge

> **前置说明**
> 这里的项目是前端项目，使用 [husky](https://github.com/typicode/husky) 管理 git hooks

## 问题解答

### 是否在合并中

通过查阅 [git hooks](https://git-scm.com/docs/githooks) 可知，merge 阶段**可能**会触发以下钩子【之所以说可能是因为 merge 有多种情况，每种情况触发的钩子不太一致】：

- `pre-merge-commit`
- `prepare-commit-msg`
- `commit-msg`
- `post-merge`

| 合并情况\触发钩子                          | `pre-merge-commit` | `prepare-commit-msg` | `commit-msg` | `post-merge` |
| ------------------------------------------ | ------------------ | -------------------- | ------------ | ------------ |
| `fast-forward merge`                       | ❌                 | ❌                   | ❌           | ✅           |
| `no fast-forward merge`                    | ✅                 | ✅                   | ✅           | ✅           |
| `merge conflict` 解决完冲突后 add & commit | ❌                 | ✅                   | ✅           | ❌           |

> `merge conflict` 会有中间态 `(当前分支 | MERGING)`，从初始态到中间态，不会触发 merge 相关的钩子。当解决完冲突后，开始 add&commit 时，会触发对应钩子

根据合并情况，使用到的钩子如下：

- `fast-forward merge` 和 `no fast-forward merge`: 使用 `post-merge` 钩子
- `merge conflict`: 使用 `prepare-commit-msg` 或 `commit-msg`【因 commit-msg 钩子无法获取到合并进来的分支名，故只能使用 `prepare-commit-msg`】

### 获取当前分支名

```sh
git rev-parse --abbrev-ref HEAD
```

> REF: https://stackoverflow.com/questions/6245570/how-to-get-the-current-branch-name-in-git

### 获取合并进来的分支名

#### post-merge

在此钩子处理 `no fast-forward merge` 和 `fast-forward merge` 情况。

`post-merge` 钩子触发时，分支已经合并了，并且 reflog 也更新了，所以可以通过 `git reflog` 获取到合并进来的分支信息

前 2 种合并情况，`git reflog -1` 返回的日志格式如下

- `no fast-forward merge`: `e7cb874 HEAD@{0}: merge feat/no-fast-forward: Merge made by the 'recursive' strategy.`
- `fast-forward merge`: `724446f HEAD@{0}: merge feat/fast-forward: Fast-forward`

可以通过正则匹配提取对应的分支名，代码如下

```js
const { execSync } = require('child_process');

function getMergeBranch() {
  // 从 reflog 提取合并进来的分支名
  function getBranchNameFromReflog(reflogMessage) {
    const reg = /@\{\d+\}: merge (.*):/;
    return reg.exec(reflogMessage)[1];
  }

  const reflogMessage = execSync('git reflog -1', { encoding: 'utf8' });
  const mergedBranchName = getBranchNameFromReflog(reflogMessage);
  return mergedBranchName;
}
```

#### prepare-commit-msg

在此钩子处理合并冲突的情况。

因冲突未解决，reflog 也不会更新，因此无法通过 reflog 获取到合并进来的分支。

不过在合并冲突阶段，`.git/MERGE_HEAD` 中会保留合并进来分支的 hash。
在 `prepare-commit-msg` 触发时，可以通过读取该文件获取对应的内容，再通过 `git name-rev [hash]` 命令获取对应的分支名

```js
const { execSync } = require('child_process');
const path = require('path');
const fs = require('fs');

// 从 .git/MERGE_HEAD (sha) 提取合并进来的分支名
function getMergeBranch() {
  try {
    const mergeHeadPath = path.resolve(process.cwd(), '.git/MERGE_HEAD');
    const mergeHeadSha = fs.readFileSync(mergeHeadPath, { encoding: 'utf8' });
    const mergeBranchInfo = execSync(`git name-rev ${mergeHeadSha}`);
    return / (.*?)\n/.exec(mergeBranchInfo)[1];
  } catch (err) {
    return '';
  }
}
```

### 合并分支是否符合要求

这个根据各自场景处理就行了。比如在合并错误分支后，进行提示，让操作者自行决定是否回滚等。

```js
function showConfirm(currentBranch, mergeBranch) {
  console.log(`检测到非法合并: ${mergeBranch} ==into==> ${currentBranch}`);
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });

  rl.question(`确定要合并 ${mergeBranch} 分支吗？(y/n) `, (answer) => {
    if (answer === 'y') {
      rl.close();
      process.exit(0);
    } else {
      console.log('撤销合并中...');
      console.log(`exec: git reset --merge HEAD@{1}`);
      execSync('git reset --merge HEAD@{1}');
      console.log('已撤销合并 done');
      rl.close();
      process.exit(0);
    }
  });
}
```

### 其他问题

细心的同学可能已经发现了，在第一个问题【是否在合并中】，最终采用了 `post-merge` 和 `prepare-commit-msg` 2 个钩子，但这 2 个钩子在`no fast-forward merge`的情况下都会触发到。此时需要进行识别，当且仅当在 `merge conflict` 才去执行 `prepare-commit-msg` 钩子中的逻辑。

思路是检测 `.git/MERGE_MSG` 文件是否存在，以及其中的内容。

```js
const { execSync } = require('child_process');
const path = require('path');
const fs = require('fs');

function isMergingConflict() {
  // 是否合并中
  const mergeMsgPath = path.resolve(process.cwd(), '.git/MERGE_MSG');
  const isMerging = fs.existsSync(mergeMsgPath);
  if (!isMerging) {
    return false;
  }

  try {
    const mergeMsg = fs.readFileSync(mergeMsgPath, { encoding: 'utf8' });
    return /\n# Conflicts:\n/.test(mergeMsg);
  } catch (err) {}
  return false;
}
```

## 总结

![git-hooks-prevent-merge-summary](https://raw.githubusercontent.com/sluggishpj/assets/main/images/git-hooks-prevent-merge-summary.svg)

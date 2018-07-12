---
title: 使用 commitizen 来格式化 Git commit message
date: 2018-07-12 10:33:42
categories:
  - 编程规范
tags:
  - git
  - commitizen
---

Git 每次提交代码，都要写 Commit message，这些说明有助于你的同事或社区的其它开发者了解你提交的代码的用途。目前，社区有多种 Commit message 的写法规范，我们介绍的工具是 [commitizen](https://github.com/commitizen/cz-cli)，它使用的是 [Angular规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)，这是目前使用最广的写法，并且有对应的工具去生成 change log。

<!--more-->

## 标准说明

每次提交， Commit message 都包括 **Header**, **Body** 和 **Footer** 三个部分。

```
<type>(<scope>): <subject>
// 空行
<body>
// 空行
<footer>
```

其中 Header 是必须的，Body 和 Footer 是可选的。

### Header

Header 部分只有一行，包括三个字段：**type**, **scope** 和 **subject** 。

**type** 用于说明提交的类别，只运行使用下面几种，

> - feat: 新功能
> - fix: 修复bug
> - docs: 文档更新
> - style: 格式更新（不影响代码运行的变动）
> - refactor: 重构（既不是新增功能，又不是bug修复）
> - test: 添加测试
> - chore: 构建过程或辅助工具的变动

如果是 **feat** 和 **fix** ，则这个 commit 将肯定出现在 change log 中，其它情况可自行决定是否放入。

**scope** 用于说明 commit 影响的范围。

**subject** 是 commit 目的的简短描述，不超过50个字符。

> - 以动词开头，使用第一人称现在时，比如 change，而不是 changed 或 changes
> - 第一个字母小写
> - 结尾不要加句号

### Body

Body 部分是对本次 commit 的详细描述，可分成多行。但是一般**我都不写**。

### Footer

Footer 部分只用于两种情况。

1. 不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以 **BREAKING CHANGE** 开头，后面是对变动的描述、以及变动理由和迁移方法。

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

2. 关闭 Issue

如果当前 commit 针对某个 issue ，那么可以在 Footer 部分关闭这个 issue 。

```
Closes #1234
```

关闭多个 issue。

```
Closes #1234, $1235, #1236
```

## 安装

使用 npm 工具进行全局安装，

```bash
npm install commitizen -g
```

然后在项目目录里，运行下面命令，使其支持 Angular 的 Commit message 格式，

```bash
commitizen init cz-conventional-changelog --save --save-exact
```

以后，凡是用到 **git commit** 命令，一律改用 **git cz** ，这时候就会出现选项，来生成符合规范的 commit message 。

如果我们希望每个使用 git 的项目都遵循这个标准，可以使用下面命令进行全局设置。

安装 cz-conventional-changelog ，

```bash
npm install -g cz-conventional-changelog
```

创建一个 **.czrc** 文件在你的 home 目录，并将 path 指向上面所安装的 commitizen 适配器，

```bash
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

现在我们可以在每个 git 项目中使用 **git cz** 提交我们的 commit message 了。

## 生成 Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成。

```bash
npm install -g conventional-changelog
```

然后进入项目目录，

```bash
conventional-changelog -p angular -i CHANGELOG.md -w
```

上面命令不会覆盖以前的 Change log，只会在 CHANGELOG.md 的头部加上自从上次发布以来的变动。

如果你想生成所有发布的 Change log，要改为运行下面的命令。

```bash
conventional-changelog -p angular -i CHANGELOG.md -w -r 0
```

为了方便使用，可以将其写入 package.json 的 script 字段。

```json
...
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -w -r 0"
  }
}
...
```

然后每次运行下面命令。

```bash
npm run changelog
```

生成的 change log 包括下面几个部分，

> - New features
> - Bug fixes
> - Breaking changes

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。

完。

**参考文章**

[https://github.com/commitizen/cz-cli](https://github.com/commitizen/cz-cli)

[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
---
layout: post
title: 前端效率提升的一些思考（下）
category: 技术
tags: [总结,开发,前端,效率,提升]
keywords: 总结,开发,前端,效率,提升]
description: 
---

## 前言

作为前端开发，想要提升效率肯定不能够仅仅依赖于别人的工具，必要时候我们可以自己操刀上。
而前端造工具的基础肯定是 `Node`。配合`commander` 等工具我们就可以自由的开发想要的命令行工具。

## 一个例子

在真实开发中，我常常会遇到这么一种情况，就是开发很多模块的时候，需要创建类似的目录，创建类似的js,类似的css，然后里面要初始化代码，虽然内容我们可以通过之前的文章提到的代码片段来生成，但是大部分创建动作是重复的，因此我需要一个工具来做到以下功能:

1. 可以选择模板列表，选中后在目录下创建对应模板
2. 可以将已有的目录迁移到模板目录。

为了开发这样的工具我们需要明确开发任务：

1、获取指定目录下对于的目录列表
2、可供命令行上下选择的功能
3、对于目录迁移的模块

因此我们可以简单的在开发前把命令列出来：

```
ct

Options:
  -h, --help  output usage information

Commands:
  list        模板列表
  new         创建模板
  add         新增模板

```

## 正式开发

初始化的目录结构如下:

```

├── bin
├── main
├── node_modules
├── package-lock.json
├── package.json
├── readme.md
├── scripts
├── template
├── tsconfig.json
└── untils

```

首先是要解决一个交互，就是列出指定目录下的模板。让用户可以选择，而且为了增强交互体验，用户可以进行模糊查询。

很容易可以找到`inquirer`模块 与 `inquirer-fuzzy-path`插件。

但是这个插件有个不符合要求的地方就是它每次输出的目录结构都是完整的绝对路径，对于想要的输出结果是只需要输出目录名称即可。

但是这个插件又不支持对输出目录的处理回调，因此看了下源码直接把这个插件修改源码放到`until`目录下。

```

const fs = require("fs");
const path = require("path");
const util = require("util");

const Choices = require("inquirer/lib/objects/choices");
const InquirerAutocomplete = require("inquirer-autocomplete-prompt");
const stripAnsi = require("strip-ansi");
const style = require("ansi-styles");
const fuzzy = require("fuzzy");

const readdir = util.promisify(fs.readdir);

function getPaths(
  rootPath,
  pattern,
  excludePath,
  itemType,
  defaultItem,
  showRegx
) {
  const fuzzOptions = {
    pre: style.green.open,
    post: style.green.close
  };
  async function listNodes(nodePath) {
    try {
      if (excludePath(nodePath)) {
        return [];
      }
      const nodes = await readdir(nodePath);
      const currentNode =
        itemType !== "file" && showRegx && nodePath.replace(showRegx, "")
          ? showRegx
            ? [nodePath.replace(showRegx, "")]
            : [nodePath]
          : [];
      if (nodes.length > 0) {
        const nodesWithPath = nodes.map(nodeName =>
          listNodes(path.join(nodePath, nodeName))
        );
        const subNodes = await Promise.all(nodesWithPath);
        return subNodes.reduce((acc, val) => acc.concat(val), currentNode);
      }
      return currentNode;
    } catch (err) {
      if (err.code === "ENOTDIR") {
        return itemType !== "directory" ? [nodePath] : [];
      }
      return [];
    }
  }
  const nodes = listNodes(rootPath);
  const filterPromise = nodes.then(nodeList => {
    const filteredNodes = fuzzy
      .filter(pattern || "", nodeList, fuzzOptions)
      .map(e => e.string);
    if (!pattern && defaultItem) {
      filteredNodes.unshift(defaultItem);
    }
    return filteredNodes;
  });
  return filterPromise;
}

class InquirerFuzzyPath extends InquirerAutocomplete {
  constructor(question, rl, answers) {
    const rootPath = question.rootPath || ".";
    const excludePath = question.excludePath || (() => false);
    const itemType = question.itemType || "any";
    const showRegx = question.showRegx || "";
    const questionBase = Object.assign({}, question, {
      source: (_, pattern) =>
        getPaths(
          rootPath,
          pattern,
          excludePath,
          itemType,
          question.default,
          showRegx
        )
    });
    super(questionBase, rl, answers);
  }

  search(searchTerm) {
    return super.search(searchTerm).then(() => {
      this.currentChoices.getChoice = choiceIndex => {
        const choice = Choices.prototype.getChoice.call(
          this.currentChoices,
          choiceIndex
        );
        return {
          value: stripAnsi(choice.value),
          name: stripAnsi(choice.name),
          short: stripAnsi(choice.name)
        };
      };
    });
  }

  onSubmit(line) {
    super.onSubmit(stripAnsi(line));
  }
}

module.exports = InquirerFuzzyPath;


```

修改前的输出

![wechatimg605.png](http://img.haoqiao.me/wikiwechatimg605.png)

修改后的输出

![wechatimg606.png](http://img.haoqiao.me/wikiwechatimg606.png)

这样复杂的第一步就解决了，接下来就是获取到对应目录的内容，然后根据参数进行功能执行。

### ct new

首先是完成命令 `ct new`的操作。

这个命令需要兼容不同的情况，当用户之间输入`ct new 模板名称`时，将在当前文件夹创建同名的目录。 如果用户输入`ct new 模板名称 目录名称`，那么将创建以目录名称为名的目录，里面的内容是指定的模板。如果用户输入`ct new` 那么将提供模板选择，并在用户输入后，告知用户输入对应的模板名称。

效果如下:

![ct.gif](http://img.haoqiao.me/wikict.gif)

这个功能的具体代码如下:

```

import * as shell from "shelljs";
const path = require("path");
const fs = require("fs");
const chalk = require("chalk");
const inquirer = require("inquirer");

// 返回模板列表
const listDir = () => {
  let fileArr = fs.readdirSync(path.join(__dirname, "../template/"), function(
    err,
    files
  ) {
    if (err) {
      console.log(err);
    }
  });
  return fileArr;
};

// 删除目录
const deleteFolderRecursive = path => {
  if (fs.existsSync(path)) {
    fs.readdirSync(path).forEach(function(file, index) {
      var curPath = path + "/" + file;
      if (fs.lstatSync(curPath).isDirectory()) {
        // recurse
        deleteFolderRecursive(curPath);
      } else {
        // delete file
        fs.unlinkSync(curPath);
      }
    });
    fs.rmdirSync(path);
  }
};

// 将模板拷贝到指定目标目录
const copyTemplates = (templatePath, targetPath, templateName) => {
  // 遍历对应模板目录的所有文件，并复制到目标目录
  const readAndCopyFile = (templatePath, tempPath) => {
    let files = fs.readdirSync(templatePath);
    files.forEach(file => {
      let curPath = `${templatePath}/${file}`;
      let stat = fs.statSync(curPath);
      let filePath = `${targetPath}/${file}`;
      if (stat.isDirectory()) {
        fs.mkdirSync(filePath);
        readAndCopyFile(`${templatePath}/${file}`, `${tempPath}/${file}`);
      } else {
        const contents = fs.readFileSync(curPath, "utf8");
        fs.writeFileSync(filePath, contents, "utf8");
      }
    });
  };
  readAndCopyFile(templatePath, templateName);
};

// 初始化模板目录
const initTemplateFolder = (
  templatePath,
  targetPath,
  templateName,
  folderName
) => {
  if (fs.existsSync(targetPath)) {
    // 如果已存在改模块，提问开发者是否覆盖该模块
    inquirer
      .prompt([
        {
          name: "overwrite",
          type: "confirm",
          message: `目录 ${folderName} 已存在，是否选择覆盖?`,
          validate: function(input) {
            if (input.lowerCase !== "y" && input.lowerCase !== "n") {
              return "Please input y/n !";
            } else {
              return true;
            }
          }
        }
      ])
      .then(answers => {
        // 如果确定覆盖
        if (answers["overwrite"]) {
          // 删除文件夹
          deleteFolderRecursive(targetPath);
          console.log(chalk.yellow(`已删除存在的目录。`));
          //创建新模块文件夹
          fs.mkdirSync(targetPath);
          // 拷贝模板
          copyTemplates(templatePath, targetPath, templateName);
          console.log(chalk.green(`模板 "${templateName}" 创建完毕!`));
        }
      })
      .catch(err => {
        console.log(chalk.red(err));
      });
  } else {
    //创建新模块文件夹
    fs.mkdirSync(targetPath);
    copyTemplates(templatePath, targetPath, templateName);
    console.log(chalk.green(`模板 "${templateName}" 创建完毕!`));
  }
};

export default async (originTemplateName, folderName) => {
  const templateArr = listDir();
  let templatePath = path.join(__dirname, "../template/");
  let templateName =
    typeof originTemplateName == "string" ? originTemplateName : "";
  let targetPath = shell.pwd().stdout;
  folderName = typeof folderName == "string" ? folderName : templateName;
  // 如果没有输入模板名称，那么让其选择一个
  if (!templateName) {
    await inquirer
      .prompt([
        {
          type: "fuzzypath",
          name: "path",
          excludePath: nodePath => nodePath.startsWith("node_modules"),
          showRegx: path.join(__dirname, "../template/"),
          itemType: "directory",
          rootPath: path.join(__dirname, "../template/"),
          message: "请选择一个模板:",
          suggestOnly: false
        }
      ])
      .then(async data => {
        templateName = data.path;
        await inquirer
          .prompt({
            type: "input",
            name: "folder",
            message: "请输入目录名称",
            default: templateName,
            validate: function(input) {
              if (!input) {
                return "不能为空";
              }
              return true;
            }
          })
          .then(data => {
            folderName = data.folder;
            targetPath = targetPath + "/" + folderName;
            templatePath = templatePath + templateName;
            initTemplateFolder(
              templatePath,
              targetPath,
              templateName,
              folderName
            );
          });
      });
  } else {
    // 如果已经输入了模板名称，如果输入的名称不在当前模板列表，那么让其重新选择
    if (templateArr.filter(item => item === templateName).length > 0) {
      targetPath = targetPath + "/" + folderName;
      templatePath = templatePath + templateName;
      initTemplateFolder(templatePath, targetPath, templateName, folderName);
    } else {
      await inquirer
        .prompt([
          {
            type: "fuzzypath",
            name: "path",
            excludePath: nodePath => nodePath.startsWith("node_modules"),
            showRegx: path.join(__dirname, "../template/"),
            itemType: "directory",
            rootPath: path.join(__dirname, "../template/"),
            message: "模板不存在,请重新选择:",
            suggestOnly: false
          }
        ])
        .then(async data => {
          templateName = data.path;
          await inquirer
            .prompt({
              type: "input",
              name: "folder",
              message: "请输入目录名称",
              default: templateName,
              validate: function(input) {
                if (!input) {
                  return "不能为空";
                }
                return true;
              }
            })
            .then(data => {
              folderName = data.folder;
              targetPath = targetPath + "/" + folderName;
              templatePath = templatePath + templateName;
              initTemplateFolder(
                templatePath,
                targetPath,
                templateName,
                folderName
              );
            });
        });
    }
  }
};


```


### ct add

`ct add` 命令的目标就是将当前目录下某个目录拷贝到模板目录，让其成为模板列表中的一个。

它的交互与`ct new` 很类似，这里就不多说。 具体看代码：

```

import * as shell from "shelljs";
const path = require("path");
const fs = require("fs");
const chalk = require("chalk");
const inquirer = require("inquirer");

// 删除目录
const deleteFolderRecursive = path => {
  if (fs.existsSync(path)) {
    fs.readdirSync(path).forEach(function(file, index) {
      var curPath = path + "/" + file;
      if (fs.lstatSync(curPath).isDirectory()) {
        // recurse
        deleteFolderRecursive(curPath);
      } else {
        // delete file
        fs.unlinkSync(curPath);
      }
    });
    fs.rmdirSync(path);
  }
};

// 新增模板
const addTemplates = (templatePath, targetPath, folderName) => {
  const readAndCopyFile = (targetPath, tempPath) => {
    let files = fs.readdirSync(targetPath);
    files.forEach(file => {
      let curPath = `${targetPath}/${file}`;
      let stat = fs.statSync(curPath);
      let filePath = `${templatePath}/${file}`;
      if (stat.isDirectory()) {
        fs.mkdirSync(filePath);
        readAndCopyFile(`${targetPath}/${file}`, `${tempPath}/${file}`);
      } else {
        const contents = fs.readFileSync(curPath, "utf8");
        fs.writeFileSync(filePath, contents, "utf8");
      }
    });
  };
  readAndCopyFile(targetPath, folderName);
};

// 初始化模板列表
const initTemplateFolder = (
  templatePath,
  targetPath,
  templateName,
  folderName
) => {
  if (fs.existsSync(templatePath)) {
    // 如果已存在模板目录
    inquirer
      .prompt([
        {
          name: "overwrite",
          type: "confirm",
          message: `模板 ${templateName} 已存在，是否选择覆盖?`,
          validate: function(input) {
            if (input.lowerCase !== "y" && input.lowerCase !== "n") {
              return "Please input y/n !";
            } else {
              return true;
            }
          }
        }
      ])
      .then(answers => {
        // 如果确定覆盖
        if (answers["overwrite"]) {
          // 删除文件夹
          deleteFolderRecursive(templatePath);
          console.log(chalk.yellow(`已删除存在的模板。`));

          //创建新文件夹
          fs.mkdirSync(templatePath);
          // 拷贝目录
          addTemplates(templatePath, targetPath, folderName);
          console.log(chalk.green(`模板 "${templateName}" 新增完毕!`));
        }
      })
      .catch(err => {
        console.log(chalk.red(err));
      });
  } else {
    //创建新模块文件夹
    fs.mkdirSync(templatePath);
    addTemplates(templatePath, targetPath, folderName);
    console.log(chalk.green(`模板 "${templateName}" 新增完毕!`));
  }
};

export default async (folderName, originTemplateName) => {
  folderName = typeof folderName == "string" ? folderName : "";
  let templateName =
    typeof originTemplateName == "string" ? originTemplateName : "";
  let targetPath = shell.pwd().stdout;
  let templatePath = path.join(__dirname, "../template/") + templateName;
  // 如果没有输入目录，或者输入的目录不存在
  if (!folderName && !fs.existsSync(folderName)) {
    await inquirer
      .prompt({
        type: "input",
        name: "folder",
        message: "目录不存在, 请输入目录名称",
        validate: function(input) {
          if (!input) {
            return "不能为空";
          }
          if (!fs.existsSync(targetPath + "/" + input)) {
            return "目录不存在";
          }
          return true;
        }
      })
      .then(async data => {
        folderName = data.folder;
        if (!templateName) {
          await inquirer
            .prompt({
              type: "input",
              name: "template",
              message: "请输入模板名称",
              validate: function(input) {
                if (!input) {
                  return "不能为空";
                }
                return true;
              }
            })
            .then(data => {
              templateName = data.template;
              templatePath =
                path.join(__dirname, "../template/") + data.template;
              targetPath = targetPath + "/" + folderName;
              initTemplateFolder(
                templatePath,
                targetPath,
                templateName,
                folderName
              );
            });
        }
      });
  } else {
    targetPath = targetPath + "/" + folderName;
    templatePath = path.join(__dirname, "../template/") + templateName;
    initTemplateFolder(templatePath, targetPath, templateName, folderName);
  }
};


```

### npm link

开发完工具后我们需要更改一下`package.json`里的内容:

```

  "name": "ct",
  "version": "1.0.0",
  "description": "create template",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "version": "standard-version -- --release-as"
  },
  "bin": {
    "ct": "bin/ct.js"
  },
  ...

```

然后在 `ct` 目录下执行 `npm link` 将工具命令全句化。 之后就能方便使用了。

![wechatimg607.png](http://img.haoqiao.me/wikiwechatimg607.png)

# 结尾

五一期间拖拖拉拉边开发工具边把这篇文章写完，这里只是给一个思路，就是如何写一个交互友好还能提升开发效率的工具，只要有方法和实现方式，配合一些想法，每个前端都能开发出属于自己的效率工具.



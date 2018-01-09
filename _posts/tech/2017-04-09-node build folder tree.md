---
layout: post
title: Node仿Tree指定层级输出树形文件目录结构
category: 技术
tags: [学习,开发,前端,总结,2018,Node开发,Source Code,Node]
keywords: 总结,2018,Node开发,source code,前端
description: 
---

## 前言

这段时间空余时间蛮少，后来特地腾出晚上的时间来开发自己的玩具。
今天讲的是为玩具所开发的一个小模块的一个功能。
具体来说它是一个仿Tree命令能够罗列给定目录的树形结构。而且我的做法是开发一个命令行工具，但这里先不提，我们就专注这个目录列表功能。
需要输出相同的结构。我们先来看下需要达到的效果

![imgn](http://haoqiao.qiniudn.com/node-tree-1.png)

## 正文

我们先来探讨一下如何获得目录结构。因为我们最终需要的是`给路径->获得目录结构->数据处理(输出/另外操作)`

在Node里有这么几个API

```

fs.readdir(path, callback)

该方法是 readdir(3) 的异步执行版本，用于读取一个目录的内容。callback 接收两个参数 (err, files)，其中 files 是一个数组，数组成员为当前目录下的文件名，不包含 . 和 ..。
fs.readdirSync(path)

该方法是 readdir(3) 的同步执行版本，返回一个不包含 . 和 .. 的文件名数组。

...这里就列两个，其他的请自行去官网查API0-0

```

一开始我试图用异步方法来获取，但是最后因为没处理好`Promise`和其它异步操作，导致最后数据没有存储下来。因此现在我们讲的是用同步方式获取。异步方式也将在这个模块写完之后再去写一版本。

基本上网上的方法都是递归调用方法来获取整个目录结构。

这里也不例外。不过我们需要注意我们需要的将整个目录关系都缓存在一个Object里。而且因为我需要指定层级目录输出。因此每个文件/目录还将有一个属性就是`deep`，它作为一个告诉调用者该目录层级的深度。如果是0，说明是指定目录的根目录下同级的文件/目录。依次类推。

当然这里就直接贴出代码

```

_getAllNames: function(level, dir) {
        var filesNameArr = []
        let cur = 0
            // 用个hash队列保存每个目录的深度
        var mapDeep = {}
        mapDeep[dir] = 0
            // 先遍历一遍给其建立深度索引
        function getMap(dir, curIndex) {
            var files = fs.readdirSync(dir) //同步拿到文件目录下的所有文件名
            files.map(function(file) {
                //var subPath = path.resolve(dir, file) //拼接为绝对路径
                var subPath = path.join(dir, file) //拼接为相对路径
                var stats = fs.statSync(subPath) //拿到文件信息对象
                    // 必须过滤掉node_modules文件夹
                if (file != 'node_modules') {
                    mapDeep[file] = curIndex + 1
                    if (stats.isDirectory()) { //判断是否为文件夹类型
                        return getMap(subPath, mapDeep[file]) //递归读取文件夹
                    }
                }

                //console.log(subPath)
            })

        }
        getMap(dir, mapDeep[dir])
            //console.log(mapDeep)

        function readdirs(dir, folderName,myroot) {
            var result = { //构造文件夹数据
                path: dir,
                name: path.basename(path),
                type: 'directory',
                deep: mapDeep[folderName]
            }
            var files = fs.readdirSync(dir) //同步拿到文件目录下的所有文件名

            result.children = files.map(function(file) {
                //var subPath = path.resolve(dir, file) //拼接为绝对路径


                var subPath = path.join(dir, file) //拼接为相对路径
                var stats = fs.statSync(subPath) //拿到文件信息对象
                    //console.log(subPath)
                if (stats.isDirectory()) { //判断是否为文件夹类型
                    // console.log(mapDeep[file])
                    return readdirs(subPath, file,file) //递归读取文件夹
                }
                return { //构造文件数据
                    path: subPath,
                    name: file,
                    type: 'file'
                }
            })

            return result //返回数据
        }

        filesNameArr.push(readdirs(dir, dir))

        return filesNameArr


    },

```

过滤掉`node_modules`是必须的，因为开发中这个目录一直存在。。。是个极大干扰源。

这里我代码都加了注释应该很好理解。

我们来输出下缓存的目录结构

```

{ path: './',
  name: '[object Object]',
  type: 'directory',
  deep: 0,
  children: 
   [ { path: '.DS_Store', name: '.DS_Store', type: 'file' },
     { path: '.babelrc', name: '.babelrc', type: 'file' },
     { path: '.gitignore', name: '.gitignore', type: 'file' },
     { path: 'README.MD', name: 'README.MD', type: 'file' },
     { path: 'bin',
       name: '[object Object]',
       type: 'directory',
       deep: 1,
       children: [Object] },
     { path: 'lib',
       name: '[object Object]',
       type: 'directory',
       deep: 1,
       children: [Object] },
     { path: 'package.json', name: 'package.json', type: 'file' },
     { path: 'test',
       name: '[object Object]',
       type: 'directory',
       deep: 1,
       children: [Object] } ] }
[ { path: './',
    name: '[object Object]',
    type: 'directory',
    deep: 0,
    children: 
     [ [Object],
       [Object],
       [Object],
       [Object],
       [Object],
       [Object],
       [Object],
       [Object] ] } ]

```

针对`./`这个当前目录的路径，程序返回上面的结构。
这样我们就得到了第一步最基础的数据。

现在第一个步骤是为了模仿Tree工具输出命令。

这里给个例子参考:

```

├── .DS_Store
├── .babelrc
├── .gitignore
├── README.MD
├── bin
│   ├── .DS_Store
│   ├── folderTree.js
│   ├── lib2
│   │   ├── .DS_Store
│   │   └── testa
│   └── testb
│       └── .DS_Store
├── lib
│   ├── .DS_Store
│   ├── folderFactory.js
│   └── testlib
│       └── testlibfile.js
├── package.json
└── test
    ├── .DS_Store
    ├── index.js
    └── testFolder
        ├── .DS_Store
        ├── a
        ├── b
        └── c

```
它有几个注意点，一个是每当你的文件或者目录是当前层级最后一个，那么输出`└──`前缀如果是普通的输出`├── `前缀。
如果是多层级，那么最前面依旧需要`│ `来进行上下层级的联系。

而且还有很多小细节需要处理。

一开始最好的解决方法其实是递归。但是我觉得有点复杂，我想试下迭代方法。最后结果如开头的效果。基本上如果根目录中最后一个file是目录类型。那么它是正常的。但是如果根目录中最后一个文件是文件类型类型。我这段迭代并没有进行处理。

我们先来看下这个有缺陷的代码：

```

     var stack = [obj[0]]
        var isFinal = false
        function printFolder(arr, folderName, isLastFolder) {
            if (arr.deep <= level) {
                for (var i = 0; i < arr.children.length; i++) {
                    var isLastFile = i == arr.children.length - 1 ? true : false
                    var isRootLast = i == arr.children.length - 1
                    isFinal = isFinal == true ? true :  arr.deep == 0 && arr.children[i].type == 'directory' && i == arr.children.length-1
                    printFile(arr.children[i], folderName, arr.deep, isLastFile, isLastFolder,isFinal)

                    if (arr.children[i].type == 'directory') {
                        //console.log('directory')
                        var t = arr.children[i].path
                        if (i == arr.children.length - 1) {
                            printFolder(arr.children[i], t, true)

                        } else {
                            printFolder(arr.children[i], t, false)

                        }

                    }

                }
            }
        }

        function printFile(file, folderName, deep, isLastFile, isLastFolder) {
            if (file[0] != '.') {
                // console.log("Folder:"+folderName)
                // console.log(deep)
                //console.log(isLastFile)
                //console.log(isLastFolder)
                var name = file.path.replace(folderName + '/', '')

                //console.log(isFinal)
                if (deep == 0) {
                    if (isLastFile) {
                        console.log('└── ' + name)

                    } else {
                        console.log('├── ' + name)
                    }
                }
                if (deep > 0) {
                    if (!isLastFolder) {
                        if (!isLastFile) {

                            console.log('│   '.repeat(deep) + '├── ' + name)
                        } else {

                            console.log('│   '.repeat(deep) + '└── ' + name)

                        }
                    } else {
                        if (!isLastFile) {
                            console.log('    '.repeat(deep) + '├── ' + name)
                        } else {
                        	 if(isFinal){
                             console.log('    '.repeat(deep) + '└── ' + name)

                        	 }else{
                        	  var str = '    '.repeat(deep) + '└── ' + name
                        	  var temp = str.split('')
                        	  temp[0] = '│'
                            console.log(temp.join(''))
                        	 }

                        }
                    }
                }

            }
        }
        printFolder(obj[0], '', false)
        console.log('目录及文件罗列完毕')


```

我做了很多判断，但还是低估了一个目录它可能的复杂性(目录中有空目录，目录中有空文件，文件和目录的多个嵌套等等)。因为我开发的时候是根据几个example来进行调节判断。当我最后写完，重新再去测试几个目录的时候发现出现上述的file缺陷。
如图：
![imgn](http://haoqiao.qiniudn.com/node-tree-3.png)

因此需要改写为递归方式。我们不可能根据别的属性来进行判断。因此我们需要依赖的还是之前的那份目录结构缓存。

这里也直接贴源码:

```

_showList(obj, level) {
        var sourceStruct = obj[0]
        var dirCount = 0
        var fileCount = 0
            // 字符集
        var charSet = {
            'node': '├── ', //节点
            'pipe': '│   ', // 上下链接
            'last': '└── ', // 最后的file或folder需要回勾
            'indent': '    ' // 缩进
        };

        function log(file, depth, parentHasNextSibling) {
           // console.log("log:")
            if (!parentHasNextSibling && depth.length > 1) {
                // Replace a pipe with an indent if the parent does not have a next sibling.
                depth[depth.length - 2] = charSet.indent;
            }
            if(file.lastIndexOf('/') > -1){
                file = file.substring(file.lastIndexOf('/')+1)
            }
            console.log(depth.join('') + file);
        }

        // 由于已经有缓存数据了，因此对数据进行遍历搜索
        // 如果type 是file 就不需要继续
        // 如果type 是directory 对临时数组
        function walk(path, depth, parentHasNextSibling) {
          //  console.log(path)

            var childrenLen = path.length - 1
           // console.log(childrenLen)
            var loop = true

            if (depth.length >= level) {
                loop = false
            }
            if (loop) {
                path.forEach(function walkChildren(child, index) {
                    var newdepth = !!depth ? depth.slice(0) : []
                    var isLast = (index >= childrenLen)

                    if (isLast) {
                        newdepth.push(charSet.last)
                    } else {
                        newdepth.push(charSet.node)
                    }
                    if(child.type == "file"){
                      log(child.name, newdepth, parentHasNextSibling)
                    }else{
                      log(child.path, newdepth, parentHasNextSibling)

                    }

                    if (child.type == "directory") {
                        var childPath = child.children
                        if (!isLast) {
                            newdepth.pop()
                            newdepth.push(charSet.pipe)
                        }
                        walk(childPath, newdepth, !isLast)
                    }

                })
                loop = !loop
            }

        }

        walk(sourceStruct.children, [])
        //console.log(sourceStruct)
        console.log('level:' + level)
        console.log('目录及文件罗列完毕')

    },
    
```

这里只需要判断文件类型，利用递归的特性将每个文件的路径存于`newpath`，然后判断长度，如果大于level就跳过。具体注释都在源码里了。

![imgn](http://haoqiao.qiniudn.com/node-tree-2.png)

## 结尾

第一个模块总算完成，四月第一篇文章居然在中旬也算对得起拖延症了~


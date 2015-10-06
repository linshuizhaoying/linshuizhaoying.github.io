---
layout: post
title: Jquery 实现原理深入学习(3)
category: 技术
tags: [Jquery，Jquery学习,项目小结,前端学习,前端总结，javascript]
keywords: 前端,资料,学习
description: 
--- 
## 前言

```
1.总体结构 √

2.构建函数 √

3.each功能函数实现 √

4.map功能函数实现 √

5.sizzle初步学习 √

6.attr功能函数实现

7.toggleClass功能函数实现(好伤)

8.val功能函数实现

9.ajax异步请求以及扩展学习

```

## 正文
今天学习sizzle，这是一个非常大的命题。以目前的水平只能浅尝则止。

sizzle是一款纯js实现的css选择权引擎。而选择器这个功能是Jquery中用的最多的部分。

看下[官方文档](https://github.com/jquery/sizzle/wiki)

```
Sizzle supports virtually all CSS 3 Selectors, including escaped selectors (.foo\+bar), Unicode selectors, and results returned in document order. The only exceptions are those that would require additional DOM event listeners to keep track of the state of elements.
```

sizzle支持几乎所有css3选择器，并按照文档位置返回结果。一些深层次的我们不会去探讨，主要研究这几个小内容.

```
1.引擎入口
2.match方法
3.find方法
4.getText方法

```

直接跳到sizzle函数

```
//选择器入口，查找与选择器表达式selector匹配的元素集合
function Sizzle( selector, context, results, seed ) {
//selector:css选择器表达式.
//context:上下文环境.
//results:可选的数组或类数组,sizzle将把查找到的元素添加到其中.
//seed:可选的元素集合.函数sizzle将从该元素集合中过滤出匹配选择器表达式的元素集合.

	var match, elem, m, nodeType,
		// QSA vars
		i, groups, old, nid, newContext, newSelector;

	if ( ( context ? context.ownerDocument || context : preferredDoc ) !== document ) {
		setDocument( context );

	}

	context = context || document; //如果未传入context,则默认为当前document对象.
	results = results || [];//如果未传入参数results,则默认为空数组[].
	nodeType = context.nodeType; //判断上下文节点类型。nodetype具体请看下面。

	if ( typeof selector !== "string" || !selector ||
		nodeType !== 1 && nodeType !== 9 && nodeType !== 11 ) {
//如果参数selector是空字符串,或者不是字符串,节点类型不是元素节点或者document节点或者DOCUMENT_FRAGMENT节点则直接返回results.
注意在1.7版本它是分selector和nodeType两种情况返回的，最新版直接一起判断了

		return results;
	}

	if ( !seed && documentIsHTML ) {
 // 尽可能快地找到目标节点, 选择器类型是id,标签和类
		if ( nodeType !== 11 && (match = rquickExpr.exec( selector )) ) {
		   // rquickExpr.exec 看下文
			// Speed-up: Sizzle("#ID")
			if ( (m = match[1]) ) { //把正则结果存到m中
				if ( nodeType === 9 ) { 这里才是真正判断节点是不是元素节点
					elem = context.getElementById( m );
              
              // 检查Blackberry 4.6返回的已经不在document中的parentNode
		
					if ( elem && elem.parentNode ) {
						// IE, Opera, Webkit有时候会返回name == m的元素
						if ( elem.id === m ) {
							results.push( elem );
							return results;
						}
					} else {
						return results;
					}
				} else {
					// 上下文不是document
					if ( context.ownerDocument && (elem = context.ownerDocument.getElementById( m )) &&
						contains( context, elem ) && elem.id === m ) {
						results.push( elem );
						return results;
					}
				}

			// Speed-up: Sizzle("TAG")
			} else if ( match[2] ) {
				push.apply( results, context.getElementsByTagName( selector ) );
				return results;
          
			// Speed-up: Sizzle(".CLASS")
			} else if ( (m = match[3]) && support.getElementsByClassName ) {
				push.apply( results, context.getElementsByClassName( m ) );
				return results;
			}
		}
    
		// QSA path
		if ( support.qsa && (!rbuggyQSA || !rbuggyQSA.test( selector )) ) {
			nid = old = expando;
			newContext = context;
			newSelector = nodeType !== 1 && selector;

			      // 使用QSA, QSA: querySelectorAll, 原生的QSA运行速度非常快,因此尽可能使用QSA来对CSS选择器进行查询
          // querySelectorAll是原生的选择器,但不支持老的浏览器版本, 主要是IE8及以前的浏览器
          // rbuggyQSA 保存了用于解决一些浏览器兼容问题的bug修补的正则表达式
          // QSA在不同浏览器上运行的效果有差异，表现得非常奇怪，因此对某些selector不能用QSA
          // 为了适应不同的浏览器，就需要首先进行浏览器兼容性测试，然后确定测试正则表达式,用rbuggyQSA来确定selector是否能用QSA			// IE 8 doesn't work on object elements
			if ( nodeType === 1 && context.nodeName.toLowerCase() !== "object" ) {
				groups = tokenize( selector );

				if ( (old = context.getAttribute("id")) ) {
					nid = old.replace( rescape, "\\$&" );
				} else {
					context.setAttribute( "id", nid );
				}
				nid = "[id='" + nid + "'] ";

				i = groups.length;
				while ( i-- ) {
					groups[i] = nid + toSelector( groups[i] );
				}
				newContext = rsibling.test( selector ) && testContext( context.parentNode ) || context;
				newSelector = groups.join(",");
			}

			if ( newSelector ) {
				try {
					push.apply( results,
						newContext.querySelectorAll( newSelector )
					);
					return results;
				} catch(qsaError) {
				} finally {
					if ( !old ) {
						context.removeAttribute("id");
					}
				}
			}
		}
	}

	// All others
	return select( selector.replace( rtrim, "$1" ), context, results, seed );
}

```

```
函数Sizzle执行的6个关键步骤如下:
1.解析选择器表达式,解析出块表达式和关系符.
2.如果存在位置伪类,则从左向右查找:
a.查找第一个块表达式匹配的元素集合,得到第一个上下文元素集合.
b.遍历剩余的块表达式和块间关系符,不断缩小上下文元素集合.
3.否则从右向左查找:
a.查找最后一个块表达式匹配的元素集合,得到候选集,映射集.
b.遍历剩余的块表达式和块间关系符,对映射集执行块间关系过滤.
4.根据映射集筛选候选集,将最终匹配的元素放入结果集.
5.如果存在并列选择器表达式,则递归Sizzle()查找匹配的元素集合,并合并,排序,去重.
6.最后返回结果集.

```
根据以上提示我们往回走看源码。

期间我们发现了一个Nodetype的东东，一开始我以为是jquery自己定义，后发翻遍源码发现并不是，那么应该是原生的，google了一下在mdn找到了详细解释。

```
概述
返回一个整数值,代表当前节点的节点类型.

语法
var type = node.nodeType;
type的值可能如下:

常量名	值
ELEMENT_NODE	1
ATTRIBUTE_NODE	2
TEXT_NODE	3
CDATA_SECTION_NODE	4
ENTITY_REFERENCE_NODE	5
ENTITY_NODE	6
PROCESSING_INSTRUCTION_NODE	7
COMMENT_NODE	8
DOCUMENT_NODE	9
DOCUMENT_TYPE_NODE	10
DOCUMENT_FRAGMENT_NODE	11
NOTATION_NODE	12

```

然后一路走下来，发现 `match = rquickExpr.exec( selector ))`

来看看它是做什么的。

```
	// Easily-parseable/retrievable ID or TAG or CLASS selectors
	rquickExpr = /^(?:#([\w-]+)|(\w+)|\.([\w-]+))$/,
```
一个解析选择器类型的正则表达式。

然后我们一路走下来发现其实之前的6个关键提示一点都没用-0- 然而我们并不打算深入词法什么的，因此直接跳到getText

```
getText = Sizzle.getText = function( elem ) {
	var node,
		ret = "",
		i = 0,
		nodeType = elem.nodeType;

	if ( !nodeType ) {
		// 如果不是节点，那么假设它是一个数组
		while ( (node = elem[i++]) ) {
			// 跳过注释节点
			ret += getText( node );
		}
	} else if ( nodeType === 1 || nodeType === 9 || nodeType === 11 ) {
		// 对元素使用textContent textContent 属性设置或返回指定节点的文本内容，以及它的所有后代。
		if ( typeof elem.textContent === "string" ) {
			return elem.textContent; //如果是字符串就返回
		} else {
			// 遍历其子节点
			for ( elem = elem.firstChild; elem; elem = elem.nextSibling ) {
				ret += getText( elem ); //递归
			}
		}
	} else if ( nodeType === 3 || nodeType === 4 ) {
		return elem.nodeValue; //如果是text节点或者CDATA_SECTION节点 那么直接返回其值
	}
	// 不包括注释或处理指令节点

	return ret;
};

```
我们可以发现 仅仅只是一个获取text内容，jquery就已经写的很细很细。希望以后有能力把sizzle里真正精髓的部分学习掉。

今天就到这里。




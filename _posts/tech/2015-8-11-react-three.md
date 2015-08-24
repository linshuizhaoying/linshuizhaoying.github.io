---
layout: post
title: React从入门到上手三部曲(3)
category: 技术
tags: [react，react学习,react实例,react入门,react总结]
keywords: 学习资料，程序语言
description: 
---

## 前言
三部曲想想应该是连贯的，因此承接上期，我们把整个包括Node.js的部分也讲述一下，可能已经不是正常的纯React教程了（似乎第二期就有点偏了。）。不过个人认为项目驱动的教程更适合入门吧。o(≧v≦)o~

## 正文
上期讲到数据的传递，我们提到props和state，如果是组件创建前自己去获取的数据应该放到哪呢？答案是state,因为自动获取的数据是变化的。props应该是一开始就不变的。
我们来看正式项目中的newsComponent.jsx和newsitem.jsx里的内容。
首先是 newsComponent.jsx里的内容
    
    getInitialState: function() {
      return {
         data : [{"title":"欢迎使用定制版每日资讯","href":"haoqaio.me"},{"title":"By 临水照		  影","href":"haoqiao.me"}]
      };
    },
    componentDidMount: function() {
      $.post("/getData" , {url:this.props.url}, function(result) {
      	var list = result;
      	if (this.isMounted()) {
      	  this.setState({
      	    data:JSON.parse(result)
      	  });
     	 }
       }.bind(this));
    },
	render: function() {
		var url = this.props.url;
		var component_style = "portlet box " + this.props.component_style;
		var item_style = this.props.item_style;
			console.log(this.state.data);
	    var datas = this.state.data;
		if(datas){
			var NewsItems = datas.map(function(Item) {
            return <NewsItem item_style={item_style} key={Item.id} title={Item.title} href={Item.href}/>
     });
     }
	
    return (
			<div className={component_style}>
				<div className="portlet-title">
					<div className="caption">
					  <i className="fa fa-cogs"></i>
					  今日资讯 From {url}
					</div>
					<div class="tools">
						<a href="javascript:;" data-original-title="" title="" class="expand"></a>
					</div>
				</div>
				<div className="portlet-body">
					  {NewsItems}
				</div>
			</div>  
          );
    }
    
    
看到我们可以看到比教程2里的演示这里多了不少东西，我们一一来分析。
首先是分析我们设计的思路，我们是想要在props里面传过来url，然后组件在渲染之前自己拿到数据，然后再渲染。这就涉及到组件的生命周期。我一开始的想法是直接在组件的componentDidMount里直接$get拿数据，但是遇到了问题，首先是数据获取异步的问题，如果是$get，我url提交过去了，后台处理中，没等后台处理返回结果它就先返回了。这样的结果就是只渲染了父组件。render的时候调用this.state.data还是是undefined。在chrome里面调试的时候可以发现过了一会state里面才有内容。
<<<<<<< HEAD

![img1](http://7s1say.com1.z0.glb.clouddn.com//react11.jpg?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
=======
![img1](http://7s1say.com1.z0.glb.clouddn.com/react11.jpg?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
>>>>>>> 50e947e626468b25132932e5bef01ba0de58fb7c

遇到这种之前没接触过react问题，当然先是去群里吼一声，然后先天不聪大神直接给出两种解决方案
	
	1.使用$.ajax的async参数，配置async=false, 这个时候为同步请求，componentWillMount会等待ajax请求完成，你能成功的在componentWillMount中拿到data。后果就是，页面卡在这里，直到你拿到数据。
	2.另外一种方式就是，给data一个默认值，第一次render让它画. 在componentDidMount函数里，$.get拿数据，拿到数据后setState会重新render，页面更新，总共2次render.
对于ajax的异步async设置为false这种方案我尝试无效后（可能是我写法有问题）我选择了第二种，先赋值，再setState。这种方案更加符合React的设计思路。在不同生命周期做不同的事情。首先是在 getInitialState 先赋值。这里提一句，由于我数据返回的是json，因此测试数据应该也是json格式的，虽然我后台拿到数据后生成的是一个数组，但是通过Json转换很容易拿到对应的格式。
在componentDidMount生命周期，也就是组件完成渲染后的周期我们去拿数据，这里我为啥不用get反而用Post呢？原因就是Node.js中，如果我用get，那么我route里面会这么写  /getdata/:url 

但是我们url里面当含有需要转义的字符比如  xxx.com/news ，这样我们就无法用req.params.url这种方式提取url，因此使用Post。当post拿到数据后我们发现有一句isMounted()的判断，从官网文档我们可以知道
   
     isMounted() returns true if the component is rendered into the DOM, false otherwise. You can use this method to guard asynchronous calls to setState() or forceUpdate().
   当组件完成渲染后isMounted()返回true，这样我们setState改变state后自然而然就会重新渲染。细心的读者会发现渲染的时候子组件多了一个props key={Item.id} ，这是为了保证唯一性。如果想测试为什么的同学可以直接在刚开始赋值的时候把数据换成之后后台传过来的数据。
   
再来看newsitem.jsx，这里面的内容改动不大，只需要把href的值拿过来在render里就行，代码就不给出了，反正在github里我已经把所有源码传上去了。

### Node.js后台处理
   
   对于后台数据的处理我们直接来看关键代码
   
    router.post('/getdata', function(req, res) {
	var url = req.body.url;
	console.log(url);
	var data_result = new Array();
	if(url == "toutiao.io"){
		superagent.get(url)
		    .end(function (err,response) {
		    	  if (err) {
	         return next(err);
	        }
		   
			 var $ = cheerio.load(response.text);
			// console.log(cheerio.load(res.text));
			$('.daily .posts .post .content .title a').each(function (idx, element) {
					    var $element = $(element);
					    var single = new Object();
					    
					    single.title = $element.text();
					    single.href = $element.attr('href');
					   // console.log("single.title content:"+single.title);
							data_result.push(single);
								//保存数据
	
			       });
		         res.send(JSON.stringify(data_result));
			       res.end();
	 
		 });
     }
     
 根据不同域名针对不同内容格式进行爬取，这个用到了几个模块，具体细节就不讲了。注意的是.end(function (err,response) 这句，官方给的示例是.end(function (err,res) ,会给后面res.send带来很大的麻烦，我调试了很久才发现这句有问题-0-，最后结果直接JSON.stringify转一下就能返回。
### 测试
代码写完了，我们可以来看下如何使用，我们直接在app.jsx里面写上


	React.render(
    	<NewsComponent component_style="green" item_style="info" url="toutiao.io"/>,
    	document.getElementById('example1')
	);
	React.render(
        <NewsComponent component_style="blue" item_style="default" url="news.dbanotes.net/	    newest" />,
     document.getElementById('example2')
	);
	
之后我们就可以看到正式运行后的界面。
<<<<<<< HEAD


![img2](http://7s1say.com1.z0.glb.clouddn.com//react12.png?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

![img3](http://7s1say.com1.z0.glb.clouddn.com//react13.png?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
=======
![img2](http://7s1say.com1.z0.glb.clouddn.com/react12.png?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
![img3](http://7s1say.com1.z0.glb.clouddn.com/react13.png?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

>>>>>>> 50e947e626468b25132932e5bef01ba0de58fb7c


这样整个系统就算打通了，剩下就是部署了=-=，部署这块我还得继续学习，毕竟一入部署深似海，从此人品是路人。Ok，本次React三部曲也算结束了。有什么问题可以Email或者在博客里留言，不过我用的是社会评论插件，想在某个专题里凭论还需要重新在页面里刷新-0-下一个阶段我就算打算自己写一个外载的评论插件。应该会去学习一下vue.js然后配合React应用到下个项目中。本次项目已经传到了github上，有兴趣可以点 [这里](https://github.com/linshuizhaoying/news.haoqaio.me)去fork或者clone。

---
layout: post
title: Jquery 实现原理深入学习(6)
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

6.attr功能函数实现 √

7.toggleClass功能函数实现(好伤) √

8.val功能函数实现 √

9.ajax异步请求以及扩展学习 √

```

## 先从XMLHttpRequest谈起

提到ajax我们肯定得先提XMLHttpRequest，jquery的ajax也是对XMLHttpRequest的封装。因此对于今天的学习将不会侧重于它封装的所有细节，而是争取掌握XMLHttpRequest。因此今天的小分类应该是这样的.

```
1.XMLHttpRequest学习
2.XMLHttpRequest封装为xhr.js的学习
3.Jquery的ajax学习

```

### XMLHttpRequest

先从[mdn文档](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)看起。

```
XMLHttpRequest 是一个 JavaScript 对象,它最初由微软设计,随后被 Mozilla,Apple, 和 Google采纳. 如今,该对象已经被 W3C组织标准化. 通过它,你可以很容易的取回一个URL上的资源数据. 尽管名字里有XML, 但XMLHttpRequest 可以取回所有类型的数据资源,并不局限于XML. 而且除了HTTP ,它还支持file 和 ftp 协议.

```
它有几个属性，看上去蛮重要的。

```
1.onreadystatechange
  一个JavaScript函数对象，当readyState属性改变时会调用它。回调函数会在user interface线程中调用。（不能在本地代码中使用. 也不应该在同步模式的请求中使用.）
  
2.readyState
  请求的五种状态
	
	值	状态	描述
	0	UNSENT (未打开)	open()方法还未被调用.
	1	OPENED  (未发送)	send()方法还未被调用.
	2	HEADERS_RECEIVED (已获取响应头)	send()方法已经被调用, 响应头和响应状态已经返回.
	3	LOADING (正在下载响应体)	响应体下载中; responseText中已经获取了部分数据.
	4	DONE (请求完成)	整个请求过程已经完毕.
	
3.response
  响应实体的类型由 responseType 来指定， 可以是 ArrayBuffer， Blob， Document， JavaScript 对象 (即 "json")， 或者是字符串。如果请求未完成或失败，则该值为 null。
  
4.responseText	
  此次请求的响应为文本，或是当请求未成功或还未发送时为 null。只读。
  
5.responseType
	设置该值能够改变响应类型。就是告诉服务器你期望的响应格式。
	
	Value	Data type of response property
	"" (空字符串)	字符串(默认值)
	"arraybuffer"	ArrayBuffer
	"blob"	Blob
	"document"	Document
	"json"	JavaScript 对象，解析自服务器传递回来的JSON 字符串。
	"text"	字符串'

6.status
  该请求的响应状态码 (例如, 状态码200 表示一个成功的请求).只读.
 
7.statusText
  该请求的响应状态信息,包含一个状态码和原因短语 (例如 "200 OK"). 只读.

8.upload	
  可以在 upload 上添加一个事件监听来跟踪上传过程。

9.withCredentials	
  表明在进行跨站(cross-site)的访问控制(Access-Control)请求时，是否使用认证信息(例如cookie或授权的header)。 默认为 false。
	
```

我们可以看到几个比较感兴趣的，分别是status,response,upload,withCredentials。
尤其是withCredentials，它可以让我们本地测试api更加的全面。
upload好像可以用来实现上传反馈。
其余几个都是常用的不多说。

接下来看下它的方法。

```
1.getResponseHeader()
  
  返回指定的响应头的值, 如果响应头还没被接受,或该响应头不存在,则返回null.

2.open()

初始化一个请求. 该方法用于JavaScript代码中;如果是本地代码, 使用 openRequest()方法代替.
如果它已经被调用了，那么再执行就是中止请求。
	参数
	method
	请求所使用的HTTP方法; 例如 "GET", "POST", "PUT", "DELETE"等. 如果下个参数是非HTTP(S)的URL,则忽略该参数.
	url
	该请求所要访问的URL
	async
	An optional boolean parameter, defaulting to true, indicating whether or not to perform the operation asynchronously. If this value is false, the send()method does not return until the response is received. If true, notification of a completed transaction is provided using event listeners. This must be true if the multipart attribute is true, or an exception will be thrown.
	user
	用户名,可选参数,为授权使用;默认参数为空string.
	password
	密码,可选参数,为授权使用;默认参数为空string.
	
3.send()
	发送请求. 如果该请求是异步模式(默认),该方法会立刻返回. 相反,如果请求是同步模式,则直到请求的响应完全接受以后,该方法才会返回.（注意: 所有相关的事件绑定必须在调用send()方法之前进行.）
	
	void send();
	void send(ArrayBuffer data);
	void send(Blob data);
	void send(Document data);
	void send(DOMString? data);
	void send(FormData data);

4.setRequestHeader()
  给指定的HTTP请求头赋值.在这之前,你必须确认已经调用 open() 方法打开了一个url.
  参数

	header
	将要被赋值的请求头名称.
	value
	给指定的请求头赋的值.

```

现在我们来看下实际用法。

```
function reqListener () {
  console.log(this.responseText);
}

var oReq = new XMLHttpRequest();
oReq.onload = reqListener;
oReq.open("get", "yourFile.txt", true);
oReq.send();

```

更多例子详解可以在[这里](https://xhr.spec.whatwg.org/)了解。

事实上我们需要关心的如同普通ajax提交，一个是Url,传输数据类型，传输数据，返回数据。如果需要更详细的应该是header头。接下来我们看一下一个已经被封装好了的xhr.js

### xhr.js

这个封装函数是我学习vue的时候别人的项目中用到的。源码如下:

```

var defaults = {};

function request(url, options) { //两个参数,url和可选参数
  var req = new XMLHttpRequest();//新建一个xmlhttprequest请求
  
  req.error = function(cb) {
    options.error = cb; 
  };

  var method = options.method || 'GET'; //如果没有传入相关参数 默认是GET
  var data = options.data; //传入的数据
  if (method === 'GET' && data) {  
  //拼凑成get提交的形式,可以看下面直观的例子图
  //Object.keys 返回一个数组，数组里是该obj可被枚举的所有属性
    url += '?' + Object.keys(data).filter(function(k){
      return k && data[k];
    }).map(function(k) {
      return k + '=' + data[k];
    }).join('&');
    data = null;
  }
  
  req.url = url;
  req.open(method, url); //GET请求发送

  if (options.onload) { //如果可选参数有Onload
    req.onload = options.onload.bind(this); //把新建的xmlHttprequest的Onload绑定上去
  }
  if (options.onerror) {
    req.onerror = options.onerror.bind(this);
  }
  if (options.onprogress) {
    req.upload.onprogress = options.onprogress.bind(this);
  }

  var startTime = new Date().getTime();
  //构建header
  var headers = {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest',
    'X-Requested-Time': startTime.toString(),
  };
  //如果数据是FormData类型，删除'Content-Type': 'application/json',
  if (data instanceof FormData) {
    delete headers['Content-Type'];
  } else if (data) {//如果是个数组就Json格式化
    data = JSON.stringify(data);
  }
  
  
  if (options.headers) {
    Object.keys(options.headers).forEach(function(k) {
      headers[k] = options.headers[k];
    });
  }
  
  Object.keys(headers).forEach(function(k) {
    req.setRequestHeader(k, headers[k]);
  });
 //处理状态
  req.onreadystatechange = function(){
    if (4 === req.readyState) {
      var duration = new Date().getTime() - startTime;
      req.data = parseResponse(req.responseText);
      if (options.afterRequest) {
        options.afterRequest(req, duration);
      }
      var type = req.status / 100 | 0;
      if (2 === type) {
        options.success && options.success(req.data, req);
      } else {
        options.error && options.error(req.data, req);
      }
    }
  };

  if (options.beforeRequest) {
    options.beforeRequest(req);
  }
//如果有数据就发送数据
  if (data) {
    req.send(data);
  } else {
    req.send();
  }

  return req;
}

function parseResponse(text) {
  try {
    return JSON.parse(text);
  } catch (e) {
    return text;
  }
}

function extend(a, b) {
  Object.keys(b).forEach(function(k) {
    if (!a[k]) {
      a[k] = b[k];
    }
  });
  return a;
}

function parseParams(method, data, success, options) {
  if (typeof data === 'function') {
    options = success;
    success = data;
    data = null;
  }
  //提交的数据格式
  options = extend({method: method, data: data, success: success}, options || {});
  return extend(options, defaults);
}

exports.defaults = defaults;

exports.http = request;

//暴露接口
exports.get = function(url, data, success, options) {
  return request(url, parseParams('GET', data, success, options));
};

exports.post = function(url, data, success, options) {
  return request(url, parseParams('POST', data, success, options));
};

exports.del = function(url, data, success, options) {
  return request(url, parseParams('DELETE', data, success, options));
};

exports.put = function(url, data, success, options) {
  return request(url, parseParams('PUT', data, success, options));
};

```

![imgn](http://haoqiao.qiniudn.com/ajax101.png)

非常简洁实用的代码，而且它有很强的扩展性。

### Jquery Ajax

接下来是进入正题。先来看看ajax的源码:

由于ajax源码超级长，我挑几个感兴趣的看看，首先是ajaxSettings

```
	ajaxSettings: {
		url: ajaxLocation,
		type: "GET", //默认是GET提交
		isLocal: rlocalProtocol.test( ajaxLocParts[ 1 ] ),
		global: true,
		processData: true,
		async: true,
		//用于定义网络文件的类型和网页的编码
		contentType: "application/x-www-form-urlencoded; charset=UTF-8",
		/*
		timeout: 0,
		data: null,
		dataType: null,
		username: null,
		password: null,
		cache: null,
		throws: false,
		traditional: false,
		headers: {},
		*/

		accepts: {
			"*": allTypes,
			text: "text/plain",
			html: "text/html",
			xml: "application/xml, text/xml",
			json: "application/json, text/javascript"
		},

		contents: {
			xml: /xml/,
			html: /html/,
			json: /json/
		},

		responseFields: {
			xml: "responseXML",
			text: "responseText",
			json: "responseJSON"
		},

		// Data converters
		// Keys separate source (or catchall "*") and destination types with a single space
		converters: {

			// Convert anything to text
			"* text": String,

			// Text to html (true = no transformation)
			"text html": true,

			// Evaluate text as a json expression
			"text json": jQuery.parseJSON,

			// Parse text as xml
			"text xml": jQuery.parseXML
		},


		flatOptions: {
			url: true,
			context: true
		}
	},
	
```
往下看把代码缩起来可以看到大概的结构。

```
1.前置过滤器
2.请求发送器
3.数据转换器
4.ajax主函数

```
我们之间来看主函数:

```
	// Main method
	ajax: function( url, options ) {

		// 如果url是个对象 模拟签名
		if ( typeof url === "object" ) {
			options = url;
			url = undefined;
		}

		// 强迫options变为一个对象
		options = options || {};

		var // Cross-domain detection vars
			parts,
			// Loop variable
			i,
			// URL without anti-cache param
			cacheURL,
			// Response headers as string
			responseHeadersString,
			// timeout handle
			timeoutTimer,

			// To know if global events are to be dispatched
			fireGlobals,

			transport,
			// Response headers
			responseHeaders,
			// Create the final options object
			s = jQuery.ajaxSetup( {}, options ),
			// Callbacks context
			callbackContext = s.context || s,
			// Context for global events is callbackContext if it is a DOM node or jQuery collection
			globalEventContext = s.context && ( callbackContext.nodeType || callbackContext.jquery ) ?
				jQuery( callbackContext ) :
				jQuery.event,
			// Deferreds
			deferred = jQuery.Deferred(),
			completeDeferred = jQuery.Callbacks("once memory"),
			// Status-dependent callbacks
			statusCode = s.statusCode || {},
			// Headers (they are sent all at once)
			requestHeaders = {},
			requestHeadersNames = {},
			// The jqXHR state
			state = 0,
			// Default abort message
			strAbort = "canceled",
			// Fake xhr
			jqXHR = {
				readyState: 0,

				// Builds headers hashtable if needed
				getResponseHeader: function( key ) {
					var match;
					if ( state === 2 ) {
						if ( !responseHeaders ) {
							responseHeaders = {};
							while ( (match = rheaders.exec( responseHeadersString )) ) {
								responseHeaders[ match[1].toLowerCase() ] = match[ 2 ];
							}
						}
						match = responseHeaders[ key.toLowerCase() ];
					}
					return match == null ? null : match;
				},

				// Raw string
				getAllResponseHeaders: function() {
					return state === 2 ? responseHeadersString : null;
				},

				// Caches the header
				setRequestHeader: function( name, value ) {
					var lname = name.toLowerCase();
					if ( !state ) {
						name = requestHeadersNames[ lname ] = requestHeadersNames[ lname ] || name;
						requestHeaders[ name ] = value;
					}
					return this;
				},

				// Overrides response content-type header
				overrideMimeType: function( type ) {
					if ( !state ) {
						s.mimeType = type;
					}
					return this;
				},

				// Status-dependent callbacks
				statusCode: function( map ) {
					var code;
					if ( map ) {
						if ( state < 2 ) {
							for ( code in map ) {
								// Lazy-add the new callback in a way that preserves old ones
								statusCode[ code ] = [ statusCode[ code ], map[ code ] ];
							}
						} else {
							// Execute the appropriate callbacks
							jqXHR.always( map[ jqXHR.status ] );
						}
					}
					return this;
				},

				// Cancel the request
				abort: function( statusText ) {
					var finalText = statusText || strAbort;
					if ( transport ) {
						transport.abort( finalText );
					}
					done( 0, finalText );
					return this;
				}
			};

		// Attach deferreds
		deferred.promise( jqXHR ).complete = completeDeferred.add;
		jqXHR.success = jqXHR.done;
		jqXHR.error = jqXHR.fail;

		// Remove hash character (#7531: and string promotion)
		// Add protocol if not provided (#5866: IE7 issue with protocol-less urls)
		// Handle falsy url in the settings object (#10093: consistency with old signature)
		// We also use the url parameter if available
		s.url = ( ( url || s.url || ajaxLocation ) + "" ).replace( rhash, "" ).replace( rprotocol, ajaxLocParts[ 1 ] + "//" );

		// Alias method option to type as per ticket #12004
		s.type = options.method || options.type || s.method || s.type;

		// Extract dataTypes list
		s.dataTypes = jQuery.trim( s.dataType || "*" ).toLowerCase().match( rnotwhite ) || [ "" ];

		// A cross-domain request is in order when we have a protocol:host:port mismatch
		if ( s.crossDomain == null ) {
			parts = rurl.exec( s.url.toLowerCase() );
			s.crossDomain = !!( parts &&
				( parts[ 1 ] !== ajaxLocParts[ 1 ] || parts[ 2 ] !== ajaxLocParts[ 2 ] ||
					( parts[ 3 ] || ( parts[ 1 ] === "http:" ? "80" : "443" ) ) !==
						( ajaxLocParts[ 3 ] || ( ajaxLocParts[ 1 ] === "http:" ? "80" : "443" ) ) )
			);
		}

		// Convert data if not already a string
		if ( s.data && s.processData && typeof s.data !== "string" ) {
			s.data = jQuery.param( s.data, s.traditional );
		}

		// Apply prefilters
		inspectPrefiltersOrTransports( prefilters, s, options, jqXHR );

		// If request was aborted inside a prefilter, stop there
		if ( state === 2 ) {
			return jqXHR;
		}

		// We can fire global events as of now if asked to
		// Don't fire events if jQuery.event is undefined in an AMD-usage scenario (#15118)
		fireGlobals = jQuery.event && s.global;

		// Watch for a new set of requests
		if ( fireGlobals && jQuery.active++ === 0 ) {
			jQuery.event.trigger("ajaxStart");
		}

		// Uppercase the type
		s.type = s.type.toUpperCase();

		// Determine if request has content
		s.hasContent = !rnoContent.test( s.type );

		// Save the URL in case we're toying with the If-Modified-Since
		// and/or If-None-Match header later on
		cacheURL = s.url;

		// More options handling for requests with no content
		if ( !s.hasContent ) {

			// If data is available, append data to url
			if ( s.data ) {
				cacheURL = ( s.url += ( rquery.test( cacheURL ) ? "&" : "?" ) + s.data );
				// #9682: remove data so that it's not used in an eventual retry
				delete s.data;
			}

			// Add anti-cache in url if needed
			if ( s.cache === false ) {
				s.url = rts.test( cacheURL ) ?

					// If there is already a '_' parameter, set its value
					cacheURL.replace( rts, "$1_=" + nonce++ ) :

					// Otherwise add one to the end
					cacheURL + ( rquery.test( cacheURL ) ? "&" : "?" ) + "_=" + nonce++;
			}
		}

		// Set the If-Modified-Since and/or If-None-Match header, if in ifModified mode.
		if ( s.ifModified ) {
			if ( jQuery.lastModified[ cacheURL ] ) {
				jqXHR.setRequestHeader( "If-Modified-Since", jQuery.lastModified[ cacheURL ] );
			}
			if ( jQuery.etag[ cacheURL ] ) {
				jqXHR.setRequestHeader( "If-None-Match", jQuery.etag[ cacheURL ] );
			}
		}

		// Set the correct header, if data is being sent
		if ( s.data && s.hasContent && s.contentType !== false || options.contentType ) {
			jqXHR.setRequestHeader( "Content-Type", s.contentType );
		}

		// Set the Accepts header for the server, depending on the dataType
		jqXHR.setRequestHeader(
			"Accept",
			s.dataTypes[ 0 ] && s.accepts[ s.dataTypes[0] ] ?
				s.accepts[ s.dataTypes[0] ] + ( s.dataTypes[ 0 ] !== "*" ? ", " + allTypes + "; q=0.01" : "" ) :
				s.accepts[ "*" ]
		);

		// Check for headers option
		for ( i in s.headers ) {
			jqXHR.setRequestHeader( i, s.headers[ i ] );
		}

		// Allow custom headers/mimetypes and early abort
		if ( s.beforeSend && ( s.beforeSend.call( callbackContext, jqXHR, s ) === false || state === 2 ) ) {
			// Abort if not done already and return
			return jqXHR.abort();
		}

		// aborting is no longer a cancellation
		strAbort = "abort";

		// Install callbacks on deferreds
		for ( i in { success: 1, error: 1, complete: 1 } ) {
			jqXHR[ i ]( s[ i ] );
		}

		// Get transport
		transport = inspectPrefiltersOrTransports( transports, s, options, jqXHR );

		// If no transport, we auto-abort
		if ( !transport ) {
			done( -1, "No Transport" );
		} else {
			jqXHR.readyState = 1;

			// Send global event
			if ( fireGlobals ) {
				globalEventContext.trigger( "ajaxSend", [ jqXHR, s ] );
			}
			// Timeout
			if ( s.async && s.timeout > 0 ) {
				timeoutTimer = setTimeout(function() {
					jqXHR.abort("timeout");
				}, s.timeout );
			}

			try {
				state = 1;
				transport.send( requestHeaders, done );
			} catch ( e ) {
				// Propagate exception as error if not done
				if ( state < 2 ) {
					done( -1, e );
				// Simply rethrow otherwise
				} else {
					throw e;
				}
			}
		}

		// Callback for when everything is done
		function done( status, nativeStatusText, responses, headers ) {
			var isSuccess, success, error, response, modified,
				statusText = nativeStatusText;

			// Called once
			if ( state === 2 ) {
				return;
			}

			// State is "done" now
			state = 2;

			// Clear timeout if it exists
			if ( timeoutTimer ) {
				clearTimeout( timeoutTimer );
			}

			// Dereference transport for early garbage collection
			// (no matter how long the jqXHR object will be used)
			transport = undefined;

			// Cache response headers
			responseHeadersString = headers || "";

			// Set readyState
			jqXHR.readyState = status > 0 ? 4 : 0;

			// Determine if successful
			isSuccess = status >= 200 && status < 300 || status === 304;

			// Get response data
			if ( responses ) {
				response = ajaxHandleResponses( s, jqXHR, responses );
			}

			// Convert no matter what (that way responseXXX fields are always set)
			response = ajaxConvert( s, response, jqXHR, isSuccess );

			// If successful, handle type chaining
			if ( isSuccess ) {

				// Set the If-Modified-Since and/or If-None-Match header, if in ifModified mode.
				if ( s.ifModified ) {
					modified = jqXHR.getResponseHeader("Last-Modified");
					if ( modified ) {
						jQuery.lastModified[ cacheURL ] = modified;
					}
					modified = jqXHR.getResponseHeader("etag");
					if ( modified ) {
						jQuery.etag[ cacheURL ] = modified;
					}
				}

				// if no content
				if ( status === 204 || s.type === "HEAD" ) {
					statusText = "nocontent";

				// if not modified
				} else if ( status === 304 ) {
					statusText = "notmodified";

				// If we have data, let's convert it
				} else {
					statusText = response.state;
					success = response.data;
					error = response.error;
					isSuccess = !error;
				}
			} else {
				// We extract error from statusText
				// then normalize statusText and status for non-aborts
				error = statusText;
				if ( status || !statusText ) {
					statusText = "error";
					if ( status < 0 ) {
						status = 0;
					}
				}
			}

			// Set data for the fake xhr object
			jqXHR.status = status;
			jqXHR.statusText = ( nativeStatusText || statusText ) + "";

			// Success/Error
			if ( isSuccess ) {
				deferred.resolveWith( callbackContext, [ success, statusText, jqXHR ] );
			} else {
				deferred.rejectWith( callbackContext, [ jqXHR, statusText, error ] );
			}

			// Status-dependent callbacks
			jqXHR.statusCode( statusCode );
			statusCode = undefined;

			if ( fireGlobals ) {
				globalEventContext.trigger( isSuccess ? "ajaxSuccess" : "ajaxError",
					[ jqXHR, s, isSuccess ? success : error ] );
			}

			// Complete
			completeDeferred.fireWith( callbackContext, [ jqXHR, statusText ] );

			if ( fireGlobals ) {
				globalEventContext.trigger( "ajaxComplete", [ jqXHR, s ] );
				// Handle the global AJAX counter
				if ( !( --jQuery.active ) ) {
					jQuery.event.trigger("ajaxStop");
				}
			}
		}

		return jqXHR;
	},
```
这里我并没有仔细去逐条学习，因为看了一下大概我发现ajax函数中对xmlHttpRequest的各个参数都做了很细的分划，但大概思路还是相同的。如果让我选择我可能会选择xhr.js的方式来组建自己的项目,毕竟ajax功能越全代表越复杂，组织格式虽然尽力在优化，但是读起来还是有点晦涩。而xhr.js的优点就显示出来了，方便学习，易于扩展。

其实说是ajax的学习，更多的还是回归到了本源xmlhttprequest的学习。而且新的api fetch好像也出了，当它被所有浏览器支持时，可能请求数据会变得更加简单。


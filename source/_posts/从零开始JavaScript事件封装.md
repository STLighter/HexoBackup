---
title: 从零开始JavaScript事件封装
date: 2016-07-17 11:25:38
categories: JavaScript
tags:
  - JavaScript
---

最近在学习某个效果的实现, 然而看着一堆`function`堆在一起的代码很难受, 想来封装一下. 然而一动起手来发现要写的东西还真不少, 所以想着分成几部分来做, 先就从事件开始.
本人接触前端时间不长, 也没有企业的项目经历(毕竟还没毕业), 只能把之前学到的一些东西东拼西凑起来, 可能封装的不是很优美. 如果有什么错误或者有更好更优美的姿势, 希望能够留言告诉我, 共同学习进步.
雷姆是我的!

---

事件处理是JS中的重头戏. 为事件指定其处理程序的方法有很多, 如HTML事件处理, DOM0级事件处理和DOM2级事件处理, DOM事件对象也不尽相同. 本文就通过封装, 提供一个跨浏览器的事件处理对象.

首先从指定事件处理程序的方式来说, 可以从HTML直接指定事件处理程序:
```HTML
<!-- Professional JavaScript for Web Developer 3rd Edition - Chinese - P348 -->
<input type="button" value="Click Me" onclick="alert('Clicked')"/>
```
更详细的用法这里不多说了, 现在几乎没人会这样做了, 因为这种写法有几个缺陷.
> * 如果这个元素显示在页面上了, 而在HTML中指定的事件处理方法还未被解析到, 这样会引发错误(例如你调用的方法在外部js脚本中, 而DOM加载并显示后, 脚本却还没下载解析完成, 这样就找不到这个方法).
> * 作用域链难以确定, 不同的JS引擎有不同的解析方式, 会导致不同浏览器下的作用域链不相同.
> * HTML与JS代码耦合, 常常需要同时修改HTML代码和JS代码.

DOM0级事件通过给事件处理程序属性赋值实现指定处理程序:
```JavaScript
// Professional JavaScript for Web Developer 3rd Edition - Chinese - P350
var btn = document.getElementByID("myBtn");
btn.onclick = function() {
	alert('Clicked');
	alert(this.id);
};
```
这样的事件被视为元素的方法, `this`指向触发事件的元素, 事件在冒泡阶段被处理, 通过赋值`null`删除事件处理程序.
DOM0级事件的缺点在于, 其只能指定一个事件处理程序, 后面指定的程序会覆盖前面指定的程序. 也就是说, 如果JS代码的两个地方为一个元素指定了不同的事件处理程序, 那么只有后面指定的程序会执行.
<!-- more -->

DOM2级事件通过两个指定的方法添加和删除事件处理程序: `addEventListener()`和`removeEventListener()`.
```JavaScript
// Professional JavaScript for Web Developer 3rd Edition - Chinese - P352
var btn = document.getElementByID("myBtn");
var handler = function() {
	alert(this.id);
};
btn.addEventListener("click", handler, false);
btn.removeEventListener("click", handler, false);
```
其中第一个参数是要处理的事件名字符串, 第二个是事件处理程序对象, 第三个表示是否在事件捕获阶段调用处理程序, true表示在事件捕获阶段调用, false表示在事件冒泡阶段调用.
使用DOM2级事件能够为一个事件添加多个事件处理程序, 添加的程序会按添加的循序触发, 程序执行的作用域和DOM0级事件一样, 是其依附的元素.

这样就完了?用DOM2级事件不就完美了么?!

然而, 他有坑爹的兼容性问题. IE8及其他更早的IE浏览器不支持这两个方法, 但他也有自己的一套方法: `attachEvnet()`和`detachEvent()`.
```JavaScript
// Professional JavaScript for Web Developer 3rd Edition - Chinese - P353
var btn = document.getElementByID("myBtn");
var handler = function() {
	alert(this === window);
	alert("Clicked");
}
btn.attachEvnet("onclick", handler);
btn.detachEvent("onclick", handler);
```
第一个参数是事件名称, 注意前面有`"on"`. 第二个参数是要指定的处理程序.
用这组方式需要注意的是, 他的作用域与之前的不同. 程序中`this`指向的是全局作用域`window`. 此外程序触发是逆序触发, 不过这点影响不大.

我们来看看《JavaScript高级程序设计(第3版)》中封装的跨浏览器的事件处理程序.
```JavaScript
//Professional JavaScript for Web Developer 3rd Edition - Chinese - 13.2.5 P354
var EventUtil = {
	addHandler: function (ele, type, handler) {
		//能力检测
		if (ele.addEventListener) {
			//alert('add');
			ele.addEventListener(type, handler, false);
		} else if (ele.attachEvent) {
			//alert('attach');
			ele.attachEvent("on" + type, handler);
		} else {
			//alert('on');
			ele["on" + type] = handler;
		}
	},
	removeHandler: function (ele, type, handler) {
		//能力检测
		if(ele.removeEventListener) {
			//alert('remove');
			ele.removeEventListener(type, handler, false);
		} else if (ele.detachEvent) {
			//alert('detach');
			ele.detachEvent("on" + type, handler);
		} else {
			//alert('on');
			ele["on" + type] = null;
		}
	}
};
```
其采用能力检测, 将事件处理程序均放在冒泡中触发, 如果DOM2级方法和IE方法都无效(现代浏览器应该不会出现这种情况)就使用DOM0级方法.

这个封装有一个小问题在于, 每次添加或删除事件处理程序的时候都会执行一次能力检测, 因此可以通过"缓存"检测的结果来达到只检测一次.
```JavaScript
// 使用函数表达式使其自动执行
var ST_on = (function () {
	// 局部私有变量, 缓存对应的方法
	var _addHandler,
		_removeHandler;

	// 添加事件
	function add (ele, type, handler) {
		// 检测缓存
		if(!_addHandler) {
			// 能力检测
			if (ele.addEventListener) {
				//alert('add');
				_addHandler = function (ele, type, handler) {
					ele.addEventListener(type, handler, false);
				}
			} else if (ele.attachEvent) {
				//alert('attach');
				_addHandler = function (ele, type, handler) {
					ele.attachEvent("on" + type, handler);
				}
			} else {
				//alert('on');
				_addHandler = function (ele, type, handler) {
					ele["on" + type] = handler;
				}
				
			}
		}
		// 执行真正的添加事件方法
		_addHandler.apply(this, arguments);
	}

	// 删除事件
	function remove (ele, type, handler) {
		if(!_removeHandler) {
			// 能力检测
			if(ele.removeEventListener) {
				//alert('remove');
				_removeHandler = function (ele, type, handler) {
					ele.removeEventListener(type, handler, false);
				}
			} else if (ele.detachEvent) {
				//alert('detach');
				_removeHandler = function (ele, type, handler) {
					ele.detachEvent("on" + type, handler);
				}
				
			} else {
				//alert('on');
				_removeHandler = function (ele, type, handler) {
					ele["on" + type] = null;
				}
			}
		}
		// 执行真正的添加事件方法
		_removeHandler.apply(this, arguments);
	}

	// 暴露接口
	return {
		addHandler: add,
		removeHandler: remove
	};
})();
```
简单的说就是第一次执行的时候进行能力检测, 并把检测结果放到私有变量中缓存, 再次执行时直接从缓存中取出对应的方法对象.

接下来处理事件对象和作用域.
事件对象同样存在兼容问题.
> * IE中用DOM0级方法添加事件处理程序, `event`对象是`window`对象的一个属性, 而不最为参数传给事件处理程序.
> * DOM中`event.target`为事件目标对象, 在IE中是`event.srcElement`.
> * DOM中用`preventDefault()`方法取消事件的默认行为, 在IE中需要将`event.returnValue`设为`false`.
> * DOM中用`stopPropagation()`方法阻止事件冒泡, 在IE中需要把`event.cancelBubble`设置为`true`.

红宝书(《JavaScript高级程序设计》)上是这样封装的:
```JavaScript
// EventUtil (Professional JavaScript for Web Developer 3rd Edition - Chinese - 13.3.3 P360)
var EventUtil = {
	addHandler: function (ele, type, handler) {
		// 省略
	},
	removeHandler: function (ele, type, handler) {
		// 省略
	},
	getEvent: function (event) {
		/*if(event) {
			alert('event');
			return event;
		} else {
			alert('window.event');
			return window.event;
		}*/
		return event? event: window.event;
	},
	getTarget: function (event) {
		if(event.target) {
			alert('target');
			return event.target;
		} else {
			alert('srcEle');
			return event.srcElement;
		}
		return event.target || event.srcElement;
	},
	preventDefault: function  (event) {
		if(event.preventDefault) {
			//alert('preventDefault')
			event.preventDefault();
		} else {
			//alert('returnValue');
			event.returnValue = false;
		}
	},
	stopPropagation: function (event) {
		if(event.stopPropagation) {
			//alert('stopPropagation')
			event.stopPropagation();
		} else {
			//alert('cancelBubble');
			event.cancelBubble = true;
		}
	}
};
```
这样用起来需要像下面这样:
```JavaScript
EventUtil.addHandler(window, 'load', function () {
	function mainClick (event) {
		alert('onclick');
		event = EventUtil.getEvent(event);
		var target = EventUtil.getTarget(event);
		alert(target);
		EventUtil.preventDefault(event);
		EventUtil.stopPropagation(event);
		EventUtil.removeHandler(document.getElementById('main'), 'click', mainClick);
	}
	EventUtil.addHandler(document.getElementById('main'), 'click', mainClick);
}
```
每次需要手动调用`EventUtil`里面的方法转换, 非常麻烦, 于是我们考虑单独封装`event`, 然后将封装后的`event`传给事件处理函数. 这也意味着, 我们需要创建一个新的事件处理函数, 在这个函数中接收并封装`event`, 再执行真正的事件处理函数, 并传入封装后的`event`.
```JavaScript
function (event) {
	// 处理对象的兼容性
	_event = event? event: window.event;
	_target = _event.target || _event.srcElement;
	_preventDefault = function () {
		if(_event.preventDefault) {
			_event.preventDefault();
		} else {
			_event.returnValue = false;
		}
	}
	_stopPropagation = 	function () {
		if(_event.stopPropagation) {
			_event.stopPropagation();
		} else {
			_event.cancelBubble = true;
		}
	}
	// 包装成一个新的对象
	var ST_event = {
		event: _event,
		target: _target,
		preventDefault: _preventDefault,
		stopPropagation: _stopPropagation
	}

	// 给真正的handler传入包装过的event
	handler(ST_event);
}
```
之前还提到了作用域不一致的问题, 也可以在我们新的事件处理函数中解决. 只需要在执行真正的处理函数时绑定执行的上下文就可以了. 综合起来我们把他写成一个`fixHandler()`方法, 并将新的函数注册为事件处理函数.
```JavaScript
// 修正handler
function fixHandler (context, handler) {
	// 返回一个修正的handler
	return function (event) {
		// 处理对象的兼容性
		_event = event? event: window.event;
		_target = _event.target || _event.srcElement;
		_preventDefault = function () {
			if(_event.preventDefault) {
				_event.preventDefault();
			} else {
				_event.returnValue = false;
			}
		}
		_stopPropagation = 	function () {
			if(_event.stopPropagation) {
				_event.stopPropagation();
			} else {
				_event.cancelBubble = true;
			}
		}
		// 包装成一个新的对象
		var ST_event = {
			event: _event,
			target: _target,
			preventDefault: _preventDefault,
			stopPropagation: _stopPropagation
		}

		// 给真正的handler传入包装过的event, 并且修正其作用域
		return handler.apply(context, [ST_event].concat(Array.prototype.slice.call(arguments, 1)));
	}
}
// 添加事件
function add (ele, type, handler) {
	// 修正handler
	var fixedHandler = fixHandler(ele, handler);

	// 检测缓存
	if(!_addHandler) {
		// 能力检测
		if (ele.addEventListener) {
			//alert('add');
			_addHandler = function (ele, type, handler) {
				ele.addEventListener(type, handler, false);
			}
		} else if (ele.attachEvent) {
			//alert('attach');
			_addHandler = function (ele, type, handler) {
				ele.attachEvent("on" + type, handler);
			}
		} else {
			//alert('on');
			_addHandler = function (ele, type, handler) {
				ele["on" + type] = handler;
			}
			
		}
	}
	// 执行真正的添加事件处理程序方法, 加入fixedHandler
	_addHandler.call(this, ele, type, fixedHandler);
}
```
然后我们就可以这样使用了:
```JavaScript
ST_on.addHandler(window, 'load', function () {
	function mainClick (event) {
		alert('onclick');
		alert(event.target);
		event.preventDefault();
		event.stopPropagation();
		// removeHandler()无效
		ST_on.removeHandler(document.getElementById('main'), 'click', mainClick);
	}
	ST_on.addHandler(document.getElementById('main'), 'click', mainClick);
});
```
但是这样做之后会产生一个问题: 我们绑定的是新的事件处理函数, 这样`removeHandler()`的时候传入旧的`handler`是没有用的. 如果返回`fixHandler`以便删除的时候使用的话, 既不优美也容易因为忘记而使用原先的`handler`.
所以我们需要在内部缓存新的`fixHandler`, 并且要使得用旧的`handler`能够快速找到对应的`fixHandler`. 

(引入缓存机制的话还能顺便将attach的顺序问题和DOM0级事件绑定不了多个处理函数的问题解决, 本来完全不打算做这么多, 似乎开了一个大坑呢. 另外自定义事件和事件委托等完全不打算做, 所以不要期待)
未完待续...
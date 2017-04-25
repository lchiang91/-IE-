# -事件的IE兼容-
## IE并不支持addEventListener和removeEventListener方法，而是实现了两个类似的方法

1. attachEvent

2. detachEvent

这两个方法都接收两个相同的参数

1. 事件处理程序名称

2. 事件处理程序方法

```
function bindEvent(node,type,handler) {
	if (node.addEventListener) { 
		node.addEventListener(type,handler); 
    }else{
		node.attachEvent("on"+type,handler); 
	}
}
```
## addEventListener和attachEvent主要有几个区别
1. 参数个数不相同，这个最直观，addEventListener有三个参数，attachEvent只有两个，attachEvent添加的事件处理程序只能发生在冒泡阶段，addEventListener第三个参数可以决定添加的事件处理程序是在捕获阶段还是冒泡阶段处理（我们一般为了浏览器兼容性都设置为冒泡阶段）

2. 第一个参数意义不同，addEventListener第一个参数是事件类型（比如click，load），而attachEvent第一个参数指明的是事件处理函数名称（onclick，onload）

3. 事件处理程序的作用域不相同，addEventListener的作用域是元素本身，this是指的触发元素，而attachEvent事件处理程序会在全局变量内运行，this是window，所以刚才例子才会返回undefined，而不是元素id

4. 为一个事件添加多个事件处理程序时，执行顺序不同，addEventListener添加会按照添加顺序执行，而attachEvent添加多个事件处理程序时顺序无规律(添加的方法少的时候大多是按添加顺序的反顺序执行的，但是添加的多了就无规律了)，所以添加多个的时候，不依赖执行顺序的还好，若是依赖于函数执行顺序，最好自己处理，不要指望浏览器

了解了这四点区别后我们可以尝试写一个浏览器兼容性比较好的添加事件处理程序方法,jQuery创始人John Resig是这样做的
```
function addEvent(node, type, handler) {
    if (!node) return false;
    if (node.addEventListener) {
        node.addEventListener(type, handler, false);
        return true;
    }
    else if (node.attachEvent) {
        node['e' + type + handler] = handler;
        node[type + handler] = function() {
            node['e' + type + handler](window.event);
        };
        node.attachEvent('on' + type, node[type + handler]);
        return true;
    }
    return false;
}
```
在取消事件处理程序的时候
```
function removeEvent(node, type, handler) {
    if (!node) return false;
    if (node.removeEventListener) {
        node.removeEventListener(type, handler, false);
        return true;
    }
    else if (node.detachEvent) {
        node.detachEvent('on' + type, node[type + handler]);
        node[type + handler] = null;
    }
    return false;
}
```

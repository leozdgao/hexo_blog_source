title: "FormData对象"
date: 2015-04-09 23:18:05
tags: javascript
categories: Javascript基础
---

兼容性：IE10+

FormData对象可以利用一些键值对来模拟表单控件的值，并在`XMLHttpRequest 2.0`中使用send方法来异步提交表单、上传文件。

**创建FormData**

通过`new`关键字调用构造器，构造器接受一个可选参数（HTMLFormElement），根据现有表单得到FormData：

```
var fData = new FormData();

var form = document.getElementById('my-form');
var oData = new FormData(form);
```

**为FormData添加数据**

FormData数据仅暴露出一个方法`append`，用于往FormData中追加数据，第一个参数为字符串，代表key，第二个参数为字符串、File对象或Blob对象，其他类型都会被转换为字符串。

```
fData.append('key1', 'value1');
fData.append('key2', 'value2');
xhr.send(fData);
```

当FormData中有File时，XMLHttpRequest的请求头中的`Content-Type`被自动设置为`multipart/form-data`。

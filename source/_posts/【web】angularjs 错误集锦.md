---
title: 【web】AngularJs错误集锦
date: 2018-08-29 15:23:22
tags: 
    - angularjs
---

错误信息：

```javascript
// IE10 及以上
TypeError: 无法获取未定义或 null 引用的属性“pager”
   at Anonymous function (http://localhost:9090/AngularProject/1/resources/files/list/component.js?bust=1535512101027:19:5)
   at processQueue (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:17169:13)
   at Anonymous function (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:17217:27)
   at Scope.prototype.$digest (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:18352:15)
   at Scope.prototype.$apply (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:18649:13)
   at done (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:12627:36)
   at completeRequest (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:12871:7)
   at requestLoaded (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:12788:9) Possibly unhandled rejection: {"description":"无法获取未定义或 null 引用的属性“pager”","number":-2146823281,"stack":"TypeError: 无法获取未定义或 null 引用的属性“pager”\n   at Anonymous function (http://localhost:9090/AngularProject/1/resources/files/list/component.js?bust=1535512101027:19:5)\n   at processQueue (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:17169:13)\n   at Anonymous function (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:17217:27)\n   at Scope.prototype.$digest (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:18352:15)\n   at Scope.prototype.$apply (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:18649:13)\n   at done (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:12627:36)\n   at completeRequest (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:12871:7)\n   at requestLoaded (http://localhost:9090/AngularProject/1/resources/js/angular.js?bust=1535512101027:12788:9)"}

// IE9
Error: 拒绝访问。
 Possibly unhandled rejection: {"message":"拒绝访问。\r\n","description":"拒绝访问。\r\n","number":-2147024891,"resource":{}}
```

解决方案：

```javascript
app.config(['$qProvider', function ($qProvider) {
    $qProvider.errorOnUnhandledRejections(false);
}]);
```


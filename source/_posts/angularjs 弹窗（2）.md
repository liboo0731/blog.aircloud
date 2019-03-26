---
title: AngularJs的UI组件ui-bootstrap —— Model
date: 2018-09-28 16:21:23
tags: 
    - angularjs
    - ui-bootstrap
    - ocLazyLoad
---

Model是用来创建模态窗口的，但是实际上，并没有Model指令，而只有`$uibModal`服务，创建模态窗口是使用`$uibModal.open()`方法。

全局的设置可以通过`$uibModalProvider.options`来配置。

##### $uibModal.open()方法可以使用的参数：

| 参数名            | 默认值                         | 备注                                                         |
| ----------------- | ------------------------------ | ------------------------------------------------------------ |
| animation         | true                           | 是否启用动画                                                 |
| appendTo          | body                           | 把模态窗口放在指定的dom元素中。例如$document.find('aside').eq(0) |
| backdrop          | true                           | 打开模态窗口时的背景设置。可设置的值有：true（显示灰色背景，在模态窗口之外单击会关闭模态窗口），false （不显示灰色背景），"static"（显示灰色背景，在模态窗口关闭之前背景元素不可用） |
| backdropClass     |                                | 为背景添加的类名                                             |
| bindToController  | false                          | 设置为true并且使用controllerAs参数时，$scope的属性会传递给模态窗口所使用的controller |
| controller        |                                | 可以设置为一个表示controller的字符串，或者一个函数，或者一个数组（使用数组标记的方式为控制器注入依赖）。 控制器中可使用$uibModalInstance来表示模态窗口的实例。 |
| controllerAs      |                                | controller-as语法的替代写法                                  |
| keyboard          | true                           | 是否允许用ESC键关闭模态窗口                                  |
| openedClass       | modal-open                     | 打开模态窗口时为body元素增加的类名                           |
| resolve           |                                | 传递到模态窗口中的对象                                       |
| scope             | $rootScope                     | 模态窗口的父作用域对象                                       |
| size              |                                | 一个字符串，和前缀“model-”组合成类名添加到模态窗口上         |
| template          |                                | 表示模态窗口内容的文本                                       |
| templateUrl       |                                | 模态窗口内容的模板url                                        |
| windowClass       |                                | 添加到模态窗口模板的类名（不是模态窗口内容模板）             |
| windowTemplateUrl | uib/template/modal/window.html |                                                              |
| windowTopClass    |                                | 添加到顶层模态窗口的类名                                     |

##### $uibModal.open()方法返回模态窗实例的属性：

| 属性名          | 类型     | 说明                                                         |
| --------------- | -------- | ------------------------------------------------------------ |
| close(result)   | function | 关闭模态窗口，传递一个结果                                   |
| dismiss(reason) | function | 取消模态窗口，传递一个原因                                   |
| result          | promise  | 一个promise，窗口关闭时为resolved，窗口取消时为rejected      |
| opened          | promise  | 一个promise，窗口打开并下载完内容解析了所有变量后，promise为resolved |
| closed          | promise  | 一个promise，窗口关闭并且动画结束后为resolved                |
| rendered        | promise  | 一个promise，窗口呈现出来后为resolved                        |

##### 静态资源引入：

```html
<link href="plugins/bootstrap/css/bootstrap.css" rel="stylesheet" />

<script src="js/angular.js"></script>
<script src="js/ocLazyLoad.js"></script>
<script src="js/ui-bootstrap-tpls"></script>
```

##### 定义模块：

```javascript
var baseModule=angular.module('base-module',[
	'ui.bootstrap',
	'oc.lazyLoad'
]);
```

##### 创建弹窗自定义指令：

```javascript
baseModule.directive('uibModal',[
	'$uibModal',
	'$ocLazyLoad',
	'$rootScope',function(
			$uibModal,
			$ocLazyLoad,
			$rootScope){
	return {
		restrict: 'A',
		scope: {
			uibModal: '='
		},
		link:function(scope,element,attr){
			element.on('click', function() {
				//动态加载组件
				$ocLazyLoad.load(scope.uibModal.componentUrl).then(function(){
					//解绑事件，否则下次执行会累计增加一个弹窗
					element.blur();
		        	//打开弹窗
					var options=angular.extend({
						animation:false,
						backdrop:'static',
						auth:true,
		            },scope.uibModal);
					var modalInstance=$uibModal.open(options);
					//下面的特性，要求静态资源也要检查登录,或者要求组件内有发起过数据获取
					if(options.auth){
						modalInstance.rendered.then(function(){
							if(!$rootScope.currentUser){
								modalInstance.close();
							}
						});
					}
				});
		    });
		}
	}
}]);
```

##### 组件中绑定：

```javascript
angular.module('user-groups').component('userGroupsForm', {
	bindings:{
		resolve:'<',
		close:'&',
	    dismiss:'&'
	}
});
```

##### 页面调用：

```html
<button type="button" uib-modal="{componentUrl:'user-groups/form/component',component:'userGroupsForm',size:'lg',resolve:{title:{title:'新建群组'}}}">新建群组</button>
```


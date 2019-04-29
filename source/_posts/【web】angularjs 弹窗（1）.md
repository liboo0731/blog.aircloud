---
title: 【web】angularjs弹窗（ui-bootstrap+directive+ocLazyLoad）
date: 2018-08-16 19:46:32
tags: 
    - web
    - angularjs
    - ui-bootstrap
    - ocLazyLoad
---

由于angularjs 项目中频繁使用弹窗，完全自行编写耗时耗力，所以结合ui-boostrap 中的modal 模块来实现功能

1. 创建一个公共弹窗服务，在使用的组件中依赖注入后调用弹窗方法
2. 在最外层组件（其余组件的父组件）注入弹窗服务，并定义调用弹窗的方法；其余组件require 此父组件，调用父组件中的方法
3. 自定义一个弹窗指令，设置仅属性调用（restrict: 'A'），在主模块注入后，即可全局调用

经过一步步实践和优化后，尽量减少中间环节，最终确认使用自定义指令来实现弹窗功能。

##### 静态资源引入：

```html
<link href="css/main.css" rel="stylesheet" />
<link href="plugins/bootstrap/css/bootstrap.css" rel="stylesheet" />

<script src="js/ocLazyLoad.js"></script>
<script src="js/drag.js"></script>
<script src="js/angular.js"></script>
<script src="js/ui-bootstrap-tpls"></script>
```

##### 自定义弹窗指令：

```javascript
angular.module('common', [
    'ui.bootstrap',
    'oc.lazyLoad'
])
.directive('uibModal',['$uibModal','$ocLazyLoad',function($uibModal,$ocLazyLoad){
    return {
        restrict: 'A',
        scope: {
            uibModal: '='
        },
        link: function(scope,element,attr){
            element.on('click', function() {
                //动态加载组件,在组件加载完成后打开弹窗
                $ocLazyLoad
                    .load(scope.uibModal.path)
                    .then(function(){
                        //弹窗打开方法
                        $uibModal.open({
                            animation:false,
                            size:scope.uibModal.size?scope.uibModal.size:'',
                            backdrop:'static',
                            component: scope.uibModal.component,
                            resolve:{
                                //获取所点击元素内容作为标题
                                title:function(){
                                    return element.context.innerHTML;
                                },
                                //传入组件的数据
                                data:function(){
                                    return scope.uibModal.data;
                                }
                            }
                            }).rendered.then(function(){
                                //弹窗显示出来后，绑定拖拽功能
                                $('.modal-content').drag(function(ev,dd){
                                    $(this).css({
                                        top: dd.offsetY,
                                        left: dd.offsetX
                                    });
                                },{
                                    handle:'.modal-header',
                                    relative:true
                                });
                            });
                });
            });
        }
    }
}]);
```

##### 参数解释：

- path：组件存放路径，按需加载
- component：组件名
- size：弹窗尺寸，默认，sm，lg
- data：传入组件的数据，Object

##### 组件传值及事件绑定：

```javascript
angular.module('users').component('usersForm',{
    templateUrl:'users/form/template.html',
    controller:[function(){
        var $ctrl = this;
        $ctrl.$onInit = function(){
            console.log(this.resolve);
        };

        $ctrl.ok = function () {
          $ctrl.close({$value: $ctrl.resolve.title});
        };

        $ctrl.cancel = function () {
          $ctrl.dismiss({$value: 'cancel'});
        };
    }],
    //close 和dismiss 被绑定自$uibModalInstance
    bindings:{
        resolve:'<',
        close: '&',
        dismiss: '&'
    }
})
```

##### 页面调用：

```html
<a uib-modal="{path:'users/detail/component',component:'usersDetail',data:user}">查看详情</a>

<button type="button" uib-modal="{path:'users/form/component',component:'usersForm'}">新建群组</button>
```




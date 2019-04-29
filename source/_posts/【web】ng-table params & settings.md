---
title: 【web】NgTable params & settings
date: 2018-09-06 19:21:31
tags: 
    - web
    - ng-table
---

### Default params：

```javascript
ngTableDefaults.params.count = 5;    // 单页显示5 条数据，number
ngTableDefaults.params.filter = {name:'a'};    // [name: string]: any
ngTableDefaults.params.group = ;    //    string | Grouping
ngTableDefaults.params.page = 2;    // 从第2 页开始，number
ngTableDefaults.params.sorting = {name:'desc'};    // 使用表名为name的列降序排列
```

### Default settings：

```javascript
ngTableDefaults.settings.$loading = true;    // 开启加载动画，boolean
ngTableDefaults.settings.counts = [5,10,20];    // 单页显示条目数组，number[]
ngTableDefaults.settings.dataOptions = {
        applyFilter:true,    // 是否过滤，boolean
        applyPaging:true,    // 是否分页，boolean
        applySort:true    // 是否排序，boolean
};
ngTableDefaults.settings.dataset = [];    // 数据源，T[]
ngTableDefaults.settings.debugMode = true;    // 开启调试，boolean
ngTableDefaults.settings.defaultSort = 'desc',    // 默认排序，asc 或desc，string
ngTableDefaults.settings.filterOptions = {
        filterComparator:true,    // 如果为true，则精确查询（Exact）；如果为false，模糊查询（ Contains），boolean
        filterDelay:0,    // 默认0，在应用过滤器之前等待用户停止键入的持续时间，number
        filterDelayThreshold:10000,    // 托管内存数组被认为很小的元素数量。 默认为10000，number
        filterFilterName:'filter',    // 实际执行过滤的名称，默认为filter，string
        filterFn:function(data,fileter,filterComparator){    // 使用提供的此函数来执行过滤，而不是选择角度$filter
            // data：数组 []
            // fileter：[name: string]: any ，在数据行上声明的字段名称的映射以及相应的过滤器值
            // filterComparator：true 或者false
            retrun [''];    // 返回一个数组
        },    
        filterLayout:'stack'    // 在单个表标题列中呈现多个html模板时要使用的布局，"stack" | "horizontal"，string
};
ngTableDefaults.settings.getData = function(params){
    return;    // 返回promise
};
ngTableDefaults.settings.getGroups = function(params){
    return;    // 返回promise
};
ngTableDefaults.settings.groupOptions = {
        defaultSort:'asc',    // 默认排序，asc 或desc，string
        isExpanded:true    // 是否展开,默认true，boolean
};
ngTableDefaults.settings.interceptors = [{    // 在数据行显示在表中之前对getData函数的调用结果的拦截器集合，集合中后一个拦截器会去拦截前一个的返回值，最终返回最后一个拦截器的处理结果
        response:function(data,params){
            // data: TData
            // params: NgTableParams<T>
            return;    // 返回数组，TData    
        },
        responseError:function(reason,params){
            // reason: any
            // params: NgTableParams<T>
            return;    // any    
        }
}];    
ngTableDefaults.settings.paginationMaxBlocks = 10;    // 分页显示的最多按钮块，number
ngTableDefaults.settings.paginationMinBlocks = 2;    // 分页显示的最少按钮块，number
ngTableDefaults.settings.sortingIndicator = 'string';    // html标记，用于在表头中显示排序指示符，string
ngTableDefaults.settings.total = 100;    // 数据总行数：100，number
```

### NgTableParams methods：

```
NgTableParams: 
count : ƒ (count)
data:(10) [{…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, visibleColumnCount: 7]
filter : ƒ (filter)
generatePagesArray : ƒ (currentPage, totalItems, pageSize, maxBlocks)
group : ƒ (group, sortDirection)
hasErrorState : ƒ ()
hasFilter : ƒ ()
hasFilterChanges : ƒ ()
hasGroup : ƒ (group, sortDirection)
isDataReloadRequired : ƒ ()
isSortBy : ƒ (field, direction)
orderBy : ƒ ()
page : ƒ (page)
parameters : ƒ (newParameters, parseParamsFromUrl)
reload : ƒ ()
reloadPages : ƒ ()
settings : ƒ (newSettings)
sorting : ƒ (sorting, direction)
total : ƒ (total)
url : ƒ (asString)
__proto__:Object
```


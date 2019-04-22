---
title: 【web】Array & Json 处理	
date: 2018-09-06 19:11:29
tags: 
    - javascript
    - jquery
    - json	
---

### JavaScript 数组操作

##### 1. 创建

```javascript
var arrayObj = new Array();　//创建一个数组
var arrayObj = new Array([size]);　//创建一个数组并指定长度，注意不是上限，是长度
var arrayObj = new Array([element0[, element1[, ...[, elementN]]]]);　//创建一个数组并赋值

//要说明的是，虽然第二种方法创建数组指定了长度，但实际上所有情况下数组都是变长的，也就是说即使指定了长度为5，仍然可以将元素存储在规定长度以外的，注意：这时长度会随之改变
```

##### 2. 访问

```javascript
var testGetArrValue=arrayObj[1]; //获取数组的元素值
arrayObj[1]= "这是新值"; //给数组元素赋予新的值
```

#####  3. 添加

```javascript
arrayObj.push([item1 [item2 [. . . [itemN ]]]]);// 将一个或多个新元素添加到数组结尾，并返回数组新长度
arrayObj.unshift([item1 [item2 [. . . [itemN ]]]]);// 将一个或多个新元素添加到数组开始，数组中的元素自动后移，返回数组新长度
arrayObj.splice(insertPos,0,[item1[, item2[, . . . [,itemN]]]]);//将一个或多个新元素插入到数组的指定位置，插入位置的元素自动后移，返回""
```

##### 4. 删除

```javascript
arrayObj.pop(); //移除最后一个元素并返回该元素值
arrayObj.shift(); //移除最前一个元素并返回该元素值，数组中元素自动前移
arrayObj.splice(deletePos,deleteCount); //删除从指定位置deletePos开始的指定数量deleteCount的元素，数组形式返回所移除的元素
```

##### 5. 截取

```javascript
arrayObj.slice(start, [end]); //以数组的形式返回数组的一部分，注意不包括 end 对应的元素，如果省略 end 将复制 start 之后的所有元素
```

##### 6. 合并

```javascript
arrayObj.concat([item1[, item2[, . . . [,itemN]]]]); //将多个数组（也可以是字符串，或者是数组和字符串的混合）连接为一个数组，返回连接好的新的数组
```

##### 7. 拷贝

```javascript
arrayObj.slice(0); //返回数组的拷贝数组，注意是一个新的数组，不是指向
arrayObj.concat(); //返回数组的拷贝数组，注意是一个新的数组，不是指向
```

##### 8. 排序

```javascript
arrayObj.reverse(); //反转元素（最前的排到最后、最后的排到最前），返回数组地址
arrayObj.sort(); //对数组元素排序，返回数组地址
```

9. 字符串化

```javascript
arrayObj.join(separator); //返回字符串，这个字符串将数组的每一个元素值连接在一起，中间用 separator 隔开
```

### jQuery 数组操作

##### $.each()

```javascript
//$.each(array/object,function(index/key,value){})
//退出 each 循环可使回调函数返回 false, 其它返回值将被忽略
//The method returns its first argument, the object that was iterated.
$.each([ 52, 97 ], function( index, value ) {
  alert( index + ": " + value );
});
//This produces two messages:
// 0: 52 
// 1: 97

//If an object is used as the collection, the callback is passed a key-value pair each time:
var obj = {
  "flammable": "inflammable",
  "duh": "no duh"
};
$.each( obj, function( key, value ) {
  alert( key + ": " + value );
});
//Once again, this produces two messages:
// flammable: inflammable 
// duh: no duh
```

##### $(selector).each() 

```html
<!-- Suppose you have a simple unordered list on the page: -->
<ul>
  <li>foo</li>
  <li>bar</li>
</ul>
```

```javascript
//$(selector).each(function(index,element){}) 
//You can select the list items and iterate across them:
$( "li" ).each(function( index ) {
  console.log( index + ": " + $( this ).text() );
});
//A message is thus logged for each item in the list:
// 0: foo 
// 1: bar

//You can stop the loop from within the callback function by returning false.
```

##### $.map()

```javascript
//$.map(array/object,function(value,index/key){return;})
//更新数组元素值,或根据原值扩展一个新的副本元素
//Map the original array to a new one and add 4 to each value.
$.map( [ 0, 1, 2 ], function( n ) {
  return n + 4;
});
//Result: [4, 5, 6]
```

##### $(selector).map()

```javascript
//$(selector).map(function(index,domElement){return;})
//The .map() method is particularly useful for getting or setting the value of a collection of elements. Consider a form with a set of checkboxes in it:
<form method="post" action="">
  <fieldset>
    <div>
      <label for="two">2</label>
      <input type="checkbox" value="2" id="two" name="number[]">
    </div>
    <div>
      <label for="four">4</label>
      <input type="checkbox" value="4" id="four" name="number[]">
    </div>
    <div>
      <label for="six">6</label>
      <input type="checkbox" value="6" id="six" name="number[]">
    </div>
    <div>
      <label for="eight">8</label>
      <input type="checkbox" value="8" id="eight" name="number[]">
    </div>
  </fieldset>
</form>

//To get a comma-separated list of checkbox IDs:
$( ":checkbox" )
  .map(function() {
    return this.id;
  })
  .get()
  .join();
//The result of this call is the string, "two,four,six,eight".

//Within the callback function, this refers to the current DOM element for each iteration. The function can return an individual data item or an array of data items to be inserted into the resulting set. If an array is returned, the elements inside the array are inserted into the set. If the function returns null or undefined, no element will be inserted.
```

##### $.inArray()

```javascript
//$.inArray(value,arrary)
//确定第一个参数在数组中的位置, 从0开始计数，如果没有找到则返回 -1
$.inArray( 5 + 5, [ "8", "9", "10", 10 + "" ] );
//indexOf()返回字符串的首次出现位置
```

##### $.merge()

```javascript
//$.merge(firstArray,secondArray)
//返回的结果会修改第一个数组的内容
//Merges two arrays, altering the first argument.
$.merge( [ 0, 1, 2 ], [ 2, 3, 4 ] )
//Result: [ 0, 1, 2, 2, 3, 4 ]
```

##### $.grep()

```javascript
//$.grep(arrary,function(value,index){return boolean;},[invert])
//$.grep() 函数使用指定的函数过滤数组中的元素，并返回过滤后的数组。
//提示：源数组不会受到影响，过滤结果只反映在返回的结果数组中。
//数组元素过滤，返回true 保留，返回false 删除；
//第三个参数可选，默认为true，如果为false 表示结果取反
//Filter an array of numbers to include only numbers bigger then zero.
$.grep( [ 0, 1, 2 ], function( n, i ) {
  return n > 0;
});
//Result: [ 1, 2 ]
```

##### $.unique()

```javascript
//$.unique(arrary)
//过滤Jquery对象数组中重复的元素(内部实现为===)(不同版本不一样，不建议使用)
//只处理删除DOM元素数组,而不能处理字符串或者数字数组.
```

#####  $.makeArray()

```javascript
//$.makeArray($(selector))
//将类数组对象转换为数组对象, 类数组对象有 length 属性,其成员索引为0至 length-1
//比如用getElementsByTagName获取的元素对象集合转换成数组对象
```

##### $(selector).toArray() 

```javascript
//把jQuery集合中所有DOM元素恢复成一个数组
//.toArray() returns all of the elements in the jQuery set:
alert( $( "li" ).toArray() );
//All of the matched DOM nodes are returned by this call, contained in a standard array:
// [<li id="foo">, <li id="bar">]
```

 

### JSON 处理

##### 1. 返回JSON数组 长度

```javascript
function getJsonLength(jsonData){
    var jsonLength = 0;
    for(var item in jsonData){
        jsonLength++;
    };
    return jsonLength;
};
```

##### 2. JSON 数组去重

```javascript
function uniqueArray(array, key){
    var result = [array[0]];
    for(var i = 1; i < array.length; i++){
        var item = array[i];
        var repeat = false;
        for (var j = 0; j < result.length; j++) {
            if (item[key] == result[j][key]) {
                repeat = true;
                break;
            }
        }
        if (!repeat) {
            result.push(item);
        }
    }
    return result;
}
```



 
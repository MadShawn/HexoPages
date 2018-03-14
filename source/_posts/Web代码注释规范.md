---
title: Web代码注释规范
date: 2018-03-14 15:22:35
tags: 编码规范
---
# 前言
为了提升代码的可读性和可维护性，同时为了提高代码总体的注释率，特制定此注释规范。本规范只针对Javascript文件，以支持ES6语法的EsDoc规范为基准，结合我们代码的现状，以及公司的相关要求，综合制定。主要包含以下几个方面的规定：
> **总的原则**：  
>+ 除类文件之外的所有文件必须有文件头注释
>+ 所有类文件必须有类注释
>+ 所有的全局变量必须有注释
>+ 所有的函数必须有注释

+ 普通脚本文件
    + 文件头
    + 全局变量
    + 函数
+ 类文件
    + 类注释
    + 成员变量
    + 成员函数
+ 组件脚本文件
    + 组件
    + 内部变量
    + 成员变量
    + 内部函数
    + 成员函数
+ 其他规定
    + 重要的公共函数
    + 复杂逻辑代码
    + 特殊处理代码
    + 特殊需求代码
    + 被注释的代码

# 普通脚本文件
普通脚本文件是指无明确的组织结构，只是变量和函数组合的文件，比如WebApp项目中定义全局变量的Application.js、定义全局公共函数对象的Util.js等。注释规范如下：

## 文件头
**重要度：** 必须有  
在文件最开始插入，描述文件的目的/内容/功能等，采用多行注释，如下Application.js文件头注释：
```javascript
/**
 * 定义web app全局命名空间，并在此空间下定义属性和函数，以便全局使用
 */
```
## 全局变量
**重要度：** 必须有  
全局变量指的是声明在window对象下的变量。如下Application.js文件中定义的全局变量的注释：
> 在使用了angular、vue、react等组件化的前端框架的单页面web app中，此种意义上的全局变量应该很少
```javascript
/**
 * App全局命名空间
 * @type {Object}
 */
var App = {};

/**
 * 全局配置信息
 * @type {Object}
 * @property {Object} ml 大陆环境配置
 * @property {Object} hm 港澳环境配置
 */
App.serverList = {
    ml: {
        /* trunk服务 */
        serviceUrl: 'http://www.xxxx.com/service',
        /* trunk地图服务 */
        subdomainsServiceUrl: 'http://www.xxxx.com/service'
    },
    hm: {
        /* 港澳服务 */
        serviceUrl: 'http://www.xxxx.com/hm/service',
        /* 港澳地图服务 */
        subdomainsServiceUrl: 'http://www.xxxx.com/hm/service'
    }
};
```
## 函数
**重要度：** 必须有 
无论是定义在脚本文件的全局函数，还是定义在类、组件、对象中的内部函数，都需要加完整的注释，注释规范如下：
```javascript
/**
 * 从url地址中获取拼接的参数值
 * @method  getUrlParam
 * @author  ChenXiao
 * @date    2017-09-13
 * @param   {String}    paramName 参数名称
 * @return  {String}    参数值
 */
getUrlParam: function (paramName) {
    var reg = new RegExp('(^|&)' + paramName + '=([^&]*)(&|$)');
    var str = window.location.search;
    var ret;
    if (!str) {
        str = window.location.hash;
    }
    if (str) {
        ret = str.substr(str.indexOf('?') + 1).match(reg);
    }
    if (ret) {
        return unescape(ret[2]);
    }
    return null;
},
```
# 类文件
类文件是指使用es6的class语法定义的文件，一般一个类的定义是一个单独的文件，因此类文件可以没有文件头注释，以类注释替代。
## 类注释
**重要度：** 必须有 
类注释与文件头注释类似，采用多行注释形式。类注释要放在import语句之后，类定义之前，且必须紧贴类定义，不能有空行。例子如下：
```javascript
import Singleton from './Singleton';

/**
 * 对象控制器类的基类
 */
export default class Controller extends Singleton {
    /**
     * 构造函数
     * @param  {object} options 可选项
     * @return {undefined}
     */
    constructor(options) {
        super(options);
    }
    /**
     * 清空
     * @return {undefined}
     */
    clear() {
    }
    /**
     * 销毁单例
     * @return {undefined}
     */
    destroy() {
        this.clear();
        super.destroy();
    }
}
```
## 成员变量
**重要度：** 原则上都要有注释，尤其重要类、类的重要变量必须有  
成员变量分为普通成员变量和静态成员变量。普通成员指的是在contructor函数中定义在this下的变量；静态变量是以`static`关键词修饰的变量，注释的规范一样，如下边两个例子中的`this._kvMap`和`instance`变量：
+ 普通成员变量
```javascript
import Singleton from './Singleton';

/**
 * 工厂类的基类
 */
export default class Factory extends Singleton {
    /**
     * 构造函数
     * @param  {object}     options 其他参数
     * @return {undefined}
     */
    constructor(options) {
        super(options);

        /**
         * 配置表
         * @type {Object}
         */
        this._kvMap = {};
    }
    /**
     * 获取配置
     * @return {object} 键值对对象
     */
    getConfig() {
        return this._kvMap;
    }
```
+ 静态成员变量
```javascript
/**
 * 单例对象的实例
 * @type {class}
 */
static instance = null;
/**
 * 获取单例对象的实例
 * @param  {...all} rest 参数
 * @return {class}  单例对象的实例
 */
static getInstance(...rest) {
    if (!this.instance) {
        this.instance = new this(...rest);
    }
    return this.instance;
}
```
## 成员函数
**重要度：** 必须有  
成员函数也分为普通成员函数和静态成员函数。普通成员指的是在contructor函数中定义在this下的变量；静态变量是以`static`关键词修饰的变量，注释的规范一样，如下边两个例子中的`getConfig`和`getInstance`函数：
+ 普通成员函数
```javascript
import Singleton from './Singleton';

/**
 * 工厂类的基类
 */
export default class Factory extends Singleton {
    /**
     * 构造函数
     * @param  {object}     options 其他参数
     * @return {undefined}
     */
    constructor(options) {
        super(options);

        /**
         * 配置表
         * @type {Object}
         */
        this._kvMap = {};
    }
    /**
     * 获取配置
     * @return {object} 键值对对象
     */
    getConfig() {
        return this._kvMap;
    }
```
+ 静态成员函数
```javascript
/**
 * 单例对象的实例
 * @type {class}
 */
static instance = null;
/**
 * 获取单例对象的实例
 * @param  {...all} rest 参数
 * @return {class}  单例对象的实例
 */
static getInstance(...rest) {
    if (!this.instance) {
        this.instance = new this(...rest);
    }
    return this.instance;
}
```
# 组件脚本文件
组件脚本文件是指采用了angular、vue等前端框架之后，开发的各个组件对应的javascript脚本文件，如angular的module、controller、service等组件，vue的component文件等。
下面是一个比较全的例子：
```javascript
/**
 * 文件头注释
 * 地图工具面板组件的控制器
 */

angular.module('app').controller('MapAllToolbarPanelCtrl', ['$scope', '$rootScope', '$compile', '$timeout',
    function ($scope, $rootScope, $compile, $timeout) {
        // var layerCtrl = fastmap.uikit.LayerController();
        /**
         * 内部变量
         * 形状编辑控制
         * @type {object}
         */
        var shapeCtrl = fastmap.uikit.ShapeEditorController();

        /**
         * 成员变量
         * 控制是否可编辑
         * @type {Boolean}
         */
        $scope.editable = false;

        /**
         * 内部函数
         * 切换可编辑状态
         * @param  {object}   event 页面事件
         * @param  {object}   data  数据
         * @return {undefined}
         */
        var toggleEditable = function (event, data) {
            $scope.editable = data.editable;
        };

        /**
         * 成员函数
         * 讲工具从常用工具栏移入/移出
         * @param  {object}   e 页面事件
         * @return {undefined}
         */
        $scope.toggleToRecent = function (e) {
            if ($scope.editable) {
                var elem = angular.element(e.currentTarget);
                var p = elem.parent();
                if (p.hasClass('selected')) {
                    $scope.$emit('MapToolbar-removeTool', {
                        ngController: p.attr('ng-controller'),
                        ngClick: p.attr('ng-click')
                    });
                    p.removeClass('selected');
                } else {
                    $scope.$emit('MapToolbar-addTool', {
                        title: p.attr('title'),
                        ngController: p.attr('ng-controller'),
                        ngClick: p.attr('ng-click'),
                        ngClass: p.attr('ng-class'),
                        icon: elem.html()
                    });
                    p.addClass('selected');
                }
                e.stopPropagation();
            }
        };
```
## 文件头
**重要度：** 必须有  
见上例。
> 文件头注释与代码之间要加一个空行，防止esdoc将其认为是一个函数注释

## 内部变量
**重要度：** 重要属性要有
内部变量是指组件内的全局变量，对组件内的所有函数可见。注释规范同变量，见上例。
## 成员变量
**重要度：** 重要变量要有
成员变量是指定义在组件特定命名空间下的变量，如上例中的`$scope`中的变量。注释规范同变量，见上例。
## 内部函数
**重要度：** 必须有
内部函数是指组件内的全局函数，对组件内的其他所有函数可见。注释规范同函数，见上例。
## 成员函数
**重要度：** 必须有
成员函数是指定义在组件特定命名空间下的函数，如上例中的`$scope`中的函数。注释规范同函数，见上例。
# 其他规定
## 重要的公共函数
**重要度：** 必须有  
+ 基类中被多个子类调用的
+ 常用的函数  

此类函数要加上详细的函数说明，并要对函数参数进行详细的说明，如值域、对象的固有属性等。
## 复杂逻辑代码
**重要度：** 必须有  
+ 判断条件超过3个的  
+ 循环/判断超过2层嵌套的

## 特殊处理代码
**重要度：** 必须有
+ 循环中的特殊判断跳出
+ 变量的特殊赋值
```javascript
// add by chenx on 2017-4-27
// 解决切换任务后，地图不能正常加载的问题
$timeout(function () {
    $scope.$broadcast('Map-Initialize');
}, 100);
```
## 特殊需求代码
**重要度：** 必须有
+ 用户提出的一些特殊处理需求

## 被注释的代码
**重要度：** 必须有
如果是由于某些情况临时注释，后续有可能有用的代码，则必须加注释信息；

# 需求/Bug/生产问题的注释
对应需求、bug和生产问题时修改的代码，如果是符合**其他规定**中的内容的，需要加上对应的注释。
正常的任务开发和bug修改只需要在提交代码时，在commit信息里写清楚具体的需求、任务、bug编号即可，以便在github上追溯。

# esdoc生成器
+ npm安装
```command
npm install esdoc
npm install esdoc-standard-plugin
```
+ esdoc使用
```command
./node_modules/.bin/esdoc
```
参考[esdoc官网](https://esdoc.org/)
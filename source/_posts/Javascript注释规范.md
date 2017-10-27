---
title: Javascript注释规范
date: 2017-09-11 14:04:49
tags: 编码规范
---
# 前言
为了规范工作中的代码注释，使得代码注释符合JsDoC规范，从而可以自动生成Api文档，特制定本规范。本规范主要包含以下几个方面的规定：
+ 文件注释
+ 类注释
+ 函数注释
+ 公共变量的注释

# 文件注释 （必须有）
文件注释主要用于对无明确的类结构的文件的注释，比如只有一个大对象定义的文件（如Application.js）、多个全局函数定义的文件（util.js）、Angular的各种对象文件（route、controller、directive、service等）。注释规范如下：
> 文件头注释中，要用 @file

+ 常规文件

        /**
         * 定义web app全局命名空间，并在此空间下定义属性和函数，以便全局使用
         * @file      Application.js
         * @author    ChenXiao
         * @date      2017-09-11
         *
         * @copyright @Navinfo, all rights reserved.
         */

        var App = {};
        // web app全局配置信息
        App.Config = {
            appName: 'FM-WebEditor',

+ Angular controller

        /**
         * 编辑页面的主controller
         * @file editorCtrl.js
         * @author ChenXiao
         * @date   2017-09-11
         *
         * @copyright @Navinfo, all rights reserved.
         */

        angular.module('app').controller('editorCtrl', ['$scope', '$rootScope', '$cookies', '$timeout', '$q',
            '$ocLazyLoad', 'ngDialog',
            'appPath', 'dsMeta', 'dsFcc', 'dsEdit', 'dsManage', 'dsColumn', 'dsLazyload',
            function ($scope, $rootScope, $cookies, $timeout, $q, $ocLazyLoad, ngDialog,
                appPath, dsMeta, dsFcc, dsEdit, dsManage, dsColumn, dsLazyload) {
                if (!$scope.testLogin()) {
                    return;
                }

    {%note danger%}注释与controller之间必须要有个空行，否则jsdoc会把controller当成函数注释，导致jsdoc报错。{%endnote%}

> 建议所有的文件头注释与代码之间要加一个空行。

# 类注释 （必须有，可以替代文件注释）
类注释主要用于符合类结构的文件的注释，比如通过L.Class创建的模拟类的文件（mapRender、dataApi、uikit下的文件）。由于代码中基本上达到了一个类一个文件的结构，因此可以用类注释替代文件注释。注释的规范如下：
> 类注释中，要用 @class

    /**
     * 交限的前端数据模型
     * @class FM.dataApi.RdRestrictionDetail
     * @author ChenXiao
     * @date   2017-09-11
     *
     * @copyright @Navinfo, all rights reserved.
     */
    FM.dataApi.RdRestrictionDetail = FM.dataApi.Feature.extend({
        /**
         * 模型转换主函数，将接口返回的数据转换为前端数据模型
         * @method setAttributes
         * @author ChenXiao
         * @date   2017-09-11
         * @param  {object} data 接口返回的数据
         * @return {undefined}
         */
        setAttributes: function (data) {
            this.geoLiveType = 'RDRESTRICTIONDETAIL';
            this.pid = data.pid || 0;
            this.restricPid = data.restricPid || 0;
            this.outLinkPid = data.outLinkPid || 0;
# 函数注释 （必须有）
所有的函数都必须加此注释，要完全符合jsdoc规范。后续会把eslint打开，不符合要求的不予合并。注释规范如下：
+ 一般情况

        /**
         * 根据的窗口的选项，创建弹出窗口对象
         * @method createDialog
         * @author ChenXiao
         * @date   2017-09-11
         * @param  {object} data 窗口选项，主要为要显示的信息类型
         * @return {object} 包含窗口标题、页面片段的信息的窗口对象
         */
        var createDialog = function (data) {
            var item = {};
            var tmplFile = FM.uikit.Config.getUtilityTemplate(data.type);
            item.title = FM.uikit.Config.getUtilityName(data.type);
            item.ctrl = appPath.scripts + tmplFile.ctrl;
            item.tmpl = appPath.scripts + tmplFile.tmpl;
            item.options = FM.Util.clone(defaultDialogOptions);
            getDlgOptions(data.type, item.options);

            return item;
        };

+ 无返回值的情况

        /**
         * 模型转换主函数，将接口返回的数据转换为前端数据模型
         * @method setAttributes
         * @author ChenXiao
         * @date   2017-09-11
         * @param  {object} data 接口返回的数据
         * @return {undefined}
         */
        setAttributes: function (data) {
            this.geoLiveType = 'RDRESTRICTIONDETAIL';
            this.pid = data.pid || 0;
            this.restricPid = data.restricPid || 0;
            this.outLinkPid = data.outLinkPid || 0;
    
    > 函数注释中，jsdoc要求必须有@return，因此无返回值时, return的类型要写为undefined。

+ 无参数有返回值

        /**
         * 将前端数据模型还原为接口数据模型
         * @method getIntegrate
         * @author ChenXiao
         * @date   2017-09-11
         * @return {Object} 接口数据模型
         */
         getIntegrate: function () {
             var data = {};
             data.pid = this.pid;
# 公共变量的注释
公共变量我们分为三种，一种是全局变量（window下的），一种是类中的全局变量（如controller中），这两种必须加注释，第三种就是除了前两种之外的其他情况（比如一个函数中定义了多个内部函数，共享一变量），这种加不加注释可以视情况而定。注释规范如下：
+ 单行注释

        // 右侧面板是否开启标识
        $scope.rightPanelFlag = false;
        // 右侧浮动面板是否开启标识
        $scope.rightFloatPanelFlag = false;
        // 所有其他的右侧浮动面板
        $scope.mapToolbarPanelFlag = false;
        $scope.clmPanelOpened = false;

+ 多行注释

        /* 右侧面板总控标识, 右侧面板对于页面的布局很重要，
        加一个总控有可能在子页面种打开/关闭右侧面板 */
        $scope.rightPanelOpened = false;

        // 将页面loading动画的开关引用赋给dsEdit的本地变量，以便在dsEdit中进行控制
        // 注意：这里利用了对象引用的特性，变量必须是个对象，不能是字符串、bool、数字等
        $scope.loading = {
            flag: false
        };

# IDE中安装jsdoc辅助工具
+ sublime
安装docBlockr
+ webstorm
请补充
+ vscode
请补充

# jsdoc生成器
+ npm安装
        npm install -g jsdoc
参考[npm jsdoc](https://www.npmjs.com/package/jsdoc)
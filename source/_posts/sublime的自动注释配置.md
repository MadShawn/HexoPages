---
title: sublime的自动注释配置
date: 2017-09-18 15:14:43
tags: 工具使用
---

## 前言
为了在sublime编辑器中能通过快捷键自动为js文件、类、函数等添加符合jsdoc规范的注释，需要安装的相关的插件，并进行自定义的配置。现将安装和配置的过程记录如下，以备不时之需。

## 文件头注释
由于sublime的DocBlockr插件只支持对函数、变量的快捷注释，不能增加文件头注释，因此需要安装File header插件。
### 安装插件
sumlime安装插件的方式：
>1. ctrl+shift+p组合键，或 prefences > package control 者打开包管理窗口
>2. 输入 install package打开包安装窗口
>3. 输入 file header进行包搜索
>4. 选中fileheader插件启动安装

### 修改配置
1. 在sublime菜单栏，点击 prefences > browse packages 打开包目录
2. 打开FileHeader > template > header 文件夹
3. 找到javascript.tmpl文件，添加如下内容：

       /**
        * [Description]
        * @file       {{file_name}}
        * @author     xxxx
        * @date       {{create_time}}
        *
        * @copyright: @Navinfo, all rights reserved.
        */

    > 注意：模版最后要加入三个空行，这样在js文件中插入此注释后才会加入一个空行

### 插件的使用
可以在sublime的菜单 prefences > package settings > File Header > key bingding-Default中，修改该插件的快捷键。你当然也可以直接用默认的。  
这里我们将command 'file_header_add_header' 的快捷键修改为 'ctrl+alt+h'。  
这样，在打开一个js文件后，按'ctrl+alt+h'组合键会打开为当前文件添加文件头注释的窗口，按enter即可完成文件头注释的插入。

## 函数注释
### 安装插件
安装sublime的DocBlockr插件，安装方法参考上一节
### 修改配置
在sumlime菜单栏，打开 prefences > package settings > DocBlockr > settings-user，打开用户配置文件，增加以下配置：

    {
        "jsdocs_extra_tags": ["@author XXXX", "@date {{date}}"],
        "jsdocs_notation_map": [{
            "prefix": "_",
            "tags": ["@private"]
        }],
        "jsdocs_autoadd_method_tag": true
    }
> 配置的详细说明可参考README文件：prefences > package settings > DocBlockr > README

### 插件的使用
在需要增加注释的函数/变量定义的上一行，输入 `/**` ，然后按 `Enter` 键，即可添加默认的注释信息。  
需要手工补充*函数*、*参数*和*返回值*的数**据类型**和**描述**。

## 批量增加函数注释
smartcomments是一个nodejs写的给文件的所有函数增加注释的工具，详细的说明可参考 [smartcomment主页](http://smartcomments.github.io/) 和 [github地址](https://github.com/smartcomments/smartcomments)

### 安装
    npm install -g smartcomments
### 自定义配置
默认的配置只支持4个@tag，不满足我们的注释需求，因此需要自定义。可以在需要批量加注释的项目下新建两个文件，smartcomments.json和templates.js，这里就不再做过多说明，直接提出代码：
* smartcomments.json

        {
            "target_dir": [
                ""
            ],
            "match_files": [
                "^((?!~).)*.(js)$"
            ],
            "backup": false,
            "private": false,
            "favor_generated": false,
            "tags": {
                "function": {
                    "name": {},
                    "desc": {
                        // 函数的默认描述
                        "value": "[Auto generated description]"
                    },
                    "author": {
                        // 要指定的作者名字
                        "value": "XXXX"
                    },
                    "date": {},
                    "params": {},
                    "rtrn": {}
                }
            },
            // 这里指向自定义的模版文件，要给出绝对地址，否则找不到
            "template": "E:\\xxxx\\template.js"
        }

* template.js    
    可以从smartcomments的[github库](https://github.com/smartcomments/smartcomments/blob/master/templates/default.js)中看到默认的default.js，拷贝一份命名为template.js，然后修改其中的 buildComment 函数即可，修改后的代码如下：

        buildComment: function (data) {
            var instance = templateInstance,                  // BaseTemplate instance
                config = instance.config,                      // User current config
                options = config.tags.function,      // Custom functions tags options
                pos,                                           // Empty comment obj
                comment = {
                    pos: data.pos,
                    tags: []
                },
                methodName = data.name,
                node = data.node;

            if (typeof (config.private) === 'undefined'
                || config.private
                || !/^_/.test(methodName)) {
                // Description statement
                if (options.desc) {
                    comment.tags.push({
                        name: '',
                        value: options.desc.value || 'Function description'
                    });
                }

                // @method statement
                if (methodName && options.name) {
                    comment.tags.push({
                        name: '@method',
                        value: methodName
                    });
                }

                // 增加作者tag
                // Author statement
                if (options.author) {
                    comment.tags.push({
                        name: '@author',
                        value: options.author.value || 'XXX'
                    });
                }

                // 增加日期tag
                // Date statement
                if (options.date) {
                    comment.tags.push({
                        name: '@date',
                        value: '  ' + (options.date.value || new Date().toLocaleDateString())
                    });
                }

                var size,
                    i = 0,
                    value;

                // @param statement
                if (options.params) {
                    var array = node.params;
                    size = array.length;
                    for (i; i < size; i++) {
                        value = ' {[type]} ' + array[i].name + ' [description]';
                        comment.tags.push({
                            name: '@param',
                            value: value
                        });
                    }
                }

                // @return statement
                if (options.rtrn) {
                    var elements = node.body.body;
                    if (elements) {
                        size = elements.length;
                        value = '';
                        for (i = 0; i < size; i++) {
                            if (elements[i].type === 'ReturnStatement') {
                                if (elements[i].argument) {
                                    value = '{[type]} [description]';
                                    break;
                                }
                            }
                        }

                        if (!value) {
                            value = '{Undefined} 无返回值';
                        }
                        comment.tags.push({
                            name: '@return',
                            value: value
                        });
                    }
                }

                if (comment.pos >= 0) {
                    // Add comment to comment_list
                    instance.comments_list.push(comment);
                }
            }
        },

### 修改smartcomments源代码
最重要的部分来了。不得不吐槽一下smartcomments的源代码，真的代码烂烂，bug多多。。。  
由于smartcomments是用npm全局安装的，这里就需要修改全局包中的代码。查看npm全局包安装目录的命令：
    
    npm config get prefix

然后修改[prefix]\node_modules\smartcomments\lib\smartcomments.js文件。修改的内容比较多，就不一一说明了，主要修改了4个函数，直接上代码：

> 修改之前备份一下是个好习惯

1. mergeSrcAndGeneratedComment

        mergeSrcAndGeneratedComment: function (instance, generatedComment, favorGenerated) {
            var srcComment = generatedComment.srcComment,
                mergedCommentLines = [];

            // process each src comment line, and remove matching comments from generatedComment
            var srcCommentTags = _.map(srcComment.value.split(/\r?\n/), function (commentLine) {
                return instance.parseTagNameValue(commentLine);
            });

            _.each(generatedComment.tags, function (tag) {
                if (favorGenerated) {
                    mergedCommentLines.push(tag);
                } else {
                    var srcTag;
                    var f = false;
                    var desc = [];
                    for (var i = 0; i < srcCommentTags.length; i++) {
                        srcTag = srcCommentTags[i];
                        if (tag.name === '' && (srcTag.name === '' || srcTag.name === '@description')) {
                            desc.push(srcTag.value);
                            f = true;
                        } else if (srcTag.name === tag.name) {
                            if (tag.name === '@param') {
                                if (instance.isSameParam(srcTag.value, tag.value)) {
                                    f = true;
                                    break;
                                }
                            } else {
                                f = true;
                                break;
                            }
                        }
                    }
                    if (!f) {
                        mergedCommentLines.push(tag);
                    } else {
                        if (tag.name === '') {
                            tag.value = desc.join('');
                            mergedCommentLines.push(tag);
                        } else {
                            mergedCommentLines.push(srcTag);
                        }
                    }
                }
            });

            instance.mergeOtherSrcTags(srcCommentTags, mergedCommentLines);
            generatedComment.tags = mergedCommentLines;
        },

2. parseTagNameValue

        parseTagNameValue: function (commentLine) {
            var clean = commentLine.replace(/^\s*\*?\s?/, ''); // strip leading ' * '
            var nameValueMatch = /^(?:(@\w+))?([^\r\n]*)/.exec(clean);

            return {
                name: nameValueMatch[1] || '',
                value: nameValueMatch[2] || '',
            };
        },

3. isSameParam

        isSameParam: function (srcValue, genValue) {
            var stripTypeRe = /\S*\s*\{\[?[\w:-]*\]?\}\s*/,
                matchValRe = /^[\w:-]+/;
            srcValue = srcValue.replace(stripTypeRe, '');
            genValue = genValue.replace(stripTypeRe, '');
            srcValue = matchValRe.exec(srcValue);
            genValue = matchValRe.exec(genValue);
            if (srcValue && genValue) {
                srcValue = srcValue[0];
                genValue = genValue[0];

                if (srcValue) {
                    return (genValue && srcValue === genValue);
                } else {
                    return !genValue;
                }
            }

            return false;
        },

4. mergeOtherSrcTags

        mergeOtherSrcTags: function (srcCommentTags, mergedCommentLines) {
            var k;
            for (k = 0; k < mergedCommentLines.length && mergedCommentLines[k].name !== '@method'; k++);

            var temp = mergedCommentLines.slice();

            _.each(srcCommentTags, function (srcTag) {
                var f = false;
                for (var i = 0; i < temp.length; i++) {
                    if (srcTag.name === temp[i].name || (srcTag.name === '@description' && temp[i].name === '')) {
                        f = true;
                        break;
                    }
                }
                if (!f) {
                    mergedCommentLines.splice(k++, 0, srcTag);
                }
            });
        },

### smartcomments的使用
1. 使用默认配置

        smartcomments --generate

    > 注意：会对执行此命令的目录下所有js文件增加函数注释，会递归遍历所有子目录
2. 使用自定义配置

        smartcomments --generate -c ./comments/smartcomments.json

    > 如果把自定义配置文件放置在执行此命令的目录下，并命名为smartcomments.json时，则可以不用指定-c参数，程序会自动识别；如果在配置文件在其他目录，则需要指定
3. 指定要增加注释的文件/目录

        smartcomments --generate -c ./comments/smartcomments.json -t ./apps/editorCtrl.js

    > 可以指定文件，也可以指定目录


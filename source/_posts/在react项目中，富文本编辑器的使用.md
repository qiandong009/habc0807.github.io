---
title: 在react项目中，富文本编辑器的使用
date: 2017-02-05 22:52:54
tags:
---

说到富文本编辑器那可是一箩筐，数不胜数。国内比较知名的百度编辑器首当其冲。外国吗？Simditor很比错，功能精简、加载速度快速，但是没有react版的，要想使用就手动封装吧。对于react的富文本编辑器还没一个定论，我是说还没有一个纯react封装的富文本编辑器被大家广为使用，也许是坑太大了，没人能很好的填上。

<!--more-->

### react 编辑器的选择

说到富文本编辑器那可是一箩筐，数不胜数。国内比较知名的百度编辑器首当其冲。外国吗？Simditor很比错，功能精简、加载速度快速，但是没有react版的，要想使用就手动封装吧。对于react的富文本编辑器还没一个定论，我是说还没有一个纯react封装的富文本编辑器被大家广为使用，也许是坑太大了，没人能很好的填上。

也不是没有人填，facebook就出一款react富文本编辑器叫**[draft-js](https://github.com/facebook/draft-js)**，可以说它的界面精简至极，缺点是没有继承上传图片这等功能，对于这种要编辑长文的编辑器然要有解决办法，**[medium-draft](http://bitwiser.in/medium-draft/)**集成上传图片，添加了更加丰富的功能，不过它的设计和交互，在国内用户习惯和思维还没那么前卫的情况，尽量不要选用。或者你能力够强，去码源码。

根据项目需求我们希望使用百度编辑器那样丰富的功能和简洁的设计，在github找到了**[react-umeditor](https://github.com/liuhong1happy/react-umeditor)**，这是**[liuhong1happy](https://github.com/liuhong1happy/react-umeditor)**同学将百度富文本编辑器用react封装的一个组件。查看了react-umeditor的demo，对该编辑器多方面评估，满足我们的项目需求，接下来就是在项目中尝试使用它了。


### 使用react-umeditor富文本编辑器

###### Install 
```
npm install react-umeditor --save
```

###### Use
javascript
```
import React from 'react'
import ReactDOM from 'react-dom'
import Editor from 'react-umeditor'

class App extends React.Component {
	constructor(props){
		super(props);
		this.state = {
			content: ""
		}
	}
	handleChange(content){
		this.setState({
			content: content
		})
	}
	getIcons(){
		var icons = [
			"source | undo redo | bold italic underline strikethrough fontborder emphasis | ",
			"paragraph fontfamily fontsize | superscript subscript | ",
			"forecolor backcolor | removeformat | insertorderedlist insertunorderedlist | selectall | ",
			"cleardoc  | indent outdent | justifyleft justifycenter justifyright | touppercase tolowercase | ",
			"horizontal date time  | image emotion spechars | inserttable"
		]
		return icons;
	}
	getQiniuUploader(){
		return {
			url:'http://upload.qiniu.com',
			type:'qiniu',
			name:"file",
			request: "image_src",
			qiniu:{
				app:{
					Bucket:"liuhong1happy",
					AK:"l9vEBNTqrz7H03S-SC0qxNWmf0K8amqP6MeYHNni",
					SK:"eizTTxuA0Kq1YSe2SRdOexJ-tjwGpRnzztsSrLKj"
				},
                domain:"http://o9sa2vijj.bkt.clouddn.com",
                genKey:function(options){
                    return options.file.type +"-"+ options.file.size +"-"+ options.file.lastModifiedDate.valueOf() +"-"+ new Date().valueOf()+"-"+options.file.name;
                }
			}
		}
	}
	render(){
		var icons = this.getIcons();
		var uploader = this.getQiniuUploader();
		var plugins = {
			image:{
				uploader:uploader
			}
		}
		var count = 100;
		var editors = [];
		for(var i=0;i<count;i++){
			editors.push({
				icons:icons,
				plugins:plugins
			})
		}
		return (<Editor ref="editor"
                                icons={icons}
                                value={this.state.content}
                                defaultValue="<p>提示文本</p>"
                                onChange={this.handleChange.bind(this)}
                                plugins={plugins} />)
	}
}
```
html

``` html
<!doctype html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <title>首页</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta http-equiv="Cache-Control" content="no-siteapp">
    <!-- 引入富文本编辑器的样式 -->
    <link rel="stylesheet" href="../../dist/third-part/mathquill/mathquill.css"/>
    <link rel="stylesheet" href="app.css" type="text/css" />
    <!-- 引入富文本编辑器的样式 -->
</head>
<body>
    <div id="root"></div>
    <!-- 引入富文本编辑器的js -->
    <script src="../../dist/third-part/jquery.min.js"></script>
    <script src="../../dist/third-part/mathquill/mathquill.js"></script>
    <!-- 引入富文本编辑器的js -->
    <script src='assets/dist/js/lib.js'></script>
    <script src='assets/dist/js/index.js'></script>
</body>
</html>
```
images(图标icons) 
```
需要把images和third-part文件夹放到项目里
```
package.json(配置)
```    
"copy": "cp -R images/. dist/images && cp -R less/. dist/less && cp -R third-part/. dist/third-part",
```

### 最终效果

![富文本编辑器.jpeg](http://upload-images.jianshu.io/upload_images/202755-2aebde24ccdbeacd.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###小结
期间对多个富文本编辑器在项目调试，最终选择了liuhong的react-umeditor，在组件使用过程中出现了很多问题。像图标引入不到项目里、需要修改配置等问题，最后都一一解决了。最后提示自己只看没用，一定要多实践，实践出真理，实践出真理，实践出真理，重要的事情说三遍。
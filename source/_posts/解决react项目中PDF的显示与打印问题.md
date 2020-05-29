---
title: 解决react项目中PDF的显示与打印问题
date: 2017-02-05 22:52:54
tags:
---

拿到这个需求，真时一头雾水。因为没有做过类似需求，不知从何下手。在查阅资料的过程中，发现有很多jQuery插件可以实现显示pdf, 但是我们是react单页面应用项目，看来这些插件并不适用，只能另寻其它方法。


<!--more-->

最近项目中有这样一个需求：
1. 页面中可以显示pdf
2. 不希望把整个页面打印下来，只打印显示PDF的部分，可以使用浏览器自带打印功能

---

## PDF文件的显示
拿到这个需求，真时一头雾水。因为没有做过类似需求，不知从何下手。在查阅资料的过程中，发现有很多jQuery插件可以实现显示pdf, 但是我们是react单页面应用项目，看来这些插件并不适用，只能另寻其它方法。

后来在 [npmjs.com](https://www.npmjs.com/) 上找到了 [react-pdf-js](https://www.npmjs.com/package/react-pdf-js) 组件, 心想显示pdf有望。就迫不及待将 react-pdf-js 依赖 通过 cnpm install react-pdf-js --save-dev 命令安装到项目中，通过 import PDF from 'react-pdf-js' 引入到项目里。将<PDF file={pdfUrl} onDocumentComplete={this.onDocumentComplete} onPageComplete={this.state.page} />插入render里。

在调试过程中发现静态pdf文件可以显示，在线pdf文件不能显示。通过控制的报错信息了解道，react-pdf-js组件要求file文件地址是url或者base64格式, 既然url行不通，就只能往base64上靠了。

##### 获取PDF文件

一开始我直接将将pdf的在线地址url转换为base64，但是不能显示。后来想明白了，只把url转换成base64格式是没有用的，需要把pdf的文件内容转换成base64才行。接下来就顺理成章，通过从后台获取到的pdf的url地址，再次请求获取到pdf文件。

在做这部分的遇到一个小问题：能请求成功，就是获取不到pdf文件，在这纠结了很久，也不知道该如何解决，把问题描述给我们公司的架构师，我们分析这是跨域问题造成的，他给nginx服务器的配置解决了跨域问题。

    'Content-Type':  'application/x-www-form-urlencoded;charset=utf-8',
    'Access-Control-Allow-Origin': 'https://www.nurse-go.cn:9091',
    'Access-Control-Allow-Methods': 'GET',
    'Access-Control-Allow-Headers': 'X-Custom-Header',
    'Access-Control-Allow-Credentials': true,

##### 将PDF文件转换为base64格式

>base64可以存储任何格式的文件，一种的很棒的存储方法。[将文件转换为base64格式](https://developer.mozilla.org/en-US/docs/Web/API/FileReader/readAsDataURL)。

这里需要注意请求pdf文件的时候要设置[responseType](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseType)为blob,  为什么使用blob类型下面解释，到这我就拿到了pdf文件，将其转化为base64格式。

    let reader  = new FileReader()
    let fileParts = []
    fileParts.push(this.props.pdfFile)
    let blob = new Blob(fileParts, {type : 'application/pdf'})
    if (blob) {
        reader.readAsDataURL(blob)
    }
    let base64
    let that = this      // 处理this的指向  
    reader.addEventListener("load", function () {
        base64 = reader.result
       that.setState({
              base64: base64,
     })
    }, false);

base64格式的转换，需要时blob格式，将转化为base64格式的pdf，在file={file}, 将其在浏览器上显示出来。实际上最终是以canvas来呈现的PDF。

pdf显示算是告一段落，接下来就是打印了。

---

## PDF文件的打印

在浏览器上，打印分整页打印和指定部分打印。项目需要打印制定部分内容打印，实现打印的方法多种多样，我使用了传统的css控制。通过[@media print](http://www.w3school.com.cn/tags/att_style_media.asp)将打印时不需要打印的部分隐藏掉，那么剩下的就是要打印的部分了。

##### PDF文件打印调试

这里有个调试的小技巧：因为只有当调用了浏览器的打印才会调用@media print 里的样式，所以可以将这部分样式放在外面，当将不需要打印的部分都隐藏掉了，再将外部的这些样式去掉，给@media print即可。

调用浏览器的打印使用的[ window.print()](http://www.w3school.com.cn/jsref/met_win_print.asp)， 虽然不能兼容所有浏览器，但是常见的高级浏览器都可以兼容，满足了我们的项目需求，这里我就没有继续深挖。

pdf的显示与打印，前前后后遇到了不少问题，以上流水做个总结。
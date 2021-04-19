---
title: 使用Layui的上传组件实现文件上传处理并响应文件流至前端保存
date: 2020-11-16 16:46:52
tags: ["layui"]
---

遇到一个需求，用户上传一个 Excel 工作簿给后端处理，响应给用户一个新的工作簿文件。

实现需求时遇到了个问题：Layui 上传组件的回调方法不支持后台返回文件流，会直接进入 error 回调提示“请求上传接口出现异常”，记录下最终实现的方法。

<!--more-->

## 前端

创建一个上传组件，设置为非自动上传，使用`choose`回调方法，该回调方法会在文件被选择后触发。

```javascript
layui.use("upload", function () {
    var upload = layui.upload;
    var uploadInst = upload.render({
        elem: "#importAndExport",
        accept: "file",
        ext: "xls|xlsx",
        auto: false,
        acceptMime:
            "application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    });
});
```

由于`choose`回调方法的参数`obj`包含了选择的文件，所以最终选择了使用该回调方法来自行实现上传并保存的功能

```javascript
choose:
function(obj){
    obj.preview(function(index, file, result){
        var form = new FormData();//创建一个formdata（表单数据）
        form.append("file",file);
        var request = new XMLHttpRequest();
        request.open('POST', "importAndExport.do", true);
        request.responseType = 'blob';
        request.onload = function(e) {
            uploadInst.config.elem.next()[0].value = ''; //清空 input file 值，以免删除后出现同名文件不可选
            if (this.status === 200) {
                var blob = this.response;
                var filename=this.getResponseHeader("Content-Disposition").split("; ")[1].split("=")[1];
                if (filename==null || ''==filename ||typeof filename == "undefined"){
                    filename="result.xlsx";
                }
                if(window.navigator.msSaveOrOpenBlob) {
                    window.navigator.msSaveBlob(blob, filename);
                }
                else{
                    var downloadLink = window.document.createElement('a');
                    var contentTypeHeader = request.getResponseHeader("Content-Type");
                    downloadLink.href = window.URL.createObjectURL(new Blob([blob], { type: contentTypeHeader }));
                    downloadLink.download = filename;
                    document.body.appendChild(downloadLink);
                    downloadLink.click();
                    document.body.removeChild(downloadLink);
                }
            }
        };
        request.send(form);
    });
}
```

## 后端

后端代码也贴出来，有个参照。

```java
@PostMapping("importAndExport.do")
@ResponseBody
public ResponseResult importAndExport(@RequestParam("file") MultipartFile file, HttpSession httpSession,HttpServletResponse response) throws Exception {
    XSSFWorkbook workbook = new XSSFWorkbook(); // 新建工作簿对象
    //处理过程略
    String fileName = "result.xlsx";
    OutputStream out = response.getOutputStream();
    response.reset();
    response.addHeader("Content-Disposition", "attachment; filename=" + fileName);
    response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8");
    workbook.write(out);
    return null;
}
```

## 效果图

![](/images/20201116152804.png)

## 参考

-   [1] [javascript - Get excel file (.xlsx) from server response in ajax - Stack Overflow](https://stackoverflow.com/questions/47134698/get-excel-file-xlsx-from-server-response-in-ajax)

-   [2] [layui 上传 Excel 更新数据并下载 - 如鱼饮水，冷暖自知 - 博客园](https://www.cnblogs.com/Jinfeng1213/p/11255584.html)

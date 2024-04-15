---
title: Powershell中将markdown转换为图片
date: 2022-04-01 21:10:53
tags: ["Powershell", "convert markdown to picture"]
---

# 前言

见本站[Powershell 执行 SSH 命令并推送结果](https://tioachan.dev/2022/04/01/Powershell%E6%89%A7%E8%A1%8CSSH%E5%91%BD%E4%BB%A4/)一文，最终决定将 markdown 格式的巡检结果转换成图片发送到群内。

# 转换

需要安装`docker`

<!--more-->

命令如下：

```powershell
$pushTitle=$current_date+"巡检结果";
#md文件保存文件夹路径
$path="C:\Users\xxx\.doc\"
#md文件名不带后缀
$filename=[guid]::NewGuid().Guid
#md文件保存路径
$savepath=$path+$filename+".md"
#先将标题输出至文件
$mdFileContent="## "+$pushTitle  | Out-File -FilePath $savepath
#换行
" " | Out-File -Append -FilePath $savepath
#将内容追加到文件
$mdTable | Out-File -Append -FilePath $savepath

# md to pdf
$filename_md=${filename}+".md"
$filename_pdf=${filename}+".pdf"
docker run  -it --rm -v ${path}:/opt/docs auchida/markdown-pdf markdown-pdf -f A4 -r landscape -o $filename_pdf $filename_md
# pdf to png
docker run -it --rm -v ${path}:/var/workdir/ -v ${path}:/var/workdir/output/ kolyadin/pdf2img -png -q -f 1 -singlefile -r 320 $filename_pdf ${filename}
$filename_png=${filename}+".png"
```

# 调用 mirai http 推送结果

```powershell
# mirai认证
$mirai_host="url"
$verifyParams = @{"verifyKey"="verifyKey"}| ConvertTo-Json
$verifyResutl=Invoke-WebRequest -Uri $mirai_host/verify -Method POST -Body $verifyParams -ContentType "application/json"

# 上传图片
$Form = @{
    type  = 'group'
    img=Get-Item -Path $path$filename_png
}
$Result = Invoke-RestMethod -Uri $mirai_host/uploadImage -Method Post -Form $Form

# mirai推送
$messageChain=@()
$messageChain=$messageChain+@{type="Image";url=$Result.url}
$pushMsgParams=@{target="QQ群号";messageChain=$messageChain}| ConvertTo-Json -Compress
$pushMsgParams
$pushResutl=Invoke-WebRequest -Uri $mirai_host/sendGroupMessage -Method POST -Body $pushMsgParams -ContentType "application/json; charset=utf-8"
$pushResutl
```

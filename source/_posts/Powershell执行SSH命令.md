---
title: Powershell执行SSH命令并推送结果
date: 2022-04-01 21:10:53
tags: ["Powershell", "SSH", "Posh-SSH", "serverchan", "mirai"]
---

# 需求

自动化巡检服务器性能指标，推送结果给项目组人员。

<!--more-->

# 依赖

安装 `Posh-SSH`

```powershell
Install-Module  -Name Posh-SSH
```

# 定义变量

由于脚本的使用场景为定时自动化服务器性能指标巡检，且所有服务器的用户名密码相同，所以将服务器 ip 定义为数组，将需要运行的命令定义为变量。

```powershell
# 代理服务器，可选http,socks4,socks5
# see https://github.com/darkoperator/Posh-SSH/blob/master/docs/New-SFTPSession.md#-proxytype
$proxy_host="x.x.x.x"
$proxy_port=12345
$proxy_type="Socks5"

# 目标服务器
$host_username = "username"
$host_password = "password"
$host_ips="1.2.3.4","5.6.7.8","9.10.11.12"

$OS_MEM="free -m |grep Mem |awk '{print `$2}'"
$OS_MEM_AVAILABLE="free -m | grep Mem |awk '{print `$7}'"
$OS_MEM_PER="free -m | grep Mem |awk '{printf (`"`%.2f\n`",`$3/`$2*100)}'"
$OS_ROOT_DISKS_USE_PRECENT="df -h / | grep '/'| awk '{print `$5}'"
$OS_CPU_US="top -bn1 | grep Cpu | awk '{print `$2}'"
$OS_CPU_load_1="uptime | sed 's/,//g' | awk '{print `$(NF-2)}'"
$OS_CPU_load_5="uptime | sed 's/,//g' | awk '{print `$(NF-1)}'"
$OS_CPU_load_15="uptime | sed 's/,//g' | awk '{print `$(NF)}'"
```

# 运行

遍历`$host_ips`，依次执行定义的命令，使用 hashtable 收集运行结果

```
$result_table = @()

$i=0
foreach ($host_ip in $host_ips) {
    # 进度条
    $i=$i+1
    $w=$i.tostring() + '/' + $host_ips.Length.toString()
    Write-Progress -Activity "执行中" -status "正在处理 $host_ip，请耐心等待，$w" -percentcomplete ($i/($host_ips.Length)*100)

    # 创建连接
    $secure = $host_password | ConvertTo-SecureString -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential($host_username,$secure)
    $session=New-SSHSession -ComputerName $host_ip -Credential $cred -AcceptKey -ProxyType $proxy_type -ProxyServer $proxy_host  -ProxyPort $proxy_port

    # 内存
    $OS_MEM_MSG = Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_MEM
    $OS_MEM_AVAILABLE_MSG = Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_MEM_AVAILABLE
    $OS_MEM_PER_MSG = Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_MEM_PER

    # 根卷
    $OS_ROOT_DISKS_USE_PRECENT_MSG= Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_ROOT_DISKS_USE_PRECENT

    # CPU
    $OS_CPU_US_MSG = Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_CPU_US
    $OS_CPU_LOAD_1_MSG = Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_CPU_load_1
    $OS_CPU_LOAD_5_MSG = Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_CPU_load_5
    $OS_CPU_LOAD_15_MSG = Invoke-SSHCommand -SessionId $session.SessionId -Command $OS_CPU_load_15

    # 关闭链接
    $null = Remove-SSHSession $session.SessionId

    $null=$OS_MEM_MSG

    # 输出结果
    $result_availabvle_mem=$OS_MEM_AVAILABLE_MSG.Output[0]+"M"
    $result_mem_persent_usage=$OS_MEM_PER_MSG.Output[0]+"%"
    $result_availabvle_root_storage=$OS_ROOT_DISKS_USE_PRECENT_MSG.Output[0];
    $result_cpu_persent_usage=$OS_CPU_US_MSG.Output[0]
    $result_cpu_load=$OS_CPU_LOAD_1_MSG.Output[0],$OS_CPU_LOAD_5_MSG.Output[0],$OS_CPU_LOAD_15_MSG.Output[0] -Join " "
    $result_table += [Pscustomobject]@{
        "IP" = $host_ip;
        "可用内存" = $result_availabvle_mem;
        "内存使用率"=$result_mem_persent_usage;
        "根卷使用占比"=$result_availabvle_root_storage;
        "CPU使用率"=$result_cpu_persent_usage;
        "CPU1/5/15分钟平均负载"=$result_cpu_load.ToString();
    }
}
```

# 推送结果

刚开始设计脚本时只考虑到自己查看巡检结果，选择`serverchan`推送结果，由于`serverchan`可以友好显示 Markdown 表格，这里选择将 HashTable 转换为 Markdown。

```powershell
$mdTable = $result_table | convertto-markdownTable -includeOpen -allColumnsAllignment Left
```

实测可用的转换方法如下：

```powershell
function convertto-markdownTable {
    [cmdletbinding()]
    Param(
        [parameter(Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="IndividualAlligned")]
        #[parameter(mandatory=$true, ValueFromPipeline=$true, ParameterSetName="AllAlligned")]
        [parameter(mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Default")]
        [object]$inputObject,

        #[parameter(ParameterSetName="IndividualAlligned")]
        #[parameter(ParameterSetName="AllAlligned")]
        [parameter(ValueFromPipeline=$true, ParameterSetName="Default")]
        [int]$width=20,

        #[parameter(ParameterSetName="IndividualAlligned")]
        #[parameter(ParameterSetName="AllAlligned")]
        [parameter(ValueFromPipeline=$true, ParameterSetName="Default")]
        [switch]$includeOpen,

        [parameter(Mandatory=$true, ParameterSetName="IndividualAlligned")]
        [parameter(ValueFromPipeline=$true, ParameterSetName="Default")]
        [hashtable]$columnAlignment,


        [parameter(Mandatory=$true, ParameterSetName="AllAlligned")]
        [parameter(ValueFromPipeline=$true, ParameterSetName="Default")]
        [string]$allColumnsAllignment
    )
    begin {
        $output = ""
        [System.Collections.ArrayList]$properties =[System.Collections.ArrayList]@()
        $firstRun=$true;
        $maxWidth = $width
        [System.Collections.ArrayList]$collection = [System.Collections.ArrayList]@()
        $hinted = $false;
    }
    process {

        $collection.add($inputObject)|out-null
        if($firstRun){
            $properties = $inputObject.psobject.properties | Where-Object {$_.MemberType -eq "NoteProperty"} | Select-Object -ExpandProperty Name
            foreach($property in $properties){
                try {
                    if($maxWidth -lt $inputObject.$property.tostring().length){
                        $maxWidth = $inputObject.$property.tostring().length
                    }
                } catch {}
                try{
                    if($includeOpen){
                        $output += "|$property$(" ".toString() * (($width - "$property".length)))"
                    } else {
                        $output += "$property$(" ".toString() * (($width - "$property".length)))|"
                    }
                } catch [system.ArgumentOutOfRangeException] {
                    if(!$hinted){
                        $hinted = $true
                    }
                }
            }
            if($includeOpen){
                $output += "|`n|"
            } else {
                $output += "`n"
            }

            foreach($property in $properties){
                if($columnAlignment.$property){
                    switch($columnAlignment.$property){
                        Right {
                            $AllignmentSymbol = @{
                                L=" ";
                                R=":";
                            }
                        }
                        Left {
                        $AllignmentSymbol = @{
                                L=":";
                                R=" ";
                            }
                        }
                        Center {
                            $AllignmentSymbol = @{
                                L=":";
                                R=":";
                            }
                        }
                        default {
                            $AllignmentSymbol = @{
                                L=" ";
                                R=" ";
                            }
                        }
                    }
                } else {
                    $AllignmentSymbol = @{
                        L=" ";
                        R=" ";
                    }

                    switch($allColumnsAllignment){
                        Left {
                        $AllignmentSymbol = @{
                                L=":";
                                R=" ";
                            }
                        }

                        Right {
                            $AllignmentSymbol = @{
                                L=" ";
                                R=":";
                            }
                        }
                        Center {
                            $AllignmentSymbol = @{
                                L=":";
                                R=":";
                            }
                        }
                        Default {
                            $AllignmentSymbol = @{
                                L=" ";
                                R=" ";
                            }
                        }
                    }
                }
                $output += "$($AllignmentSymbol.L)"+("-"*($width-2)) +"$($AllignmentSymbol.R)|"
            }
                $AllignmentSymbol = " ";

            if($includeOpen){
                $output += "`n"
                $output += "|"
            } else {
                $output += "`n"
            }
            foreach($property in $properties){
                try {
                    if([string]::IsNullOrEmpty($inputObject.$property)){
                        $output += " "*$width +"|"
                    } else {
                        $output += "$($inputObject.$property.ToString() + (" "*($width - ($inputObject.$property.toString().length))))|"
                    }
                } catch [system.ArgumentOutOfRangeException] {
                    if(!$hinted){
                        $hinted = $true
                    }
                }
            }
            $output += "`n"
            $firstRun=$false
        } else {
            foreach($property in $properties){
                try {
                    if($maxWidth -lt $inputObject.$property.tostring().length){
                        $maxWidth = $inputObject.$property.tostring().length
                    }
                } catch {}
                if([string]::IsNullOrEmpty($inputObject.$property)){
                    $output += " "*$width +"|"
                } else {
                    if($includeOpen){
                        if($properties.IndexOf($property) -eq 0){
                            $output += "|"
                        }
                    }
                    try {
                        if($includeOpen){
                            $output += "$($inputObject.$property.toString() + (" "*($width - ($inputObject.$property.tostring().length))))"
                        } else {
                            $output += "$($inputObject.$property.toString() + (" "*($width - ($inputObject.$property.tostring().length))))|"
                        }
                    } catch [system.ArgumentOutOfRangeException] {
                        if(!$hinted){
                            $hinted = $true
                        }
                    }
                    if($includeOpen){
                        $output +="|"
                    }
                }
            }
            $output += "`n"
        }
    }
    end {
        if($maxWidth -gt $width){
            write-verbose "This dataset requires a minimum -width of $($maxWidth)"
            write-verbose "Reruning with -width $maxwidth"
            $collection | convertto-markdownTable -width $maxWidth -includeOpen:$includeOpen.IsPresent
        } else {
            write-output $output
        }
    }
}
```

然后开始调用`serverchan`进行推送

```powershell
$postParams = @{title=$pushTitle;desp=$mdTable;channel='66|18'}
$Response=(Invoke-WebRequest -Uri https://sctapi.ftqq.com/<key>.send -Method POST -Body $postParams).Content| ConvertFrom-Json
$ResponsePushId=$Response.data.pushid
$ResponseReadkey=$Response.data.readkey
# 阅读链接
$callbackUrl="https://sct.ftqq.com/readpush?id=${ResponsePushId}&readkey=${ResponseReadkey}"
```

# 后续

想要将结果推送给项目组内所有人，考虑到不可能让所有人注册 serverchan，且 pushdeer 没有对 md 内容进行转换美化，显示效果非常拉胯。

最终决定组建 QQ 群，先调用 serverchan 推送一次消息，获取阅读链接，使用 mirai 的 http api 插件将链接发送到群消息。

但 serverchan 的推送内容保存时间极短，所以需要考虑新的推送方式。

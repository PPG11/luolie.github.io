---
title: win 终端设置vpn
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-22 13:38:43
password:
su
tags:
categories:
---

在 Windows 下给终端（cmd，Git Bash，PowerShell）配置代理

其实命令很简单，跟在 Linux 下没什么区别。

注意 ping 是不同代理，所以试验时建议使用 curl

```Bash
curl -vv http://www.google.com
```

但是在不同终端下命令不一样

1. cmd 下

```Bash
set http_proxy=http://127.0.0.1:1080

set https_proxy=http://127.0.0.1:1080
```

2. Git Bash

```Bash
export http_proxy ...
```

3. PowerShell

```Bash
# NOTE: registry keys for IE 8, may vary for other versions
$regPath = 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings'

function Clear-Proxy
{
    Set-ItemProperty -Path $regPath -Name ProxyEnable -Value 0
    Set-ItemProperty -Path $regPath -Name ProxyServer -Value ''
    Set-ItemProperty -Path $regPath -Name ProxyOverride -Value ''

    [Environment]::SetEnvironmentVariable('http_proxy', $null, 'User')
    [Environment]::SetEnvironmentVariable('https_proxy', $null, 'User')
}

function Set-Proxy
{
    $proxy = 'http://example.com'

    Set-ItemProperty -Path $regPath -Name ProxyEnable -Value 1
    Set-ItemProperty -Path $regPath -Name ProxyServer -Value $proxy
    Set-ItemProperty -Path $regPath -Name ProxyOverride -Value '<local>'

    [Environment]::SetEnvironmentVariable('http_proxy', $proxy, 'User')
    [Environment]::SetEnvironmentVariable('https_proxy', $proxy, 'User')
}
```

参考文献
https://zcdll.github.io/2018/01/27/proxy-on-windows-terminal/

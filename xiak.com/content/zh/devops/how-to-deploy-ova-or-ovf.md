---
title: "使用 govc 部署 ova 和 ovf"
date: 2019-11-28T12:35:00+08:00
author: Xiak
image : "images/blog/cafe.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["DevOps"]
tags: ["devops", "vmware"]
description: ""
draft: false
type: "post"
---

# 使用 govc 部署 ova 和 ovf

`govc` 是 `vmware` 官方开发的一个命令行 `client`, 用 `golang` 编写, 实现了大部分 `vmware sdk` 的功能, 详细的请参阅 [govmomi](https://github.com/vmware/govmomi)

## 下载 govc

Windows下载

```bash
https://github.com/vmware/govmomi/releases/download/v0.21.0/govc_windows_amd64.exe.zip
```

Linux 下载

```bash
https://github.com/vmware/govmomi/releases/download/v0.21.0/govc_linux_amd64.gz
```

## 解压

解压下载的压缩包, 把可执行文件放入加入了环境变量的路径下

在控制台输入 `govc version` 测试


## 设置环境变量

Windows

```bash
set GOVC_INSECURE=1
set GOVC_URL=https://10.121.229.8
set GOVC_USERNAME=administrator
set GOVC_PASSWORD=password
set GOVC_DATASTORE=229-6-local-storage2
set GOVC_NETWORK="10.121.229.5-254"
set GOVC_RESOURCE_POOL='10.121.229.6/Resources/AVE'
```

Linux

```bash
export GOVC_INSECURE=1
export GOVC_URL=https://10.121.229.8
export GOVC_USERNAME=administrator
export GOVC_PASSWORD=password
export GOVC_DATASTORE=229-6-local-storage2
export GOVC_NETWORK="10.121.229.5-254"
export GOVC_RESOURCE_POOL='10.121.229.6/Resources/AVE'
```

## 从 ova 或则 ovf 导出 配置文件 config.json

```bash
govc import.spec your-ova-or-ovf.ova.ovf | python -m json.tool > config.json
```

在 `config.json` 里设置锚点, 如 `__REPLACE_FQDN__` , `__VMNAME__` , 之后方便通过 `sed -i` 修改

把 `DiskProvisioning` 的 修改为 `thin`

`config.json` 的例子:

```bash
{
  "DiskProvisioning": "thin",
  "IPAllocationPolicy": "fixedPolicy",
  "IPProtocol": "IPv4",
  "PropertyMapping": [
    {
      "Key": "vami.ipv4.Avamar_Virtual_Edition",
      "Value": "__REPLACE_IPV4__"
    },
    {
      "Key": "vami.ipv6.Avamar_Virtual_Edition",
      "Value": ""
    },
    {
      "Key": "vami.gatewayv4.Avamar_Virtual_Edition",
      "Value": "__REPLACE_GW4__"
    },
    {
      "Key": "vami.gatewayv6.Avamar_Virtual_Edition",
      "Value": ""
    },
    {
      "Key": "vami.DNS.Avamar_Virtual_Edition",
      "Value": "__REPLACE_DNS__"
    },
    {
      "Key": "vami.searchpaths.Avamar_Virtual_Edition",
      "Value": ""
    },
    {
      "Key": "vami.FQDN.Avamar_Virtual_Edition",
      "Value": "__REPLACE_FQDN__"
    },
    {
      "Key": "vami.NTP.Avamar_Virtual_Edition",
      "Value": "__REPLACE_NTP__"
    }
  ],
  "NetworkMapping": [
    {
      "Name": "VM Network",
      "Network": "__REPLACE_NETWORK__"
    }
  ],
  "Annotation": "This is an Avamar Virtual Edition.",
  "MarkAsTemplate": false,
  "PowerOn": true,
  "InjectOvfEnv": false,
  "WaitForIP": true,
  "Name": "__VMNAME__"
}
```

# 修改 config.json

```bash
sed -i "s#__VMNAME__#10.121.229.2#" config.json
```

# 部署 ova

```bash
govc import.ova -options=config.json your-ova-or-ovf.ova.ovf 
```

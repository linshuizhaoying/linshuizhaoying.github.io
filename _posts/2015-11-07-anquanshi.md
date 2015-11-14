## 安全师

### 第一章

### Rip

`配置rip协议`

```
router rip 
network ip（直连两端的IP，比如 192.168.1.254 -> 192.168.1.0)

```

icmp 控制列表 第一步

```
access-list 101(看题目要求) permit（看题目要求） icmp host ip1 host ip2

ip1是题目中被限制一方（例题中是分公司） ip2是服务器(题目中是总公司) 

```

其它服务

```
ftp:

access-list 101(看题目要求) permit（看题目要求） tcp  host ip1 host ip2 eq ftp

tcp类似：

access-list 101(看题目要求) permit（看题目要求） tcp host ip1 host ip2 eq www 

奇葩的dns:

access-list 101(看题目要求) permit（看题目要求） tcp host ip1 host ip2 eq 53

access-list 101(看题目要求) permit（看题目要求） udp host ip1 host ip2 eq 53

```


应用 第二步

```
int fa0/0 题目一般给出对某个路由器的端口进行应用配置

ip access-group 101 in

```

安全认证登录:

```
console加密:

conf t
line console 0
password 要求设置的密码
login (必须加上)

全局加密:

conf t
enable password 要求设置的密码
service password-encryption

```



### Vlan

vtp安全协议配置：

```
  vtp mode ...(+? 自动列出)
  vtp domain ...
  vtp password ...
  
```



```
vlan配置:

 第一步 valn 50(题目给的vlan编号)
   -> name Sales（题目给的别名 注意大小写）
   
   
 第二步 将指定设备的端口号划入vlan
 int fa0/1 (题目给的，需要先跳到该端口)
 switchp access vlan 50
 switchp mode access (这句必须)
 
 
```

```

最后一步 先save 再export

```

## 操作系统安全管理

```
基本的安全：

1.文件
  -> 描述 是 普通用户可以修改
     
     实际上 添加users组 默认权限 不加修改
           在高级中 勾选拒绝（右边那侧）  删除 删除子文件夹
 
 2.本地安全设置
   ->本地策略
   ->安全选项 
   
 ```

```
特殊:

文件夹审计：
  高级里
  ->审核
  ->添加everyone用户
    审计 删除 删除子文件夹 （成功和失败都勾选）

部署策略：
本地安全设置
   ->本地策略
   ->审核选项 
     对象访问 账户登录事件
     勾选 成功和失败
     
屏蔽命令行:

打开组策略 或者 gpedit.msc
用户设置
-> 管理模板
直接可以看到
注册表也限制

使用管理模板禁用注册表:
打开组策略 或者 gpedit.msc
用户设置
-> 管理模板
注册表限制


屏蔽注册表：
打开本地安全策略->软件限制策略
->其它规则->哈希规则


密钥导出:
certmgr.msc
->个人->证书



     
```

## 应用服务安全管理

### web
```
默认文档:
右键网站属性->文档->添加index.asp ->置顶

确保页面正常显示:
右键网站属性->主目录->应用程序配置->选项->启用父路径

安全加固:

网站->数据库database/data->右键权限->给来宾用户添加写入权限

服务扩展:除了active server page 启用 其它都禁用

网站配置->主目录->应用程序配置->映射->删光除了.asp


其它：
证书：
右键网站属性->目录安全性->服务器证书（题目有给路径）

目录安全性->安全通信->编辑 
    启用ssl 要求客户端证书
    指定证书
 
IP范围： 
 
目录安全性->IP地址和域名限制


日志:
   右键网站属性->日志
   
```

最后一步

`本地计算机->所有任务->将配置保存到磁盘`


## 


### 数据库安全

查询分离器:

```
use master
drop proc ......,...,,..
```

## 应急响应

```
chdisk e: /f >d:\Report.txt 
```


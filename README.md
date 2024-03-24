# TuLink - 一个生成重定向链接的api，可用于短域名

<img src="https://img.shields.io/badge/Features 特性-json文件储存代替数据库，直接杜绝注入风险，可内存溢出-blue">

起因就是qqbot只能发送备案域名，有人向我借二级域名，不放心就写了一个短域名程序便于监控，同时温习长久未动的flask

## 使用方法
向/creat_redirect发送请求，参数如下

| token                                     | address        |
| ----------------------------------------- | -------------- |
| token1                                    | 要重定向的域名 |

（token可在../data/users.json查看和设置）

返回如下消息

| code | address                                              | timestamp     |
| ---- | ---------------------------------------------------- | ------------- |
| 0    | http://127.0.0.1/redirect?address_token=ttdux3w7yt61 | 1711293984967 |

/redirect的参数address_token随机生成，timestamp是时间戳

访问返回的address就能重定向到指定的网址

自己部署的就暂时不放出来了（怕被打）

还有就是../data/redirect_datas.json可查看修改已有的重定向

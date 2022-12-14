---
weight: 3
title: 评估测试
---

# 评估测试

## 概念

评估测试用于验证在当前的安全防护措施下网络、系统抵抗黑客攻击的能力。

评估测试可采用黑盒测试。即测试小组在不知晓目标网络环境的情况下，以某一URL或IP地址为目标，尝试突破其网站/网络的测试。类 似于一次真实的黑客攻击。测试中应尽量不影响正常业务运行。


## STRIDE 威胁建模

- Spoofing：仿冒。冒充他人。
- Tampering：篡改。修改数据或代码，包括持久化的数据和网络传输中的数据。
- Repudiation：否认。否认做过某种行为。
- Information Disclosure：信息泄露。信息被泄露或被窃取。
- Dos：拒绝服务。拒绝服务攻击消耗资源，导致服务不可用。
- Elevation of privilege：权限提升。未经授权访问。

## 常见威胁

- 访问控制：低权限用户不能通过以下手段查看高权限用户的信息和功能。
	+ 直接通过url访问无权页面；
	+ 直接通过url下载其它用户上传的图片；
- 短信轰炸：手机号应加密，针对同一用户增加发送频率限制。
- 重放攻击：防止数据包被窃取后进行重放。接口请求前可先获得Token或增加请求时间字段。
- 文件上传：限制格式，禁止上传exe、php等文件。
- 信息泄露：设计全局异常信息和页码，防止异常页面泄露代码的运行信息。
- HTTPS：TLS1.0安全性不足，应使用TLS1.2或更高版本。
- CORS+CSRF：限制 origin 和 refere 及 Access-Control-Allow-Origin 的来源只能为信任的网站。
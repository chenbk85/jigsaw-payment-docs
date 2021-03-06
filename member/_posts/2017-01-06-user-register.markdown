---
layout: post 
title: "简易开户"  
subtitle: "CIF-1001"  
date: 2017-01-06 20:00:00  
author: "shamphone"  
header-img: "img/home-bg-post.jpg"  
catalog: true  
chapter : "2.1"
tag: [api]  
---

## 接口定义

```html
PUT /member
```

## 功能概述

用户注册开户，建立支付账户。 

## 输入参数

| 参数名称         | 可选         | 说明                                           |  备注                      |
|------------------|--------------|------------------------------------------------|----------------------------|
| 登录手机号       | 可选         | 国内手机号为11位纯数字，国外手机号按照地区来设定| 登录手机与登录邮箱必输其一 |
| 登录邮箱         | 可选         | 标准邮箱，最长50个字母，存储时全部转小写       |                            |
| 托管平台ID       | 必传         | 托管平台的ID号。                               | 托管平台开户时必须提供     |
| 安全手机         | 可选         | 如果是邮箱要求必选                             | 如果是登录手机不为空，安全手机与登录手机相同 |
| 安全邮箱         | 可选         | 如果是登录邮箱不为空，安全邮箱与登录邮箱相同   |                            |
| 登录密码         | 必选         | 密码采用MD5加密后上送，不能包含空格和特殊字符  |                            |
| 支付密码         | 必选         |  MD5标准加密上送 支付密码不能与登录密码相同    |                            |
| 订购业务类型列表 | 可选         | 以逗号分隔的业务类型名称列表                   |                            |
| 托管账户业务类型列表 | 可选     | 以逗号分隔的账户业务类型列表                   |             |
| 验证码编号       | 可选         | 由验证码发送接口提供                           | 同时选输(手机或邮箱验证码) |
| 验证码           | 可选         | 验证码在规定的时间段内且一次性验证有效         |                            |
| 客户姓名         | 可选         |                                                | 建议业务系统进行上送       |
| 证件国别         | 可选         | 参见国别数据字典                               |                            |
| 证件类型         | 可选         | 参见证件类型数据字典                           |                            |
| 证件号码         | 可选         | 根据证件类型进行号码检查                       |                            |

## 输出参数

| 参数名称         | 可选         | 说明                                           |  备注                      |
|------------------|--------------|------------------------------------------------|----------------------------|
| 执行结果         | 必选         |                                                |                            |
| 失败原因         | 必选         |                                                |                            |
| 用户号           | 必选         |                                                |                            |
| 用户状态         | 必选         |                                                |                            |

 

## 业务处理流程

1. 【会员系统】进行接口必输参数及其合法性校验：  
   - 【会员系统】进行**手机号、登录邮箱、登录密码、支付密码参数缺失**校验，如果不通过，则提示“接口输入参数缺失”异常（异常码：\$｛接口输入参数缺失｝）；    
   - 【会员系统】检查业务系统是否同时上送**（同时为空或同时不为空）客户姓名、证件国别、证件类型、证件号码，**如果业务系统上送的不完整，则提示“客户参数不完整”异常（异常码：\${客户参数不完整}）；    
   - 【会员系统】检查业务系统是否同时上送了**（同时为空或同时不为空）短信验证码编号、短信验证码**，  
         - 如果业务系统上送不完整，则提示“短信验证参数不完整”异常（异常码：\${短信验证参数不完整}）；  
         - 如果同时上送了短信验证码编号、短信验证码，则进行短信验证码签权（*开户流程中只关注签权结果，具体签权过程由参见短信验证码签权的功能流程设计*），如果签权不通过，则将签权异常抛出  

2. 【会员系统】进行业务校验：
   - 检查证件号是否合法， 证件号必须为全部号码，  
     - 如果证件类别为身份证，则通过专用工具类校验身份证是否合法，  
	 - 如果不合法，则提示“身份证号不符合规则”异常（异常码：\${身份证号不符合规则}）。  
   - 校验手机号或者邮箱是否被占用：
     - 通过查询*用户标识表（用户标识字段）*，确认手机号或者是否已经存在，如果存在，则： 
     - 如果业务系统上送的互联网支付密码不为空，则比对互联网支付密码是否匹配，如果比对信息不通过，则提示“手机号已注册”或者“邮箱已注册”异常，（异常码：\$｛用户手机号已存在\|用户邮箱已存在｝）；

3. 【会员系统】进行用户开户：
    - 如果用户未存在，则进行新用户创建：  
        1. 进行用户基本信息创建（个人用户基本信息表）、用户标识创建（个人用户标识表）、用户密码信息创建（个人用户登录密码表（互联网登录密码类型为：1）、个人用户支付密码表（互联网支付密码类型为：1）  
        2. 如果登录手机不为空，则将用户登录手机设置同步为用户安全手机。如果登录邮箱不为空则将用户登录邮箱设置同步为用户安全邮箱
		3. 【会员系统】进行用户管理信息设置（*个人用户管理信息表*）的默认设置：用户的资金止付状态：未止付（0），用户的锁定状态：为未锁定（0）
		4. 如果业务订购列表不为空，进行业务订购；

4. 如果业务系统上送了客户信息，则进行客户信息处理：  
	- 将证件三要素和客户姓名、性别插入到客户信息临时表。

5. 【会员系统】检查是否需要回写： 
	- 如果需要回写，则将交易编号、客户号、用户号（报文内容可不写）写入*回写日志表*中。

6. 【会员系统】进行维护日志登记  
	- 调用相应工具进行*系统维护日志表、系统维护日志明细表*进行信息记录。  

7. 【会员系统】登记系统事件登记表  

8. 【会员系统】如果用户是手机注册，则发送短信通知  
	1. 通过业务获取短信模板，拼装短信内容  
	2. 调用短信发送接口，发送短信（短信发送失败不影响业务，接口中已实现重发机制）  
	
9. 【会员系统】如果用户是邮箱注册，则发送邮箱通知  
	1. 通过业务获取邮箱模板，拼装邮箱内容  
	2. 调用邮箱发送接口，发送邮件   
	
10. 【会员系统】系统返执行结果  
	- 如果失败则返回失败及失败原因，返回用户号、用户状态、用户锁定状态、用户资金止付状态、客户号（如果有关联客户）

【结束流程】
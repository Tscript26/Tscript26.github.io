---
title: 12月2号-本日工作
author: 一三
date: 2025-12-02 09:00:00 +0800
categories: [Work]
tags: [balabala]
---

## 本日工作内容
1. 小艺A2A接入调研
    - 借用ChatGPT以及Codex完成了一个基于fastapi的样例
    - [接口要求](https://developer.huawei.com/consumer/cn/doc/service/agent2agent-0000002498656261)
    1. 调研结果，
        - 一共需要写9个接口对接当前的所有项目
        UserId,如果未发现，由前端生成的，规则是uuid.uuid1
        - 
2. 沙盒代码审查
    - 发现环境修改类API未添加并行检查（同一时间只允许一人进行环境修改） --- 已修复
    - 发现创建虚拟环境为对默认类型进行限制（default, base）， --- 已修复
    - 发现创建虚拟环境的libpath在代码层硬写了版本号 --- 已修复
    - 发现修改环境类的API没有mutation check，可能会造成干扰 --- 已修复
    - 发现添加依赖，检查虚拟环境依赖接口没有增加虚拟环境是否存在的检测 --- 已修复



## Jekyll 修改
1. 修改的当前项目的名称和description，添加了对应邮箱
2. 发现有两个找不到的min.js，有机会要处理一下


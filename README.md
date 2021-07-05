---
description: 为了实现SDDC的POC部署而定义的文档
---

# 简介

## 为什么要学SDDC

2013年前后，SDX（软件定义XX）广泛流行，而SDDC的概念则是集大成者。参与这方面的厂商包括Vmware、微软、IBM、HP等等，不过能够将这种理念顺利延续下来的厂商屈指可数。

顾名思义，这是一项私有云下的技术，但在最新的[Windows Server Software-Defined Datacenter](https://docs.microsoft.com/en-us/windows-server/sddc)中，Azure Stack HCI 解决方案也涵盖在内。

众所周知，Azure Stack HCI 解决方案是Azure现阶段给予很多重视的产品，而Azure Stack HCI 大量运用了SDDC的概念和技术，如果不了解SDDC，在学习Azure Stack HCI 的时候，可能会遇到很多疑问。

当然，这两者的学习能够互补，我们可以站在不同的角度，考量产品的适用场景。

## 什么是SDDC

简言之，SDDC就是计算、存储、网络都使用软件定义的方式实现。以实现对硬件设备的解耦依赖。因此一个完整的SDDC方案，一定会包含计算、存储、网络的软件定义。

![image](http://ny9s.com/picupdate/20210705153125.png)

## 前置知识

全系列文章会尽量详细，读者只需要有基本的Windows操作经验即可。

每个人知识背景不同，理解可能也不同，如果发现部分章节的内容不易理解，请向我提交issues。

## 收益

通过`纯手工`的方式搭建SDDC平台，了解基本的SDDC使用方法。

> 部署方式仅适用于POC，由于缺少必要的高可用设计，不能直接应用于生产环境。
>
> 深入的理论知识，可以参考附录的推荐章节。

## 许可

[MIT许可](LICENSE.md)：请任意使用全部或部分，但请注明出处

## 反馈

请通过[Github](https://github.com/kukisama/SDDCPOC)站点的issues来提交反馈，以及希望增加的内容，只支持`中文`。

## 保密

发布内容不涉及NDA，且主要面向初级入门，因此发布内容在特定情况下会缺少深入细节，但原则上不会影响阅读




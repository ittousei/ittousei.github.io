---
title: 使用Cloudflare R2搭建个人图床
description: 
author: MsEspeon
date: 2026-05-23 22:10:00 +0800
categories: [Tutorial, Cloudflare]
pin: false
math: true
mermaid: true
---

本文简单探讨了使用Cloudflare R2对个人网站的图片资源实现批量化管理，算是[桃子圈数据备份与微博镜像站搭建](/posts/peachring-archive/)的一个额外延伸，不过文中涉及到的相关技术工具同样适用于个人博客的图床搭建。

在搭建微博镜像站点的过程中，图片资源的存储和访问是一个重要的问题。此前对于个人博客的图片资源，我采用的管理方案是手动维护：

- 对于微博镜像项目，需要备份的图片数量超过10k张，手动维护显然不再可行，需要诉诸一个专业的图片托管服务来管理这些资源
- 对于个人博客，随着文章数量的堆叠，需要管理的图片也在不断增加，未来还会有增量更新的需求，继续手动一张张添加图片也不现实

基于以上两点考量，批量化图片资源管理的需求提上了日程。

对于个人站点的图片资源管理，需要满足以下要求：

1、批量上传且保留文件目录结构（批量引用）：批量化的自动管理是这个项目的初衷
2、公开访问且不借助科学上网：这是一个很重要的需求，毕竟这个博客还是主要面向国内的，如果博客能国内直连，图片资源要科学上网，会让网站的设计很割裂
3、免费或低成本：白嫖总是好的
4、稳定性：图片资源的访问稳定性直接关系到用户体验，尤其是对于需要一次性访问上百张图片的微博镜像站点

综合所有考量，我最终选择了Cloudflare R2作为图片资源的托管方案。

## 什么是Cloudflare R2？

Cloudflare R2是Cloudflare提供的对象存储服务，适合保存图片、视频、备份文件、静态资源等。R2的几个核心特点是：

- 基于对象存储
- 面向静态资源、媒体文件、归档数据
- 国内直连
- 没有公网流出流量费（egress fee）

R2的官方文档：

- R2 介绍: https://developers.cloudflare.com/r2/how-r2-works/

## R2的计费方式

Cloudflare R2的计费方式是先用免费档位的Free Plan把用户骗进来，再通过进阶套餐额外收费。免费套餐每月提供10GB的存储空间，以及百万次的数据请求额度：

- Storage: `10 GB-month / month`
- Class A (Write): `1 million requests / month`
- Class B (Read): `10 million requests / month`
- Egress (出口流量费): `Free`

如果用户的使用超出免费额度，则需要根据超额次数额外收取费用：

- Storage: `$0.015 / GB-month`
- Class A (Write): `$4.50 / million requests`
- Class B (Read): `$0.36 / million requests`
- Egress (出口流量费): `Free`

套餐的免费档位给出的资源相当丰厚，10GB的存储空间足以应付博客图床这种轻量级别的存储，百万次/月的数据请求也远超出底边博客的日常访问需求了。不过套餐的收费档性价比并不高，且计费采用的是取上界的方式，稍微超出一点免费额度就会被收取一整档的费用，因此在使用过程中需要注意监控使用量，算是一个潜在的小坑。当然只要不超出免费额度，就不需要担心费用问题了。

官方定价页：

- https://developers.cloudflare.com/r2/pricing/

## 批量上传文件

### 创建Bucket

Cloudflare R2以Bucket为单位进行对象存储，上传对象时需要指定Bucket名称和对象的Key。对于批量。首先，需要在网页端创建一个Bucket，步骤如下：

1. 登录[Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入`R2 Object Storage`
3. 点击`Create bucket`
4. 输入bucket名称
5. 选择存储类型（选取`Standard`类型，`Infrequent Access`并不包含在Free Plan内）

新建bucket的官方文档：

- Create new buckets: https://developers.cloudflare.com/r2/buckets/create-buckets/

### 上传对象

Cloudflare支持多种方式上传：

- Dashboard
- Wrangler
- S3 API
- `rclone`

其中，Dashboard为在网页端手动上传，但只支持上传少量、小体积文件，无法满足批量上传的需求；Wrangler和`rclone`是Cloudflare提供的命令行工具；S3 API提供了编程接口，可以通过编写代码实现上传。后三种方式均支持批量化上传文件，我选用的方法是`rclone`，运行环境是Mac的`terminal`。

上传对象的官方文档：

- Upload objects: https://developers.cloudflare.com/r2/objects/multipart-objects/
- CLI: https://developers.cloudflare.com/r2/get-started/cli/
- S3 API: https://developers.cloudflare.com/r2/api/s3/

### rclone的安装与配置

`rclone`是一个开源的命令行工具，支持多种云存储服务的文件管理，包括Cloudflare R2。首先，在命令行安装`rclone`：

```bash
brew install rclone
```

安装完毕后，对`rclone`进行配置：

```bash
rclone config
```

按照提示创建一个新的remote，选择Cloudflare R2作为存储类型，并填入以下信息：

- Access Key ID
- Secret Access Key
- S3 endpoint

你需要访问网页端的[Cloudflare Dashboard](https://dash.cloudflare.com/) ，在先前创建的bucket中设置R2 API Token，拿到这三个值。

配置完毕后，就能批量上传文件了，输入以下命令（自行替换{}部分），上传的文件夹将保留本地的目录结构：

```bash
rclone copy {本地文件夹名} r2:{bucket名}/{文件夹名}/
```

配置`rclone`的官方文档：

- https://developers.cloudflare.com/r2/examples/rclone/

## 公开访问

以上步骤实现了将本地图片资源上传到Cloudflare R2，但默认情况下上传的对象是私有的，无法直接通过URL访问。为了让个人站点能够直接引用这些图片资源，需要开启公开访问权限。

Cloudflare R2提供了两种方式实现公开访问：

- Public Development URL：Cloudflare会为每个bucket分配一个公共URL，开启后可以通过该URL访问存储在R2中的对象
- 自定义域名：将R2绑定到一个自定义域名上，通过该域名访问存储在R2中的对象

公开访问的官方文档：

- Public buckets: https://developers.cloudflare.com/r2/data-access/public-buckets/

### Public Development URL

Public Development URL为Cloudflare官方默认分配的访问地址，自带Rate Limit访问限额，只适合小规模的开发调试，不适用于商业网站的高强度访问。考虑到使用场景是一个月也用不到几次的个人博客，访问强度不会太大，因此这个默认分配的URL其实完全够用了。

开启Public Development URL，需要完成以下步骤：

1. 进入 `R2 Object Storage`
2. 选择 bucket
3. 打开 `Settings`
4. 找到 `Public Development URL`
5. 点击 `Enable`

开启后会获得一个地址，形式通常类似：

```text
https://pub-xxxxxxxx.r2.dev
```

如果图片资源的相对路径是：

```text
/images/image_01.jpg
```

那么完整的访问地址就是：

```text
https://pub-xxxxxxxx.r2.dev/images/image_01.jpg
```

### 自定义域名

如果有运营商业网站的需求，或者需要更高的访问限额，可以选择绑定一个自定义域名。绑定自定义域名后，可以开启Caching、Access Control等高级功能，不过对于个人博客的使用，这些功能暂时不是很必要，所以这里就不继续深入研究了。

绑定自定义域名，需要完成以下步骤：

1. 进入 bucket 的 `Settings`
2. 找到 `Custom Domains`
3. 点击 `Add`
4. 输入子域名，例如 `assets.example.com`

注意：

- 这个域名必须已经在当前 Cloudflare 账号中
- 必须由 Cloudflare DNS 托管

## 官方文档汇总

- R2介绍: https://developers.cloudflare.com/r2/how-r2-works/
- R2定价: https://developers.cloudflare.com/r2/pricing/
- 创建Bucket: https://developers.cloudflare.com/r2/buckets/create-buckets/
- 上传对象: https://developers.cloudflare.com/r2/objects/multipart-objects/
- CLI: https://developers.cloudflare.com/r2/get-started/cli/
- S3 API: https://developers.cloudflare.com/r2/api/s3/
- 公开访问: https://developers.cloudflare.com/r2/data-access/public-buckets/
- Limits: https://developers.cloudflare.com/r2/platform/limits/
- R2与Cache: https://developers.cloudflare.com/cache/interaction-cloudflare-products/r2/
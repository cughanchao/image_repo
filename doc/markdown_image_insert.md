# 使用GitHub和PicGo搭建markdown图床



## 为什么使用GitHub作为图床

markdown可以很方便的进行文档写作，被广泛应用于技术博客创作。但是插入图片一直是markdown的痛点。

通解决方案法是把图片上传到服务器，然后将图片的链接插入文档，从而解决文档在不同设备阅读或者分享的问题。所以需要图床来保存图片返回图片链接。markdown图床的方案有很多，可以购买专业的服务器，也可以使用免费的代码仓自己搭建，具体可以见参考链接中的介绍。

本文选择**免费**的GitHub代码仓搭建图床。对应的还有Gitee代码仓，它的服务器在国内，速度快，但是对图片有1MB大小的限制，因此基本不可用。GitHub也有访问速度慢的问题，因此需要使用CDN加速，不然大一点的图片加载很慢，常用的方案是使用免费的CDN服务[jsDelivr](https://www.jsdelivr.com/)，该服务加速效果好，免费且配置简单。

>**gitee文件大小有1mb限制, 所以超过1mb的文件无法通过外链获取**（[zhanghuid/picgo-plugin-gitee](https://github.com/zhanghuid/picgo-plugin-gitee)）



## PicGo

有了图床之后，我们在写作的时候需要上传图片，然后得到图片的链接，再插入文档中。如果每张图片都这么手动操作则效率低下，我们需要一个工具帮我们快速上传照片返回链接，这个功能就是[Molunerfinn/PicGo](https://github.com/Molunerfinn/PicGo)。PicGo是一款夸平台的开源软件，还能跟Typora联合使用，使得插图一气呵成。



## 方案

GitHub创建仓库，创建三个分支，main、dev、test，把笔记存在test分支。





## 参考：

[markdown百度百科](https://baike.baidu.com/item/markdown)

[markdown图床](https://www.jianshu.com/p/ea1eb11db63f)

[Markdown写作必须知道的图床知识](https://blog.csdn.net/weixin_44441012/article/details/88016189)

[使用免费CDN服务加速Github博客](https://blog.csdn.net/qq_38877888/article/details/103995674)

[jsDelivr](https://www.jsdelivr.com/)

[PicGo官方文档](https://picgo.github.io/PicGo-Doc/)

[PicGo官方文档：配置GitHub图床](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A)


---
title: "Bing高级搜索语法"
date: 2023-09-25T15:17:59+08:00
categories:
- category
- subcategory
tags:
- tag1
- tag2
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---
日常使用关键字加空格的方式搜索，一般都能满足需求。
<!--more-->

## 使用以下关键字可以缩小搜索范围：
　　关键字定义示例

    contains:只搜索包含指定文件类型的链接的网站。若要搜索包含MicrosoftWindowsMediaAudio(.wma)文件链接的网站，请键入：音乐contains:wma。

    filetype:仅返回以指定文件类型创建的网页。若要查找以PDF格式创建的报表，请键入主题，后面加filetype:pdf。

    inanchor:、inbody:、intitle:这些关键字将返回元数据中包含指定搜索条件(如定位标记、正文或标题等)的网页。为每个搜索条件指定一个关键字，您也可以根据需要使用多个关键字。若要查找定位标记中包含msn、且正文中包含seo和sem的网页，请键入inanchor:msninbody:seoinbody:sem。

    ip:查找托管在特定IP地址 的网站。IP地址必须是以英文句点分开的地址。键入关键字ip:，后面加网站的IP地址。键入IP:207.46.249.252。

    language:返回指定语言的网页。在关键字language:后面直接指定语言代码。使用搜索生成器中的“语言”功能也可以指定网页的语言。若只需查看有关古董文物的英文网页，请键入"antiques"language:en。

    loc:或location:返回特定国家或地区的网页。在关键字loc:后面直接指定国家或地区代码。若要搜索两种或两种以上语言，请使用逻辑运算符OR对语言分组。若要查看有关美国或英国雕塑的网页，请键入sculpture(loc:USORloc:GB)。若要查看可用于Bing的语言代码列表，请参阅国家、地区和语言代码。

    prefer:着重强调某个搜索条件或运算符，以限定搜索结果。若要查找足球的相关网页，但搜索内容主要限定在某球队，请键入足球prefer:球队。

    site:返回属于指定网站的网页。若要搜索两个或更多域，请使用逻辑运算符OR对域进行分组。您可以使用site:搜索不超过两层的Web域、顶级域及目录。您还可以在一个网站上搜索包含特定搜索字词的网页。 若要在“滚来滚去，在互联网的世界里”网站上搜索有关SEO的网页，请键入site:www.tangshanseo.cnseo。

    feed:在网站上查找搜索条件的RSS(ReallySimpleSyndication)或Atom源。若要查找关于足球的RSS或Atom源，请键入feed:足球。

    hasfeed:在网站上查找包含搜索条件的RSS或Atom源的网页。若要在NewYorkTimes网站上查找包含与足球有关的RSS或Atom源的网页，请键入site:www.nytimes.comhasfeed:足球。

    url:检查列出的域或网址是否位于Bing索引中。若要验证“滚来滚去，在互联网的世界里”网站是否位于索引中，请键入url:feedbb.com。
    　　注意：在这些关键字中，请勿在冒号后面接空格。

## 一. 基本语法

1.intext

把网页中的正文内容中的某个字符做为搜索条件。例如在google里输入:intext:38.5.将返回所有在网页正文部分包含”38.5”的网页

2.allintext

使用方法和intext类似

3.intitle

搜索网页标题中是否有我们所要找的字符。例如搜索:intitle:38.5.将返回所有网页标题中包含”38.5”的网页.同理allintitle:也同intitle类似

4.cache

搜索google里关于某些内容的缓存,有时候往往能找到一些好东西

5.define

搜索某个词的定义,例如搜索:define:38.5,将返回关于“38.5”的定义。

6.filetype

搜索制定类型的文件，例如：filetype:doc.将返回所有以doc结尾的文件URL

http://7.info

查找指定站点的一些基本信息

8.inurl

搜索我们指定的字符是否存在于URL中.例如输入:inurl:admin,将返回N个类似于这样的连接:http://xxx/admin,常用于查找通用漏洞、注入点、管理员登录的URL

9.allinurl

也同inurl类似,可指定多个字符

10.linkurl

例如搜索:inurl:http://hdu.edu.cn可以返回所有和http://hdu.edu.cn做了链接的URL

11.site

搜索指定域名,如site:hdu.edu.cn.将返回所有和http://hdu.edu.cn有关的URL
## 二.基本运用

1.双引号

把搜索词放在双引号中，代表完全匹配搜索，也就是说搜索结果返回的页面包含双引号中出现的所有的词，连顺序也必须完全匹配。bd和Google 都支持这个指令。例如搜索：“seo方法图片”

2.减号

减号代表搜索不包含减号后面的词的页面。使用这个指令时减号前面必须是空格，减号后面没有空格，紧跟着需要排除的词。Google 和bd都支持这个指令。

例如：搜索 -引擎

返回的则是包含“搜索”这个词，却不包含“引擎”这个词的结果

3.星号

星号是常用的通配符，也可以用在搜索中。百度不支持号搜索指令。

比如在Google 中搜索：搜索*擎

其中的*号代表任何文字。返回的结果就不仅包含“搜索引擎”，还包含了“搜索收擎”，“搜索巨擎”等内容。

4.inurl

inurl: 指令用于搜索查询词出现在url 中的页面。bd和Google 都支持inurl 指令。inurl 指令支持中文和英文。

比如搜索：inurl:搜索引擎优化

返回的结果都是网址url 中包含“搜索引擎优化”的页面。由于关键词出现在url 中对排名有一定影响，使用inurl:搜索可以更准确地找到竞争对手。

5.inanchor

inanchor:指令返回的结果是导入链接锚文字中包含搜索词的页面。百度不支持inanchor。

比如在Google 搜索 ：inanchor:点击这里

返回的结果页面本身并不一定包含“点击这里”这四个字，而是指向这些页面的链接锚文字中出现了“点击这里”这四个字。

可以用来找到某个关键词的竞争对收，而且这些竞争对手往往是做过SEO 的。研究竞争对手页面有哪些外部链接，就可以找到很多链接资源。

6.intitle

intitle: 指令返回的是页面title 中包含关键词的页面。Google 和bd都支持intitle 指令。

使用intitle 指令找到的文件是更准确的竞争页面。如果关键词只出现在页面可见文字中，而没有出现在title 中，大部分情况是并没有针对关键词进行优化，所以也不是有力的竞争对手。

7.allintitle

allintitle:搜索返回的是页面标题中包含多组关键词的文件。

例如 ：allintitle:SEO 搜索引擎优化

就相当于：intitle:SEO intitle:搜索引擎优化

返回的是标题中中既包含“SEO”，也包含“搜索引擎优化”的页面

8.allinurl

与allintitle: 类似。

allinurl:SEO 搜索引擎优化

就相当于 ：inurl:SEO inurl:搜索引擎优化

9.filetype

用于搜索特定文件格式。Google 和bd都支持filetype 指令。

比如搜索filetype:pdf SEO

返回的就是包含SEO 这个关键词的所有pdf 文件。

10.site

site:是SEO 最熟悉的高级搜索指令，用来搜索某个域名下的所有文件。

11.linkdomain

linkdomain:指令只适用于雅虎，返回的是某个域名的反向链接。雅虎的反向链接数据还比较准

确，是SEO 人员研究竞争对手外部链接情况的重要工具之一。

比如搜索 linkdomain:链接：http://cnseotool.com -site:链接：http://cnseotool.com

得到的就是点石网站的外部链接，因为-site:链接：http://cnseotool.com 已经排除了点石本身的页面，也就是内部链接，剩下的就都是外部链接了

12.related

related:指令只适用于Google，返回的结果是与某个网站有关联的页面。比如搜索

related:链接：http://cnseotool.com

我们就可以得到Google 所认为的与点石网站有关联的其他页面。这种关联到底指的是什么，Google 并没有明确说明，一般认为指的是有共同外部链接的网站

上面介绍的这几个高级搜索指令，单独使用可以找到不少资源，或者可以更精确地定位竞争对手，把这些指令混合起来使用则更强
## 三.谷歌黑语法

inurl:gitlab 公司 filetype:txt inurl:gitlab 公司 intext:账号 site:. http://gitee.com intext:账号 （ ftp://: 密码 地址） site:. http://gitee.com filetype:txt 账号 （ ftp://: 密码 地址） site:gitlab.*.com intext:密码 site: http://code.aliyun.com 2021 服务器地址
1.对公网的语法

inurl:备份 filetype:txt 密码 inurl:config.php filetype:bak inurl:data "index of/" xxx (mp3等等) Index of /password "Index of /" +passwd "Index of /" +password.txt inurl:phpMyAdmin inurl:ewebeditor intitle:后台管理 intitle:后台管理 inurl:admin inurl:http://baidu.com inurl:php?id=1 inurl:login site: "身份证" "学生证" "1992" "1993" "1994" "1995" "1996" "1997" "1998" "1999" "2000" site:.cn filetype:xls "服务器" "地址" "账号" site:file.*.net（.com/.cn） inurl:ali "管理平台" cache

类似于百度快照功能，通过cache或许可以查看到目标站点删除的敏感文件 也可以用来解决找目标站点的物理路径不报错，而无法找到物理路径
2.“ ” + - | AND

将要搜索的关键字用引号括起来，搜索引擎将会搜索完全匹配关键字的网页 “房产” +南京 //搜索与南京有关的房产 “房产” -南京 //搜索结果除去南京的房产 房产|酒店 //搜索房产或者酒店有关的页面 房产 AND 酒店 //搜索同时匹配房产与酒店这两个关键词的页面
3.对限定目标类型的语法

inurl:gitlab 公司 filetype:txt inurl:gitlab 公司 intext:账号 site:.http://gitee.com intext:账号 （ftp://: 密码 地址） site:.http://gitee.com filetype:txt 账号 （ftp://: 密码 地址） site:gitlab.*.com intext:密码 site:http://code.aliyun.com 2019 服务器地址
4.对限定域名的语法

site:http://xxxx.com site:http://xxxx.com filetype: txt (doc docx xls xlsx txt pdf等) site:http://xxxx.com intext:管理 site:http://xxxx.com inurl:login site:http://xxxx.com intitle:管理 site:http://a2.xxxx.com filetype:asp(jsp php aspx等) site:http://a2.xxxx.com intext:ftp://:(地址 服务器 虚拟机 password等) site:http://a2.xxxx.com inurl:file(load) site:http://xxxx.com intext:*@http://xxxx.com //得到N个邮件地址
5.个人信息

site:http://xxxx.com intext:电话 //N个电话

## bing高级搜索语法
|符号|函数|
|---|---|
|+|查找包含前面带有 + 符号的所有术语的网页。 还允许包括通常被忽略的术语。|
|" "|查找短语中的确切字词。|
|()|查找或排除包含一组单词的网页。|
|AND 或 &|查找包含所有术语或短语的网页。|
|NOT 或 –|排除包含术语或短语的网页。|
|OR 或 \||查找包含任一术语或短语的网页。|
### 注意
- 默认情况下，所有搜索都是 AND 搜索。
- 必须将 NOT 和 OR 运算符大写。 否则，必应将忽略它们作为停止字词，这些字词和数字是常见的单词和数字，为加快全文搜索速度而省略。
- 停止字词和除本主题中介绍的符号外的所有标点符号将被忽略，除非它们由引号括起或前面带有 + 符号。
- 仅前 10 个术语用于获取搜索结果。
- 以下首选顺序支持术语分组和布尔运算符：
- 由于 OR 是优先级最低的运算符，因此当与搜索中的其他运算符结合使用时，将 OR 词括在括号中。
- 此处介绍的一些特性和功能可能在你的国家/地区不可用。

|关键字|定义|示例|
|---|---|---|
|contains：|将结果重点放在具有指向指定文件类型的链接的网站上。|若要搜索包含指向媒体音频 Windows.wma (.wma) 的链接的网站，请键入 music contains：wma|
|ext：|仅返回具有指定文件名扩展名的网页。|若要查找仅以 DOCX 格式创建的报表，请键入主题，然后键入ext：docx。|
|filetype：|仅返回在指定的文件类型中创建的网页。|若要查找以 PDF 格式创建的报表，请键入主题，然后键入filetype：pdf。|
|inanchor：或inbody：或intitle：|这些关键字将分别返回元数据中包含指定术语的网页，例如网站的定位点、正文或标题。 每个关键字仅指定一个术语。 你可根据需要对多个关键字条目进行字符串处理。|若要查找定位点中包含"msn"的网页以及正文中包含术语"spaces"和"magog"，请键入inanchor：msn inbody：spaces inbody：magog。|
|ip：|查找由特定 IP 地址托管的站点。 IP 地址必须是点形象限地址。 键入ip：关键字，后跟网站的 IP 地址|键入IP：207.46.249.252。|
|language：|返回特定语言的网页。 直接在 language： keyword 之后指定语言代码。|若要仅以英语查看有关小人的网页，请键入"翻译公司"语言：en。|
|loc：或位置：|返回来自特定国家/地区的网页。 直接在 loc： 关键字后指定国家/地区代码。 若要关注两种或多种语言，请使用逻辑OR对语言进行分组。|若要查看有关来自美国或大不列颠的艺术家的网页，请键入 " (loc：US OR loc：GB) 。 有关可用于语言代码的列表必应，请参阅国家/地区、区域以及语言代码。|
|prefer：|添加对搜索词或其他运算符的强调，以帮助关注搜索结果。|若要查找与足球有关但主要与组织相关的结果，请键入"足球 prefer:球队"。|
|site：|返回属于指定网站的网页。 若要专注于两个或多个域，请使用逻辑 OR 对域进行分组。 可以使用站点：搜索深度不超过两个级别的 Web 域、顶级域和目录。 还可以搜索网站上包含特定搜索词的网页。|若要从 BBC 或 CNN 网站查看有关心疾病的网页，请键入"心 (网站：bbc.co.uk OR site：cnn.com) 。 若要在 Microsoft 网站上查找有关 HALO 电脑版本的网页，请键入site：www.microsoft.com/games/pc halo。|
|feed：|在网站上找到 RSS 或 Atom 源，获取搜索的术语。|若要查找有关足球的 RSS 或 Atom 源，请键入 feed：football。|
|hasfeed：|查找网站上包含 RSS 或 Atom 源的网页，以查找搜索的术语。|若要在包含 RSS 或 Atom 源的 New York Times 网站上查找网页，请键入site：www.nytimes.com hasfeed：football。|
|url：|检查列出的域或 Web 地址是否位于必应索引中。|若要验证 Microsoft 域是否位于索引中，请键入url：microsoft.com。|

## 参考链接
[搜索引擎高级语法 csdn](https://blog.csdn.net/qq_44761480/article/details/102985000#:~:text=%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8EBing%E5%BF%85%E5%BA%94%E9%AB%98%E7%BA%A7%E6%90%9C%E7%B4%A2%E8%AF%AD%E6%B3%95%201%20contains%3A%E5%8F%AA%E6%90%9C%E7%B4%A2%E5%8C%85%E5%90%AB%E6%8C%87%E5%AE%9A%E6%96%87%E4%BB%B6%E7%B1%BB%E5%9E%8B%E7%9A%84%E9%93%BE%E6%8E%A5%E7%9A%84%E7%BD%91%E7%AB%99%E3%80%82%20%E8%8B%A5%E8%A6%81%E6%90%9C%E7%B4%A2%E5%8C%85%E5%90%ABMicrosoftWindowsMediaAudio%20%28.wma%29%E6%96%87%E4%BB%B6%E9%93%BE%E6%8E%A5%E7%9A%84%E7%BD%91%E7%AB%99%EF%BC%8C%E8%AF%B7%E9%94%AE%E5%85%A5%EF%BC%9A%E9%9F%B3%E4%B9%90contains%3Awma%E3%80%82%202%20filetype%3A%E4%BB%85%E8%BF%94%E5%9B%9E%E4%BB%A5%E6%8C%87%E5%AE%9A%E6%96%87%E4%BB%B6%E7%B1%BB%E5%9E%8B%E5%88%9B%E5%BB%BA%E7%9A%84%E7%BD%91%E9%A1%B5%E3%80%82%20%E8%8B%A5%E8%A6%81%E6%9F%A5%E6%89%BE%E4%BB%A5PDF%E6%A0%BC%E5%BC%8F%E5%88%9B%E5%BB%BA%E7%9A%84%E6%8A%A5%E8%A1%A8%EF%BC%8C%E8%AF%B7%E9%94%AE%E5%85%A5%E4%B8%BB%E9%A2%98%EF%BC%8C%E5%90%8E%E9%9D%A2%E5%8A%A0filetype%3Apdf%E3%80%82,...%207%20prefer%3A%E7%9D%80%E9%87%8D%E5%BC%BA%E8%B0%83%E6%9F%90%E4%B8%AA%E6%90%9C%E7%B4%A2%E6%9D%A1%E4%BB%B6%E6%88%96%E8%BF%90%E7%AE%97%E7%AC%A6%EF%BC%8C%E4%BB%A5%E9%99%90%E5%AE%9A%E6%90%9C%E7%B4%A2%E7%BB%93%E6%9E%9C%E3%80%82%20...%208%20site%3A%E8%BF%94%E5%9B%9E%E5%B1%9E%E4%BA%8E%E6%8C%87%E5%AE%9A%E7%BD%91%E7%AB%99%E7%9A%84%E7%BD%91%E9%A1%B5%E3%80%82%20...%20%E6%9B%B4%E5%A4%9A%E9%A1%B9%E7%9B%AE)
[搜索引擎高级语法 知乎](https://zhuanlan.zhihu.com/p/349614983)
[bing高级搜索选项](https://support.microsoft.com/zh-cn/topic/%E9%AB%98%E7%BA%A7%E6%90%9C%E7%B4%A2%E9%80%89%E9%A1%B9-b92e25f1-0085-4271-bdf9-14aaea720930)
[bing高级搜索关键字](https://support.microsoft.com/zh-cn/topic/%E9%AB%98%E7%BA%A7%E6%90%9C%E7%B4%A2%E5%85%B3%E9%94%AE%E5%AD%97-ea595928-5d63-4a0b-9c6b-0b769865e78a)
[google 搜索语法](https://zhuanlan.zhihu.com/p/136076792)
```

1. 链接规整。
a. 链接不要用参数的模式。
比如要https://www.nbd.com.cn/jijin/01/  或：https://www.nbd.com.cn/jijin/01.html
不能 https://www.nbd.com.cn/jijin?id=01

b. 不能同一个页面有多种露出链接.
比如：https://www.nbd.com.cn/jijin/01.html,  https://www.nbd.com.cn/jijin/01/

c. 不是.html，.js这种类似链接后面要有\符号
https://www.nbd.com.cn/jijin/01 => https://www.nbd.com.cn/jijin/01/

d. 不允许用相对链接,必须用完整链接
/jin/01/ => https://www.nbd.com.cn/jin/01/
//www.nbd.com.cnjin/01/ => https://www.nbd.com.cn/jin/01/


2. 完善TDK
Title：{基⾦产品名称}基⾦-净值-收益-最新估值-代码-每经⽹ 
• Keywords：{基⾦产品名称},{基⾦产品名称}基⾦,{基⾦产品名称}净值,{基⾦产品名称}最新估值,{基 
⾦产品名称}代码 
• Description：每经⽹基⾦频道为您提供{基⾦产品名称}基⾦最新估值、净值、⾏情⾛势、经理介 
绍、收益、档案等详情，想了解{基⾦产品名称}基⾦更多详情，请登录每经⽹。

3. 要有 ⾯包屑导航内容规则为
“每经⽹⾸⻚>基⾦产品>{基⾦产品名称}”。

4. 图片，需要添加alt属性
alt=“{文章标题}”

5. 建立并随时更新sitemap

6. 频道页，文章页等提供热门文章列表(是隐藏的)。
文章取昨天浏览最高的稿件。 要24小时才变一次。

7. 频道页，增加最新咨询模块(可隐藏)
文章要取昨天最后的15篇文章，24小时不变。

8. 标题链接要是完整的，由前端截短。
<a href=’’>粉丝近千万，网红夫妇被追缴、罚款超2300万元！23人实名举报其偷逃税：他们找各种理由不开发票</a>

9. 404页面下方增加热门文章模块。
404页面下方增加整个网站的导航栏.

10.文章发布后主动调用百度搜索资源平台的接口，提交链接。

11. 对首页，频道页的分类栏，用h1,h2等标签

12.文章页加入JSON-LD


13. 文章页下方加入 上一篇,下一文章。

14. 文章页下方加入 相关文章 模块

15. 友情链接
严禁拒绝黑链、暗链、链接工厂等垃圾链接，同时不能带有Ajax，Flash，含有nofollow标签，JS，frame，iframe等网站，进行友情链接交换。

16. 文章列表页等，翻页后，title上要显示页数：  “第2页-基金频道”

17. 所有页面最好都要有pc端和移动端两种。并且两端建立关联：
• 例如，PC版⽹址为https://www.nbd.com.cn/jijing_wds/，且对应的移动版⽹址为 
https://m.nbd.com.cn/web_app/column/2013/，那么此⽰例中的注释如下所⽰： 

• 在PC版⽹⻚(https://www.nbd.com.cn/jijing_wds/) 上，添加： 
<link rel="alternate" media="only screen and (max-width: 640px)" 
href="https://m.nbd.com.cn/web_app/column/2013/" > 
<meta name="mobile- 
agent"content="format=html5;url=https://m.nbd.com.cn/web_app/column/2013/"> 
<link rel="canonical" href="https://www.nbd.com.cn/jijing_wds/"> 

• ⽽在移动版⽹⻚(https://m.nbd.com.cn/web_app/column/2013/) 上，所需的注释应为： 
<link rel="canonical" href="https://www.nbd.com.cn/jijing_wds/"> 
• 备注：以上URL请使⽤标准的PC和移动端的URL；

并且，用户请求时,服务器用user-agent来跳转到相应的端。

18. 开发热词关联的后台 (词汇 → 文章链接)，文章正文中自动根据关键词增加超链接。


```
# ImageSpider
图片抓取

`
scrapy startproject ImageSpider
`

一、首先我们创建蜘蛛项目，

命令行输入：
scrapy startproject ImageSpider
创建蜘蛛文件，最终生成的目录结构如下：
scrapy下载图片

二、定义item

既然我们要下载图片，肯定要得到图片的链接，也就是src里面的内容，因此我们需要定义一个item，用来存放图片链接，于是乎：items.py文件定义如下：
import scrapy

class ImagespiderItem(scrapy.Item):
    imgurl = scrapy.Field()
    pass
定义了一个item：imgurl，这里你随便定义！好了之后我们开始来编写我们的蜘蛛！

三、创建蜘蛛文件

我们进入spiders目录，创建：ImgSpider.py 文件，并编写爬虫：

import scrapy
from ImageSpider.items import ImagespiderItem

class ImgspiderSpider(scrapy.Spider):
    name = 'ImgSpider'
    allowed_domains = ['lab.scrapyd.cn']
    start_urls = ['http://lab.scrapyd.cn/archives/55.html']

    def parse(self, response):
        item = ImagespiderItem()  # 实例化item
        imgurls = response.css(".post img::attr(src)").extract() # 注意这里是一个集合也就是多张图片
        item['imgurl'] = imgurls
        yield item
        pass
若对上面细节不清楚，请查看：scrapy中文文档

四、图片下载中间件pipeline编写：

打开pipeline.py进行中间件编写，这里的话主要继承了scrapy的：ImagesPipeline这个类，我们需要在里面实现：def get_media_requests(self, item, info)这个方法，这个方法主要是把蜘蛛yield过来的图片链接执行下载，灰常的简单，代码如下：
class ImagespiderPipeline(ImagesPipeline):

    def get_media_requests(self, item, info):
        # 循环每一张图片地址下载，若传过来的不是集合则无需循环直接yield
        for image_url in item['imgurl']:
            yield Request(image_url)
五、设置，启动图片下载

万事俱备，接下来就启动我们的下载中间件，还有一个问题，那图片存在哪里呢？scrapy给我们提供了一个常量：IMAGES_STORE，我们只需要在settings.py里面设置即可，来我们看一下settings里面的设置：
#图片存储位置
IMAGES_STORE = 'D:\ImageSpider'
#启动图片下载中间件
ITEM_PIPELINES = {
   'ImageSpider.pipelines.ImagespiderPipeline': 300,
}
六、启动爬虫

进入ImageSpider目录，命令行输入：
scrapy crawl ImgSpider
你将会发现，图片已经乖乖的躺在：D:\ImageSpider目录里了

scrapy图片下载


总结：

scrapy下载图片和我们平时开发一模一样，唯一多的就是需要你多定义一个item保存图片的地址，这也很合理，木有图片我们怎么下载呀？然后再把地址传给自定义的pipeline下载，pipeline也非常的简单，可以看到就几行代码。

不满足的同学可能就有疑问了，为什么里面的图片名称都是些乱七八糟的字符，如果我要个性化的命名，要肿么办呢？若你有同样需求，不妨继续观看scrapy中文网下一篇文章：《scrapy图片下载（二）：scrapy图片重命名放入不同目录》

源码地址：github
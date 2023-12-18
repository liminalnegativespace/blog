+++
title = "一个搭建blog的「meta教程」"
summary = " "
image = "https://i.imgur.com/rdeOHzq.gif"
categories = [
    "故纸堆",
]
tags = [
    "技术",
    "教程",                         
]
date = "2023-12-14"
menu = "main"
+++

之所以叫“meta教程”是因为这并不是一篇完整的教程，而是对别人写的教程的实验和补充。

### Intro

我自己在搭建博客的时候用的是这篇<a href="https://chrisjhart.com/Creating-A-Simple-Free-Blog-Hugo/" target="_blank">教程</a>，用到的只有Hugo和GitHub Pages，选择的theme是<a href="https://themes.gohugo.io/themes/hugo-book/" target="_blank">Book</a>。但是因为我已经有GitHub帐号，authentication什么的也都已经设置好了，并不能完全算是从零开始，况且上面的教程里还是有很多需要在本地进行的command line操作，对没有编程和使用git经验的用户可能还是不够友好。后来发现了这篇<a href="https://allisoniscoding.vercel.app/posts/how_to_build_a_blog_with_your_browser/" target="_blank">中文教程</a>，于是就打算试试完全从零开始跟随这个教程，看看能不能成功。

### Process

为此我注册了一个新的GitHub帐号，然而第一步试图用GitHub帐号登录Vercel的时候就遇到了问题，Vercel的页面上出现了一个`Account not found`的报错信息😂···上网查了一下发现这好像是一个常见问题，于是按照这篇<a href="https://github.com/orgs/vercel/discussions/3113" target="_blank">帖子</a>里官方的回复给客服发了邮件，然后过了一会竟然就可以登录了！（不知道是因为本来就是transient error还是后台真的有人在处理···

从这之后就很顺利了，按照教程中写的成功搭建了可以公开访问的博客网站。然而到这里就又出现了一个问题，就是在没有本地环境的情况下要怎样预览新的博文？

GitHub本身可以提供一些markdown的预览，但是并不支持theme里自带的一些styling，而且在进行一些更复杂的排版或加入图片、链接等元素的时候，在发布前确认一切都可以正常显示还是很重要的。

我试图像在本地环境一样，在浏览器中GitHub workspace的terminal中直接运行`hugo server`命令来打开localhost，虽然没有报错，但是只能显示出下图这样的“毛坯网页”，而且链接也是无效的:
{{< figure src="images/hs.png" >}}
于是便想到了在教程第4步设置environment variable的时候，页面上列出了除production之外的其他环境：
{{< figure src="images/env.png" >}}
其中的Preview环境应该可以用于公开发布前的预览。那么接下来的问题就是如何把新的内容发布到Preview，而不是默认的Production环境。根据<a href="https://vercel.com/docs/deployments/preview-deployments" target="_blank">官方文档</a>的说明，只要是通过git上传到除`main`之外的其他branch上的内容，就会被发布到Preview环境里：
{{< figure src="images/doc.png" >}}
于是现在的任务就变成了新建一个branch来进行preview deployment，这一步也可以在浏览器中进行：
{{< figure src="images/branch0.png" caption="点击source control中的branch按钮，在出现的输入框中新建名为preview的branch">}}
接下来在这个branch里对博客内容进行更新，并通过commit&push上传到GitHub，完成后Vercel会自动将内容发布到Preview环境中：
{{< figure src="images/preview1.png" >}}
点进上图中最上面的条目就可以看到这次deployment的详细信息。在preview中的deployment会生成一个临时url，点击这个页面中的visit按钮就可以访问了：
{{< figure src="images/preview2.png" >}}
然而又遇到了一个问题，虽然主页可以正常显示，但主页上博文的链接点进去却会出现404。用浏览器自带的object inspector检查了一下，发现这是因为尽管现在是在Preview环境中，但页面上的链接使用的还是Production环境下的url：
{{< figure src="images/preview3.png" >}}
对比了一下我自己的blog，我觉得这是theme本身设计的问题，如果生成链接的时候使用的是relative而不是absolute path应该就不会出现这个问题了。但这个问题解决起来也很简单，只要在临时url的后面加上博文的路径就可以了：
{{< figure src="images/preview4.png" >}}
如果确认一切都可以正确显示，就可以将Preview中的内容同步到Production环境中了。这一步可以通过GitHub中的pull request来实现：
{{< figure src="images/pr0.png" caption="点击New pull request按钮">}}
{{< figure src="images/pr1.png" caption="将compare的选项设置为preview">}}
{{< figure src="images/pr2.png" caption="提交pull request">}}
{{< figure src="images/pr3.png" caption="merge pull request，这样新的内容就会出现在main branch中了">}}

当这一步完成之后，Vercel中又会自动进行deployment，将内容发布到Production环境中：
{{< figure src="images/prod0.png" >}}
这时再打开production url就会看到博文中的内容已经更新了：
{{< figure src="images/prod1.png" >}}

### Conclusion
总之我觉得这个教程写的很好，确实不需要任何本地的配置，也很容易follow，根据这篇教程我成功完成了“从零开始搭建静态博客”的实验，遇到的唯一blocker也是Vercel平台本身的问题。

这个教程中提供的方法和我自己使用的之间主要的区别就是这里用了Vercel作为host，而我用的是GitHub Pages。在这个实验中，我觉得Vercel相比GitHub Pages的pros&cons是这样的：
{{< columns >}} <!-- begin columns block -->
**Pros**
<br>
✅不需要本地配置，全部可在浏览器中完成
<br>
✅几乎不需要command line操作，non-tech友好
<br>
✅不需要额外设置Environment
<br>
✅可以使用private GitHub repository
<---> <!-- magic separator, between columns -->
**Cons**
<br>
❌不够透明，如果Vercel出了什么问题很难自己debug
<br>
❌预览需要进行branch management，不如在本地电脑上方便即时
{{< /columns >}}
关于Vercel和GitHub Pages，这里还有一个更详细的<a href="https://bejamas.io/compare/github-pages-vs-vercel/" target="_blank">对比</a>。

写到这里又意识到，如果只是不想在本地安装软件或library的话，或许通过在浏览器中的GitHub workspace进行编辑再发布到GitHub pages上的方式也能实现，只不过没有了本地电脑，该如何预览的问题就又出现了。但GitHub pages或许也有设置不同的Environment再连接到不同的branch上的方法，有时间可以试验一下···
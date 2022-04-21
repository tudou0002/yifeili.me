---
title: "算法的力量"
date: 2022-04-06T04:24:02-04:00
draft: false
tags:  ['reading']
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "一些读后感"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/tudou0002/yifeili.me/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
> 如今， 最重要的革命没有在哲学系发生，甚至也没有发生在议会和城市广场上，而是在实验室，研究机构，科技公司和数据中心里默默上演，其中大部分都涉及数字技术领域的发展。然而这些非凡的进步是在一种警惕文化，孤立智识的氛围中发生的。

最近读了Jamie Susskind写的《算法的力量：人类如何共同生存》（李大白译）这本书，有一些想写下来的东西

:bulb:**Ethics in algorithms**  
算法的公平性是很重要的，自从接触算法以来好像时不时地就被提醒这一点。Machine learning 第一节课上教授就介绍了训练集的bias会在算法预测中体现出来，提醒我们要注意ethics in algorithm. 好朋友是做matching algorithm的, 之前她跟我讲过一个matching algorithm在实际生活中可能被用到的例子：当医院里有一个新的肾源时，要怎样设计分配算法才能算是能较为公平有效地把这个分配给等待移植名单上的病人。对这个例子印象深刻是因为我以前从没想到过算法还能被用在这个地方，实际上，每当意识到算法在我意料之外的情境中影响了我所能接触到的东西时，我都会觉得有点毛骨悚然。因此我对算法总抱有一些敬畏之心。

因为研究课题的特殊性，小组里也有一个专门研究ethics的实习生姐姐，常常为我们提供一些从technical角度容易被忽视的观点。我在做data collection的时候也会考虑自己的流程有没有做到尊重别人的隐私，公平透明等，这里有一篇有关使用数据注意事项的文章，值得一读。（[‘Just What do You Think You’re Doing, Dave?’A Checklist for Responsible Data Use in NLP](https://aclanthology.org/2021.findings-emnlp.414.pdf)）

在各种算法大步前进的时候，很高兴能看到身边的人也在注重ethics，因为我觉得这是在做正确的事。

:bulb:**未来的可能性**

另外一个书中讨论的大方向是未来会被算法与数字生活影响的方面。以下是一些我觉得很有意思的想法：
- 工作能为人带来收入，地位和幸福，在机器取代人类工作的未来，失去工作的人又该如何满足自身的物质需求与精神需求。他们会在赛博空间里找到自己更热爱的生活方式吗
- 算法在决定我们对一些重要资源的获取方式上发挥了越来越大的作用。公司用算法来筛选简历，信用评分决定了贷款的额度，物品的定价由购买者的居住地... 究竟什么样的算法可以被称得上是实现了分配正义的算法
- 在虚拟世界可以实现绝对自由吗，仅存于人们脑海中的想法该不该受到法律的约束。假如一个人对虚拟空间里的虚拟人物做出了有违道德的事， 那么ta该受到谴责吗


一直很想读一些分析算法对社会的影响的书， 这本书更多的像是一块敲门砖，涵盖了政治，哲学，社会的一些议题。 因为很少读法律或者哲学相关的书， 所以书中的很多观点都很新颖，也花了一些时间理解。在以后的实际工作中，也要时刻谨记作为一个程序员的影响力，不仅仅是对当下的项目，也有对社会公平公正的思考。

又想到匡扶摇画的《创造10101010101010》里的一段话：
![截自《创造10101010101010》](/future_politics.jpg#center)
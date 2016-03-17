---
layout: post
title: "How to separate storyboard for iPhone and iPad"
date: 2016-03-17 19:29:51 +0800
comments: true
categories: 
---

##Storyboard for iPhone and iPad

在Xocde6時Apple提供了Universal Storyboard的選項讓一個storyboard可以適應任何size的裝置。但有時候還是會因為iPhone與iPad的大小不同來呈現不同的效果, 在Standford CS193 Class11中的例子就使用iPad額外添加splitView的效果。



1.增加iPad的Storyboard

  ![Create Storyboard](/images/CreateStoryboard.png)


2.取名(這裏我用`Main_iPad`)

  ![Give Name](/images/GiveName.png)


3.在Project setting中選擇Info

  ![Info](/images/Info.png)


4.右鍵新增並取名為的Main storyboard file base name (iPad)

  ![Add New Storyboard File](/images/AddNewStoryboardfile.png)


5.Value中給定你在步驟3中設定iPad storyboard的名字(這裏我的是`Main_iPad`)


6.別忘了開啟iPad Storyboard設定Initial view, 才找得到你的Viewcontroller

  ![AddInitialView](/images/AddInitialView.png)


##Reference
1.https://irawd.wordpress.com/2014/10/21/xcode-6-separate-storyboard-for-ipad-and-iphone/


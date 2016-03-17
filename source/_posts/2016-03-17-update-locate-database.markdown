---
layout: post
title: "How to Update Locate Database on OS X"
date: 2016-03-17 09:51:29 +0800
comments: true
categories: 
---

##在Mac上使用locate指令尋找檔案

最近開始學在Mac上用Terminal做事情, ㄧ直不知道怎樣找檔案最快, 直到讀到了locate這個指令。

但是第一次使用locate前需要update database, 若是沒有更新會遇到以下問題:


```
WARNING: The locate database (/var/db/locate.database) does not exist.
To create the database, run the following command:

  sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.locate.plist

  Please be aware that the database can take some time to generate; once
  the database has been created, this message will no longer appear.
```


而怎麼更新database在"Learning Unix for OS X" 這本書上並沒有講得很清楚, 因此上網google了一下。

需要執行`$sudo /usr/libexec/locate.updatedb`

讓你的Mac做第一次更新, 第一次會花些時間來建立你的database。

成功後就可以使用locate查詢, ex: `$locate test`

但是接下來每次更新都需要下相同指令`$sudo /usr/libexec/locate.updatedb`,

變得稍嫌麻煩, 可以做一些更改將指令簡化成updatedb(跟linux上相同的指令)。

`cd /usr/local/bin`

`ln -s /usr/libexec/locate.updatedb updatedb`

接下來每次更新database只需要下`updatedb`就可以了。

##Referrence
1.http://aross.se/2015/07/19/how-to-update-locate-database-on-osx.html




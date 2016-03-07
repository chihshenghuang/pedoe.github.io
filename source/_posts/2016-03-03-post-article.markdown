---
layout: post
title: "Post Article"
date: 2016-03-03 21:28:18 +0800
comments: true
categories: 
---

##前言

  成功架好blog後要做的第一件事, 就是來發表一篇文章充充版面。這篇文章中會教大家如何發表文章以及基礎的排版編輯。首先要來談談使用Octopress+Github Pages架設網頁的主要優點。 

1. GitHub是一使用Git版本控制並且提供存放軟體代碼與內容服務的平台。而Git是一免費的分散式版本控制系統, 由Linus Torvalds(Linux Kernel的創作者)在2005年左右所創造出來的, 目前是被廣泛應用的版本控制系統之一。Git的好處是可以在本機端就有一儲存庫(Repository)可進行修改編輯並運作版本控制, 不必連線到主機端取得資料, 使得開發非常快速且方便。因此使用Github來存放blog好處是可以用強大的git進行blog內容的控制, 也容易讓多個作者維護同一個blog。最棒的是可以學習熟悉目前世界上最潮的版本控制系統。

2. Octopress支援markdown語法來進行編輯, 大大簡化編寫網頁的過程。Markdown是一種輕量級的標記式語言, 目的為好讀好寫, 讓使用者免去學習HTML複雜繁長的語法由靠編寫markdown文件再轉換成HTML文件來產生網頁。這篇文章將會介紹一些常用的語法並舉簡單的例子實作markdown語法。而使用Octopress還有一個好處是內部一些方便的指令, 例如幫助你把網頁部署到github pages與local端預覽blog呈現效果等等。


##發表你第一篇文章
 
1. 進到你的Octopress資料夾內

   `$cd octopress`

2. 產生一篇名為'title'的文章

   `$rake new_post['title']`

3. 接下來你就會看到產生一個名字為時間與title的markdown文件, 類似下方的訊息

   `Creating new post: source/_posts/2016-03-04-title.markdown`

4. 產生文章後我們來預覽看看

   `$rake preview`


   ![Alt Text](/images/new_post_article.png)

   有沒有看到上面一篇名為'Title'的文章？恭喜你!在blog上發表了第一篇文章。


##編輯文章內容
要開始編輯文章, 我們必須進到文章所在的資料夾去開始編輯。

1. 預設中是放在/octopress/source/\_posts

   `$cd octopress/source/_posts`

2. 檢查文章是否存在, 輸入ls來觀看資料夾內所有內容

   `$ls`
   
   此時你應該會看到剛剛所建立的文章, 在本文的例子中是'2016-03-04-title.markdown'這篇文章

   ![Alt Text](/images/ls.png)

3. 由於我是用vim進行文章編輯, 網路上也有許多方便好用的markdwon編輯軟體, 就依照個人喜好囉。首先開啟文章進行編輯

   `$vim 2016-03-04-title.markdown`

   開啟後你應該會看到如下的畫面

   ![Alt Text](/images/content.png)

   title這行就是顯示你這篇文章的title

   data就是你產生文章的時間

   comment則是你可以選擇這篇文章是否允許評論(要能成功使別人留言還需做點額外努力, 可以自行google或是以後有機會談到blog設定時再分享給大家)

   categories則是可以添加你希望支援的語法, 例如CSS3, Sass等等

4. 接下來隨意輸入內容進行測試

   ![Alt Text](/images/input content.png)

   開始預覽後應該可以看到文章內容如下

   ![Alt Text](/images/my_first_article.png)

   這樣就完成我們的第一篇文章囉!


##Markdown Format
文章中只能寫純文字實在是太單調了, 這裡分享常用到的基本語法

1. Code blocks, 將需要加註code blocks的部分用`符號左右封裝起來

   `Code Blocks`

2. 貼上圖片
   
   圖片檔案在Local端語法為:
   
      `![Alt Text](圖片路徑)`, 其中中括號內輸入你對圖片的敘述, 而一般會將圖片放在/source/images

   圖片檔案由網路取得:

      `![Alt Text](圖片網址)`, 其中中括號內輸入你對圖片的描述

3. 網址, 網址這部分直接照打就可以了

   https://pedoe.github.io

4. header文字大小, 使用`#`符號加在標題前方例如`#header1`, 依照`#`符號的數     不同有不同的效果, 產生的效果如下
   
#header1

###header1

還有許多語法在這邊沒有詳細列出, 有需要可以上官網查詢。配合這些功能就可以依照自己喜歡的樣子排版編輯你自己的文章, 別忘了編輯好後還要下命令讓網頁產生以及上傳, 其他人才能看到你blog的更新。

##Reference
1. http://octopress.org/docs/blogging/
2. https://guides.github.com/features/mastering-markdown/
3. http://markdown.tw















   

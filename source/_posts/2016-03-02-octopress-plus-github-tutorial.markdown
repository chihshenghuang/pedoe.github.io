---
layout: post
title: "Octopress + GitHub Tutorial"
date: 2016-03-02 23:38:45 +0800
comments: true
categories:
- CSS3
- Sass
- Media Queries
---

##前言
  當初查找資料時。 看到許多人的blog都是簡潔乾淨的風格, 查詢之後才知道這是Octopress開源框架加上GitHub Pages服務來建立的blog。 在安裝搭載blog時也遇到了不少問題, 所以將整個流程記錄下來, 希望能幫助跟我一樣遇到困難的人。


##基本設定
1. 安裝Xcode Command Line Tool:

   `$xcode-select --install`

   (否則在octopress下install bundle 會產生error:`Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.`)

2. 檢查Ruby版本是否為2.0

   `$ruby --version`

   若沒有Ruby的話請google如何安裝(Mac OS X 10.10 已內建Ruby)



##建立Github Repository
在GitHub內create new repository, Repository name請打"yourname.github.io",
   ex:我的github帳號為pedoe, repository name:pedoe.github.io(帳號與repository name須一致, 一開始我隨便打下場很慘)
  
   ![Alt Text](/images/github_repository.png)
   
   Create成功後請記下SSH位置, ex:git@github.com:pedoe/pedoe.github.com.git
   


##安裝Octopress
1. Clone Octopress
   
   `$git clone git://github.com/imathis/octopress.git octopress`

2. 進入clone下來的octopress file
   
   `$cd octopress`

3. 安裝octopress所需的dependencies
   
   `$gem install bundler`
   
   `$bundle install`

4. 安裝octopress默認的主題

   `$rake install`

5. Local端安裝完畢, 可以在octopress file內執行命令
   
   `$rake preview`,
   
   接著開啟web browser輸入網址http://localhost:4000/ , 就可以看到初步的樣子囉!



##SSH Key 
1. 產生ssh key:(see tutorial) https://help.github.com/articles/generating-a-new-ssh-key/

2. 增加ssh key到github上:https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/

3. Add ssh key to ssh-agent:https://help.github.com/articles/adding-a-new-ssh-key-to-the-ssh-agent/



##Generate and Depoly Web
1. 在Octopress file內設定got pages
   
   `$rake setup_github_pages`
   
   輸入完command後會要求你輸入ssh位址, 將github上創建完成的repository位址輸入(步驟3中所得到的)

2. 生成頁面
   
   `$rake generate`

3. 部署頁面

   `$rake deploy`

   但是輸入此命令後我遇到push的問題, 出現error如下

   `![rejected]master -> master (non-fast-forward)`

   這個問題google過後得到http://stackoverflow.com/questions/21356212/failed-to-deploy-to-github-pages-using-octopress, 我照著他的做法解決了, 步驟列在下方

4. 處理git的問題:

   `$git config branch.master.remote origin`
   
   `$git config branch.master.merge refs/heads/master`
   
   `$git pull`

5. 再一次部署頁面
  
   `$rake deploy`

   等個幾分鐘後在browser輸入"http://urname.github.io", 像我的就是http://pedoe.github.io , 輸入完成後應該就能看到你的blog囉!

6. 將生成的code上傳至github中:
   
   `$git add .`
   
   `$git commit -m 'create blog'`
   
   `$git push origin source`

7. 準備開始寫文章吧!



##Reference
1. http://octopress.org
2. http://msching.github.io/blog/2014/04/11/starting/
3. http://shengmingzhiqing.com/blog/setup-octopress-with-github-pages.html/




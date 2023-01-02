---
title: 持续集成和Jenkins杂谈
date: 2018-11-28 14:35:09
tags: 
- Jenkins
- CI/CD
categories:
---
关于持续集成/持续部署/持续交付的认识和看法，以及我设计的一个CI/CD流程。
<!-- more -->
## 持续集成、部署和交付

在写Jenkins的时候觉得还是把CI、CD和CD说一下吧。
- 持续集成(Continuous Integration)
- 持续部署(Continuous Deployment)
- 持续交付(Continuous Delivery)

开发工作流程一般是
> ①code -> ②build -> ③integrate -> ④test -> ⑤delivery -> ⑥deploy


[持续集成]对应①到④，我对持续集成的理解是利用CI工具对提交的代码进行及时的集成、**测试**(测试真的很重要)，频繁的集成可以帮助开发者在代码开发阶段提前发现问题，发现问题的时间越早付出的代价就越小。

[持续交付]对应①到⑤，持续集成是持续交付的子集，在持续交付阶段需要将代码部署到类生产环境中，比如在持续交付过程中连接数据库做更多更丰富的测试,缩短迭代周期。

[持续部署]对应①到⑥，持续部署则是持续交付后面接着部署到生产环境，但是想要做到持续部署的全自动化还是很难的，因为要经历很多不同的环境，这些环境的搭建配置和管理都很复杂。

也有把这三个统称为持续集成的说法，只是最后部署与否的区别，喊CI又不准确，CI/CD比较合适吧，哈哈哈。

#### 常用的集成工具
- Travis-ci，是一个在线托管的CI服务，Github的好基友，每次commit都会触发Travis重构，开源项目免费，私人项目就有点贵了。使用Travis构建项目的时候还可以生成覆盖率报告等，最后你的repo上会有很多看起来很高大上的徽章。
- Jenkins，Jenkins是一个用Java编写的集成工具,需要自己本地安装，像我这种普通玩家集成小项目用Jenkins就很舒服，不过很吃内存，我的云服务器升级到2G内存才能玩得动，现在还不敢用Jenkins去集成比较大的项目。

#### 对于CI/CD的理解
首先，一定要写好测试! * 3

其次，CI/CD是**提高代码开发效率的手段**，**而不是目的**，在实际使用中，选择自己喜欢的、用得顺手的、适合项目的工具并且根据需求做到④⑤⑥中的任一步，一般来说做到⑤就可以了。

最后，在玩CI/CD的时候也看看Docker，Docker可以提供的是干净的测试环境也方便之后的部署。


## Jenkins

> Jenkins是一个用Java编写的开源的[持续集成](https://zh.wikipedia.org/wiki/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88)工具


![LOGO](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/Jenkins_logo_with_title.svg/1200px-Jenkins_logo_with_title.svg.png)

官网奉上[https://jenkins.io/](https://jenkins.io/),Jenkins使用范围很广，文档和教程很全，拥有众多插件，企业级使用一般是基于Jenkins1.0进行二次开发来适应相应的构建流程（有空研究一下二次开发）。

在2018年上半年因为实习的缘故与Jenkins结缘，从此开始折腾，Jenkins的文档又臭又长，推荐看一遍Jenkins的文档目录，然后在自己的机器上或者是云服务器上自己安装一个Jenkins慢慢折腾，遇到问题再查[文档](https://jenkins.io/doc/)。


#### 安装配置
Linux下添加apt源的方式，或是下载war包部署到Tomcat中。

安装运行后，默认运行端口是8080，访问`serverurl:8080`,会有一个unlock阶段，新建用户需要管理员权限，管理员密码在`cat /var/lib/jenkins/secrets/initialAdminPassword`中，不熟悉插件的话就按照推荐的安装一遍，使用教程不赘述，下面说一下Jenkins可以被用来做什么。

#### 使用流程
使用Jenkins集成测试项目时需要在Github中设置Webhook，每次Commit会发一个带有项目更新信息的Post给Jenkins服务器，要求Jenkins服务器的必须在公网下。Jenkins服务器接到了Webhook后拉去相应分支代码，在本地测试，docker构建镜像然后运行，可以很方便地看到每次提交后代码的构建状态和构建历史。

在使用的时候我比较喜欢BlueOcean——Jenkins的一个UI插件，美观顺手无脑，主要用来构建Pipeline项目，Pipeline的构建是根据Jenkinsfile走的，类似于Dockerfile，如果原先项目代码中没有Jenkinsfile，则会通过可视化操作引导创建一个使用流程，最后自动生成一个Jenkinsfile，也可以手动编写。Jenkinsfile使用groovy语法，指定agent、创建stages、执行steps等，下面是一个简单案例
```
pipeline{
    agent : any
    stages{
        stage('start'){
            sh 'echo hello,'
        }
        stage('end'){
            sh 'echo world'
        }
    }
}
```

Jenkins通过自定义的流程可以完成很多事，官方文档中包括[Build a Java app with Maven](https://jenkins.io/doc/tutorials/build-a-java-app-with-maven)、[Build a Node.js and React app with npm](https://jenkins.io/doc/tutorials/build-a-node-js-and-react-app-with-npm)、[Build a Python app with PyInstaller](https://jenkins.io/doc/tutorials/build-a-python-app-with-pyinstaller)，这些都是一套完整的流程，一些可以插入的额外功能比如说设置环境变量、邮件提示、记录测试结果等。

使用的代码仓库github也好，自建的gitlab也行，设置好webhook就行。

![](/img/Use-Jenkins/process.png)

使用阿里云的镜像仓库和Docker Swarm搭建服务（适用于中小型企业）的流程基本上如上图所示。Jenkins根据分支选择条件将打包的镜像推送到对应的镜像仓库，设置推送镜像自动触发构建，Docker Swarm就拉取镜像构建了。ps.阿里云的镜像仓库和Docker Swarm之间可以走内网，省一些流量费用。

关于这个模型的反思，根据[关于两种CI/CD策略以及git分支模型的思考 – Bu・log](https://yaowenjie.github.io/devops/thinking-in-two-kinds-of-ci-cd-strategies-and-git-branch-models)，模型应该适当拉长流程，分隔出一个测试环境，理论来说，为了项目镜像的完全正确性，需要每次初始化一个新的环境进行测试，会耗费大量时间延长构建过程的周期，这里的取舍就见仁见智了。

#### 注意点
1. 修改Jenkins默认端口
    ```
    vi /etc/sysconfig/jenkins 
    // 修改JENKINS_PORT为8081 
    JENKINS_PORT="8081" 
    ```
2. Jenkins分为master节点和slave节点，master分配任务给slave，在构建任务并发的时候缓解压力。
3. 可以使用参数化构建，根据变量回滚前几次的构建情况（没试验过）。参考[使用jenkins进行项目的自动构建部署&&回滚 - 简书](https://www.jianshu.com/p/dceaa1c7bb49)


#### 参考文章
[谈谈持续集成，持续交付，持续部署之间的区别](https://segmentfault.com/a/1190000006166538)

[持续集成系统的演进之路](http://jolestar.com/ci-teamcity-vs-jenkins/)

[关于两种CI/CD策略以及git分支模型的思考 – Bu・log](https://yaowenjie.github.io/devops/thinking-in-two-kinds-of-ci-cd-strategies-and-git-branch-models)

[Who is using Jenkins? - Jenkins - Jenkins Wiki](https://wiki.jenkins.io/pages/viewpage.action?pageId=58001258)

[使用jenkins进行项目的自动构建部署&&回滚 - 简书](https://www.jianshu.com/p/dceaa1c7bb49)


#### 拓展阅读
[前端自动化测试探索](http://fex.baidu.com/blog/2015/07/front-end-test/)
    
#### Log

[2018-5-5]最近在忙毕设，我先把坑占好，原先Jenkins的东西就很多很杂，不然等做完毕业设计啥都不想干了全忘了那岂不是亏大发了。虽然我不是一个经验丰富的开发者，接触过的持续集成工具目前只有Jenkins，但是我希望我的想法能给你带来一些帮助。之后我也会去使用Travis-ci，对比一下,补充本文内容。
[2018-5-9]添加了一些内容，但是还没有补全，未完待续。。。但是写着写着就像是使用教程了，之后应该还会修改，写博客的目的是为了传达idea，而不是简单的教程，警！限于认识的局限性，我对workflow的认识还很浅，暂时就没有写上去了，希望在后续的学习中能够弄明白，更新待续。。
[2018-8-15]摸鱼了快两个月，好多东西都忘得差不多了，趁现在赶紧修改文章。在写博客的时候，想到什么写什么，很多地方不是使用过且深有体会的人是没办法跟上的，就先写成这样，之后多练练文笔，好好组织一下文章也许会好些。
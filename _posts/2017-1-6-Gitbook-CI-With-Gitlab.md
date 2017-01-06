---
layout: post
title: Gitbook CI With Gitlab
---

Previously the document engineers of our group use MS Word to write and publish products documents. They want me to work out a solution to let them write with markdown, manage versions with git and publish using gitbook. After days of google and trying, I was finally able to set up a Gitbook+Gitlab CI enviroment to do the exact thing they want. Here's the steps.

## Enviroment

I use two hosts have centos 7 installed, one for gitlab and another for gitlab runners and apache which will be used to host the gitbook.

| host |  |
| :--- | :--- |
| 10.0.0.3 | gitlab |
| 10.0.7.237 | apache, runners |

## Requisitions

[install](https://about.gitlab.com/ "install") gitlab on 10.0.0.3, then create an project named **gitbook-test**

![](/images/2017-1-6-Gitbook-CI-With-Gitlab/create01.png)

install git, apache, nodejs on 10.0.7.237, for Centos it's like:

```
sudo yum install git httpd npm -y
```

similar for the other systems.

## Set up repository and file structure

Clone you repository to local.

```
git clone http://10.0.0.3:8088/lijingjing/gitbook-test.git
cd gitbook-test
```

then create gitbook file structure, here I use gitbook to do the initilization works for me, you could use whatever tools you like or simple write static htmls directly which do not neet any build tools.

```
npm install gitbook-cli
gitbook init
```

now you directory may look like:

```
[root@host-10-0-7-237 gitbook-test]# tree
.
├── README.md
└── SUMMARY.md

0 directories, 2 files
```

## Set up `.gitlab-ci.yml`

Gitlab uses .gitlab-ci.yml script to describe build process. A common .gitlab-ci.yml usually contains three parts: build, test and deploy. Now here we need to build with`gitbook build`, do nothing in test and deploy our built htmls to apache web server.

```
build:
    stage: build
    artifacts:
        paths:
         - _book
    script: 
     - npm install gitbook-cli && node node_modules/gitbook-cli/bin/gitbook.js build
    tags:
     - build-capable

test:
    stage: test
    script: 
     - echo "no tests."
    tags:
     - test-capable

deploy:
    stage: deploy
    script:
     - rm -rf /var/www/html/gitbook-test/*
     - cd _book
     - cp -rf . /var/www/html/gitbook-test/
     - echo "this is where it'll distribute itself wherever is necessary!"
    tags:
     - deploy-capable
    only: 
     - master
```

then commit our changes and push to gitlab server.

## Set up gitlab runners

Now open you project page in the brower, you'll see a icon besides the name of your project:

![](/images/2017-1-6-Gitbook-CI-With-Gitlab/pending.png)

that because we told gitlab we need 3 runners, build capable, test-capable and deploy-capable , to do our CI job, but we haven't set up any runners currently. So just set up 3 runners on host 10.0.7.237 by running `gitlab-ci-multi-runner register` 3 times\(here we use ssh runners\).

Now you get 3 runners set up in you gitlab and you could see the status of CI Job become running or passed if you're not quick enough.

![](/images/2017-1-6-Gitbook-CI-With-Gitlab/runners.png)

![](/images/2017-1-6-Gitbook-CI-With-Gitlab/running.png)

![](/images/2017-1-6-Gitbook-CI-With-Gitlab/passed.png)

if the status is **failed, **just click it to see the details.

now your gitbook has been built and deployed to the apache web server on 10.0.7.237, open your browser and navigate to [http://10.0.7.237/gitbook-test/](http://10.0.7.237/gitbook-test/), you'll see your gitbook here. Have fun!

![](/images/2017-1-6-Gitbook-CI-With-Gitlab/gitbook.png)


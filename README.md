# docker-jenkins
<br>

# 使い方

## 1.リポジトリをクローンする

https://github.com/f4samurai/docker-jenkins 

<br>

## 2. Jenkins masterを起動し、初期設定をする

### 2-1.masterのコンテナを起動し、Jenkinsの初回起動をする

$ cd docker-jenkins/docker

$ docker-compose -f docker-compose-master-init.yml up -d

ブラウザでhttp://localhost:8080/にアクセスし、Unlock Jenkinsの画面が表示されるまで待つ

<br>

### 2-2.初期設定をする

Administrator passwordの入力を求められるので以下を実行する

$ docker-compose exec master /bin/bash -c 'cat /var/jenkins_home/secrets/initialAdminPassword'

標準出力されるパスワードをコピーする

<br>

ブラウザに戻り、入力欄にペーストし、Continueを押す

Customize Jenkinsの画面が表示されるので、Install suggested pluginsを押す

Create First Admin Userの画面が表示されるので、Adminユーザーの情報を設定し、Save and Continueを押す

Instance Configurationの画面が表示されるので、http://JenkinsホストのIPアドレス:8080/を入力し、Save and Finishを押す

Jenkins is ready!の画面が表示されるので、Start using Jenkinsを押す

<br>

## 3.Jenkins agentを作成する

### 3-1.masterからagentにSSH接続するためのSSHキーペアを作成する

$ docker-compose exec -u jenkins master /bin/bash -c 'ssh-keygen -t rsa -N "" -f /var/jenkins_home/.ssh/agent001_id_rsa'

<br>

### 3-2.公開鍵をagent起動時に環境変数で渡せるようにする

以下を実行して作成したキーの公開鍵を標準出力し、コピーする

$ docker-compose exec -u jenkins master /bin/bash -c 'cat /var/jenkins_home/.ssh/agent001_id_rsa.pub'

コピーした値をdocker-jenkins/docker/docker-compose.ymlのJENKINS_AGENT_SSH_PUBKEYの右辺にペーストする

JENKINS_AGENT_SSH_PUBKEY=ここにペースト

<br>

### 3-3.agentのコンテナを起動する

$ docker-compose up -d

<br>

### 3-4.Jenkins master上でagentを作成する

#### 3-4-1.新規ノードの作成

[Jenkinsの管理] > [ノードの管理] > [新規ノードの作成]

ノード名をagent001、TypeのPermanent Agentを選択し、Createを押す

<br>

#### 3-4-2.ノード名、リモートFSルートを設定

ノード名：agent001

リモートFSルート：/var/jenkins_home

その他は自由

<br>

#### 3-4-3.起動方法を設定

起動方法：SSH経由でUnixマシンのスレーブエージェントを起動

ホスト：agent001

<br>

#### 3-4-4.認証情報を新規作成する

[認証情報] > [追加]

種類：SSHユーザー名と秘密鍵

ユーザー名：jenkins

以下を実行し標準出力される値をコピーし、[直接入力] > [鍵]にペーストする

$ docker-compose exec -u jenkins master /bin/bash -c 'cat /var/jenkins_home/.ssh/agent001_id_rsa' 

[追加]を押す

<br>

#### 3-4-5.[認証情報]で今作成したものを選ぶ

<br>

#### 3-4-6.agentのJavaのパスを指定する

[高度な設定] 

以下を実行し標準出力される値をコピーし、[高度な設定] > [Javaのパス]にペーストする

$ docker-compose exec -u jenkins agent001 /bin/bash -c 'which java'  

<br>

#### 3-4-7.新規ノードの作成完了

[保存]を押す

<br>

## 4.Jenkins masterからJenkins agentが接続できていることを確認

[Jenkinsの管理] > [ノードの管理] でagent001が起動できていることを確認

<br>
<br>

# 参考資料

## Configuration as Code

https://www.m3tech.blog/entry/2020/11/27/110000 

https://www.kaizenprogrammer.com/entry/2018/09/23/030646 

## Masterについて

https://hub.docker.com/r/jenkins/jenkins

https://github.com/jenkinsci/docker/blob/master/README.md#plugin-installation-manager-cli 

https://github.com/jenkinsci/docker/blob/master/README.md#preinstalling-plugins

## Slaveについて

https://kacfg.com/aws-ec2-docker-jenkins 

https://qiita.com/robozushi10/items/0298a01cdffcf835fe38 

上の記事に出てくるjenkinsci/ssh-slaveはJavaのバージョンが古くてすんなり使えなかったので、
jenkins/ssh-slaveを採用

https://hub.docker.com/r/jenkins/ssh-slave

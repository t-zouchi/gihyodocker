## コンテナを上げる 
docker-compose up -d

docker container ls

##  swarm を起動
docker container exec -it manager docker swarm init

join token
--token SWMTKN-1-22w3y7skgc0sne2krplnrrbvqc7r7vkfqz1v84vit7lkq9m3y5-0ixp6daifxn737qkxl0dciuhz 172.18.0.3:2377


worker01~03 追加
docker container exec -it worker03 docker swarm join --token SWMTKN-1-22w3y7skgc0sne2krplnrrbvqc7r7vkfqz1v84vit7lkq9m3y5-0ixp6daifxn737qkxl0dciuhz 172.18.0.3:2377


## manager 確認
docker container exec -it manager docker node ls

## todoapp ネットワークを作成
docker container exec -it manager docker network create --driver=overlay --attachable todoapp 

## tododb 
tododbをビルド、 ch04/tododb:tatest としてpush

docker image build -t ch04/tododb:latest .
docker image tag ch04/tododb:latest localhost:5000/ch04/tododb:latest
docker image push localhost:5000/ch04/tododb:latest

todoappを swarmつかってデプロイ

docker container exec -it manager vi /stack/todo-mysql.yml

docker container exec -it manager docker stack deploy -c /stack/todo-mysql.yml todo_mysql
確認 
docker container exec -it manager docker service ls

## 初期データ投入
docker container exec -it manager docker service ps todo_mysql_master --no-trunc --filter "desired-state=running"

コンテナの確認
ID                          NAME                  IMAGE         NODE                DESIRED STATE       CURRENT STATE           ERROR        PORTS
bmwo02qwz0zc6cqndf5dddcs9   todo_mysql_master.1   registry:5000/ch04/tododb:late
st@sha256:95e311680cb8a66f0e3398bd415184b123daf831a622c84b18350e95eb11c4ab   57061d5237d0        Running             Running 5 minutes ago

コンテナに入る
docker container exec -it 57061d5237d0 docker container exec -it todo_mysql_master.1.bmwo02qwz0zc6cqndf5dddcs9 bash 
↑めんどいのでシェル芸でコマンドを作る

docker container exec -it manager docker service ps todo_mysql_master --no-trunc --filter "desired-state=running" --format "docker container exec -it {{.Node}} docker container exec -it {{.Name}}.{{.ID}} bash"

docker container exec -it 162095d62521 docker container exec -it todo_mysql_master.1.oghabvog3s51adqz04y27iiur bash

docker container exec -it 2054c1bf4ebd docker container exec -it todo_mysql_master.1.vs0pndg1v56y62csv0rbg16kk bash

user/local/bin/init-data.sh
mysql -u gihyo -pgihyo tododb
``` 文字コードのエンコードエラーで中身確認できてないけど多分入ってるから きにすんな ```

### slaveにもデータが入っているか確認する
docker container exec -it manager docker service ps todo_mysql_slave --no-trunc --filter "desired-state=running" --format "docker container exec -it {{.Node}} docker container exec -it {{.Name}}.{{.ID}} bash"

docker container exec -it 2054c1bf4ebd docker container exec -it todo_mysql_slave.1.ke576jteabe43uk3hzfl93rwq bash
docker container exec -it 2054c1bf4ebd docker container exec -it todo_mysql_slave.2.vyq012jzdyhn1yz0dhfd83ddi bash


### master/slave 構成のDB構築完了


## apiサーバを立てる
api image の作成

docker image build -t ch04/todoapi:latest .
docker image tag ch04/tododb:latest localhost:5000/ch04/todoapi:latest
docker image push localhost:5000/ch04/todoapi:latest



todo-app.yml の作成

docker container exec -it manager vi /stack/todo-app.yml

docker container exec -it manager docker stack deploy -c /stack/todo-app.yml todo_app 

docker container exec -it manager docker service logs -f todo_app_api

You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD

error: database is uninitialized and password option is not specified

 2018-10-29T04:33:39.289702Z 0 [Note] mysqld (mysqld 5.7.24-log) starting as process 96 ...
2018-10-29T04:33:39.399558Z 0 [Note] InnoDB: Setting file './ibtmp1' size o 12 MB. Physically writing the file full; Please wait ...

docker container exec -it manager docker service ps todo_app_api --no-trunc --filter "desired-state=running" --format "docker container exec -it {{.Node}} docker container exec -it {{.Name}}.{{.ID}} bash"

docker container exec -it ccb3322fa65c docker container exec -it todo_app_api.1.ostrqgoxlyrpafcpdtdg5ctol bash
docker container exec -it 50b4c4510b7b docker container exec -it todo_app_api.2.tebnyd8anaq8vk8p85byk339l bash

## KeyChat
Keychat is a chat application which includes chat widget and server. It uses Matrix/Synapse as a chat engine. React is used to build a chat widget and Postgres as database. All application run as a docker container. There are four components 
– keychat-server: API server to connect main synapse chat engine and running business logic.
– keychat-widget is a UI widget written in React. This component is embedded in a website as a chat bubble to start the chat.
– keychat-web-chorus is a UI application which is used by agents/owner. Owner is a service provider hosting the website on which this chat is running. It's a fork of riot-web and customized for this project.
– keychat-notifier is a background service written in golang. It scans and processes messages from the Synapse server and saves it in the database. Websockets at client end receives these Messages.
– keychat-android: TODO


TODO's:
– Refactor code apply best REST service practises.
– Refactor code to apply best golang lang code practises
– Lots of Testing and testing various scenarios.
– Synapse is overkill for this project. Explore other chat engine.
– Work on UI redesign


KeyChat aims to provide a chat service for websites. It will show as chat bubble at bottom of website. It tries to connect end user visiting the website to **customer service of product/service website**


Installation:
All components run as docker container. Installation steps include
– Install Postgres DB and load the schema
– Install Synapse matrix implementation and upate the config
– Install and run components
– Run the demo

#### Install Postgres and create keychat db.

```
git clone    https://github.com/madhutry/keychat-db.git
cd keychat-db
docker pull postgres
rm -rf mkdir ${HOME}/dockervol/
mkdir ${HOME}/dockervol/
docker run -d \
    --name prod-pg \
    -e POSTGRES_PASSWORD=keychatusr1 \
    -v ${HOME}/dockervol/postgres-data-prod/:/var/lib/postgresql/data \
    -p 5432:5432\
    postgres
```
    
##### Check DB:
```
docker exec -it prod-pg /bin/bash
su - postgres
psql -U keychatusr1 keychatdb
\dt
```

#### Install Synapse and configure it.

```
docker pull matrixdotorg/synapse
docker run -it --rm \
	--mount type=volume,src=prod-synapse-data,dst=/data \
	-e SYNAPSE_SERVER_NAME=keychat.matrix \
	-e SYNAPSE_REPORT_STATS=yes \
	matrixdotorg/synapse:latest generate
```
##### Update Image.Update homeserver.yaml in host to enable registration
```
sudo su -
sed -i -e 's/#enable_registration: false/enable_registration: true/g' /var/lib/docker/volumes/prod-synapse-data/_data/homeserver.yaml
```
##### Create Synapse Container:

```
docker run -d --name prod-synapse \
    --mount type=volume,src=prod-synapse-data,dst=/data \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```
##### Check Synapse:
```
docker exec -it prod-synapse register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml
```

#### Componenent Installation
```
git clone    https://github.com/madhutry/keychat-server.git
git clone    https://github.com/madhutry/keychat-notifier.git
git clone    https://github.com/madhutry/keychat-widget.git
git clone    https://github.com/madhutry/keychat-web-chorus.git
git clone    https://github.com/madhutry/keychat-demo.git
```
##### Build and Run keychat api server
```
cd keychat-server/
docker build -t keychat-server  .
./createenv.sh 
docker run -it --rm -p 6060:6060 --env-file envlist.txt  --name chat-server keychat-server
```
##### Initialize Users Owner side users
```
curl --location --request POST 'http://localhost:6060/chat/registeragent'    --header 'Content-Type: application/json; charset=utf-8'    --data-raw '{         "username": "admin25",         "domainname": "localhost",         "fullname": "AdminMad25",         "type": "admin"     }'  

curl --location --request POST 'http://localhost:6060/chat/registeragent'    --header 'Content-Type: application/json; charset=utf-8'    --data-raw '{         "username": "owner25",         "domainname": "localhost",         "fullname": "Owner 25",         "type": "owner"     }'  

curl --location --request POST 'http://localhost:6060/chat/registeragent'    --header 'Content-Type: application/json; charset=utf-8'    --data-raw '{         "username": "agent25",         "domainname": "localhost",         "fullname": "Agent 25",         "type": "agent"     }'
```
##### Install Keychat Notifier component

```
cd ../keychat-notifier/
docker build -t keychat-notifier .
./createenv.sh 
docker run -it --rm --env-file envlist.txt  --name keychat-notifier keychat-notifier
```
##### Build and Run client Keychat Widget
```
cd ../keychat-widget
docker build -t keychat-widget .
./updateip.sh 

docker run -p 3000:80 -it --rm  -v ${PWD}/default.conf:/etc/nginx/conf.d/default.conf --name keychat-widget keychat-widget
```

##### Build and Run Keychat Owner side chat interface
```
cd ../keychat-web-chorus
docker build -t keychat-web-chorus .

docker run -p 8081:80 -v ${PWD}/config.json:/app/config.json keychat-web-chorus
```
##### Start Demo website
```
cd ../keychat-demo
npm install --global http-server
http-server --p 8082
```
##### Run Demo 

Open in browser the owner interface:
http://localhost:8081/
Login with owner25/keychat

In another tab open the end client interface:
http://localhost:8082

Everything is up and running. There are specialized messages which owner end can post to client like #contact to send the contact info and #rating to get to know experience of the client.







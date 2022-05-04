# companion-demo

### Prerequisite for running the demo
This demo relies on docker compose [https://docs.docker.com/compose/](https://docs.docker.com/compose/). So a recent version of docker compose should be available in order to run this demo.

### Run the demo
- clone this project in your working directory
```
git clone https://github.com/search-companion/companion-demo.git
```
- start up the companion_demo network
```
cd ./companion-demo
docker compose up
```

This starts up 4 containers and you should see the log output of the 4 containers:
 - companion_zoo: 1 zookeeper host (zoo1)
 - companion_solr: 1 solr server (solr1) with solr admin on [http://localhost:8983/](http://localhost:8983/)
 - companion_db: 1 db server (db1)
 - companion_karaf: 1 companion karaf server (karaf1) with admin on [http://localhost:8181/hawtio/](http://localhost:8181/hawtio/) (with username=karaf and password=karaf)

The companion dataimport routine imports the data from the source db (database companion on db1) into the target solr collection (collection gettingstarted on solr1) and keeps it in sync when source db is being updated.

There are 2 types of imports: 
- full import: copies all source records to the target solr collection, optionally with alias handling (new collection and alias assignment)
- delta import: copies only the changed records to the target solr collection

The full import process requires to be started manually (by starting the "data-import-full" camel route). 
The full import will create a new collection ("gettingstarted" concatenated with the timestamp), read all records from the source db and push those to the target solr collection. After the final commit, an alias "gettingstarted"
will be generated that points to the new collection in order to make it available for searching via "gettingstarted".

In the demo configuration, the delta import process will run periodically (every 20s): when 1 or more records are added to the source db, this delta import will pick up the record(s), perform some mapping towards the solr document and push the solr documents to the target solr collection.

An additional camel route is set in this demo config that adds every minute a new record in the source db. This way, the delta import functionality can be followed as records are changed in the source db.

The processes can be follewed up via the log of the karaf container.
The camel routes and the log can be accessed via:
1) the hawtio admin console on our karaf container available on [http://localhost:8181/hawtio/](http://localhost:8181/hawtio/) after login (with username=karaf and password=karaf) and selecting the "Logs" tab.
This hawtio console also allows you to inspect the details and metrics of the different camel routes via the "Camel" tab.


2) the karaf CLI client for the companion_karaf container:
```
docker exec -it companion_karaf client
```
At the prompt of the karaf client, look at the log via the "log:tail" or the "log:display" (or using the "ld" alias) command:
```
karaf@root() > log:tail
```
To view the camel routes, use the following command in the CLI:
```
karaf@root()> camel:route-list
```
At this karaf CLI, the source db and the target solr collection can be queried via jdbc within karaf:
```
karaf@root() > ds-list
karaf@root()> query db-ds "select * from items"
karaf@root()> query solr-ds "select id,date_dt,status_s from gettingstarted where id like 'item_id_*'"
```
In order to stop the companion_demo network, press ^C in the terminal.
To clean up the companion_demo network, run the command "docker compose down" in the folder where it was started.

It should be clear by now that this is only a limited overview of the capabilities. I hope this demo inspires you to take a deeper dive in the source code.



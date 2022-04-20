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

This starts up 4 containers and you should see the log output of these containers:
 - companion_zoo: 1 zookeeper host (zoo1)
 - companion_solr: 1 solr server (solr1)
 - companion_db: 1 db server (db1)
 - companion_karaf: 1 companion karaf server (karaf1)

The companion dataimport functionality imports the data from the source db (database companion on db1) into the target solr collection (collection gettingstarted on solr1).
The delta import process will run periodically (every 20s): when 1 or more records are added to the source db, this delta import will pick up the record(s), perform some mapping towards the solr document and push the solr documents to the target solr collection.
The full delta import process requires to be started manually (by starting the "data-import-full" camel route). The full import will create a new collection ("gettingstarted" concatenated with the timestamp), read all records from the source db and push those to the target solr collection. After the final commit, an alias "gettingstarted"
will be generated that points to the new collection in order to make it available for searching via "gettingstarted".

The process can be followed via the log of the karaf container.
This log can be accessed via the hawtio console on our karaf container available on [http://localhost:8181/hawtio/](http://localhost:8181/hawtio/) after login (with username=karaf and password=karaf) and selecting the "Logs" tab.
This hawtio console also allows you to inspect the details and metrics of the different camel routes.

The log and the camel routes can also be viewed via the CLI, using the karaf client for the companion_karaf container:
```
docker exec -it companion_karaf client
```
At the prompt of the karaf client, look at the log via the "log:tail" or the "log:display" (or using the "ld" alias) command:
```
karaf@root() > log:tail
```
To view the camel routes, use the following command:
```
karaf@root()> camel:route-list
```
Both the source db and the target solr collection can be queried via jdbc within the karaf client as both are configured as datasources within karaf:
```
karaf@root() > ds-list
karaf@root()> query db-ds "select item_id from items"
karaf@root()> query solr-ds "select id from gettingstarted"
```
It should be clear by now that this is only a limited overview of the capabilities. I hope this inspires you to take a deeper dive in the source code in order to explore how you can implement this into your environment.

In order to stop the companion_demo network, press ^C in the terminal.
To clean up the companion_demo network, run the command "docker compose down" in the folder where it was started.


+++
title = "WebUI"
+++

## WebUI

The Pilosa server comes packaged with in-browser WebUI.  When you run a local Pilosa server on the default host, you can access it at [localhost:10101](http://localhost:10101)
This can be used for constructing queries and viewing the cluster status.

#### Console

The [Console view](http://localhost:10101/#console) allows you to enter PQL queries and run them against your locally running server.  First you must select an Index with the Select index dropdown.  

Each query's result will be displayed in the Output section along with the query time. 

The Console will keep a record of each query and its result with the latest query on top.

![console](/img/docs/webui-console.png)

#### Cluster Admin

Use the [Cluster Admin tab](http://localhost:10101/#admin) to view the current status of your cluster.  This contains information on each node in the cluster, plus the list of Indexes and Frames.
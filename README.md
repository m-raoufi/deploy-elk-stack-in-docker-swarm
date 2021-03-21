# deploy-elk-stack-in-docker-swarm
In this blog I’ll show how we can create a centralized logging solution where we collect all our Docker logs from different containers. Here we will use the well-known ELK stack (Elasticsearch, Logstash, Kibana).

We will use docker-compose to deploy our ELK stack. Docker-compose offers us a solution to deploy multiple containers at the same time.
Prerequisites:
Docker
Docker-Compose


$ git clone
$ cd docker-elk
$ docker stack deploy -c docker-stack.yml docker-elk

The first resource which is deployed is Elasticsearch. Elasticsearch is a system for quickly analyzing and analyzing different types of data. We’re able to use a single instance and disable xpack (paying feature) by using environment variables. It is deployed in the logging-networkwhich is a Docker bridge network.

Now we will deploy Logstash. Logstash is an open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to tools like Elasticsearch.

Logstash depends on Elasticsearch which needs to be deployed first. Port 12201 is exposed and mapped on the server. Other Docker containers will send their logs to Logstash by connecting to this UDP port. As volume we will mount the logstash.conf inside the container.
We will expect Gelf input. Gelf is the Graylog Extended Log Format and is a great choice for logging from within applications. We will send our logs to our ElasticSearch container. They can communicate with each other using their service name because they are deployed in the same Docker Bridge network.

The file /usr/share/logstash/config/pipelines.yml contains the available pipelines which Logstash run. Here we can define specific files (pipelines) but we are using only one pipeline now. The default file describes the directory where our logstash.conf is available so we don’t need to update our pipeline configuration. If you want to run multiple pipelines you can add additional id’s and point to specific .conf files instead of pointing to a directory like it is now (by default). For more info, take a look here.

If you configure multiple pipelines it can be useful to debug your Logstash container. You can use docker exec to debug your Logstash container and check configuration files. This is out of scope for our current basic setup.
$ docker exec -it docker-elk_logstash_1 bash
bash-4.2$ cat /usr/share/logstash/pipeline/logstash.conf
input {
  gelf {}
}
output {
  elasticsearch {
    hosts => "elasticsearch:9200"
  }
}
The last ELK stack related service in our template is the setup of Kibana. Kibana lets us visualize our Elasticsearch data.

Kibana depends on Logstash and will expose and map port 5601. Like all other ELK stack related resources Logstash is deployed inside the logging-network. This allows the containers to communicate with each other by using service names.
Now we can test our setup. One Apache (httpd) container is already started by using Docker-compose. Users of Docker Desktop for Mac should edit the gelf-address in the docker-compose.yml to udp://host.docker.internal:12201. Check here for more information. Don’t forget to update your environment by running docker-compose up -d again.
We will start an additional Nginx container using the Docker CLI.
Users of Docker Desktop for Mac should update the gelf-address to udp://host.docker.internal:12201. Check here for more information.
$ docker run -d -p 8080:80 --log-driver gelf --log-opt gelf-address=udp://localhost:12201 nginx:latest
Let’s visit both applications in the browser.

Now Access Kibana on http://localhost:5601. Click on the left on ‘Discover’ and on ‘Create Index Pattern’. Create a pattern using ‘*’ and ‘@timestamp’.

Now click on discover and verify the logs.

We have logs for our Nginx and for our Apache container!

Nginx

Apache (httpd)
Conclusion
In this blog I showed you how easy it is to deploy a basic ELK stack in Docker to monitor your other Docker containers. The focus here was on the setup of the ELK stack and not on the pipeline or additional Logstash and Elasticsearch configurations.
I hope you enjoyed it! Feel free to ask any question if you have one!

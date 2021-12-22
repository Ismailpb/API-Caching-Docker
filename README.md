# ElastiCaching-IPstack containerization-Docker

Here we are going to setup 3 containers with same application to find out the GeoIP location of the IP address mentioned in the url by using the ports as 8081,8082,8083.

### Access Key

Before going to craete the containers we need a access key from "https://ipstack.com/" [You can generate the access key using free tier].

### Network

Next we need to create a seperate brdige network by using the below command
```
# docker network create api-net
```
### ElastiCache

Here we are going to create an  ElastiCache From AWS Console[https://console.aws.amazon.com] >> ElastiCache and create an elasticache for redis.

Once we setup the cache, we need to pull an image to create the containers. Here am going to take a simple python code to print the GeoIP location and Hostname of the container(From which container the request received).

To pull the image to local system, use the command as 
```
docker image pull ismailpb/ipstack:latest

]# docker image ls
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
ismailpb/ipstack   latest    e07a48470df2   16 months ago   101MB
```
### Creating Containers

Once the image is ready, we can move to create 3 containers for IPstack by using the below command.

```
docker container run -d --name ipstackapp1 --network api-net --restart always -p 8081:8080 -e CACHING_SERVER=ipstack-ro.nqpsgt.ng.0001.aps1.cache.amazonaws.com -e IPSTACK_KEY=xyxyxyxyxyxyxyxyxyyxyxy ismailpb/ipstack:latest

docker container run -d --name ipstackapp2 --network api-net --restart always -p 8082:8080 -e CACHING_SERVER=ipstack-ro.nqpsgt.ng.0001.aps1.cache.amazonaws.com -e IPSTACK_KEY=xyxyxyxyxyxyxyxyxyyxyxy ismailpb/ipstack:latest

docker container run -d --name ipstackapp3 --network api-net --restart always -p 8083:8080 -e CACHING_SERVER=ipstack-ro.nqpsgt.ng.0001.aps1.cache.amazonaws.com -e IPSTACK_KEY=xyxyxyxyxyxyxyxyxyyxyxy ismailpb/ipstack:latest


]# docker container ls -a
CONTAINER ID   IMAGE                     COMMAND            CREATED              STATUS              PORTS                                       NAMES
70456bd9334a   ismailpb/ipstack:latest   "python3 app.py"   39 seconds ago       Up 23 seconds       0.0.0.0:8083->8080/tcp, :::8083->8080/tcp   ipstackapp3
53b71cd78ca3   ismailpb/ipstack:latest   "python3 app.py"   About a minute ago   Up About a minute   0.0.0.0:8082->8080/tcp, :::8082->8080/tcp   ipstackapp2
d24a994aca7a   ismailpb/ipstack:latest   "python3 app.py"   4 minutes ago        Up 3 minutes        0.0.0.0:8081->8080/tcp, :::8081->8080/tcp   ipstackapp1
]#
```
Here the section "CACHING_SERVER=ipstack-ro.nqpsgt.ng.0001.aps1.cache.amazonaws.com" is the elasticache that we have created previously. You may use your own  primary endpoint elasticache in the container creation.

Now the application is available, but the url is little big as we may need to provide the port number. So to simply the procedure we are goint to craete a Load Balancer so that we can point/create CNAME for a subdomain with the Load Balancer public DNS.

### Creating Load Balancer

Before creating Load Balancer, we need to create a target group with minimum requirements. For creating target group you may login to your aws console >> EC2 >> Load Balancing >> Target Groups. From there we can create the target group and need to attach the instance(Instance that we have created the containers)and once the target group is ready just move on to create Load Balancer. Here am using the Load Balancer as "Application Load Balancer".


For creating Load Balancer log in to aws console >> EC2 >> Load Balancing >> Load Balancers. Once the load balancer is ready we can set a  CNAME record for a subdomain (The domain that we owned) to the  dns name in application load balancer.

ie 
```
 stack.ismail.host  CNAME ipstack-1XXXXYYY.ap-south-1.elb.amazonaws.com (stack.ismail.host is my subdomain).
 ```
 
 









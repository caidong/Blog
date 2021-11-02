#### 拉取镜像 

``` 	sh
sudo docker pull elasticsearch:7.8.0 
```



#### 启动容器

```sh
 ## 创建网络  
 docker network create esnet
 ## 改主机配置
 在宿主机执行：sudo sysctl -w vm.max_map_count=262144
 docker 命令增加参数：  -e ES_JAVA_OPTS="-Xms1g -Xmx1g"           
sudo docker run -d --name elasticsearch --net esnet -p 9200:9200 -p 9300:9300  \
-e ES_JAVA_OPTS="-Xms1g -Xmx1g" \
-e "discovery.seed_hosts=172.26.8.100,172.26.8.105,172.26.8.108" \
-e "node.name=es01" \
-e "cluster.initial_master_nodes=es01,es02,es03" \
-e "network.publish_host=172.26.8.100" \
elasticsearch:7.8.0



sudo docker run -d -p 9200:9200 -p 9300:9300 --name elasticsearch \
 -e ES_JAVA_OPTS="-Xms1g -Xmx1g" \
-e "discovery.seed_hosts=172.26.8.100,172.26.8.105,172.26.8.108" \
-e "node.name=es02" \
-e "cluster.initial_master_nodes=es01,es02,es03" \
-e "network.publish_host=172.26.8.105" \
elasticsearch:7.8.0



sudo docker run -d -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e ES_JAVA_OPTS="-Xms1g -Xmx1g" \
-e "discovery.seed_hosts=172.26.8.100,172.26.8.105,172.26.8.108" \
-e "node.name=es03" \
-e "cluster.initial_master_nodes=es01,es02,es03" \
-e "network.publish_host=172.26.8.108" \
elasticsearch:7.8.0
```

### 查看集群状态 

http://172.26.8.100:9200/_cat/nodes



![状态](https://gitee.com/vole_store/pic-bed/raw/master/BlogImage/image-20211101154639849.png)

## 拉取启动Kibana

```sh
docker run -d --name kibana --net somenetwork -p 5601:5601 kibana:7.8.0
```

#### 配置Kibana 文件

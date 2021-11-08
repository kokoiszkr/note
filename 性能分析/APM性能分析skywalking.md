## 安装skywalking-ui
使用docker-compose.yml文件部署
```
version: '3.3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.0.1
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
    environment:
      discovery.type: single-node
    ulimits:
      memlock:
        soft: -1
        hard: -1
  oap:
    image: apache/skywalking-oap-server:8.0.1-es6
    container_name: oap
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    environment:
      SW_STORAGE: elasticsearch
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
  ui:
    image: apache/skywalking-ui:8.0.1
    container_name: ui
    depends_on:
      - oap
    links:
      - oap
    restart: always
    ports:
      - 8081:8080
    environment:
      SW_OAP_ADDRESS: oap:12800
```
### 运行容器
运行docker-compose命令启动skywalking。启动后使用浏览器打开`http://localhost:8081`地址访问skywalking界面。
```
docker-compose up -d
```

## Python应用嵌入skywalking
### 1. 安装skywalking
参考文档 https://pypi.org/project/apache-skywalking/0.6.0/
```
pip install apache-skywalking==0.6.0
```
### 2. 应用头部嵌入skywalking

```
from skywalking import agent, config

config.init(collector='localhost:11800', service='服务名')
agent.start()
```

### 3. 修改插件增加更多信息
```
/usr/local/lib/python3.6/dist-packages/skywalking/plugins

parameter = ",".join([str(arg) for arg in args])
span.tag(Tag(key=tags.DbStatement, val=op+' '+parameter))
```

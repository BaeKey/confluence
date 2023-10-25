# confluence

默认端口: 8090

+ 最新版本: v8(8.5.2)
+ 最新的修复中文乱码问题的版本: [v7](https://github.com/haxqer/confluence/tree/latest-zh) (感谢: [sunny1025g](https://github.com/sunny1025g) for the `zh` image. [#issues/16](https://github.com/haxqer/confluence/issues/16) )

## 环境要求
- docker-compose: 17.09.0+

## 使用 docker-compose 启动

- start confluence & mysql

```yaml
version: '3.4'
services:
  confluence:
    image: shiguang2021/confluence:8.5.2
    container_name: confluence-srv
    environment:
      - TZ=Asia/Shanghai
    #      - JVM_MINIMUM_MEMORY=1g
    #      - JVM_MAXIMUM_MEMORY=12g
    #      - JVM_CODE_CACHE_ARGS='-XX:InitialCodeCacheSize=1g -XX:ReservedCodeCacheSize=8g'
    depends_on:
      - mysql
    ports:
      - "8090:8090"
    volumes:
      - home_data:/var/confluence
    restart: always
    networks:
      - network-bridge

  mysql:
    image: mysql:8.0
    container_name: mysql-confluence
    environment:
      - TZ=Asia/Shanghai
      - MYSQL_DATABASE=confluence
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_USER=confluence
      - MYSQL_PASSWORD=123123
    command: ['mysqld', '--character-set-server=utf8mb4', '--collation-server=utf8mb4_bin', '--transaction-isolation=READ-COMMITTED', '--innodb_log_file_size=256M', '--max_allowed_packet=256M','--log_bin_trust_function_creators=1']
    volumes:
      - mysql_data:/var/lib/mysql
    restart: always
    networks:
      - network-bridge

networks:
  network-bridge:
    driver: bridge

volumes:
  home_data:
    external: false
  mysql_data:
    external: false
```

- 以守护进程的方式启动 confluence & mysql

```
    docker-compose up -d
```

- 默认的 数据库(mysql8.0) 配置:

```bash
    driver=mysql
    host=mysql-confluence
    port=3306
    db=confluence
    user=root
    passwd=123456
```


## 破解 confluence

```
docker exec confluence-srv java -jar /var/agent/atlassian-agent.jar \
    -p conf \
    -m Hello@world.com \
    -n Hello@world.com \
    -o your-org \
    -s you-server-id-xxxx
```

## 破解 confluence 的插件

- 例如: 你想要破解 BigGantt 插件
1. 从 confluence marketplace 中安装 BigGantt 插件
2. 查看 BigGantt 的 `App Key` 是 : `eu.softwareplant.biggantt`
3. 然后执行 :

```
docker exec confluence-srv java -jar /var/agent/atlassian-agent.jar \
    -p eu.softwareplant.biggantt \
    -m Hello@world.com \
    -n Hello@world.com \
    -o your-org \
    -s you-server-id-xxxx
```

4. 最后粘贴生成的 licence

## `Datacenter` license

添加 `-d` 参数即可生成 `datacenter` license

```
docker exec confluence-srv java -jar /var/agent/atlassian-agent.jar \
    -d \
    -p conf \
    -m Hello@world.com \
    -n Hello@world.com \
    -o your-org \
    -s you-server-id-xxxx
```

## How to upgrade

```shell
cd confluence && git pull
docker pull haxqer/confluence:latest && docker-compose stop
docker-compose rm
```

enter `y`, then start server

```shell
docker-compose up -d
```


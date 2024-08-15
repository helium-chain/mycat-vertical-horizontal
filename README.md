- mycat - mysql 垂直拆分，全局表

| 名称 | IP | 端口 |
| :-: | :-: | :-: |
| MyCat | 172.16.0.10 | 8066,9066 |
| mysql1 | 172.16.0.101 | 3306 |
| mysql2 | 172.16.0.102 | 3307 |
| mysql3 | 172.16.0.103 | 3308 |

- 请先删除{master01,master02,master03}/data/ 文件下的所有数据(你需要保持一个空的挂载目录)。
- mycat/data 目录里是初始化sql数据。

1. 启动服务
```sh
$ docker-compose -p master01 up -d
```

2. 连接MyCat的9066端口查看运行情况。

```sh
$ mysql -h localhost -uroot -P9066 -p123456
```
```sql
-- 查看数据节点，是否所有节点都出现
show @@datanode;
-- 查看节点心跳，RS_CODE=1，则表示后端数据库连接正常
-- 如果失败，查看日期，可能是库没有创建
show @@heartbeat;
```

3. 通过客户端，比如Navicat连接MyCat。
```sh
$ mysql -h localhost -uroot -P8066 -p123456
```

4. 执行测试数据导入，在MyCat中
```sql
-- 导入表结构
source ./shopping-table.sql

-- 导入测试数据
source ./shopping-insert.sql
source ./tb_log.sql
```

5. 在MyCat中执行测试sql。
```sql
select 
    ua.user_id, ua.contact, p.province, c.city, r.area , ua.address 
from    
    tb_user_address ua ,tb_areas_city c , tb_areas_provinces p ,tb_areas_region r
where 
    ua.province_id = p.provinceid and ua.city_id = c.cityid and ua.town_id = r.areaid;

SELECT 
    order_id , payment ,receiver, province , city , area 
FROM 
    tb_order_master o, tb_areas_provinces p , tb_areas_city c , tb_areas_region r 
WHERE
    o.receiver_province = p.provinceid AND o.receiver_city = c.cityid AND o.receiver_region = r.areaid;

-- 更新
UPDATE tb_order_pay_log SET user_id = "xxx" WHERE out_trade_no = "918835712999055360";
```

6. 测试水平分片。
```sql
select * from tb_log;
```
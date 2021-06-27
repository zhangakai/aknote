#广告页缓存

1.实现
    
    1.如下思路：
    首先访问nginx ，我们可以采用缓存的方式，先从nginx本地缓存中获取，获取到直接响应
    如果没有获取到，再次访问redis，我们可以从redis中获取数据，如果有 则返回，并缓存到nginx中
    如果没有获取到,再次访问mysql,我们从mysql中获取数据，再将数据存储到redis中，返回。

    2.使用openResty
    OpenResty(又称：ngx_openresty) 是一个基于 nginx的可伸缩的 Web 平台
    相当于封装了nginx,并且集成了LUA脚本，开发人员只需要简单的其提供了模块就可以实现相关的逻辑
    
    修改配置:
    cd /usr/local/openresty/nginx/conf
    vi nginx.conf

    3.缓存载入与读取
    a.连接mysql ，按照广告分类ID读取广告列表，转换为json字符串。
    b.连接redis，将广告列表json字符串存入redis 。
    ngx.header.content_type="application/json;charset=utf8"
    local cjson = require("cjson")
    local mysql = require("resty.mysql")
    local uri_args = ngx.req.get_uri_args()
    local id = uri_args["id"]
    
    local db = mysql:new()
    db:set_timeout(1000)
    local props = {
    host = "192.168.211.132",
    port = 3306,
    database = "changgou_content",
    user = "root",
    password = "123456"
    }
    
    local res = db:connect(props)
    local select_sql = "select url,pic from tb_content where status ='1' and category_id="..id.." order by sort_order"
    res = db:query(select_sql)
    db:close()
    
    local redis = require("resty.redis")
    local red = redis:new()
    red:set_timeout(2000)
    
    local ip ="192.168.211.132"
    local port = 6379
    red:connect(ip,port)
    red:set("content_"..id,cjson.encode(res))
    red:close()
    
    ngx.say("{flag:true}")
    
    配置nginx
    server {
    listen       80; //端口
    server_name  localhost; //server 名

    location /update_content {
        content_by_lua_file /root/lua/update_content.lua; //脚本位置
    }
    }

    4.限流
    nginx提供两种限流的方式 一是控制速率 二是控制并发连接数
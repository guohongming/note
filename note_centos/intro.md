苔客小程序 
大体业务：
    注册成为途牛会员->
    成为店主-> 
    添加途牛的产品到自己的店铺->
    分享自己的店铺 和 产品 给其他人-> 
    用户从店主这里下的单会给对应的店主结算佣金

结构：
    前端：微信小程序
    网关：gateway 
    服务端：框架 springboot + mybatis
           服务注册到geteway ：etcd
           rpc：pebble 基于thrift
           数据库：mysql
           中间件：redis 消息队列
           
           
           
           
途牛国际站：
    服务开发

pc/m 列表页的开发    
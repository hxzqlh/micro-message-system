# 基于go micro的分布式即时通信系统


## 技术架构
```$xslt
1.微服务框架：go micro 基于go-plugins可插拔模式
2.服务发现：etcd
3.服务端限流：令牌桶(ratelimit)
4.熔断机制：hystrix
5.消息中间件：kafka
6.web框架：gin  gorm
7.数据库：mysql
8.token认证：jwt-go
```

## 系统模块
```$xslt
1. 用户服务:
    用户API服务 - 用户注册登录
    用户RPC服务 - 用户鉴权 网关gate调用
2. IM服务：
    IM消息发送服务 - produce消息到kafka 网关gate调用
    IM消息订阅服务 - 订阅kafka消息供websocket展示
3. 网关：
    调用rpc发送消息到kafka
    获取并绑定用户与IM websocket服务地址
```

## 系统环境要求
```$xslt
golang >= 1.14
protoc >= 3.6.1
```

## 用户服务-userservice
### 生成pb
```$xslt
cd userserver/protos
protoc --proto_path=. --micro_out=. --go_out=. user.proto
```
### 启动user rpc服务
```$xslt
cd /userserver/cmd/rpc
go run user_rpc.go -f ../config/config_rpc.json
```
### 启动user api
```$xslt
cd /userserver/cmd/api
go run user_api.go -f ../config/config_api.json
```

## IM通信服务-imservice
### 生成pb
```$xslt
cd imserver/protos
protoc --proto_path=. --micro_out=. --go_out=. im.proto
```
### 启动kafka消息推送rpc服务 供网关gateway调用
```$xslt
cd imserver/cmd/
go run rpcproducer/im_rpc.go
```

### 启动kafka消息订阅消费者服务(进程) 订阅kafka消息推送到websocket
```$xslt
cd imserver/cmd/
go run imconsumer/im_server.go -f ./config/config_im_1.json
go run imconsumer/im_server.go -f ./config/config_im_2.json
```

## 网关API-gateway 需token认证并调用userservice和imservice
### 调用rpc发送消息到kafka(/send)
```$xslt
cd /userserver/cmd/rpc
go run user_rpc.go -f ../config/config_rpc.json

cd imserver/cmd/
go run rpcproducer/im_rpc.go

cd gateway/cmd
go run api/gateway_api.go
```
### 获取并绑定用户与IM服务地址(/address)
```$xslt
cd /userserver/cmd/rpc
go run user_rpc.go -f ../config/config_rpc.json

cd gateway/cmd
go run api/gateway_api.go
```
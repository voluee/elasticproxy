# elasticproxy
elasticproxy 代理elasticsearch 提供给kibana 调用， 可以在该项目中实现传输数据的修改，例如修改请求报文以及响应报文等

## 已实现功能
- 修改请求报文支持kibana基于filebeat的offset字段做二级排序 
>> 做过elk的人应该了解kibana排序至支持到秒级别，但同一秒内出现多个日志的时候那么kibana展示的日志就会混轮，加上该代理可以解决该问题

# 使用介绍
## 二进制文件
> 前提条件 已经有elastcsearch和kibana服务， 并且日志收集器为filebeat且日志数据中有offset字段
> - 在release中下载elasticproxy可执行文件 
> - 启动elasticproxy 通过` -elastic_host` 启动参数指定elasticsearch的地址, 默认elasticproxy的端口为8899
> - 启动kibana修改kibana的配置文件中elasticsearch的地址改为elasticproxy的地址

## Docker
> - docker run -it --rm -p 8899:8899 zhangyuming/elasticproxy elasticproxy -elastic_host elastichost:9200
> - 修改kibana 指向elasticproxy地址


## kubernetes
> - 切换到elasticsearch 命名空间
> - `kubectl run elasticproxy --image=zhangyuming/elasticproxy --command -- elasticproxy -elastic_host elasticsearch:9200` 
> - `kubectl expose deployment elasticproxy --target-port=8899 --port=8899 `
> - 切换kibana elasticsearch地址指向 `elasticproxy:8899`


## 项目介绍  
- proxy 项目的一些实体以及接口定义 
- runner 真正修改报文的方法 
- vlog 日志服务 
- main.go 程序入口 
### 开发方式
例如你需要修改请求报文 
- 在runner包下创建一个结构体实体，且该实体需要实现 RequestModifyer接口, 该接口在proxy包中定义
```
type RequestModifyer interface {
	RequestModify(request *LocalRequest)
}
```
- 在runner包的init方法中注册你的实体， 当前init方法在runner包中的kibana.go 中
```
func init() {
	proxy.RegistryRequestModifyer(&kibana{})
}
```

在main.go中会逐个执行注册的服务并把修改后的报文发送出去

# faq

## grpc是什么

    rpc框架，标签是modern open-source
    目的是让c/s透明交互
    cncf项目之一

    grpc项目没有lts release版本，都是滚动升级
    可以在浏览器上使用 grpc-web
    支持多种序列化方式，默认是pb，也支持json等

## 使用grpc的好处

    低延时 + 高可伸缩 + 分布式
    移动端开发 + 云服务
    设计一个新的协议需要注意3点：精准 有效 语言
    分层设计方便扩展：认证、负载均衡、日志、监控

## 对移动端的支持

    ios android
    更少的带宽，更小的cpu消耗，更省电

## 为啥grpc比基于http2的二进制块好

    跨平台
    应用层进行交互和流控
    调用取消级联式
    负载均衡和故障迁移

## gRPC的发音

    Jee-Arr-Pee-See

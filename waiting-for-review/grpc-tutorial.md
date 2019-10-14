# c++使用grpc

  总共3步：

* proto文件中定义一个service
* pb编译proto文件，生成c++的client、server代码
* 基于上面定义的service，使用c++ grpc api实现一个小例子

  .pb.h .pb.cc message的头文件和源文件
  .grpc.pb.h .grpc.pb.cc service的头文件的和源文件

## 理解

  里面包含了一个对代码的初步了解

## server端对service的处理

  service，在rpc中指远程调用中的哪个调用。
rpc是ipc的一种，ipc是进程间调用，更具体一点：
不同机器进程间的调用，说的更简单一点：我在本机
的代码中调用一个函数，实际上这个函数的实现和执行在
服务端。这里的`函数`，有的地方叫方法，有的地方叫
`服务`，这样的函数一般功能单一，很多地方也叫
`微服务`。下面针对函数，统一叫service

  先看看服务端是如何实现service的

```c++
    class RouteGuideImpl final : public RouteGuide::Service {
    }
```

  上面也说了service的定义，在.grpc.pb.h中。
在.proto文件中定义的rpc方法，在RouteGuide::Service类中定义。
RouteGuide::Service是一个基类，实现还是在server逻辑中。
不管grpc中的逻辑是怎么样，grpc保证了client的调用，会调用
到RouteGuideImpl中的同名函数，参数类型也是一样的。

```c++
    Status GetFeature(ServerContext* context,
                  const Point* point,
                  Feature* feature) override {
    }
```

```proto
    rpc GetFeature(Point) returns (Feature) {}
```

  针对定义的rpc service，对应到c++上如上。
第一个参数应该是固定的，表示上下文；
第二个参数是函数参数；
第三个参数是函数的返回；

  在不细究grpc实现细节的基础上，服务端对rpc是这么处理的：

* 从public RouteGuide::Service派生一个子类
* 子类里定义一个方法，和proto上定义的rpc调用要一致

  这里讲了服务端处理的一半逻辑，
剩下一半是创建一个server来处理client来的请求。

  通用处理：  
  上面派生了一个子类，下面称为实现类。
* 创建一个实现对象，也称为service对象
* 创建一个server构造器对象
* 为server构造器对象指明server要监听的ip和端口，基于认证情况
* 告诉server构造器对象，有哪些service可以在请求时调用
* 万事具备，构造一个server，这个server服务负责处理rpc请求

  `同步异步下篇再考虑`，grpc默认都是同步rpc，本篇也是

  service在实现需要考虑并发，实际场景下也一定会出现。
service在实现时一定要保证是线程安全的。

  因为server端的处理太重要了，我们需要从多角度来理解grpc，
下面我们结合route的例子来看。

  json数据里包含了很多组数据，每组数据包含一个坐标和地名。  
  一元rpc，传一个坐标，得到坐标和地名  
  服务端流式rpc，传两个坐标，得到所有在范围里的坐标和地名  
  客户端流式rpc，传多个坐标，得到一个综合的计算信息
  双向流式rpc，传多个坐标和消息，服务器返回重复的

  下面看看函数上的区别：

```c++
    Status GetFeature(ServerContext* context,
                      const Point* point,
                      Feature* feature) override {}

    Status ListFeatures(ServerContext* context,
                        const routeguide::Rectangle* rectangle,
                        ServerWriter<Feature>* writer) override {}

    Status RecordRoute(ServerContext* context,
                       ServerReader<Point>* reader,
                       RouteSummary* summary) override {}

    Status RouteChat(ServerContext* context,
                     ServerReaderWriter<RouteNote, RouteNote>* stream)
                     override {}
```

  不同类型的rpc，落地到c++的函数参数也是不一样的，
估计client在实现时也是有区别的。

  上面就是服务端所有的操作，接下来看看客户端的处理

## client端对service的处理

  client第一件事是创建一个stub，有了stub，rpc调用就像
本地调用一样了。

  创建stub之前要先创建一个channel，channel指明了rpc
的服务端访问方式：ip端口和认证信息。

  stub在调用service时都会加上一个context上下文，
这个上下文不可以复用，可用作设置超时，rpc配置信息，
就是之前提到的元数据。

  对于流式rpc，client的处理也各不一样：

```c++
    Status status = stub_->GetFeature(&context, point, feature);

    std::unique_ptr<ClientReader<Feature> > reader(
          stub_->ListFeatures(&context, rect));

    std::unique_ptr<ClientWriter<Point> > writer(
          stub_->RecordRoute(&context, &stats));

    std::shared_ptr<ClientReaderWriter<RouteNote, RouteNote> > stream(
          stub_->RouteChat(&context));
```

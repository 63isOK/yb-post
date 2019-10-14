# grpc的说明

  一个高性能，开源的通用rpc框架，有4个主要特征：

* 服务定义简单 - 使用pb
* 跨语言跨平台 - client和server的stubs自动生成
* start quickly and scale - 运行时安装，方便scale
* 带认证的双向流 - 基于http2的认证，插件式

## 一个简单的c++例子

## 安装grpc

  在grpc中，pb起了很大的作用：服务定义和数据序列化，
虽然pb也可以用其他方案代替，在学习过程中依然使用pb。

  安装pb3：  

```shell
    cd third_party/protobuf
    make && sudo make install
```

  安装grpc：  

```shell
    sudo apt-get install build-essential autoconf libtool pkg-config
    sudo apt-get install libgflags-dev libgtest-dev
    sudo apt-get install clang libc++-dev

    git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
    cd grpc
    git submodule update --init
    make
```

  如果要测试例子，需要使用pkg-config:  
  sudo apt-get install pkg-config

  c++ linux make方式安装的不容易卸载干净 例子需要安装grpc和pb

## grpc的介绍

  在grpc中pb的意义重大，
不仅是接口定义语言idl，也是底层消息交换的数据格式。

  grpc的主要目的：调用服务的方法(本机或远程机器)就像调用
本机方法一样，这是rpc的定义，也是分布式程序和微服务的基石。
grpc的宗旨是定义服务，指明方法的参数和返回值。基于这点，
在服务端：有一个grpc服务，这个服务实现了接口，
并处理客户端发送的调用请求；在客户端：有一个stub
提供了方法，和服务端暴露的能力一样。这个stub可用多种语言实现。

  stub，百度翻译是`桩`，可以理解为客户端代理

  先熟悉以下名词：  

---

1. ipc - 进程间调用
2. rpc - 是ipc的一种，一般指不同机器上进程间通信
3. rpc调用过程 - 一般有3个角色：client clientStub server  
    第一步：clent发起一个call，发给clientStub  
    第二步：clientStub对请求进行封包(包括序列化)，发给server  
    第三步：server收到消息，丢给serverStub，进行解包  
    第四步：serverStub调用子程序，处理完原路返回给client

---

  一般stub都是自动生成的，先定义好idl，用工具生成。
手动编写也可以，很麻烦。stub要处理序列化，大小端问题。

## pb的使用介绍

  在grpc中使用pb，第一件事是定义一个proto文件，
在文件里定义了要序列化数据的格式，也定义了rpc接口。
.proto后缀结尾。

  在pb中，数据的结构(格式)称为message，一个message里
包含很多field，一个field就是一个name-value对。

```proto
    message people {
      string name = 1;
      int32 a = 2;
      bool b = 3;
      char d = 4;
    }
```

  使用protoc工具，生成一些访问类，在c++里，会有一个
people类，有一些访问field的简单方法(name() set_name()),
还有序列化、反序列化方法

  message用来进行数据格式定义，pb也可以定义services

```proto
    service hello {
      rpc hi(SaySomething) returns (Something) {}
    }

    message SaySomething {
      string aa = 1;
    }

    message Something{
      string aa = 1;
    }
```

  这样就定义了一个rpc，使用protoc工具，可生成server
和client的代码

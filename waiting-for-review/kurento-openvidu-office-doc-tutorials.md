# openvidu的基础教程

openvidu支持纯客户端和客户端+服务端的模式

纯客户端模式：openvidu官网教程提供了3种客户端：
js、angular、lonic。
js是老的js代码写的；angular是基于angular框架写的；
lonic是基于手机项目写的

客户端+服务端模式：客户端由js写的，
服务端有java或nodejs写

## 第一个例子: webcomponent

这是一个最简单的例子，多人群聊，还可以文字聊天，
可以控制自己发布的流状态，也可以控制她人流的显示与否

### 环境准备

```shell
git clone https://github.com/OpenVidu/openvidu-tutorials.git
npm install -g http-server
```

### 例子运行

```shell
http-server openvidu-tutorials/openvidu-webcomponent/web
sudo docker run -p 4443:4443 --rm -e  \
  openvidu.secret=MY_SECRET \
  openvidu/openvidu-server-kms:2.6.0
```

demo访问地址：localhost:8080

### 理解

* openvidu-webcomponent-{VERSION}.js  
  里面封装了所有的细节，包括了webrtc和kurento工具包的
* openvidu-webcomponent-{VERSION}.css  
  定了样式
* app .js  
  这里面是demo的逻辑
* index.html  
  页面代码

webcomponent里的秘密:  
    页面定义了两个渲染对象，一个是main用于用户输入信息，
一个是openvidu-webcomponent,用于显示后面的会话画面，
两个对象相互切换是通过joinSession和leaveSession事件触发，
页面上点击进入按钮进入会话，实际上app.js中做了两件事：
第一：获取标识会话的sessionId和标识用户的tokenId；
第二：用参数(tokenId，其他两个参数是附加的)配置
openvidu-webcomponent对象。配置完，会自动触发后面的操作：
进入到指定会话(推自己的流，并接收同一会话里其他人的流)

所以openvidu-webcomponent.js将所有的跟流相关的操作全封装了，
webrtc js的引用，到和kms的交互，在本例中，对外暴露的就
两个http接口和一个webcomponent对象。

`说好的open source，为啥js里都是用混淆后的代码，
要不要这么快商业`

## 第二个例子: hello world

http-server openvidu-tutorials/openvidu-hello-world/web

### hello world 理解

  这个例子相对webcomponent来说简单很多，少了很多会话控制，
只有一个简单的看自己和看其他人

* openvidu-browser.js  
  封装了流细节的js，类似与上个例子中的webcomponent.js
* app .js
  demo 逻辑
* style.css index.html
  页面和样式

    前一个例子将所有都封装在js中，通过一个对象的配置来触发，
这个例子中js暴露了一个OpenVidu对象，通过这个对象可以找到
session对象，发布对象，通过会话对象可以订阅所有新流事件，
但是大部分还是封装在js里.

## 第三个例子: insecure-js

  不安全的一个js demo，纯客户端模式  
  openvidu包含了3部分：浏览器中的js；后台的openvidu服务和kms  
  http-server openvidu-tutorials/openvidu-insecure-js/web

## insecure-js 理解

  这个例子功能和上一个不一样，上一个主要是简单的群聊，
这个是在群聊的基础上，多了一个渲染画面，通过画面列表还可以
进行切换

  上个例子在获取tokenId进行发布时，只是简单用了默认设置，
这个例子设置了很多，音视频关联的video对象，音视频设备的选择，
音视频控制，分辨率，帧率，本地画面是否做镜像翻转

## 第四个例子: insecure-angular

  这个例子和上一个例子没区别，只是用angular框架实现的  
熟悉前端技术的推荐从这个例子看起，

## 第五个例子: openvidu-library-angular

  这个展示了angular的基础库，换句话说：
  这是openvidu提供的angular库，用与前端开发
  运行了一下，提供了webcomponent所需要的全部功能

## 第六个例子: openvidu-library-react

## 第七个例子: openvidu-insecure-react

  这和angular库一样，openvidu也提供了基于react框架的封装库

## 第八个例子: openvidu-ionic

  这个例子是为手机端设计的，使用了lonic v4和angular 7，

## 第九个例子: openvidu-js-java

  这个是前后端结合的例子，可以使用ssl，
前面的例子都是客户端demo，所以称为insecure，不安全的，
因为没有后端，不走ssl，所以是insecure

spa：single page application

整个例子分4部分：
openvidu提供的js库，用于浏览器前端；
java业务服务端库，用于实现java业务服务；
openvidu server，用于连接kms；
kms，提供媒体处理能力

### 运行

```shell
  sudo apt install -y maven
  mvn package exec:java
```

  编译时报错，说tomcat的一个什么库没找到，
  使用mvn dependency:analyze修复依赖，
  发现打印信息中报确实确实了那个库，但是运行没什么用，
  之后在网上找了一些其他方法，都没有什么用，
  之后重新配了下spring boot的环境，也就是mvn和jdk的环境，
  才ok，因为以前spring boot环境在虚拟机中配了，
  在本机还没配  
  之后在<https://localhost:5000>访问

### openvidu-js-java理解

  这个例子相对于前面的例子，多加了一个用户的概念，
有用户就有角色(有人推流，有人不推流)，其他与会者列表
可以切换，但并不是一个全功能的，缺少了很多控制功能。

  整个程序，js前端，java后端：

* app.java 服务端入口
* LoginController.java 管理用户的登录和登出
* SessionController.java 管理用户的会话
* openvidu-browser-VERSION.js 前端js sdk，封装了流操作
* app.js 页面逻辑
* index.html 页面渲染
* style.css 页面样式

  页面有3块：登录 + 选择会话 + 进入会话。  
  登录：  
调用http接口api-login/login，成功就进入选择会话展示阶段，  
  选择会话：  
和前面讲的一样，调用http接口api-sessions/get-token获取tokenId，
这个例子将token的的相关处理丢在服务端，其他的和前面例子一样  
  进入会话：  
推自己的流，看别人的音视频流  

  代码中并没有直接调用webrtc或kurento的地方，都被封装成sdk了，
通过sdk和openvidu服务交互，通过openvidu和kurento交互。

## 其他例子要么是mvc要么是node

  暂时不看

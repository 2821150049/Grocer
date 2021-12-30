﻿# 第二章、ROS通信机制

## 一、话题通信（发布订阅）

### 1.话题通信理论模型

![1](https://img-blog.csdnimg.cn/img_convert/44ba785ba7814cce29a50f99e395ab03.png)

1. 角色

+ ROSMaster      管理者（媒婆）



+ talker	发布者（男方）



+ listener  订阅者（女方）

2. 流程

+ ROSMaster可以根据话题建立发布者和订阅者的连接

3. 注意事项

+ RPC协议
+ TCP协议
+ 步骤0和步骤1没有顺序要求
+ talker和listener可以存在多个
+ talker个listener建立联系后，master就可以关闭了
+ 上述实现流程已经被封装，直接调用就可以了

4. 关注点

+ 大部分实现已经被封装了
+ 设置一致的话题
+ 主要关注发布者的实现，订阅者的实现，消息载体

### 2.话题通信基本操作

1. C++实现

+ 流程
  + 编写发布者
  + 编写订阅者
  + 配置文件
  + 编译执行
+ 发布者

```cpp
#include "ros/ros.h"
#include "std_msgs/String.h"
#include "sstream"
/*
发布方实现：
    1.包含头文件
    2.初始化ROS节点
    3.创建节点句柄
    4.创建发布者对象
    5.编写发布逻辑并发布数据
*/
int main(int argc,char* argv[]) {
    ros::init(argc,argv,"结点名称");
    ros::NodeHandle nh;
    ros::Publisher pub = nh.advertise<std_msgs::String>("话题名称",10);
    std_msgs::String msg;
    // 发布频率10Hz
    ros::Rate rate(10);
    // 设置编号
    int count = 0;
    while(ros::ok()){
        setlocale(LC_ALL,"");
        count++;
        // msg.data = "hello";
        std::stringstream ss;
        ss << "hello--->" << count;
        msg.data = ss.str();
        ROS_INFO("发布的数据%s",ss.str().c_str());
        pub.publish(msg);
        rate.sleep();
    }
    return 0;
}
```

+ 检查是否有问题

```cpp
rostopic echo 话题名称
```

+ 订阅者实现

```cpp
#include "ros/ros.h"
#include "std_msgs/String.h"
/*
订阅方实现：
    1.包含头文件
    2.初始化ROS节点
    3.创建节点句柄
    4.创建订阅者对象
    5.编写订阅逻辑并订阅数据
    6.声明spin();函数处理回调函数
*/
void 回调函数名(const std_msgs::String::ConstPtr& msg) {
    // 通过msg获取得到的数据
    ROS_INFO("翠花订阅到的数据%s",msg->data.c_str());
}
int main(int argc,char* argv[]) {
    setlocale(LC_ALL,"");
    ros::init(argc,argv,"结点名");
    ros::NodeHandle nh;
    ros::Subscriber sub = nh.subscribe("订阅的话题名",10,回调函数名);
    ros::spin();
}
```

2. python实现

+ 发布者

```python
#! /usr/bin/env python
import rospy
from std_msgs.msg import String
if __name__ == "__main__":
    rospy.init_node("sanDai")
    pub = rospy.Publisher("che",String,queue_size=10)
    msg = String()
    rate = rospy.Rate(1)
    count = 0
    while not rospy.is_shutdown():
        count += 1
        msg.data = "hello" + str(count)
        pub.publish(msg)
        rospy.loginfo("发布的数据是%s",msg.data)
        rate.sleep()
```

+ 订阅者

```python
#! /usr/bin/env python
import rospy
from std_msgs.msg import String

def doMsg(msg):
    rospy.loginfo("我订阅的数据%s",msg.data)


if __name__ == "__main__":
    rospy.init_node("huahua")
    sub = rospy.Subscriber("che",String,doMsg,queue_size=10)

    rospy.spin()
    pass
```

+ 注意事项

*数据丢失*：加延时

### 3.自定义话题通信

**简介：**ROS 中通过 std_msgs 封装了一些原生的数据类型,比如:String、Int32、Int64、Char、Bool、Empty....但是，这些数据一般只包含一个 data 字段，结构的单一意味着功能上的局限性。

**可以使用的字段类：**

- int8, int16, int32, int64 (或者无符号类型: uint*)
- float32, float64
- string
- time, duration
- other msg files
- variable-length array[] and fixed-length array[C]

1. **自定义msg文件**

msg目录下文本文件

```cpp
string name
uint16 age
float64 height
```

2. **编辑文件配置**

package.xml中添加*编译依赖*与*执行依赖*

```xml
  <build_depend>message_generation</build_depend>
  <exec_depend>message_runtime</exec_depend>
  <!-- 
  exce_depend 以前对应的是 run_depend 现在非法
  -->
```

CMakeList.txt编辑msg相关配置

```cmake
# 需要加入 message_generation,必须有 std_msgs
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  message_generation
)
## 配置 msg 源文件
add_message_files(
  FILES
  Person.msg
)
# 生成消息时依赖于 std_msgs
generate_messages(
  DEPENDENCIES
  std_msgs
)
#执行时依赖
catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES demo02_talker_listener
  CATKIN_DEPENDS roscpp rospy std_msgs message_runtime
#  DEPENDS system_lib
)
# 防止编译时，编译顺序错误
add_dependencies(demo02_pub ${PROJECT_NAME}_generate_messages_cpp)
```

3. **编译及VC配置**

```json
{
  "configurations": [
      {
          "browse": {
              "databaseFilename": "",
              "limitSymbolsToIncludedHeaders": true
          },
          "includePath": [
              "/opt/ros/noetic/include/**",
              "/usr/include/**",
              "/home/zhf/Learn_ROS/demo03_ws/devel/include/**" //配置 head 文件的路径 
          ],
          "name": "ROS",
          "intelliSenseMode": "gcc-x64",
          "compilerPath": "/usr/bin/gcc",
          "cStandard": "c11",
          "cppStandard": "c++17"
      }
  ],
  "version": 4
}
```

4. C++实现

+ pub

```cpp
#include "ros/ros.h"
#include "plumbing_pub_sub/Person.h"
int main(int argc, char *argv[])
{
    setlocale(LC_ALL,"");
    ros::init(argc,argv,"Person");
    ros::NodeHandle nh;
    // 设置话题
    ros::Publisher pub = nh.advertise<plumbing_pub_sub::Person>("teacher",20);
    plumbing_pub_sub::Person stu;
    stu.age = 1;
    stu.name = "张三";
    stu.height = 1.7;
    ros::Rate rate(1);
    while (ros::ok()) {
        rate.sleep();
        pub.publish(stu);
        ROS_INFO("name:%s,age:%d,height:%0.2f",stu.name.c_str(),stu.age,stu.height);
        stu.age ++;
        ros::spinOnce();
    }
    return 0;
}
```

+ sub

```cpp
#include "ros/ros.h"
#include "plumbing_pub_sub/Person.h"
using namespace ros;
void doMsg(const plumbing_pub_sub::Person::ConstPtr & msg) {
    ROS_INFO("%s,%d,%0.2f",msg->name.c_str(),msg->age,msg->height);
}
int main(int argc, char *argv[])
{
    setlocale(LC_ALL,"");
    init(argc,argv,"student");
    NodeHandle np;
    Subscriber sub = np.subscribe("teacher",20,doMsg);

    spin();
    return 0;
}
```

*这个函数的参数是针对消息的一个指针，const plumbing_pub_sub::Person::ConstPtr & msg
 这是在ros里面固定格式的一个调用，plumbing_pub_sub::Person 对应于我们主函数里面话题内容/turtle1/pose是一样的，后面的ConstPtr是一个常指针，msg以一个常指针的形式指向plumbing_pub_sub::Person所有的信息的数据内容*

+ python实现

VC 配置d

```json
{
    "python.autoComplete.extraPaths": [
        "/opt/ros/noetic/lib/python3/dist-packages",
        "/home/zhf/Learn_ROS/demo03_ws/devel/lib/python3/dist-packages" // 导入生成发python 路径
    ],
    "python.pythonPath": "/usr/bin/python",
    "cmake.configureOnOpen": false
}
```

*pub*

```python
#! /usr/bin/env python
import rospy
from plumbing_pub_sub.msg import Person 

if __name__ == "__main__":
    # 初始化节点
    rospy.init_node("father")
    pub = rospy.Publisher("liaotian",Person,queue_size=10)
    fat = Person()
    fat.name = "李四"
    fat.age = 1
    fat.height = 1.8
    rate = rospy.Rate(1)
    while not rospy.is_shutdown():
        pub.publish(fat)
        fat.age += 1
        rate.sleep()
```

*sub*

```python
#! /usr/bin/env python

import rospy
from plumbing_pub_sub.msg import Person 

def doMsg(p):
    rospy.loginfo("%s,%d,%0.2f",p.name,p.age,p.height)

if __name__ == "__main__":
    # 初始化节点
    rospy.init_node("mather")
    sub = rospy.Subscriber("liaotian",Person,doMsg,queue_size=10)
    rospy.spin()
    
```

## 二、服务通信（请求响应）

### 1.概念

> 以请求响应的方式实现不同节点之间数据交互的通信模式。
>
> 用于偶然的、对时时性有要求、有一定逻辑处理需求的数据传输场景。

### 2.模型

+ ROS master
+ Talker(Server)
+ Listener(Client)

![2](https://img-blog.csdnimg.cn/img_convert/d71ec22001d776cd63241694f724bf7c.png)

+ 注意事项

**1.服务端是先启动的，客户端后请求**

**2.客户端和服务端都可以存在多个**

**3.话题需要一致**

**4.数据格式自定义**

### 3.自定义服务通信数据类型srv

1. 按照固定格式创建srv文件

+ 包含两部分，请求部分，和响应部分

![3](https://img-blog.csdnimg.cn/img_convert/94c6fc92f4fbf63351d9977040c5e961.png)

+ 编辑配置文件

*package.xml*

```xml
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```

*CMakeList.txt*

```cmake
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  message_generation
)

add_service_files(
  FILES
  AddInts.srv
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES plumbing_server_client
 CATKIN_DEPENDS roscpp rospy std_msgs message_runtime
#  DEPENDS system_lib
)
```

+ 编译文件就好了

### 4.C++实现

*配置vscode*

```json
// 配置路径
/home/zhf/Learn_ROS/demo03_ws/devel/include/**
```

*服务端*

```cpp
#include "ros/ros.h"
#include "plumbing_server_client/AddInts.h"
using namespace ros;
/* 
服务器实现:
        1.包含头文件
        2.初始化 ROS 节点
        3.创建 ROS 句柄
        4.创建 服务 对象
        5.回调函数处理请求并产生响应
        6.由于请求有多个，需要调用 ros::spin()
*/
bool doNums(plumbing_server_client::AddInts::Request & request,
    plumbing_server_client::AddInts::Response & response) {
    int num1 = request.num1;
    int num2 = request.num2;
    int sum = num1 + num2;
    ROS_INFO("%d+%d=%d",num1,num2,sum);
    response.sum = sum;
    return true;
}
int main(int argc, char *argv[])
{
    // 初始化ROS节点
    init(argc,argv,"server");
    // 创建句柄
    NodeHandle nh;
    // 创建服务对象
    ServiceServer server = nh.advertiseService("addInts",doNums);
    // 处理请求并产生响应
    spin();
    return 0;
}
```

*CMakeList.txt*

```cmake
add_executable(demo01_server src/demo01_server.cpp)
# 后面的参数可以使用默认值
add_dependencies(demo01_server ${PROJECT_NAME}_gencpp)
target_link_libraries(demo01_server
  ${catkin_LIBRARIES}
)
```

*测试指令*

```cpp
rosservice call addInts "num1: 6
num2: 5" 
```

![4](https://img-blog.csdnimg.cn/img_convert/fbb5e6bb446873dbc5e9eb16146ad0d7.png)

*客户端*

```cpp
#include "ros/ros.h"
#include "plumbing_server_client/AddInts.h"
using namespace ros;
int main(int argc, char *argv[])
{
    setlocale(LC_ALL,"");
    init(argc,argv,"client");
    NodeHandle nh;
    plumbing_server_client::AddInts msg;
    msg.request.num1 = 5;
    msg.request.num2 = 10;
    ServiceClient client = nh.serviceClient<plumbing_server_client::AddInts>("addInts");
    bool flag=true;
    Rate rate(10);
    while (flag) {
        msg.request.num1++;
        flag = client.call(msg);
        ROS_INFO("响应结果%d",msg.response.sum);
        rate.sleep();
    }
    return 0;
}
```

*实现参数的动态提交*

```cpp
// 在client.call(msg);之前加一个函数
client.waitForExistence();
// 或者
ros::service::waitForService("等待的服务名称");
```

### 5.python实现

*服务端*

```python
#! /usr/bin/env python

import rospy
from plumbing_server_client.srv import *
# 参数必须用request
def doNum(request):
    sum = request.num1 + request.num2
    response = AddIntsResponse()
    response.sum = sum
    rospy.loginfo(response.sum)
    return response.sum
if __name__ == "__main__":
    rospy.init_node("demo01_server")
    ser = rospy.Service("server_1",AddInts,doNum)
    rospy.loginfo("服务器已经启动")
    rospy.spin()
```

*客户端*

```python
#! /usr/bin/env python
import rospy
from plumbing_server_client.srv import *
# 优化
import sys
if __name__ == "__main__":
    rospy.init_node("demo01_client")
    cli = rospy.ServiceProxy("server_1",AddInts)
    num1 = int(sys.argv[1])
    num2 = int(sys.argv[2])
    # 等待服务器启动
    # cli.wait_for_service()  rospy.wait_for_service("话题名称")
    rospy.wait_for_service("server_1")
    response = cli.call(num1,num2)
    rospy.loginfo("响应数据%d",response.sum)
```

## 三、参数服务器（参数共享）

### 1.概念

> 参数服务器在ROS中主要用于实现不同节点之间的数据共享。参数服务器相当于是独立于所有节点的一个公共容器，可以将数据存储在该容器中，被不同的节点调用，当然不同的节点也可以往其中存储数据。

+ *以共享的方式实现不同节点之间的数据交互*
+ 作用
  + 存储多借点共享数据
+ 参数服务器的增删改查

### 2.模型

![5](https://img-blog.csdnimg.cn/img_convert/1d2c5b0e1c45c283463bcec4ce529cee.png)

+ 注意：

参数服务器不是为高性能而设计的

+ 数据类型

```cpp
32-bit integers  // 4字节int
booleans	// 布尔数据
strings		// 字符数据
doubles		// 浮点数据
iso8601 dates	// 时间数据
lists		// 列表
base64-encoded binary data	// 二进制数据
字典
```

### 3.C++实现

+ **实现的AIP**

```cpp
ros::NodeHandle
    setParam("键",值)
ros::param
    set("键",值)
```

+ **增加与修改**

```cpp
#include "ros/ros.h"
using namespace ros;

int main(int argc, char *argv[])
{
    init(argc,argv,"ros_param_C");
    NodeHandle nh;
    // 增加数据
    // 方案1：nh  参数一：书记名，参数二：数据
    nh.setParam("type","xiaoHuang");
    nh.setParam("radius_1",0.15);
    // 方案2：参数一：书记名，参数二：数据
    param::set("type_param","xiaoBai");
    param::set("radius_2",0.15);
    // 修改， 相当于覆盖
    // 方案一
    nh.setParam("radius_1",0.3);
    // 方案二
    param::set("radius_2",0.25);
    return 0;
}
```

+ 调用指令获取数据

```cpp
rosparam list // 获取数据列表
rosparam get /数据名 // 获取数据
```

![6](https://img-blog.csdnimg.cn/img_convert/f802d15a885bf71d9d129574057ffd72.png)

+ **查找**

```cpp
#include "ros/ros.h"
using namespace ros;
/*
ros::NodeHandle  -----------------------------------------
        param(键,默认值) 
            存在，返回对应结果，否则返回默认值
        getParam(键,存储结果的变量)
            存在,返回 true,且将值赋值给参数2
            若果键不存在，那么返回值为 false，且不为参数2赋值
        getParamCached(键,存储结果的变量)--提高变量获取效率
            存在,返回 true,且将值赋值给参数2
            若果键不存在，那么返回值为 false，且不为参数2赋值
        getParamNames(std::vector<std::string>)
            获取所有的键,并存储在参数 vector 中 
        hasParam(键)
            是否包含某个键，存在返回 true，否则返回 false
        searchParam(参数1，参数2)
            搜索键，参数1是被搜索的键，参数2存储搜索结果的变量
ros::param  -------------------------------------------------
        和上面的一样
*/
int main(int argc, char *argv[])
{
    setlocale(LC_ALL,"");
    init(argc,argv,"ros_param_C");
    NodeHandle nh;
    // 查询
    //ros::NodeHandle  --------------------------------------------------
    //1. 找到返回该结果的值
    float radius = nh.param("radius",0.0);
    ROS_INFO("radius:%0.2f",radius);
    float radius_1;
    // 返回bool类型数据
    // 2. 查找并返回true/flase
    bool result = nh.getParam("radius_111",radius_1);
    if(result)
        ROS_INFO("radius:%0.2f",radius_1);
    else
        ROS_INFO("没有找到数据");
    // getParamCached// 是性能提升
    // 3. 查找并返回true/flase提升效力
    result = nh.getParamCached("radius_1",radius_1);
    if(result)
        ROS_INFO("radius:%0.2f",radius_1);
    else
        ROS_INFO("没有找到数据");
    // 4. 获取所以参数
    std::vector<std::string> names;
    nh.getParamNames(names);
    for (auto &&name : names){
        ROS_INFO("便利到的元素%s",name.c_str());
    }
    // 5. 判断键是否存在，存在返回true,参数是键名  nh.hasParam("radius_1");
    result = nh.hasParam("radius_1");
    if(result)
        ROS_INFO("键存在");
    else
        ROS_INFO("没有找到数据");
    // 6. 查询键，并保存键名
    std::string key;
    nh.searchParam("type",key);
    ROS_INFO("搜索结果：%s",key.c_str());
    //ros::param 函数和上面的一样 -------------------------------------------------
    return 0;
}
```

![7](https://img-blog.csdnimg.cn/img_convert/4827faa5543240152280c34eedb70304.png)

+ **删除**

```cpp
#include "ros/ros.h"
using namespace ros;
/*
ros::NodeHandle
    deleteParam()
ros::param
    del()
*/
int main(int argc, char *argv[])
{
    setlocale(LC_ALL,"");
    init(argc,argv,"ros_param_C");
    NodeHandle nh;
    // 参数的删除
    nh.setParam("radius",0.5);
    // 方案一 参数一：键
    bool flag1 = nh.deleteParam("radius");
    if (flag1) {
        ROS_INFO("删除成功");
    }else{
        ROS_INFO("删除失败");
    }
    // 方案二
    flag1 = ros::param::del("radius");
    if (flag1) {
        ROS_INFO("删除成功");
    }else{
        ROS_INFO("删除失败");
    }
    // 打印所以参数
    std::vector<std::string> names;
    ros::param::getParamNames(names);
    for (auto && name :names) {
        ROS_INFO("节点有%s",name.c_str());
    }
    return 0;
}
```

![8](https://img-blog.csdnimg.cn/img_convert/249b88873f9bbe0b31c971b92936962a.png)

### 4.Python实现

+ **增加和修改**

```python
#! /usr/bin/env python
"""
    参数服务器操作之新增与修改(二者API一样)_Python实现:
"""
import rospy
if __name__ == "__main__":
    rospy.init_node("set_update_paramter_p")
    # 设置各种类型参数
    rospy.set_param("p_int",10)
    rospy.set_param("p_double",3.14)
    rospy.set_param("p_bool",True)
    rospy.set_param("p_string","hello python")
    rospy.set_param("p_list",["hello","haha","xixi"])
    rospy.set_param("p_dict",{"name":"hulu","age":8})
    # 修改
    rospy.set_param("p_int",100)
```

+ **查找**

```python
#! /usr/bin/env python
"""
    参数服务器操作之查询_Python实现:    
        get_param(键,默认值)
            当键存在时，返回对应的值，如果不存在返回默认值
        get_param_cached
        get_param_names
        has_param
        search_param
"""
import rospy
if __name__ == "__main__":
    rospy.init_node("get_param_p")
    #获取参数
    int_value = rospy.get_param("p_int",10000)
    double_value = rospy.get_param("p_double")
    bool_value = rospy.get_param("p_bool")
    string_value = rospy.get_param("p_string")
    p_list = rospy.get_param("p_list")
    p_dict = rospy.get_param("p_dict")

    rospy.loginfo("获取的数据:%d,%.2f,%d,%s",
                int_value,
                double_value,
                bool_value,
                string_value)
    for ele in p_list:
        rospy.loginfo("ele = %s", ele)

    rospy.loginfo("name = %s, age = %d",p_dict["name"],p_dict["age"])
    # get_param_cached
    int_cached = rospy.get_param_cached("p_int")
    rospy.loginfo("缓存数据:%d",int_cached)
    # get_param_names
    names = rospy.get_param_names()
    for name in names:
        rospy.loginfo("name = %s",name)
        
    rospy.loginfo("-"*80)
    # has_param
    flag = rospy.has_param("p_int")
    rospy.loginfo("包含p_int吗？%d",flag)
    # search_param
    key = rospy.search_param("p_int")
    rospy.loginfo("搜索的键 = %s",key)
```

+ **删除**

```python
#! /usr/bin/env python
"""
    参数服务器操作之删除_Python实现:
    rospy.delete_param("键")
    键存在时，可以删除成功，键不存在时，会抛出异常
"""
import rospy
if __name__ == "__main__":
    rospy.init_node("delete_param_p")

    try:
        rospy.delete_param("p_int")
    except Exception as e:
        rospy.loginfo("删除失败")
```

## 四、常用命令

+ **如何获取话题和消息载体、可以通过下面命令得到**

+ **动态获取结点数据**

+ 官方链接：

  [ROS常用命令链接]: http://wiki.ros.org/ROS/CommandLineTools

### 1. rosnode:操作结点

+ 是用于获取结点信息的命令

+ 用法

```cpp
rosnode ping 节点名称  	// 测试结点的连接情况，看是否连接正常
    // rosnode ping /Person
rosnode list	// 列出活动结点
    // rosnode list
rosnode info 结点名称	// 打印结点日志
    //rosnode info /Person
rosnode machine	设备名称 // 列出指定设备上的节点
	// rosnode machine zhf
rosnode kill 结点名称	// 杀死某个节点
    // rosnode kill /Person
rosnode cleanup // 清除不可连接的节点，也叫僵尸结点
    // rosnode cleanup
```

### 2. rostopic:操作话题

+ 用于获取ROS主题调试信息，包括订阅者，发布者，发布频率和ROS消息
+ 用法

```cpp
rostopic bw     // 显示主题使用的带宽
rostopic delay  // 显示带有 header 的主题延迟
rostopic echo   // 打印消息到屏幕、需要找到对应的包下面，source一下
    // rostopic echo 话题名
rostopic find   // 根据类型查找主题
rostopic hz     // 显示主题的发布频率
    // rostopic hz liaotian(话题名称)
rostopic info   // 显示主题相关信息
    // rostopic info Person（话题名）
rostopic list   // 显示所有活动状态下的主题(和rosnode list差不多)
    // rostopic list
rostopic pub    // 将数据发布到主题
rostopic type   // 打印主题类型
```

### 3. rosmsg:操作msg消息

+ 显示消息类型

```cpp
rosmsg show    		// 显示消息描述
rosmsg info    		// 显示消息信息
rosmsg list    		// 列出所有消息
rosmsg md5    		// 显示 md5 加密后的消息
rosmsg package    	// 显示某个功能包下的所有消息
rosmsg packages    	// 列出包含消息的功能包
```



### 4. rosservice:操作服务

+ 列出和查询服务通信的命令工具
+ 用法

```cpp
rosservice args 	// 打印服务参数
rosservice call    	// 使用提供的参数调用服务
    // rosservice call addIns(服务名称) 数据
rosservice find   	// 按照服务类型查找服务
rosservice info    	// 打印有关服务的信息
    // rosservice info addIns(服务名称)
rosservice list    	// 列出所有活动的服务
rosservice type    	// 打印服务数据类型
rosservice uri    	// 打印服务的 ROSRPC uri
```

### 5. rossrv:操作srv消息

+ 显示服务类型信息的命令

```cpp
rossrv show    	// 显示服务消息详情
rossrv info    	// 显示服务消息相关信息
rossrv list    	// 列出所有服务信息
rossrv md5    	// 显示 md5 加密后的服务消息
rossrv package	// 显示某个包下所有服务消息
rossrv packages // 显示包含服务消息的所有包
```

### 6. rosparm:操作参数

+ 用于使用YAML编码文件在参数服务器上获取和设置ROS参数。
+ 用法

```cpp
rosparam set    	// 设置参数
rosparam get    	// 获取参数
rosparam delete    	// 删除参数
rosparam list    	// 列出所有参数
rosparam load    	// 从外部文件加载参数
    // rosparam load 名称.yaml
rosparam dump    	// 将参数写出到外部文件
    // rosparam dump 名称.yaml
```

## 五、通信实操

### 案例一、话题发布

1. **命令获取结点话题名称和数据类型**

![11](https://img-blog.csdnimg.cn/img_convert/f7d2c47ca94276b0d827f06a08354c57.png)

2. **C++实现**

```cpp
#include "ros/ros.h"
// 数据类型
#include "geometry_msgs/Twist.h"
/*
发布方实现：
    1.包含头文件
    2.初始化ROS节点
    3.创建节点句柄
    4.创建发布者对象
    5.编写发布逻辑并发布数据
*/
int main(int argc,char* argv[]) {
    ros::init(argc,argv,"test_pub");
    ros::NodeHandle nh;
    // /turtle1/cmd_vel发布的话题名称
    ros::Publisher pub = nh.advertise<geometry_msgs::Twist>("/turtle1/cmd_vel",100);
    // 发布频率10Hz
    ros::Rate rate(100);
    geometry_msgs::Twist msg;
    msg.linear.x = 0.1;
    msg.linear.y = 0.0;
    msg.linear.z = 0.0;
    msg.angular.z = 0.8;
    msg.angular.y = 0.0;
    msg.angular.x = 0.0;
    while(ros::ok()){
        pub.publish(msg);
        msg.linear.x += 0.001;
        ROS_INFO("haha");
        rate.sleep();
        ros::spinOnce();
    }
    return 0;
}
```

### 案例二、话题订阅

+ **命令查找数据**

![12](https://img-blog.csdnimg.cn/img_convert/bd6ccc88c5acb1c63a1e433893f3c51e.png)

![13](https://img-blog.csdnimg.cn/img_convert/6c38654f6b1fb91f8a77089edc0593cb.png)

+ **C++实现**

```cpp
#include "ros/ros.h"
#include "turtlesim/Pose.h"
using namespace ros;
void doTurtlesim(const turtlesim::Pose::ConstPtr & pose) {
	ROS_INFO("位置信息：");
	ROS_INFO("位置(x:%0.2f,y:%0.2f)朝向(theta:%0.2f)线速度(linear_velocity:%0.2f)加速度(angular_velocity:%.2f)",pose->x,pose->y,pose->theta,pose->linear_velocity,pose->angular_velocity);
}
int main(int argc,char* argv[]) {
	
	setlocale(LC_ALL,"");
	init(argc,argv,"test_sub");
	NodeHandle nh;
	Subscriber sub = nh.subscribe("/turtle1/pose",100,doTurtlesim);
	spin();
	return 0;
}
```

+ **效果**

![14](https://img-blog.csdnimg.cn/img_convert/aeaa5dcdf67255c5daa7f4be3e61e143.png)

#### 补、

**案例一和案例二需要手动添加依赖**

*pakeage.xml*

![15](https://img-blog.csdnimg.cn/img_convert/f0e02706e2bf5d4094638855965fab46.png)

*CMakeList.txt*

![16](https://img-blog.csdnimg.cn/img_convert/6f17e6a4506516f2d03c39433a82aea2.png)



### 案例三、服务调用

+ **命令查询数据**

![17](https://img-blog.csdnimg.cn/img_convert/60789575d567c71d8eedb2269de77c8b.png)

+ **命令调用服务**

![18](https://img-blog.csdnimg.cn/img_convert/240504e0b364194af124e47bf28dd860.png)

*需要依赖包`turtlesim`*

+ **C++实现**

```cpp
#include "ros/ros.h"
#include "turtlesim/Spawn.h"
using namespace ros;
int main(int argc,char* argv[]) {
	setlocale(LC_ALL,"");
	init(argc,argv,"test_ser");
	NodeHandle nh;
	ServiceClient client = nh.serviceClient<turtlesim::Spawn>("/spawn");
	turtlesim::Spawn msg;
	msg.request.x = 1.0;
	msg.request.y = 1.0;
	msg.request.theta = 1.0;
	msg.request.name = "turtle3";
    client.waitForService();
	bool flag = client.call(msg);
	if (flag)
    {
        ROS_INFO("新的乌龟生成,名字:%s",msg.response.name.c_str());
    } else {
        ROS_INFO("乌龟生成失败！！！");
    }
	spinOnce();
	return 0;
}
```

+ *结果：*

![19](https://img-blog.csdnimg.cn/img_convert/4f2f83c5a310e7abefad915fd60a7a1c.png)

### 案例四、参数服务调用

+ **命令行查看数据**

![20](https://img-blog.csdnimg.cn/img_convert/fe92ec0b204d06edf09b53eb85f4a101.png)

+ **C++实现**

````c++
#include "ros/ros.h"
using namespace ros;
int main(int argc,char* argv[]){
	init(argc,argv,"test_param");
	param::set("/turtlesim/background_r",255);
	param::set("/turtlesim/background_g",0);
	param::set("/turtlesim/background_b",0);
	return 0;
}
````


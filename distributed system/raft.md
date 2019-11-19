## brpc

### 安装

```bash
git clone https://github.com/apache/incubator-brpc.git
# read https://github.com/apache/incubator-brpc/blob/master/docs/cn/getting_started.md
sudo apt-get install -y git g++ make libssl-dev libgflags-dev libprotobuf-dev libprotoc-dev protobuf-compiler libleveldb-dev
mkdir build && cd build && cmake .. && make
```

### 使用

+ [rpc及protobuf的语法](https://developers.google.com/protocol-buffers/docs/proto#options)

+ `protobuf`的字段值范围为`[1, 2^29-1]`，其中`19000-19999`为内部实现用的保留字段

+ 字段规则：

  + `required`：只有一个

  + `optional`：最多一个

  + `repeated`：可以有`0`到多个

    ```protobuf
    repeated int32 samples = 4 [packed=true]; // 可以使其更高效的编码
    ```

+ `braft::StateMachine`：每个`raft node`的所有事件的汇聚点

+ 

### 示例：echo

+ `proto`文件

  ```protobuf
  option cc_generic_services = true;
   
  message EchoRequest {
        required string message = 1;
  };
  message EchoResponse {
        required string message = 1;
  };
   
  service EchoService {
        rpc Echo(EchoRequest) returns (EchoResponse);
  };
  ```

+ 

## braft

### 安装

+ 需要先安装`brpc`，安装后找到`brpc`的安装路径，在`bash`输入如下两行后即可，或者不输直接安装`brpc`的时候`make install`

```bash
export CMAKE_INCLUDE_PATH=/path/to/brpc/build/output/include
export CMAKE_LIBRARY_PATH=/path/to/brpc/build/output/lib
```

### 问题

+ 父类的`public`方法，子类以`private`去`override`，有什么作用，来自`class Counter : public braft::StateMachine`

## grpc

### 安装

+ 参考[BUILDING.md](https://github.com/grpc/grpc/blob/master/BUILDING.md)

## gflags

+ [简单介绍](http://dreamrunner.org/blog/2014/03/09/gflags-jian-ming-shi-yong/)
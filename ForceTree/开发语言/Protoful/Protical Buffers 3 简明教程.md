### Protical Buffers 3 简明教程

> https://developers.google.com/protocol-buffers/docs/proto3

#### 一、简介

Protocol Buffers 是一种语言无关、平台无关的可扩展机制，它用于序列化结构化数据。使用 Protocol Buffers 定义依次结构化的数据，然后使用特殊生成的源代码，轻松地在各种数据流中使用各种语言编写和读取结构化数据。

现在有许多框架在使用 Protocol Buffers。流行的RPC框架 gRPC 也是基于 Prococol Buffers。当前仅有 2.0 和 3.0 两个版本。

优点：

1. 性能较 xml、json、thrift 等要好，效率高
2. 代码生成机制，数据解析类自动生成
3. 支持向后兼容和向前兼容
4. 支持多重编程语言（Java、C++、Python）

缺点：

1. 二进制格式，可读性差
2. 缺乏自描述

#### 二、文档结构

Protocol Buffers 使用 **.proto** 文件来保存文档。

##### 1、版本声明

Protocol Buffers 文档的第一个非注释行是版本申明，若不填写默认为 2.0 版本。

``` protobuf
syntax = "proto3"; // 或 proto2
```

##### 2、Packages

与 Java 的 package 申明类似，Protocol Buffers 也可以声明 package，来防止命名冲突，不同的是 Package 是可选的。

``` protobuf
package foo.bar;
message Open { ... }
```

使用的时候也需要加上命名空间：

``` protobuf
message Foo {
   ...
   foo.bar.Open open = 1;
   ...
}
```

不同的语言，生成的代码不相同：

1. C++ 是 namespace

   ``` protobuf
   namespace foo::bar
   ```

2. Java

   Java使用package声明，如果在 .proto 中显示，则使用 option java_package 定义包名

3. Go
   Golang使用 go package name，如果在 .proto 中显示，则使用 option go_package 定义包名

##### 3、Import

若要使用官方定义或其他贡献者定义的文件消息，与Java的import或C++的include类似，使用 import 来导入：

``` protobuf
import "myproject/other_protos.proto"
```

##### 4、主体

使用 ``message`` 定义消息，它类似结构体、类。

使用 ``service`` 定义服务，它表征RPC通信的方法。

##### 5、注释

Protocol Buffers 使用 C/C++ 的样式，使用 // 和 /* 来表示或包裹注释：

``` protobuf
/*
 * Tool 类
 */
message Tool {
	// 名字
	String name = 1;
	/* 使用方法 */
	String method = 2;
}
```

#### 三、数据类型

Protocol Buffers 中的数据类型分为 标量类型 和 复合类型，类似 Java 的基本变量和类。复合类型包括 枚举 和 其他消息类型。

##### 1、标量类型

| .proto   | 说明                                             | Java    | Python     |
| -------- | ------------------------------------------------ | ------- | ---------- |
| double   | 双精度浮点数                                     | Double  | float      |
| float    | 单精度浮点数                                     | float   | float      |
| int32    | 32位整数，使用变长编码，对负数编码效率低         | Int     | int        |
| int64    | 64位整数，使用变长编码，对负数编码效率低         | long    | int/long   |
| uint32   | 32位无符号整数，使用变长编码                     | int     | int        |
| uint64   | 64位无符号整数，使用变长编码                     | long    | int/long   |
| sint32   | 32位带符号整数，对负数编码比int32高效            | int     | int        |
| sint64   | 64位带符号整数，对负数编码比int64高效            | long    | int/long   |
| fixed32  | 4字节编码，若变量经常大于 2^(28)，则比uint32高效 | int     | int        |
| fixed64  | 8字节编码，若变量经常大于 2^(56)，则比uint64高效 | long    | int/long   |
| sfixed32 | 4字节编码                                        | int     | int        |
| sfixed64 | 8字节编码                                        | long    | int/long   |
| bool     | 布尔值                                           | boolean | bool       |
| string   | 字符串，必须包含 utf-8编码 或 7-bit ASCⅡ text    | String  | string     |
| bytes    | 任意的字节序列                                   | String  | ByteString |
|          |                                                  |         |            |

##### 2、复合类型

**枚举**

在 Protocol Buffers 中，我们可以定义 枚举 和 枚举类型：

``` protobuf
enum MediaType {
	option allow_alias = true; // 使用别名
	PCM = 0;
	MP3 = 1;
	MP4 = 2;
	WAV = 3;
	AVI = 4;
}
MediaType medisType = 1;
```

枚举定义在一个消息内部或消息外部。若定义在message消息内部，而其他message又想使用，那么可以通过 MessageType.EnumType 的方式引用。

> 注意：
>
> 1、枚举定义的时候，必须要整第一个枚举值必须是 **0**，这样我们可以用 0 作为默认值和第一个元素，枚举值不能重复，除非使用 option allow_alias = true 选项来开启别名。
>
> 2、枚举值的范围是 32-bit int，但因为枚举值使用变长编码，所以不推荐使用负数作为枚举值，这样会带来效率问题。

**复合消息**
















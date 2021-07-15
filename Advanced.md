# 1 SSH

## 1.1 生成 SSH 公钥

如前所述，许多 Git 服务器都使用 SSH 公钥进行认证。为了向 Git 服务器提供 SSH 公钥，如果某系统用户尚未拥有密钥，必须事先为其生成一份。这个过程在所有操作系统上都是相似的。首先，你需要确认自己是否已经拥有密钥。默认情况下，用户的 SSH 密钥存储在其 ~/.ssh 目录下。进入该目录并列出其中内容，你便可以快速确认自己是否已拥有密钥：

```
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```

我们需要寻找一对以 id_dsa 或 id_rsa 命名的文件，其中一个带有 .pub 扩展名。.pub 文件是你的公钥，另一个则是与之对应的私钥。如果找不到这样的文件（或者根本没有 .ssh 目录），你可以通过运行 ssh-keygen 程序来创建它们。

```
$ ssh-keygen -o
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local
```

首先 ssh-keygen 会确认密钥的存储位置（默认是 .ssh/id_rsa），然后它会要求你输入两次密钥口令。如果你不想在使用密钥时输入口令，将其留空即可。然而，如果你使用了密码，那么请确保添加了 -o 选项，它会以比默认格式更能抗暴力破解的格式保存私钥。你也可以用 ssh-agent 工具来避免每次都要输入密码。

现在，进行了上述操作的用户需要将各自的公钥发送给任意一个 Git 服务器管理员 （假设服务器正在使用基于公钥的 SSH 验证设置）。他们所要做的就是复制各自的 .pub 文件内容，并将其通过邮件发送。公钥看起来是这样的：

```
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
NrRFi9wrf+M7Q== schacon@mylaptop.local
```

## 1.2 使用记录

尽量把 id_rsa 文件保存在 ~/.ssh/ 目录下面，改个后缀。

```
admin@host-172-28-34-29 ~ $ ssh-add ~/.ssh/id_rsa_hpd
Identity added: /home/admin/.ssh/id_rsa_hpd (/home/admin/.ssh/id_rsa_hpd)
admin@host-172-28-34-29 ~ $ cat ~/.ssh/id_rsa_hpd.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC8OOQAF7FE8W1xYmuQjjUZ542BmVPJouW9cUAl+ievHoLfo8yocB9JaCaC3cJInoixbj/1/AkoxpFefsGe90RU3D04zWogV9AARUC9hgPcctEKWIJlypr14c1W99nL53yIQR552vaH8cFCpwynYRWY+QJI+HfjDUiLcTbmDf12/aZVruFUfrL087RXyXymw5w5PZKk0Rs+C5gshHed/j0G92UDqCE9zgfSUA0dQaTfwncgjZ5kQaoZ1VC3tvqqAC5vWH+cBMMA6FKqc6iTsIKE/3iyKORSUNmakEkayYcEI04YevVxxveY6HHG1PgOf8Qkc2oR/YSBihiZnPZOu58V admin@host-172-28-34-29
```

如果执行 ssh-add 时出现 **Could not open a connection to your authentication agent** 在执行 ssh-add ~/.ssh/id_ras 时发生此错，

执行如下命令　`ssh-agent bash`
然后再执行 `ssh-add ~/.ssh/id_ras` 即可。

## 1.3 笔记

1. 生成自己的公钥和私钥

```
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_hupeidong -C "hupeidong@jd.com"

$ ssh-agent bash (如果下面命令报错，则执行此命令)

$ ssh-add ~/.ssh/id_rsa_hupeidong
```

在 git 和 gerrit 中添加生成的 `id_rsa_hupeidong.pub`

# 2 Pending Cache

可以建立一个 Pending Cache 服务，用于存放一个分片计算 common 数据的结果，其他分片则等待这个分片计算完成并且取到计算结果。取结果需要 rpc 调用。由于 prepare 和 取数是并行的，且取数的耗时更长，所以对于等待计算结果的机器也不会有很大的影响，而且能够减少 CPU。


# Protocol Buffers

## 1 Language Guide

### 1.1 定义一个消息类型

```
syntax = "proto3";

message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}
```

**[指定字段类型]**

在上面的例子中，所有字段都是标量类型：两个整型（page_number和result_per_page），一个string类型（query）。当然，你也可以为字段指定其他的合成类型，包括枚举（enumerations）或其他消息类型。

**[分配标识号]**

正如你所见，在消息定义中，每个字段都有唯一的一个数字标识符。这些标识符是用来在消息的二进制格式中识别各个字段的，一旦开始使用就不能够再改变。**注：[1,15]之内的标识号在编码的时候会占用一个字节。[16,2047]之内的标识号则占用2个字节。**所以应该为那些频繁出现的消息元素保留 [1,15]之内的标识号。切记：要为将来有可能添加的、频繁出现的标识号预留一些标识号。

最小的标识号可以从1开始，最大到2^29 - 1, or 536,870,911。不可以使用其中的[19000－19999]（ (从FieldDescriptor::kFirstReservedNumber 到 FieldDescriptor::kLastReservedNumber)）的标识号，Protobuf协议实现中对这些进行了预留。如果非要在.proto文件中使用这些预留标识号，编译时就会报警。同样你也不能使用早期保留的标识号。

**[指定字段规则]**

所指定的消息字段修饰符必须是如下之一：

singular：一个格式良好的消息应该有0个或者1个这种字段（但是不能超过1个）。
repeated：在一个格式良好的消息中，这种字段可以重复任意多次（包括0次）。重复的值的顺序会被保留。

在proto3中，repeated的标量域默认情况下使用packed。

**[添加更多消息类型]**

在一个.proto文件中可以定义多个消息类型。在定义多个相关的消息的时候，这一点特别有用——例如，如果想定义与SearchResponse消息类型对应的回复消息格式的话，你可以将它添加到相同的.proto文件中，如：

```
message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}

message SearchResponse {
    ...
}
```

**[保留标识符（Reserved）]**

如果你通过删除或者注释所有域，以后的用户可以重用标识号当你重新更新类型的时候。如果你使用旧版本加载相同的.proto文件这会导致严重的问题，包括数据损坏、隐私错误等等。现在有一种确保不会发生这种情况的方法就是指定保留标识符），protocol buffer的编译器会警告未来尝试使用这些域标识符的用户。

```
message Foo {
    reserved 2, 15, 9 to 11;
    reserved "foo", "bar";
}
```

注：不要在同一行reserved声明中同时声明域名字和标识号

**[从.proto文件生成了什么？]**

当用protocol buffer编译器来运行.proto文件时，编译器将生成所选择语言的代码，这些代码可以操作在.proto文件中定义的消息类型，包括获取、设置字段值，将消息序列化到一个输出流中，以及从一个输入流中解析消息。

对C++来说，编译器会为每个.proto文件生成一个.h文件和一个.cc文件，.proto文件中的每一个消息有一个对应的类。

### 1.2 默认值

当一个消息被解析的时候，如果被编码的信息不包含一个特定的singular元素，被解析的对象锁对应的域被设置位一个默认值，对于不同类型指定如下：

- 对于strings，默认是一个空string
- 对于bytes，默认是一个空的bytes
- 对于bools，默认是false
- 对于数值类型，默认是0
- 对于枚举，默认是第一个定义的枚举值，必须为0;
- 对于消息类型（message），域没有被设置，确切的消息是根据语言确定的。
- 对于可重复域的默认值是空（通常情况下是对应语言中空列表）。

注：对于标量消息域，一旦消息被解析，就无法判断域释放被设置为默认值（例如，例如boolean值是否被设置为false）还是根本没有被设置。你应该在定义你的消息类型时非常注意。例如，比如你不应该定义boolean的默认值false作为任何行为的触发方式。也应该注意如果一个标量消息域被设置为标志位，这个值不应该被序列化传输。

### 1.3 枚举

当需要定义一个消息类型的时候，可能想为一个字段指定某“预定义值序列”中的一个值。例如，假设要为每一个SearchRequest消息添加一个 corpus字段，而corpus的值可能是UNIVERSAL，WEB，IMAGES，LOCAL，NEWS，PRODUCTS或VIDEO中的一个。其实可以很容易地实现这一点：通过向消息定义中添加一个枚举（enum）并且为每个可能的值定义一个常量就可以了。

在下面的例子中，在消息格式中添加了一个叫做Corpus的枚举类型——它含有所有可能的值——以及一个类型为Corpus的字段：

```
message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
    enum Corpus {
        UNIVERSAL = 0;
        WEB = 1;
        IMAGES = 2;
        LOCAL = 3;
        NEWS = 4;
        PRODUCTS = 5;
        VIDEO = 6;
    }
    Corpus corpus = 4;
}
```

如你所见，Corpus枚举的第一个常量映射为0：每个枚举类型必须将其第一个类型映射为0，这是因为：
- 必须有有一个0值，我们可以用这个0值作为默认值。
- 这个零值必须为第一个元素，为了兼容proto2语义，枚举类的第一个值总是默认值。

你可以通过将不同的枚举常量指定为相同的值。如果这样做你需要将allow_alias设定位true，否则编译器会在别名的地方产生一个错误信息。

```
enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
}
enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    // RUNNING = 1;  // Uncommenting this line will cause a compile error inside
    // Google and a warning message outside.
}
```

枚举常量必须在32位整型值的范围内。因为enum值是使用可变编码方式的，对负数不够高效，因此不推荐在enum中使用负数。如上例所示，可以在一个消息定义的内部或外部定义枚举——这些枚举可以在.proto文件中的任何消息定义里重用。当然也可以在一个消息中声明一个枚举类型，而在另一个不同的消息中使用它——采用MessageType.EnumType的语法格式。

当对一个使用了枚举的.proto文件运行protocol buffer编译器的时候，生成的代码中将有一个对应的enum（对Java或C++来说），或者一个特殊的EnumDescriptor类（对Python来说），它被用来在运行时生成的类中创建一系列的整型值符号常量（symbolic constants）。

### 1.4 使用其他消息类型

你可以将其他消息类型用作字段类型。例如，假设在每一个SearchResponse消息中包含Result消息，此时可以在相同的.proto文件中定义一个Result消息类型，然后在SearchResponse消息中指定一个Result类型的字段，如：

```
message SearchResponse {
    repeated Result results = 1;
}

message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
}
```

**[导入定义]**

在上面的例子中，Result消息类型与SearchResponse是定义在同一文件中的。如果想要使用的消息类型已经在其他.proto文件中已经定义过了呢？你可以通过导入（importing）其他.proto文件中的定义来使用它们。要导入其他.proto文件的定义，你需要在你的文件中添加一个导入声明，如：

```
import "myproject/other_protos.proto";
```

默认情况下你只能使用直接导入的.proto文件中的定义。然而，有时候你需要移动一个.proto文件到一个新的位置，可以不直接移动.proto文件，只需放入一个伪 .proto 文件在老的位置，然后使用import public转向新的位置。import public 依赖性会通过任意导入包含import public声明的proto文件传递。例如：

```
// 这是新的proto
// All definitions are moved here

// 这是旧的proto
// 这是所有客户端正在导入的包
import public "new.proto";
import "other.proto";

// 客户端proto
import "old.proto";
// 现在你可以使用新旧两种包的proto定义了。
```

通过在编译器命令行参数中使用-I/--proto_pathprotocal 编译器会在指定目录搜索要导入的文件。如果没有给出标志，编译器会搜索编译命令被调用的目录。通常你只要指定proto_path标志为你的工程根目录就好。并且指定好导入的正确名称就好。

**[使用proto2消息类型]**

在你的proto3消息中导入proto2的消息类型也是可以的，反之亦然，然后proto2枚举不可以直接在proto3的标识符中使用（如果仅仅在proto2消息中使用是可以的）。

### 1.5 嵌套类型

你可以在其他消息类型中定义、使用消息类型，在下面的例子中，Result消息就定义在SearchResponse消息内，如：

```
message SearchResponse {
    message Result {
        string url = 1;
        string title = 2;
        repeated string snippets = 3;
    }
    repeated Result results = 1;
}

如果你想在它的父消息类型的外部重用这个消息类型，你需要以Parent.Type的形式使用它，如：

message SomeOtherMessage {
    SearchResponse.Result result = 1;
}

当然，你也可以将消息嵌套任意多层，如：

message Outer {          // Level 0
    message MiddleAA {   // Level 1
        message Inner {  // Level 2
            int64 ival = 1;
            bool booly = 2;
        }
    }
    message MiddleBB {   // Level 1
        message Inner {  // Level 2
            int32 ival = 1;
            bool booly = 2;
        }
    }
}
```

### 1.6 更新一个消息类型

支持更新消息类型。

https://developers.google.com/protocol-buffers/docs/proto#updating

### 1.7 Any

Any类型消息允许你在没有指定他们的.proto定义的情况下使用消息作为一个嵌套类型。一个Any类型包括一个可以被序列化bytes类型的任意消息，以及一个URL作为一个全局标识符和解析消息类型。为了使用Any类型，你需要导入import google/protobuf/any.proto

```
import "google/protobuf/any.proto";

message ErrorStatus {
    string message = 1;
    repeated google.protobuf.Any details = 2;
}
```

对于给定的消息类型的默认类型URL是type.googleapis.com/packagename.messagename。

不同语言的实现会支持动态库以线程安全的方式去帮助封装或者解封装Any值。例如在C++ 中会有PackFrom() 和UnpackTo() 方法。

```
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
    if (detail.Is<NetworkErrorDetails>()) {
        NetworkErrorDetails network_error;
        detail.UnpackTo(&network_error);
        ... processing network_error...
    }
}
```

目前，用于Any类型的动态库仍在开发之中。

### 1.8 Oneof

如果你的消息中有很多可选字段，并且同时至多一个字段会被设置，你可以加强这个行为，使用oneof特性节省内存.

Oneof字段就像可选字段，除了它们会共享内存，至多一个字段会被设置。设置其中一个字段会清除其它字段。你可以使用case()或者WhichOneof() 方法检查哪个oneof字段被设置，看你使用什么语言了.

**[使用Oneof]**

为了在.proto定义Oneof字段，你需要在名字前面加上oneof关键字, 比如下面例子的test_oneof:

```
message SampleMessage {
    oneof test_oneof {
        string name = 4;
        SubMessage sub_message = 9;
    }
}
```

然后你可以增加oneof字段到oneof定义中. 你可以增加任意类型的字段, 但是不能使用repeated关键字.

在产生的代码中，oneof字段拥有同样的 getters 和setters，就像正常的可选字段一样。也有一个特殊的方法来检查到底那个字段被设置。

**[Oneof 特性]**

设置oneof会自动清除其它oneof字段的值.所以设置多次后，只有最后一次设置的字段有值。

```
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();  // Will clear name field.
CHECK(!message.has_name());
```

如果解析器遇到同一个oneof中有多个成员，只有最会一个会被解析成消息。**oneof不支持repeated。**反射API对oneof字段有效.如果使用C++,需确保代码不会导致内存泄漏.下面的代码会崩溃，因为sub_message已经通过set_name() 删除了 SampleMessage message;

```
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");  // Will delete sub_message
sub_message->set_...       // Crashes here
```

在C++中，如果你使用Swap()两个oneof消息，每个消息，两个消息将拥有对方的值，例如在下面的例子中，msg1会拥有sub_message并且msg2会有name。

```
SampleMessage msg1;
msg1.set_name("name");
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```

**[向后兼容性问题]**

当增加或者删除oneof字段时一定要小心. 如果检查oneof的值返回None/NOT_SET, 它意味着oneof字段没有被赋值或者在一个不同的版本中赋值了。你不会知道是哪种情况，因为没有办法判断如果未识别的字段是一个oneof字段。

Tage 重用问题：

- 将字段移入或移除oneof：在消息被序列化或者解析后，你也许会失去一些信息（有些字段也许会被清除）
- 删除一个字段或者加入一个字段：在消息被序列号或者解析后，这也许会清除你现在设置的oneof字段
- 分离或者融合oneof：行为与移动常规字段相似。

### 1.9 Map（映射）

如果你希望创建一个关联映射，protocol buffer提供了一种快捷的语法：

```
map<key_type, value_type> map_field = N;
```

其中key_type可以是任意Integer或者string类型（所以，除了floating和bytes的任意标量类型都是可以的）value_type可以是任意类型。

例如，如果你希望创建一个project的映射，每个Projecct使用一个string作为key，你可以像下面这样定义：

```
map<string, Project> projects = 3;
```

Map的字段可以是repeated。

序列化后的顺序和map迭代器的顺序是不确定的，所以你不要期望以固定顺序处理Map。当为.proto文件产生生成文本格式的时候，map会按照key的顺序排序，数值化的key会按照数值排序。

从序列化中解析或者融合时，如果有重复的key则后一个key不会被使用，当从文本格式中解析map时，如果存在重复的key。

**[向后兼容性问题]**

map语法序列化后等同于如下内容，因此即使是不支持map语法的protocol buffer实现也是可以处理你的数据的：

```
message MapFieldEntry {
    key_type key = 1;
    value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

### 2.0 包

当然可以为.proto文件新增一个可选的package声明符，用来防止不同的消息类型有命名冲突。如：

```
package foo.bar;
message Open {
    ...
}
```

在其他的消息格式定义中可以使用包名+消息名的方式来定义域的类型，如：

```
message Foo {
    ...
    required foo.bar.Open open = 1;
    ...
}
```

对于C++，产生的类会被包装在C++的命名空间中，如上例中的Open会被封装在 foo::bar空间中。

**[包及名称的解析]**

Protocol buffer语言中类型名称的解析与C++是一致的：首先从最内部开始查找，依次向外进行，每个包会被看作是其父类包的内部类。当然对于（foo.bar.Baz）这样以“.”分隔的意味着是从最外围开始的。

ProtocolBuffer编译器会解析.proto文件中定义的所有类型名。对于不同语言的代码生成器会知道如何来指向每个具体的类型，即使它们使用了不同的规则。

### 2.1 定义服务(Service)

如果想要将消息类型用在RPC(远程方法调用)系统中，可以在.proto文件中定义一个RPC服务接口，protocol buffer编译器将会根据所选择的不同语言生成服务接口代码及存根。如，想要定义一个RPC服务并具有一个方法，该方法能够接收SearchRequest并返回一个SearchResponse，此时可以在.proto文件中进行如下定义：

```
service SearchService {
    rpc Search(SearchRequest) returns(SearchResponse);
}
```

最直观的使用protocol buffer的RPC系统是gRPC，一个由谷歌开发的语言和平台中的开源的PRC系统，gRPC在使用protocl buffer时非常有效，如果使用特殊的protocol buffer插件可以直接为您从.proto文件中产生相关的RPC代码。

如果你不想使用gRPC，也可以使用protocol buffer用于自己的RPC实现。

### 2.2 JSON 映射

Proto3 支持JSON的编码规范，使他更容易在不同系统之间共享数据，在下表中逐个描述类型。

如果JSON编码的数据丢失或者其本身就是，这个数据会在解析成protocol buffer的时候被表示成默认值。如果一个字段在protocol buffer中表示为默认值，在转化成JSON的时候编码的时候忽略掉以节省空间。

### 2.3 选项

在定义.proto文件时能够标注一系列的options。Options并不改变整个文件声明的含义，但却能够影响特定环境下处理方式。完整的可用选项可以在google/protobuf/descriptor.proto找到。

cc_enable_arenas(文件选项):对于C++产生的代码启用arena allocation

### 2.4 生成访问类

通过如下方式调用protocol编译器：

```
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --javanano_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

IMPORT_PATH声明了一个.proto文件所在的解析import具体目录。如果忽略该值，则使用当前目录。如果有多个目录则可以多次调用--proto_path，它们将会顺序的被访问并执行导入。-I=IMPORT_PATH是--proto_path的简化形式。
当然也可以提供一个或多个输出路径：
--cpp_out 在目标目录DST_DIR中产生C++代码，可以在C++代码生成参考中查看更多。
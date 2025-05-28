# mOSPF 报文
分为 Hello 报文和 LSU 报文
- **Hello报文**用于检测自己周围的链路信息，维护邻居表。
- **LSU报文**用于与网络中的其他所有节点同步各自检测到的链路信息，维护**LSDB**。**LSDB**可以看做每个节点上存储的所有链路的状态集合。

不同的 header 构造都有库函数可以使用
不同的报文头大小，都有定义的宏

### 通用报文头(Header)

![](https://netdocs.lab427.top/images/lab7/header.png)

- version：为mOSPF的版本号
- type：用于标识该报文是Hello类型还是LSU类型
- length：mOSPF头部以及负载的总长度
- Router ID：标识产生该mOSPF报文的节点
- Aera ID：区域ID，本次实验中无用
- checksum：检测报文是否损坏
- padding：填充，为0
**在 `mospf_proto.h`**中，有详细介绍
![[mospf-hdr-data-struct.png]]

### Hello报文

Hello报文用于维持邻居之间的关系，那么该报文中需要提供以下必要的信息：
- 发送方Router ID，
- 发送接口的IP地址，
- 接收接口的IP地址，
- 子网网段。
其中，Router ID可以从mOSPF Header中获取，IP地址可以直接从IP Header中获取，子网网段则通过mask与IP地址计算得到。
#### hello 内容
其数据结构封装为
```c
struct mospf_hello {

u32 mask; // network mask associated with this interface

u16 helloint; // number of seconds between hellos from this router


u16 padding; // set to zero

}__attribute__ ((packed));

#define MOSPF_HELLO_SIZE sizeof(struct mospf_hello)
```


![](https://netdocs.lab427.top/images/lab7/hello.png)

- mask：表示发送端口和接收端口之间的子网掩码
- interval：表示两个节点之间约定的Hello报文发送间隔，用于确定超时时间。在本实验中，所有路由器都以相同的间隔发送Hello报文，因此该字段意义不大。

#### 发送
- 周期性地在所有端口**多播**Hello报文，以维护邻居关系。**这里存在一个多播地址**的问题
	- `#define MOSPF_ALLSPFRouters 0xe0000005 // 224.0.0.5` ，这个宏可以使用。主机序存储。

### LSU报文

**LSU除了上述字段外，还会附带一系列的LSAs**

![](https://netdocs.lab427.top/images/lab7/lsu.png)

- seq：**同一节点**发送的LSU的该字段随时间严格递增，表示LSU的新旧
- ttl：控制LSU的生命周期，**LSU每被转发一次，该字段都要自减**
- nadv：表示该LSU中附带的LSAs的数量

### LSA

**LSA是附带在LSU中的信息。每一个LSA描述节点与它的一个邻居的链路信息。**

![](https://netdocs.lab427.top/images/lab7/lsa.png)

- network：邻居之间的子网网段
- mask：邻居之间的子网掩码
- Router ID则是邻居节点的编号

# Router 实例
Router 实例实际上就是每个节点运行程序的实例，其是一个路由程序。
结构如下
```c
typedef struct {

struct list_head iface_list; // the list of interfaces

int nifs; // number of interfaces

struct pollfd *fds; // structure used to poll packets among

// all the interfaces
 

#ifdef DYNAMIC_ROUTING

// used for mospf routing

u32 area_id;

u32 router_id;

u16 sequence_num;

int lsuint;

#endif

} ustack_t;

  

extern ustack_t *instance;
```
**有不同的接口**，存在 `iface_list` 中。
其中 `sequence_num` 是标识其版本信息，及其最新的 LSU 状态信息是什么样的，来保证系统的状态是最新的。

在 OSPF（Open Shortest Path First）协议中，序列号（Sequence Number）是用来标识链路状态公告**LSA，Link State Advertisement** 的版本号，它的主要作用是区分最新的 LSA 和旧的 LSA，确保网络中的路由器可以使用最新的链路状态信息来构建拓扑。

`area_id` 表示区域id，在本实验中没有意义；
`router_id` 表示该路由器的唯一标识；
`sequence_num` 表示LSU的序列号，用于表示LSU报文的新旧；`lsuint` 表示发送LSU的时间间隔（s）。
	序列号的更新在这个函数
```c
void mospf_init_lsu(struct mospf_lsu *lsu, u32 nadv)
{
	lsu->seq = htons(instance->sequence_num);
	lsu->unused = 0;
	lsu->ttl = MOSPF_MAX_LSU_TTL;
	lsu->nadv = htonl(nadv);
}
```

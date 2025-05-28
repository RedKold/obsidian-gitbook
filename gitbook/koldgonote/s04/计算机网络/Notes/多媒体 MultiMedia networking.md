# Video
- **定义**：
	- Video: sequence of images displayed at constant rate
	- Digital image: array of pixels
		- each pixel represented by bits
	- Coding: use redundancy *within* and between images to decrease # bits used to encode image
		- Example：
			- MPEG 1
			- MPEG2
			- MPEG4 (often used in Internet)
## 三种应用类型
## *Streaming, stored* audio video
- **streaming:** can begin playout before downloading entire file 
- **stored (at server):** can transmit faster than audio/video will be rendered (implies storing/buffering at client) 
- e.g., YouTube, Netflix

**缓存技术应用**

### Streaming multimedia：**UDP**
- Server sends at rate appropriate for client
	- often: send rate = encoding rate = constant rate
	- transmission rate can be oblivious to congestion levels
- 特点：短 playout delay (2-5 secs) to remove network jitter
- error recovery:
	- application-level, time permitting
- RTP: multimedia payload types
- UDP may *not* go through **firewalls**(由于是非安全协议)
### HTTP
通过 HTTP GET 来获得多媒体文件。（一块一块的数据拿过来）
- 以 *TCP*协议下的最大速度发送

- HTTP/TCP 更容易通过防火墙

### DASH
- ***DASH***：***D**ynamic **A**daptive **S**treaming over **H**TTP*
- 客户端是智能的，客户端决定：
	- 什么时候请求块 (When to request chunk)
	- 请求多大的码率。(higher quality when more bandwidth available)


## VoIP：packet loss, delay
- interactive nature of human-to-human conversation limits delay tolerance



# Internet QoS
为了适应实时通讯、大量客户端/服务端应用，大量图像的网站——所以必须支持 ***Quality of Service***(QOS) within TCP/IP
- In place of "best-effort"
- Add traffic control to routers
- Provide means of requesting QOS

**两大 QoS 框架**
- **Integrated Services Architecture(ISA)**
- **Differentiated Services(DS)**

## Integrated Services Architecture (ISA)
- Associate a distinguishable stream of IP packets with a flow 
	- With the same QOS parameters 
	- Identified by source and destination IP address, port numbers, protocol type (TCP or UDP) 
	- Unidirectional, Can be multicast

### ISA Functions
- **Routing Algorithm**
	- Link cost based on a variety of QOS parameters
	- Routing / forwarding based on classes of flows with similar QoS
- **Queuing discipline**
	- Priority queuing
	- Multiple queues instead of one, taking account of different flow requirements
- **Discard policy**
	- Selective discard instead of just new **comings**
- **Reservation protocol**
	- ***RSVP***, reserve resource for new flow at a given level of QOS
	- **Admission control**
		- Determines if sufficient resources are available for the flow at the requested QOS
	- **Traffic control database**
		- Parameters of traffic control
	- **Management agent**
		- Modifies the traffic control database
		- Directs the admission control module to set policies

## Differential Services (DS)


## Policing
- **目标**：限制 traffic 不超过给定的参数
- 三种常用的名词
	- （long term）average rate: how many pkts can be sent per unit time
	- **peak rate**:
	- (max) **burst size**: max number of pkts sent consecutively (with no intervening idle)
- 在包的流(packet flow, traffic)进入网络之前，塑造它的样子
	- *Control the rate* at which packets are sent
- Two traffic shaping algorithm
	- Leaky Bucket
	- Token Bucket


### Leaky Bucket
![[leaky-bucket-1.png]]
![[leaky-bucket-2.png]]
### Token Packet


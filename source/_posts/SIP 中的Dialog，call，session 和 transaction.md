---
title: SIP 中的Dialog，call，session 和 transaction
date: 2019-3-30
tags: [sip] 
---


### 一、基本概念

**1、Messages(消息)**

---

<span style="font-family:'楷体';font-size:17px;">消息是在服务器和客户端之间交换的独立文本, 有两种类型的消息,分别是*请求(Requests)和响应(Responses*)。</span>

<span style="font-size:14px;">两种类型的消息都由一个起始行、一个或多个头字段、一个标识头字段结束的空行、一个可选的消息体组成。</span>

<!--more-->

**2、Transaction(事务)** 

---

<span style="font-family:'楷体';font-size:17px;">事务发生于客户端和服务器端之间,包含从客户端发出请求给服务器,到服务器响应给客户端的最终消息(non-1xx message)之间的所有消息(***也就是说，事务是一次完整的请求***)。如果请求是一个"Invite"消息,并且最终的响应是一个non-2xx消息,那么该事务包含一个"Ack"响应消息.如果服务器的响应是一个2xx消息,那么,随后的ACK是一个单独的事务.</sapn>

<div style="font-size:14px;"> Branch是一个事务ID（Transaction ID），用于区分同一个Client所发起的不同Transaction。</div>
<br>
<span style="font-size:14px;">对于遵循RFC3261规范的实现，**这个branch参数的值必须用magic cookie”z9hG4bK”打头**。其它部分是对“To, From, Call-ID头域和Request-URI”按一定的算法加密后得到。</span>

<span style="font-size:14px">这7个字母是一个乱数cookie（定义成为7位的是为了保证旧版本的RFC2543实现不会产生这样的值），这样服务器收到请求之后，可以很方便的知道这个branch ID是否由本规范所产生的（就是说，全局唯一的）。</span>

**事务分类**

---

<span style="font-family:'楷体';font-size:17px;">事务根据类型还分Invite和Non-Invite型，*即邀请和非邀请类型*。Non-Invite类型事务主要处理的是除Invite和ACK类型外的所有Sip信息。而非Invite里的ACK信息要处理的话就不属于事务处理的范围了，一般由程序自己把信息发送给传输层直接发送。Invite需要三次握手，所以需要的时间比较长；而Non-Ivite类型只需两次握手，要求回应时间短。</span>


INVITE事务三次握手：

![title](/api/file/getImage?fileId=5aed0f3a09eb7d1b300000bd)

<span style="font-size:15px;">注意在上图这两个UA中，每一个代理服务器都将自己的地址加入返回的ACK的Via头域中，而非成功的transaction则不会加入。CSeq头域的值必须与INVITE相同，并且CSeq的方法必须是ACK。

**中间响应消息1xx的使用则是为了节省网络开销设计的，一旦 UC 收到任何一个中间响应消息，则UC必须停止消息重发定时器，不再重发这个请求消息，反之则直到收到最终响应消息或重发定时器超时。**一旦客户端UAC的事务在Calling状态收到任何中间响应消息1xx，事务则自动切换到Processing状态，停止请求消息的重发。并且需要将中间响应消息传送给TU事务用户。在呼叫业务中，TU以及上层应用可以根据中间响应消息在用户界面上提示用户。一旦事务切换到Processing状态，任何其他中间响应消息也都要传送给TU。</span>

<span style="font-family:'楷体';font-size:15px">**注意，从INVITE到200OK都是一个事务(branck值相同)，而最后的确认消息ACK却是一个的单独的事务。后面的BYE到200OK又属于另一个新的事务。**</span>

INVITE响应:

![invite](/api/file/getImage?fileId=5aed121009eb7d1b300000c0)

ACK响应:

![title](/api/file/getImage?fileId=5aed11f909eb7d1b300000bf)

<span style="font-family:'楷体';font-size:17px;">这里抓包和上面提到的一致。</span>


非INVITE二次握手：

![title](/api/file/getImage?fileId=5aed12b509eb7d1b300000c1)
<span style="font-size:14px;">当UAC发出非INVITE请求时，它就会在事务管理子层上开启定时器F（TCP）或者是E（UDP），确保超时的时候进行重传。这适用于除了 ACK请求外的其他非INVITE请求。每次超时重传时E的时间都被翻倍，直到最大的4秒。而F超时时，UAC就会认为是Timeout，这个事务将被删除。</span>


**3、Dialog(对话)**

---

<span style="font-family:'楷体';font-size:17px;">对话是两个UAs(user agent) 之间持续一段时间的端到端(peer-to-peer)的SIP 关系.&nbsp;`一个对话由一个Call-ID, 一个local tag 和 一个remote tag来标识.对话过去也叫做 "call leg"`。一个对话由SIP消息建立，就像用2xx响应INVITE请求。</sapn>


<span style="font-size:14px">**dialog的建立是客户端收到UAS的响应（To tag）时开始建立的。**收到180响应时建立dialog叫做早期对话（early dialog）,收到2XX的应答开始才是真正的dialog建立。</span>

**4、Session(会话)**

---

<span style="font-family:'楷体';font-size:16px;">用于进行媒体流传送。当一方发出请求，而另外一方或多方接受请求并通过信令交互成功后才能建立会话。**具体而言就是通过offer/answer方式交换sdp的媒体。** </span>

<span style="font-family:'楷体';font-size:17px;">具体来说，INVITE中的消息体用sdp语言来描述自己可处理的媒体类型，200OK中带回UAS端可处理的媒体类型。这个时候媒体交换就算是完成了。也就是session建立起来了。 </span>**只有当媒体协商成功后，会话才能被建立起来。**

<span style="font-family:'楷体';font-size:17px;">一次呼叫只能建立一次会话，但可以建立多个对话（Dialog），因为接受请求的可能不止一个。</span>


**5、Call(呼叫) **

---

<span style="font-family:'楷体';font-size:17px;">一个呼叫是由一个会议中被同一个发起者邀请加入的所有成员组成的。*一个 SIP 呼叫用全局唯一呼叫标识（CALL_ID）来识别。*因此，如果一个用户被不同的人邀请参加同一个多点会议，每个邀请都有一个唯一的呼叫。</sapn>

---

### SIP几个重要的参数
---

<span style="font-family:'楷体';font-size:17px;">1) 如下三个值相同代表同一个dailog（会话）</span>

<span style="font-family:'楷体';font-size:17px;">Call-id</span>

<span style="font-family:'楷体';font-size:17px;">Form  tag</span>

<span style="font-family:'楷体';font-size:17px;">To  tag</span>


<span style="font-family:'楷体';font-size:17px;">2）branch值相同，代表同一个 transaction(事务)</span>

<span style="font-family:'楷体';font-size:17px;">Branch</span>


<span style="font-family:'楷体';font-size:17px;">3） cseq</span>

<span style="font-family:'楷体';font-size:17px;">Cseq</span>

<span style="font-family:'楷体';font-size:17px;">其生存域是一个会话。用于将一个会话中的请求消息序列化，以便用于重复消息、“迟到”消息的检测，响应消息与相应请求消息的匹配等。包含两部分：一个32位的序列号，一个请求方法。
通常在会话开始时确定一个初始值，其后再发送消息时将该值加1。主叫方与被叫叫各自维护自己的CSeq序列，互不干扰，这有点像TCP/IP中IP包的序列号。
一个响应消息有与其对应的请求消息相同的CSeq值。</span>


<span style="font-size:14px">【注意】SIP中CANCEL消息与ACK消息总是比较特殊。CANCEL消息的CSeq中的序列号总是跟其要cancel的消息的相同，而对于ACK消息：如果它所要确认的是INVITE请求的non-2xx响应，则ACK消息的CSeq中的序列号与对应INVITE请求的相同；如果是2xx响应，则不同，此时ACK被当作一个新的事务。</sapn>


---

### Dialog，call，session 和 transaction关系图
---


![1.png](https://i.loli.net/2019/03/30/5c9f19baad7bd.png)

<span style="font-family:'楷体';font-size:17px;">Transaction:维护hop to hop状态，包括一个请求和其触发的所有响应，包括若干暂时响应和一个最终响应。生命周期从请求产生到收到最终响应。 </span>

<span style="font-family:'楷体';font-size:17px;">Dialog：维护peer to peer状态，目前只有invite和subscribe请求会触发dialog。其生命周期贯穿一个端到端会话的始终。</span>


---

#### Early dialog、Session、Dialog、Transaction等的在一个UA-UA的呼叫中的体现：

---

![2.png](https://i.loli.net/2019/03/30/5c9f19bb16f4c.png)


<span style="font-size:14px;">在这个例子中，通过INVITE事务而成功建立起来的dialog必须有一个ACK进行回应，这是第二个transaction的开始，尽管ACK并没有回复，但是由于新的 branch-value被填入，所以这个ACK代表了一个新的Transaction的开始。注意，此时 transaction number (CSeq) 并没有根据INVITE而增加--也就是说若收到的最终响应不是2XX（是3XX--6XX），则该transaction中包含ACK，若最终响应是2XX，则ACK属于一个新的transaction。</span>


---

**总的来说**:

- 1.对话和事务处于信令层，而会话处于媒体传输层。SIP使用SDP来通知传输层（RTP）来创建、增加、移除和修改会话。
- 2.一般来说，在会议应用中SIP可以通过请求来让另一方加入已有会话中。在这种情况下，新的对话会被创建。
- 3.对话是end-point对end-point的关系，即真实的通信双方，而transaction 是hop by hop的关系，即路由过程中交互的双方。




---
参考资料

[<span style="font-size:14px;">SIP协议入门：初学者必须明白的几个重要概念（原创）</span>](http://blog.sina.com.cn/s/blog_60e1d7bb0100f6er.html)
[<span style="font-size:14px;">SIP 中的Dialog，call，session 和 transaction</span>](https://www.cnblogs.com/matthew-2013/p/4939723.html)























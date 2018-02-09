该文档介绍所有子模块——`客户端`，`multipart`，`协议`和`工具类`中要被加载到aiohttp命名空间的名称信息。


# WebSocket 工具类
*class aiohttp.WSCloseCode*   
&ensp;&ensp;&ensp;一个保留关闭消息码的整数枚举类。   
&ensp;&ensp;&ensp;**OK**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 正常结束，表示目标连接已经成功建立。   

&ensp;&ensp;&ensp;**GOING_AWAY**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示服务器正在关闭或浏览器已离开页面。   

&ensp;&ensp;&ensp;**PROTOCOL_ERROR**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示由于协议错误引起的终止连接。   

&ensp;&ensp;&ensp;**UNSUPPORTED_DATA**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示因接收到不能接受到的数据类型引起的终止连接。（比如只能接受文本的端口却接受了二进制消息）   

&ensp;&ensp;&ensp;**INVALIED_TEXT**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示因接受到的数据中含有不能理解的消息类型引起的终止连接。（如在文本消息中出现非UTF-8编码的内容）    

&ensp;&ensp;&ensp;**POLICY_VIOLATION**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示因接收到的信息违反规定引起的终止连接。如果没有合适的状态码会返回通用状态码（比如`unsupported_data`或`message_too_big`）。

&ensp;&ensp;&ensp; **MESSAGE_TOO_BIG**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示因接受的消息（数量）太大引起的终止连接。    

&ensp;&ensp;&ensp; **MANDATORY_EXTENSION**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  表示因客户端期望与服务器协商更多的扩展类型但服务器没用在响应消息中回复此类内容引起的终止连接。扩展列表需要在Close帧中的/reason/部分显示。注意该状态码不会被服务器端使用，因为此状态码已代表WebSocket握手失败。  

&ensp;&ensp;&ensp; **INTERNAL_ERROR**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示服务器端因遇到一个期望之外的错误无法完成请求而引起的终止连接。   

&ensp;&ensp;&ensp; **SERVICE_RESTART**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  服务重启。客户端需要重新连接，如果确定重连需要等5-30S不等的时间。   

&ensp;&ensp;&ensp; **TRY_AGAIN_LATER**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  服务过载。客户端需要连接到不同的IP地址（如果有的话）或尝试重新连接。   

*class aiohttp.WSMsgType*  
&ensp;&ensp;&ensp; 描述`WSMessage`类型的`整数枚举(IntEnum)`类。   

&ensp;&ensp;&ensp; **CONTINUATION**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  用于连接帧的标记，用户不会收到此消息类型。  

&ensp;&ensp;&ensp; **TEXT**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  文本消息，值为字符串。   

&ensp;&ensp;&ensp; **BINARY**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  二进制类型，值为字节码。    

&ensp;&ensp;&ensp; **PING**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  代表Ping帧（由客户端发送）。    

&ensp;&ensp;&ensp; **PONG**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 代表Pong帧，用于回复ping。由服务器发送。    

&ensp;&ensp;&ensp; **CLOSE**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  代表Close帧。  

&ensp;&ensp;&ensp; **CLOSED FRAME**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  不是一个帧，只是一个代表websocket已被关闭的标志。   

&ensp;&ensp;&ensp; **ERROR**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  不是一个帧，只是一个代表websocket接受到一个错误的标志。   

*class aiohttp.WSMessage*   
&ensp;&ensp;&ensp; WebSocket信息，由 `.receive()`调用得到。  
&ensp;&ensp;&ensp;**type（类型）**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  消息类型，是一个`WSMsgType`实例对象。   

&ensp;&ensp;&ensp;**data（数据）**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  消息载体。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  1. `WSMsgType.TEXT`消息的类型为`str`。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  2. `WSMsgType.BINARY`消息的类型为`bytes`。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  3. `WSMsgType.CLOSE`消息的类型为`WSCloseCode`。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  4.`WSMsgType.PING`消息的类型为`bytes`。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  5. `WSMsgType.PONG`消息的类型为`bytes`。    

&ensp;&ensp;&ensp; **extra**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 额外信息，类型为字符串。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 只对`WSMsgType.CLOSE`消息有效，内容包含可选的消息描述。    

&ensp;&ensp;&ensp; **json(\*, loads=json.loads)**   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  返回已解析的JSON数据。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  新增于0.22版本。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  **参数：** loads - 自定义JSON解码函数。    

&ensp;&ensp;&ensp; **tp**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 不赞成使用的**type**别名函数。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 1.0版本后不再建议使用。   

# 信号
信号是一个包含注册过的异步回调函数的列表。   
信号的生命周期有两个阶段: 在信号的内容被标准列表操作所填充之后: sig.append()之类的。     
第二个是sig.freeze()调用之后，在这之后信号会被冻结: 添加，删除和丢弃回调函数都是被禁止的。     
唯一可做的就是调用之前已经注册过的回调函数: await sig.send(data) 。   

更多实用例子请看<a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/ServerUsage.md#信号">aiohttp.web中的信号</a>章节。    

*class aiohttp.Signal*     
&ensp;&ensp;&ensp;  信号组件，具有collections.abc.MutableSequence接口。    

&ensp;&ensp;&ensp; *coroutine send(\*args, \*\*kwargs)*    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 从列表头部开始逐个调用已注册的回调函数。    

&ensp;&ensp;&ensp; *frozen*    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;如果 freeze()被调用过则为True。该属性只读。    

&ensp;&ensp;&ensp; *freeze()*   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  冻结列表。在这之后所有内容均不允许改动。    












 







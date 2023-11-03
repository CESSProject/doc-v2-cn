本节描述了节点之间的消息设计，包括消息交换格式和消息协议。

# 消息交换格式

节点之间的消息交换格式是采用 [**protobuf**](https://protobuf.dev/)。 protobuf 是一种灵活、高效的结构数据序列化方法，具有编码后体积更小，编解码速度更快等高效的时间效率和空间效率优势。

# 消息协议

通用消息是所有节点通信中必须携带的信息，通用消息中包括了对等节点的使用版本号、时间戳、消息 ID、是否广播此条消息给邻近节点、节点 ID、节点公钥、节点数据签名，通用消息的主要作用是验证节点的身份以及数据的签名。通用消息协议设计如下：

```protobuf
message MessageData {
  string clientVersion = 1; // client version
  int64 timestamp = 2;      // unix timestamp
  string id = 3;            // allows requesters to use request data when processing a response
  bool gossip = 4;          // true to have receiver peer gossip the message to neighbors
  string nodeId = 5;        // id of node that created the message (not the peer that may have sent it). =base58(multihash(nodePubKey))
  bytes nodePubKey = 6;     // Authoring node Secp256k1 public key (32bytes) - protobufs serielized
  bytes sign = 7;           // signature of message data + method specific data by message authoring node.
}
```

# 写入数据消息协议

写数据消息协议描述了一个节点如何将自己的数据写给另一个节点，其中包括请求消息和响应消息，写数据消息协议格式定义如下：

```protobuf
message WritefileRequest {
  MessageData messageData = 1;
  string Roothash =2;  // Roothash uniquely identifies a user data
  string Datahash = 3; // Datahash is the currently written data hash value
  uint32 Length = 4;   // Length is the length of the data written this time
  uint32 Offset = 5;   // Offset is the offset of this write
  bytes Data = 6;      // Data is the data written this time
}

message WritefileResponse {
  MessageData messageData = 1;
  uint32 Code = 2;  // Code indicates the result of this transfer
  uint32 Offset =3; // Offset is the write offset the receiver wants
}
```

写入数据请求消息协议 `WritefileRequest` 中定义了数据的 offset，基于此可以实现断点续写功能，只要对等方在写数据响应消息中告诉写入方 offset，写入方就可以从 offset 指定的位置开始写入，不用每次都从起始位置开始写入。

# 读取数据消息协议

读取数据消息协议描述了一个节点如何从另一个节点读取自己想要的数据，其中包括请求消息和响应消息，读数据消息协议格式定义如下：

```protobuf
message ReadfileRequest {
  MessageData messageData = 1;
  string Roothash =2;   // Roothash uniquely identifies a user data
  string Datahash = 3;  // Datahash is the currently written data hash value
  uint32 Offset = 4;    // Offset is the offset that the reader wants to read
}

message ReadfileResponse {
  MessageData messageData = 1;
  uint32 Code = 2;   // Code indicates the result of this transfer
  uint32 Offset =3;  // Offset is the data offset returned by the peer
  uint32 Length = 4; // Length is the returned data length
  bytes Data = 5;    // Data is the returned data
}
```

`ReadfileRequest` defines the data offset of reading, which enables breakpoint and continuation setup in the reading process. As long as the reader informs the peer about the offset in `ReadfileResponse` message, the peer can start reading from the position specified by the offset.

读数据请求消息协议 `ReadfileRequest` 中定义了数据的读取 offset，基于此可以实现断点续读功能，只要读取方在请求消息中告诉对等方 offset，对等方就可以从 offset 指定的位置开始返回。

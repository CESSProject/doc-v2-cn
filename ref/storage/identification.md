一个存储节点使用 **multi-address** 和 **Peer ID** 的组合来作为自己唯一的身份标识，如下例所示：

```
/ip4/127.0.0.1/tcp/80/p2p/16Uiu2HAkucvAj8FaUwfMZunCnKB8dGTE4GzX91hVuRV33zLxzZq6
```

- *ip4*: 基于一个 IPv4 地址的通信
- *127.0.0.1*: 通信 IP 地址
- *tcp*: 使用 TCP 协议进行通信
- *p2p*: 这是一种 P2P 通信
- *16Ui...zZq6*: 节点 ID

通过这个唯一标识符，网络中的其它对等方就能与自己建立正确的连接，还能验证对等方的身份和数据信息。

有关 multi-address 的更多信息请参考：[libp2p-Addressing](https://docs.libp2p.io/concepts/fundamentals/addressing/)。而有关 Peer ID 的更多信息请参考：[libp2p-PeerID](https://docs.libp2p.io/concepts/fundamentals/peers/)。

https://blog.csdn.net/SFC0511/article/details/101454415

1.打开手机中的蓝牙功能有哪些方法？
法1：使用Intent  ==>new Intent(BluetoothAdaper.ACTION_REQUEST_ENABLE); startActivityForResult(intent,1);
法2：使用BluetoothAdapter ==>BluetoothAdapter.getDefaultAdapter().enable();
方法1会弹出一个对话框，方法2直接打开，但是需要在清单中配置权限。
 

2.搜索蓝牙的过程会经过哪些状态？
搜索到蓝牙设备状态，采用了广播BluetoothDevice.ACTION_FOUND
搜索完成，采用了广播BluetoothAdapter.ACTION_DISCOVERY_FINISHED
 
3.两个蓝牙设备之间通信需要指定哪些信息？
蓝牙设备的地址
UUID
 
4.如何获得Wi-Fi相关的信息？
通过WifiManager获取wifi管家
从wifi管家中获取WifiInfo对象
	
---
https://my.oschina.net/huangmc/blog/2414475
Android Bluetooth API (1) - 蓝牙的打开、发现、连接与管理
**BluetoothAdapter**
表示本地蓝牙适配器（蓝牙无线装置）。 BluetoothAdapter 是所有蓝牙交互的入口点。
利用它可以发现其他蓝牙设备，查询绑定（配对）设备的列表，使用已知的 MAC 地址实例化 BluetoothDevice，以及创建 BluetoothServerSocket 以侦听来自其他设备的通信。

**BluetoothDevice**
表示远程蓝牙设备。利用它可以通过 BluetoothSocket 请求与某个远程设备建立连接，或查询有关该设备的信息，例如设备的名称、地址、类和绑定状态等。

**BluetoothSocket**
表示蓝牙套接字接口（与 TCP Socket 相似）。
这是允许应用通过 InputStream 和 OutputStream 与其他蓝牙设备交换数据的连接点。

**BluetoothServerSocket**
表示用于侦听传入请求的开放服务器套接字（类似于 TCP ServerSocket）。 
要连接两台 Android 设备，其中一台设备必须使用此类开放一个服务器套接字。 
当一台远程蓝牙设备向此设备发出连接请求时， BluetoothServerSocket 将会在接受连接后返回已连接的 BluetoothSocket。

#### 服务器端连接

设置服务器套接字并接受连接的基本过程：

1. 通过 BluetoothAdapter#listenUsingRfcommWithServiceRecord(String UUID) 获取 BluetoothServerSocket 该字符串是您的服务的可识别名称，系统会自动将其写入到设备上的新服务发现协议 (SDP) 数据库条目（可使用任意名称，也可直接使用您的应用名称）。 
2. 通过调用 BluetoothServerSocket#accept() 开始侦听连接请求 这是一个阻塞调用。它将在连接被接受或发生异常时返回。 
仅当远程设备发送的连接请求中所包含的 UUID 与向此侦听服务器套接字注册的 UUID 相匹配时，连接才会被接受。 操作成功后，accept() 将会返回已连接的 BluetoothSocket。
3. 除非您想要接受更多连接，否则请调用 BluetoothServerSocket#close 这将释放服务器套接字及其所有资源，但不会关闭 accept() 所返回的已连接的 BluetoothSocket。 与 TCP/IP 不同，RFCOMM 一次只允许每个通道有一个已连接的客户端

#### 客户端的连接
要发起与远程设备（保持开放的服务器套接字的设备）的连接，必须首先获取表示该远程设备的 BluetoothDevice 对象。
 然后必须使用 BluetoothDevice 来获取 BluetoothSocket 并发起连接。
 过程主要是：
 1. 调用BluetoothDevice#createRfcommSocketToServiceRecord(UUID) 获取 BluetoothSocket。 要使用相同的 UUID。
 2. 通过调用 BluetoothSocket#connect发起连接。  此方法为阻塞调用。 如果由于任何原因连接失败或 connect() 方法超时（大约 12 秒之后），它将会引发异常。

#### 管理连接 
利用 BluetoothSocket，传输任意数据的一般过程非常简单：
- 获取 InputStream 和 OutputStream，二者分别通过套接字以及 getInputStream() 和 getOutputStream() 来处理数据传输。
- 使用 read(byte[]) 和 write(byte[]) 读取数据并写入到流式传输。 就这么简单。

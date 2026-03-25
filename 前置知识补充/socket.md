# socket介绍

我们可以通过Socket技术（它是计算机之间进行**通信**的**一种约定**或一种方式），实现两台计算机之间的通信，Socket也被翻译为`套接字`，是操作系统底层提供的一项通信技术，它支持TCP和UDP。

而Java就对Socket底层支持进行了一套完整的封装，我们可以通过Java来轻松实现Socket通信。

要实现Socket通信，我们必须创建一个数据发送者和一个数据接收者，也就是客户端和服务端，我们需要提前启动服务端，来等待客户端的连接，而客户端只需要随时启动去连接服务端即可，它们默认采用的是TCP协议进行连接。

## ServerSocket

首先编写服务端，服务端使用ServerSocket对象来实现，它代表我们的Socket服务端，还记得我们之前说过每个应用程序都需要一个端口来进行TCP通信吗，我们可以为其绑定一个用于通信的端口以便客户端可以进行连接：

```java
ServerSocket server = new ServerSocket(8080)  //参数为绑定的端口，之后一律使用此端口进行通信
```
### try-with-resources
创建好后，由于服务端会一直占用资源，我们在使用完成后也需要对其资源进行释放，和之前IO一样，这里我们直接使用try-with-resource语法来编写：

```java
try(ServerSocket server = new ServerSocket(8080)) {
    
} catch (IOException e) {
    e.printStackTrace();
}
```

## socket
### accept

接着，我们可以调用`accept()`方法来监听客户端连接，如果没有客户端连接，程序会阻塞在此位置等待连接：

```java
try(ServerSocket server = new ServerSocket(8080)) {
    server.accept();   //等待客户端连接
} catch (IOException e) {
    e.printStackTrace();
}
```

当客户端连接到来后，`accept()`方法会返回应该Socket对象作为结果，它代表一个客户端Socket连接，我们可以打印看看客户端连接的相关信息：

```java
Socket socket = server.accept();
System.out.println("接受到来自客户端的连接: " + socket.getInetAddress() + ":" + socket.getPort());
```

接着就是编写客户端了，我们可以使用Socket对象来完成，其中参数分别是服务端地址和端口，这里我们因为是本地启动的服务端，相当于连接自己，所以说直接使用我们本机的IP即可：

```java
Socket socket = new Socket("localhost", 8080)  //填写连接服务端的信息
```
### ip地址
各位小伙伴可以尝试输入一下`ipconfig`命令查看网络列表，这里解释一下localhost、192.168.x.x、127.0.0.1的区别：

> 所有计算机都有一个特殊的网络和IP地址，127.0.0.1，它被称作是本地环回地址，我们访问此IP地址等于访问这台电脑自己，这个地址主要用于本地主机和本地服务之间的通信，所以说如果我们要访问自己电脑上的服务端，只需要填写这个IP地址即可。
> 
> 那这个跟我们从路由器得到的IP地址有什么区别吗，路由器得到的192.168.XX.XX是路由器为我们分配的一个局域网地址，使用自己的地址同样可以代表这台计算机本身，但是当我们切换网络时，局域网地址可能会出现变化，所以说它不适合作为本地连接的IP地址使用。
> 
> localhost实际上是一个域名，但是它等价于127.0.0.1，操作系统自带的域名解析（通过本地主机文件hosts实现）可以将其自动解析到127.0.0.1上，所以说很多时候我们使用localhost也可以代表本地主机。但是注意，在我们之后学习Web服务器时，浏览器会将它们认为是两个不同的站点。


当Socket对象创建时，就会自动进行连接了：

```java
try (Socket socket = new Socket("localhost", 8080)){
    System.out.println("已连接到服务端！");
}catch (IOException e){
    e.printStackTrace();
}
```

现在我们先启动一下服务端接着再启动客户端，此时就可以完成连接了：

![QQ_1721286252875](https://oss.itbaima.cn/internal/markdown/2024/07/18/ecTbi7128ZvIE6t.png)

注意客户端不需要指定自己的端口，一般都是自动分配，所以说我们收到来自客户端的连接时一般都是一个随机的端口号，然后前面的IP地址就是客户端用于访问我们服务端的IP地址，如果各位小伙伴使用局域网IP地址访问的话，这里会出现一些变化。



# socket通信
## 传输文件

server端
~~~java
public class Server {  
    static void main(String[] args) {  
        try(ServerSocket server = new ServerSocket(8080)){  
            Socket socket = server.accept();  
            System.out.println("已建立连接，ip为 "+socket.getInetAddress()+ "端口为 "+socket.getPort());  
            InputStream in = socket.getInputStream();  
            FileOutputStream out = new FileOutputStream("filetest");  
            long len = 0,total =0; //len是read到的字节流的长度，total是总共读取了多少字节  
            byte[] buffer = new byte[1024]; //缓冲区  
            while((len = in.read(buffer))>0){  
                total+=len;  
                System.out.println("服务器已读取到 "+ total + " 字节数据");  
                out.write(buffer,0,(int)len);  
            }  
        }catch (IOException r){  
            r.printStackTrace();  
        }  
    }  
}
~~~

client端
~~~java
public class Client {  
    static void main(String[] args){  
        try(Socket socket = new Socket("localhost",8080)){  
            System.out.println("请输入要传输的文件的路径：");  
            Scanner scanner = new Scanner(System.in);  
            String path = scanner.nextLine();  
            OutputStream out = socket.getOutputStream();  
            FileInputStream in = new FileInputStream(path);  
            in.transferTo(out);  
        }catch (IOException e){  
            e.printStackTrace();  
        }  
    }  
}
~~~


## socket实现UDP通信

我们接着来看如何使用Socket实现UDP通信，Java为我们提供了DatagramSocket API，它是一种UDP Socket，用于发送和接收UDP数据报。

由于UDP不像TCP那样需要提前连接，所以我们只需要创建一个Socket等待数据到来即可：

```java
try(DatagramSocket socket = new DatagramSocket(8080)) {   //绑定8080端口

} catch (IOException e) {
    e.printStackTrace();
}
```

接着我们即可连续读取客户端发来的数据：

```java
while(true) {
  	//UDP数据包，类似于数据缓冲区，数据会先被缓存到里面
    DatagramPacket packet = new DatagramPacket(new byte[1024], 1024);
    socket.receive(packet);   //将收到的数据缓冲到DatagramPacket中
    System.out.println(new String(packet.getData()));  //取出数据并打印
}
```

客户端这边由于不需要预先建立连接，所以我们直接创建一个UDP数据包，并在数据包中指定发送的IP地址和端口等内容，发送后直接就可以到达：

```java
try (DatagramSocket socket = new DatagramSocket();
     Scanner scanner = new Scanner(System.in)) {
    while (true) {
        String str = scanner.nextLine();
        byte[] data = str.getBytes();
        InetAddress address = InetAddress.getByName("127.0.0.1");  //直接在数据包中填写要发送到的目标主机IP地址和端口信息
        DatagramPacket packet = new DatagramPacket(data, data.length, address, 8080);
        socket.send(packet);
    }
} catch (IOException e){
    e.printStackTrace();
}
```

这个相比TCP就简单很多了，直接发送就能收到


### DataGramSocket

DatagramPacket作为一个纯粹的内存对象，其核心任务是处理**字节数组**与**网络地址**之间的绑定。
- **数据载体**：它内部封装了一个 `byte[]` 数组，这是你要发送或接收的原始二进制数据（无格式字节流）。    
- **地址标注**：对于**发送方**，它记录了目标机器的 IP 地址和端口号；对于**接收方**，它记录了发送者的 IP 地址和端口号（操作系统会自动填入）。



# 浏览器访问服务端（用socket
最后我们来研究一下HTTP协议，我们前面说过，浏览器访问一个网站实际上用的就是HTTP协议，并且HTTP协议是基于TCP协议的，所以说我们可以创建一个ServerSocket来处理浏览器的访问请求，看看HTTP请求到底长啥样。首先我们还是直接编写一个服务端来等待连接：

```java
try(ServerSocket server = new ServerSocket(8080)){
    Socket socket = server.accept();
    InputStream in = socket.getInputStream();
    while (!socket.isClosed()) {
        int i = in.read();
        if(i == -1) break;
        System.out.print((char) i);
    }
}catch (IOException e){
    e.printStackTrace();
}
```

此时我们使用Chrome浏览器访问：\[http://localhost:8080]
可以直接在服务端收到以下数据：

![QQ_1721298866890](https://oss.itbaima.cn/internal/markdown/2024/07/18/eQdhX6LrRwJbpTK.png)

这正是我们之前说的HTTP协议对应的报文格式，可以看到当我们使用浏览器访问服务端时，浏览器会默认向我们的服务端发送一个GET请求，路径为根路径"/"，并且还包含了很多请求标头。

但是此时我们会发现，浏览器会一直处于加载中状态：

![QQ_1721299010236](https://oss.itbaima.cn/internal/markdown/2024/07/18/REX4Q2Og8em1WGK.png)

这是因为HTTP规定请求完成之后服务端需要给一个响应结果，所以说浏览器实际上一直在等待我们的服务端发送一个请求结果给它，但是我们这里没有做任何处理，所以说就会一直卡住。

现在我们来尝试给它返回一个简单的HTML页面（看不懂代码没关系，这里只是测试一下）

```java
try(ServerSocket server = new ServerSocket(8080)){
    Socket socket = server.accept();
    OutputStreamWriter writer = new OutputStreamWriter(socket.getOutputStream());
    String html = """
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <title>测试网站</title>
            </head>
            <body>
                <h1>欢迎访问我们的测试网站</h1>
                <p>这个网站包含很多你喜欢的内容，但是没办法展示出来，因为我们还没学会</p>
            </body>
            """;
    writer.write("HTTP/1.1 200 OK\r\n");   //根据HTTP协议规范，返回对应的响应格式
    writer.write("Content-Type: text/html;charset=utf-8\r\n");  //务必加一下内容类型和编码，否则会乱码
    writer.write("\r\n");
    writer.write(html);
    writer.flush();
}catch (IOException e){
    e.printStackTrace();
}
```

此时我们使用Chrome浏览器再次访问服务端，就可以展示出我们返回的数据了：

![QQ_1721299638581](https://oss.itbaima.cn/internal/markdown/2024/07/18/CT3YFcXf5HOiPne.png)
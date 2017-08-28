---
title: '[每天进步一点点]NIO-Channel(一)'
author: lujing
tags:
  - java
  - nio
categories:
  - nio
date: 2017年08月28日 14时48分10秒
---
## Channel
&emsp;&emsp;Java NIO的通道类似IO的流，但又有些不同：
* 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
* 通道可以异步地读写。
* 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。
它实际上是双向交流的通道。
![enter description here][1]

## Channel的实现

&emsp;&emsp;Channel是一个接口，只有两个方法`close()`和 `isOpen()`，从JDK文档中，所有已知的实现如下，其中重要的标注下：
- AbstractInterruptibleChannel
- AbstractSelectableChannel
- AsynchronousFileChannel
- AsynchronousServerSocketChannel
- AsynchronousSocketChannel
- `DatagramChannel`
- `FileChanne`
- Pipe.SinkChannel
- Pipe.SourceChannel
- SelectableChannel
- `ServerSocketChannel`
- `SocketChannel`

`FileChannel`从文件中读写数据。

`DatagramChannel`能通过UDP读写网络中的数据。

`SocketChannel`能通过TCP读写网络中的数据。

`ServerSocketChannel`可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

## FileChannel示例
&emsp;&emsp;下面代码使用RandomAccessFile对文件进行读写操作：
```
public class ChannelFile {
    public static void main(String[] args){
        try{
            //RandomAccessFile 随机存取文件的读和写
            RandomAccessFile aFile = new RandomAccessFile("E:\\githubWorkspace\\javaLearn\\src\\main\\resources\\nio\\channel\\data\\nio-data.txt", "rw");
            FileChannel inChannel = aFile.getChannel();
            ByteBuffer buf = ByteBuffer.allocate(48);
            int bytesRead = inChannel.read(buf);
            while (bytesRead != -1) {
                System.out.println("Read " + bytesRead);
                buf.flip();
                while(buf.hasRemaining()){
                    System.out.print((char) buf.get());
                }
                buf.clear();
                bytesRead = inChannel.read(buf);
            }
            String newData = "\nNew String to write to file..." + System.currentTimeMillis();

            buf.clear();
            buf.put(newData.getBytes());

            buf.flip();

            while(buf.hasRemaining()) {
                inChannel.write(buf);
            }
            inChannel.close();
            aFile.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

原文整理：http://tutorials.jenkov.com/java-nio/index.html


  [1]: http://otqn63yyl.bkt.clouddn.com/github/images/170828/overview-channels-buffers.png
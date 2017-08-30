---
title: '[每天进步一点点]NIO-Buffer(二)'
author: lujing
tags:
  - java
  - nio
categories:
  - nio
date: 2017年08月29日 14时15分59秒
---
## Buffer

&emsp;&emsp;Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

 1. Buffer的基本用法
 2. Buffer的capacity,position和limit
 3. Buffer的类型
 4. Buffer的分配
 5. 向Buffer中写数据
 6. flip()方法
 7. 从Buffer中读取数据
 8. clear()与compact()方法
 9. mark()与reset()方法
 10. equals()与compareTo()方法
 
### Buffer的基本用法
&emsp;&emsp;使用Buffer读写数据一般遵循以下四个步骤：
 1. 写入数据到Buffer
 2. 调用`buffer.flip()`
 3. 从Buffer中读取数据
 4. 调用`buffer.clear()`或者`buffer.compact()`
 
&emsp;&emsp;当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过`flip()`方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。

&emsp;&emsp;一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用`clear()`或`compact()`方法。`clear()`方法会清空整个缓冲区。`compact()`方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

&emsp;&emsp;下面是一个使用Buffer的例子：
```
	public static void main(String[] args){
        try{
            //RandomAccessFile 随机存取文件的读和写
            RandomAccessFile aFile = new RandomAccessFile("E:\\githubWorkspace\\javaLearn\\src\\main\\resources\\nio\\channel\\data\\nio-data.txt", "rw");
            FileChannel inChannel = aFile.getChannel();
            //create buffer with capacity of 48 bytes
            ByteBuffer buf = ByteBuffer.allocate(48);

            int bytesRead = inChannel.read(buf); //read into buffer.
            while (bytesRead != -1) {
                buf.flip();  //make buffer ready for read
                while(buf.hasRemaining()){
                    System.out.print((char) buf.get()); // read 1 byte at a time
                }
                buf.clear(); //make buffer ready for writing
                bytesRead = inChannel.read(buf);
            }
            aFile.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
### Buffer的capacity,position和limit
&emsp;&emsp;缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

&emsp;&emsp;为了理解Buffer的工作原理，需要熟悉它的三个属性：

| 属性 |  描述   |
| --- | --- |
|  capacity   |   	&emsp;&emsp;作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。   |
|  limit   |  &emsp;&emsp;在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。</br>&emsp;&emsp;当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）   |
|  position   |  &emsp;&emsp;当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1。</br>&emsp;&emsp;当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。  |

&emsp;&emsp;position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。

&emsp;&emsp;这里有一个关于capacity，position和limit在读写模式中的说明，详细的解释在上面表格中。

![enter description here][1]
&emsp;&emsp;以下不变保持标记、位置、限制和容量值：

0 <= mark <= position <= limit <= capacity 

&emsp;&emsp;一个新创建的缓冲区始终有一个零的位置和一个未定义的标记。最初的限制可能是零，或者它可能是一些其他的值取决于缓冲区的类型和它的方式，它是构造。一个新分配的缓冲区的每个元素都被初始化为零。

### Buffer的类型
&emsp;&emsp;JDK文档中的，所有直接子类：`ByteBuffer`， `CharBuffer`， `DoubleBuffer`， `FloatBuffer`， `IntBuffer`， `LongBuffer`， `ShortBuffer`，ByteBuffer的子类`MappedByteBuffer`是专门用于内存映射的ByteBuffer。
```
public abstract class MappedByteBuffer
    extends ByteBuffer
	
```
### Buffer的分配
&emsp;&emsp;所有的缓冲区类都不能直接使用new关键字实例化，它们都是抽象类，但是它们都有一个用于创建相应实例的静态工厂方法，以ByteBuffer类为例子：
```
ByteBuffer buff = ByteBuffer.allocate(10);  
```
&emsp;&emsp;上面代码将会从堆空间中分配一个容量大小为10的byte数组作为缓冲区的byte数据存储器。对于其他缓冲区类上面方式也适用，如创建容量为10的CharBuffer：
```
CharBuffer buff = CharBuffer.allocate(10);
```

&emsp;&emsp;如果想用一个指定大小的数组作为缓冲区的数据的存储器，可以使用wrap()方法：
```
byte[] bytes = new byte[10];  
ByteBuffer buff = ByteBuffer.wrap(bytes);  
```
&emsp;&emsp;上面代码中缓冲区的数据会存放在bytes数组中，bytes数组或buff缓冲区任何一方中数据的改动都会影响另一方。还可以创建指定初始位置(position)和上界(limit)的缓冲区： 
```
byte[] bytes = new byte[10];  
ByteBuffer buff = ByteBuffer.wrap(bytes, 3, 8);  

```
### 向Buffer中写数据
&emsp;&emsp;写数据到Buffer有两种方式：
* 从`Channel`写到`Buffer`。
* 通过`Buffer`的`put()`方法写到`Buffer`里。

&emsp;&emsp;从Channel写到Buffer的例子
```
	int bytesRead = inChannel.read(buf); //read into buffer.
```
&emsp;&emsp;通过put方法写Buffer的例子：
```
	buf.put(127);
```
&emsp;&emsp;put方法有很多版本，允许你以不同的方式把数据写入到Buffer中。例如， 写到一个指定的位置，或者把一个字节数组写入到Buffer。 更多Buffer实现的细节参考JavaDoc。

### flip()方法
&emsp;&emsp;flip方法将Buffer从写模式切换到读模式。调用flip()方法会将`position`设回0，并将`limit`设置成之前`position`的值。

&emsp;&emsp;换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。

### 从Buffer中读取数据
&emsp;&emsp;从Buffer中读取数据有两种方式：
* 从`Buffer`读取数据到`Channel`。
* 使用`get()`方法从`Buffer`中读取数据。

&emsp;&emsp;从Buffer读取数据到Channel的例子：
```
	//read from buffer into channel.
	int bytesWritten = inChannel.write(buf);
```
&emsp;&emsp;使用get()方法从Buffer中读取数据的例子
```
	byte aByte = buf.get();
```
&emsp;&emsp;get方法有很多版本，允许你以不同的方式从`Buffer`中读取数据。例如，从指定`position`读取，或者从`Buffer`中读取数据到字节数组。更多Buffer实现的细节参考JavaDoc。

### rewind()方法

&emsp;&emsp;`Buffer.rewind()`将`position`设回`0`，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

&emsp;&emsp;`flip()`方法和`rewind()`方法很相似，区别在于`rewind()`方法不会影响limit，可以从JDK源码中看出：
```
public final Buffer flip() {  
	limit = position;  
	position = 0;  
	mark = -1;  
	return this;  
}  

public final Buffer rewind() {  
    position = 0;  
    mark = -1;  
    return this;  
}
```



### clear()与compact()方法
&emsp;&emsp;一旦读完`Buffer`中的数据，需要让`Buffer`准备好再次被写入。可以通过`clear()`或`compact()`方法来完成。

&emsp;&emsp;如果调用的是`clear()`方法，`position`将被设回`0`，`limi`t被设置成 `capacity`的值。换句话说，`Buffer`被清空了。`Buffer`中的数据并未清除，只是这些标记告诉我们可以从哪里开始往`Buffer`里写数据。

&emsp;&emsp;如果`Buffer`中有一些未读的数据，调用`clear()`方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

&emsp;&emsp;如果`Buffer`中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用`compact()`方法。

&emsp;&emsp;`compact()`方法将所有未读的数据拷贝到Buffer起始处。然后将`position`设到最后一个未读元素正后面。`limit`属性依然像`clear()`方法一样，设置成`capacity`。现在`Buffer`准备好写数据了，但是不会覆盖未读的数据。
```
    // 创建一个容量为10的byte数据缓冲区  
    ByteBuffer buff = ByteBuffer.allocate(10);  
    // 填充缓冲区  
    buff.put((byte)'A');  
    buff.put((byte)'B');  
    buff.put((byte)'C');  
    buff.put((byte)'D');  
    System.out.println("first put : " + new String(buff.array()));  
    //翻转  
    buff.flip();  
    //释放  
    System.out.println((char)buff.get());  
    System.out.println((char)buff.get());  
    //压缩  
    buff.compact();  
    System.out.println("compact after get : " + new String(buff.array()));  
    //继续填充  
    buff.put((byte)'E');  
    buff.put((byte)'F');  
    //输出所有  
    System.out.println("put after compact : " + new String(buff.array()));  
```
&emsp;&emsp;输出结果： 
```
first put : ABCD

A
B

compact after get : CDCD

put after compact : CDEF
```

### mark()与reset()方法
&emsp;&emsp;通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如：
```
	buffer.mark();
	//call buffer.get() a couple of times, e.g. during parsing.
	buffer.reset();  //set position back to mark.
```
### equals()与compareTo()方法

&emsp;&emsp;可以使用`equals()`和`compareTo()`方法两个`Buffer`。

#### equals()

&emsp;&emsp;当满足下列条件时，表示两个Buffer相等：
* 有相同的类型（byte、char、int等）。
* Buffer中剩余的byte、char等的个数相等。
* Buffer中所有剩余的byte、char等都相同。

&emsp;&emsp;如你所见，`equals`只是比较`Buffer`的一部分，不是每一个在它里面的元素都比较。实际上，它只比较`Buffer`中的剩余元素。

#### compareTo()方法

&emsp;&emsp;`compareTo()`方法比较两个`Buffer`的剩余元素(byte、char等)， 如果满足下列条件，则认为一个`Buffer`“小于”另一个`Buffer`：
* 第一个不相等的元素小于另一个Buffer中对应的元素 。
* 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。


原文整理：http://tutorials.jenkov.com/java-nio/buffers.html


  [1]: http://otqn63yyl.bkt.clouddn.com/github/images/170829/buffers-modes.png
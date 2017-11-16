# Java IO

## 1、新IO示例

### 1.1 复制文件程序的新的实现

下面也是复制文件的程序，这里使用了`FileChannel`来解决这个问题：

    public static void main(String ...args) {
        File fileToCopy = new File("." + File.separator + "who will succeed.mp4");
        File fileCopied = new File("." + File.separator + "who will succeed(copy).mp4");
        try {
            boolean succeed = fileCopied.exists() ? (fileCopied.delete() && fileCopied.createNewFile()) : fileCopied.createNewFile();
            copyWithFileChannel(fileToCopy, fileCopied);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void copyWithFileChannel(File fileToCopy, File fileCopied) throws IOException {
        long st = System.currentTimeMillis();
        FileChannel fcr = new FileInputStream(fileToCopy).getChannel();
        FileChannel fcw = new FileOutputStream(fileCopied).getChannel();
        ByteBuffer bfr = ByteBuffer.allocate(1024 * 15);
        while (fcr.read(bfr) != -1) {
            bfr.flip();
            fcw.write(bfr);
            bfr.clear();
        }
        System.out.println(System.currentTimeMillis() - st);
    }

输出结果:

    97

从输出的结果中可以看出，相比于我们之前的使用缓冲的输入和输出方式，这里使用了FileChannel之后效率又有了大幅度的提升。

在这里，注意一下ByteBuffer.clip()和ByteBuffer.clear()方法的调用。

### 1.2 通道连接：复制文件的方法的又个一实现

此外，还可以通过通道连接，将一个通道与另一个通道相连，来达到复制文件的目的。代码：

    public static void main(String ...args) {
        File fileToCopy = new File("." + File.separator + "who will succeed.mp4");
        File fileCopied = new File("." + File.separator + "who will succeed(copy).mp4");
        try {
            boolean succeed = fileCopied.exists() ? (fileCopied.delete() && fileCopied.createNewFile()) : fileCopied.createNewFile();
            copyByConnection(fileToCopy, fileCopied);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void copyByConnection(File fileToCopy, File fileCopied) throws IOException {
        long st = System.currentTimeMillis();
        FileChannel fcr = new FileInputStream(fileToCopy).getChannel();
        FileChannel fcw = new FileOutputStream(fileCopied).getChannel();
        fcr.transferTo(0, fcr.size(), fcw);
        System.out.println(System.currentTimeMillis() - st);
    }

输出结果：

    16

平均时间比上面的方式又提升了许多。

以上就是使用新IO之后复制文件操作的效率的提升，而新IO的应用范围当然不局限于简单地复制文件。它在许多领域都有重要的应用，而且它对读写性能的提升，可能直接影响程序的整体性能。因此，我们有必要了解和学习新IO相关的知识。

## 2、Buffer和Channel

新IO速度的提升来自于所使用的结构更接近于操作系统执行IO的方式：**通道和缓冲器**。我们可以将其想象成一个煤矿，**通道**是一个包含煤层的矿藏，而**缓冲器**则是派送到矿藏的卡车。也就是我们没有直接和通道交互，而是与缓冲器交互，并把缓冲器派送到通道。通道要么从缓冲器中获取数据，要么向缓冲器发送数据。

1. 这里的缓冲器是ByteBuffer，它是**Buffer**的子类，Buffer还有几个其他的实现。

2. 旧的IO库中有三个类被修改，分别是：**FileInputStream, FileOutputStream和RandomAccessFile**，它们内部均添加了一个获取Channel的方法**getChannel()**。也就是说，Channel就是与文件之间的一个“通道”。有了Channel之后，我们将Buffer与之关联来实现我们想要的逻辑。

3. 对于字节流，因为Reader和Writer是字符模式类，不能产生通道，所以为其提供了**Channels**类，用于在通道中产生Reader和Writer. 








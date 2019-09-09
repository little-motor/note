[toc]
# 1. 引言
Java的I/O建立于流(stream)之上，输入流读取字节，输出流写入字节。过滤器(filter)流可以串联到输入/出流上，可以用于加密、压缩或只是提供额外到方法。阅读器（reader）和书写器（writer）可以串联到输入/出流上，允许程序读/写文本（即字符）而不是字节。
流是同步的，即某一时刻读/写字节会阻塞其他操作，Java还支持使用通道和缓冲区的非阻塞I/O。
# 2. 输出流
```
public abstract class OutputStream implements Closeable, Flushable {
    /**
     * Writes the specified byte to this output stream. The general
     * contract for <code>write</code> is that one byte is written
     * to the output stream. The byte to be written is the eight
     * low-order bits of the argument <code>b</code>. The 24
     * high-order bits of <code>b</code> are ignored.
     * <p>
     * Subclasses of <code>OutputStream</code> must provide an
     * implementation for this method.
     */
    public abstract void write(int b) throws IOException;

    /**
     * Writes <code>b.length</code> bytes from the specified byte array
     * to this output stream. The general contract for <code>write(b)</code>
     * is that it should have exactly the same effect as the call
     * <code>write(b, 0, b.length)</code>.
     */
    public void write(byte b[]) throws IOException {
        write(b, 0, b.length);
    }

    /**
     * Writes <code>len</code> bytes from the specified byte array
     * starting at offset <code>off</code> to this output stream.
     * The general contract for <code>write(b, off, len)</code> is that
     * some of the bytes in the array <code>b</code> are written to the
     * output stream in order; element <code>b[off]</code> is the first
     * byte written and <code>b[off+len-1]</code> is the last byte written
     * by this operation.
     * <p>
     * The <code>write</code> method of <code>OutputStream</code> calls
     * the write method of one argument on each of the bytes to be
     * written out. Subclasses are encouraged to override this method and
     * provide a more efficient implementation.
     * <p>
     * If <code>b</code> is <code>null</code>, a
     * <code>NullPointerException</code> is thrown.
     * <p>
     * If <code>off</code> is negative, or <code>len</code> is negative, or
     * <code>off+len</code> is greater than the length of the array
     * <code>b</code>, then an <tt>IndexOutOfBoundsException</tt> is thrown.
     */
    public void write(byte b[], int off, int len) throws IOException {
        if (b == null) {
            throw new NullPointerException();
        } else if ((off < 0) || (off > b.length) || (len < 0) ||
                   ((off + len) > b.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }
        for (int i = 0 ; i < len ; i++) {
            write(b[off + i]);
        }
    }

    /**
     * Flushes this output stream and forces any buffered output bytes
     * to be written out. The general contract of <code>flush</code> is
     * that calling it is an indication that, if any bytes previously
     * written have been buffered by the implementation of the output
     * stream, such bytes should immediately be written to their
     * intended destination.
     * <p>
     * If the intended destination of this stream is an abstraction provided by
     * the underlying operating system, for example a file, then flushing the
     * stream guarantees only that bytes previously written to the stream are
     * passed to the operating system for writing; it does not guarantee that
     * they are actually written to a physical device such as a disk drive.
     * <p>
     * The <code>flush</code> method of <code>OutputStream</code> does nothing.
     */
    public void flush() throws IOException {
    }

    /**
     * Closes this output stream and releases any system resources
     * associated with this stream. The general contract of <code>close</code>
     * is that it closes the output stream. A closed stream cannot perform
     * output operations and cannot be reopened.
     * <p>
     * The <code>close</code> method of <code>OutputStream</code> does nothing.
     */
    public void close() throws IOException {
    }

}
```
OutputStream的子类使用这些方法向某种特定的介质写入数据，例如，FileOutputStream使用这些方法将数据写入文件。TelnetOutputStream使用这些方法将数据写入网络连接。ByteArrayOutputStream使用这些方法将数据写入可扩展的字节数组。
write（int b）接受0-255之间的整数作为参数，将对应的字节写入到输出流当中，虽然是接受的一个int值但其实他代表了一个无符号字节，但是超过255之后，之后接受4字节int值但最低位字节，其他三个字节都会被忽略。
```
public class OutPutStreamTest {
  public static void main(String[] args) throws IOException {
    FileOutputStream fileOutputStream = new FileOutputStream("filePath");
    //a
    fileOutputStream.write(97);
  }
}
```
由于每次写入一个字节效率不是很高，所以OutPutStream还提供了write(byte[] data)和wirte(bute[] data, int offset, int length)。
另外flush()方法用于立刻刷新缓冲区，close()方法用于关闭资源。在Java7中，引入了try with resources构造方法
```
//try块参数表中声明的所有AutoCloseable对象会被自动调用close()方法
try(OutputStream out = new FileOutputStream("filePath")){
    //处理输出流
} catche (IOException ex){
    //处理错误
}
```
# 3. 输入流
Java的基本输入类是java.io.InputStream这个类提供将数据读取为原始字节所需的基本方法。
```
public abstract class InputStream implements Closeable {

    // MAX_SKIP_BUFFER_SIZE is used to determine the maximum buffer size to
    // use when skipping.
    private static final int MAX_SKIP_BUFFER_SIZE = 2048;

    /**
     * Reads the next byte of data from the input stream. The value byte is
     * returned as an <code>int</code> in the range <code>0</code> to
     * <code>255</code>. If no byte is available because the end of the stream
     * has been reached, the value <code>-1</code> is returned. This method
     * blocks until input data is available, the end of the stream is detected,
     * or an exception is thrown.
     *
     * <p> A subclass must provide an implementation of this method.
     */
    public abstract int read() throws IOException;

    /**
     * Reads some number of bytes from the input stream and stores them into
     * the buffer array <code>b</code>. The number of bytes actually read is
     * returned as an integer.  This method blocks until input data is
     * available, end of file is detected, or an exception is thrown.
     *
     * <p> If the length of <code>b</code> is zero, then no bytes are read and
     * <code>0</code> is returned; otherwise, there is an attempt to read at
     * least one byte. If no byte is available because the stream is at the
     * end of the file, the value <code>-1</code> is returned; otherwise, at
     * least one byte is read and stored into <code>b</code>.
     *
     * <p> The first byte read is stored into element <code>b[0]</code>, the
     * next one into <code>b[1]</code>, and so on. The number of bytes read is,
     * at most, equal to the length of <code>b</code>. Let <i>k</i> be the
     * number of bytes actually read; these bytes will be stored in elements
     * <code>b[0]</code> through <code>b[</code><i>k</i><code>-1]</code>,
     * leaving elements <code>b[</code><i>k</i><code>]</code> through
     * <code>b[b.length-1]</code> unaffected.
     *
     * <p> The <code>read(b)</code> method for class <code>InputStream</code>
     * has the same effect as: <pre><code> read(b, 0, b.length) </code></pre>
     *
     */
    public int read(byte b[]) throws IOException {
        return read(b, 0, b.length);
    }

    /**
     * Reads up to <code>len</code> bytes of data from the input stream into
     * an array of bytes.  An attempt is made to read as many as
     * <code>len</code> bytes, but a smaller number may be read.
     * The number of bytes actually read is returned as an integer.
     *
     * <p> This method blocks until input data is available, end of file is
     * detected, or an exception is thrown.
     *
     * <p> If <code>len</code> is zero, then no bytes are read and
     * <code>0</code> is returned; otherwise, there is an attempt to read at
     * least one byte. If no byte is available because the stream is at end of
     * file, the value <code>-1</code> is returned; otherwise, at least one
     * byte is read and stored into <code>b</code>.
     *
     * <p> The first byte read is stored into element <code>b[off]</code>, the
     * next one into <code>b[off+1]</code>, and so on. The number of bytes read
     * is, at most, equal to <code>len</code>. Let <i>k</i> be the number of
     * bytes actually read; these bytes will be stored in elements
     * <code>b[off]</code> through <code>b[off+</code><i>k</i><code>-1]</code>,
     * leaving elements <code>b[off+</code><i>k</i><code>]</code> through
     * <code>b[off+len-1]</code> unaffected.
     *
     * <p> In every case, elements <code>b[0]</code> through
     * <code>b[off]</code> and elements <code>b[off+len]</code> through
     * <code>b[b.length-1]</code> are unaffected.
     *
     * <p> The <code>read(b,</code> <code>off,</code> <code>len)</code> method
     * for class <code>InputStream</code> simply calls the method
     * <code>read()</code> repeatedly. If the first such call results in an
     * <code>IOException</code>, that exception is returned from the call to
     * the <code>read(b,</code> <code>off,</code> <code>len)</code> method.  If
     * any subsequent call to <code>read()</code> results in a
     * <code>IOException</code>, the exception is caught and treated as if it
     * were end of file; the bytes read up to that point are stored into
     * <code>b</code> and the number of bytes read before the exception
     * occurred is returned. The default implementation of this method blocks
     * until the requested amount of input data <code>len</code> has been read,
     * end of file is detected, or an exception is thrown. Subclasses are encouraged
     * to provide a more efficient implementation of this method.
     */
    public int read(byte b[], int off, int len) throws IOException {
        if (b == null) {
            throw new NullPointerException();
        } else if (off < 0 || len < 0 || len > b.length - off) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        }

        int c = read();
        if (c == -1) {
            return -1;
        }
        b[off] = (byte)c;

        int i = 1;
        try {
            for (; i < len ; i++) {
                c = read();
                if (c == -1) {
                    break;
                }
                b[off + i] = (byte)c;
            }
        } catch (IOException ee) {
        }
        return i;
    }

    /**
     * Skips over and discards <code>n</code> bytes of data from this input
     * stream. The <code>skip</code> method may, for a variety of reasons, end
     * up skipping over some smaller number of bytes, possibly <code>0</code>.
     * This may result from any of a number of conditions; reaching end of file
     * before <code>n</code> bytes have been skipped is only one possibility.
     * The actual number of bytes skipped is returned. If {@code n} is
     * negative, the {@code skip} method for class {@code InputStream} always
     * returns 0, and no bytes are skipped. Subclasses may handle the negative
     * value differently.
     *
     * <p> The <code>skip</code> method of this class creates a
     * byte array and then repeatedly reads into it until <code>n</code> bytes
     * have been read or the end of the stream has been reached. Subclasses are
     * encouraged to provide a more efficient implementation of this method.
     * For instance, the implementation may depend on the ability to seek.
     */
    public long skip(long n) throws IOException {

        long remaining = n;
        int nr;

        if (n <= 0) {
            return 0;
        }

        int size = (int)Math.min(MAX_SKIP_BUFFER_SIZE, remaining);
        byte[] skipBuffer = new byte[size];
        while (remaining > 0) {
            nr = read(skipBuffer, 0, (int)Math.min(size, remaining));
            if (nr < 0) {
                break;
            }
            remaining -= nr;
        }

        return n - remaining;
    }

    /**
     * Returns an estimate of the number of bytes that can be read (or
     * skipped over) from this input stream without blocking by the next
     * invocation of a method for this input stream. The next invocation
     * might be the same thread or another thread.  A single read or skip of this
     * many bytes will not block, but may read or skip fewer bytes.
     *
     * <p> Note that while some implementations of {@code InputStream} will return
     * the total number of bytes in the stream, many will not.  It is
     * never correct to use the return value of this method to allocate
     * a buffer intended to hold all data in this stream.
     *
     * <p> A subclass' implementation of this method may choose to throw an
     * {@link IOException} if this input stream has been closed by
     * invoking the {@link #close()} method.
     *
     * <p> The {@code available} method for class {@code InputStream} always
     * returns {@code 0}.
     *
     * <p> This method should be overridden by subclasses.
     */
    public int available() throws IOException {
        return 0;
    }

    /**
     * Closes this input stream and releases any system resources associated
     * with the stream.
     *
     * <p> The <code>close</code> method of <code>InputStream</code> does
     * nothing.
     */
    public void close() throws IOException {}

    /**
     * Marks the current position in this input stream. A subsequent call to
     * the <code>reset</code> method repositions this stream at the last marked
     * position so that subsequent reads re-read the same bytes.
     *
     * <p> The <code>readlimit</code> arguments tells this input stream to
     * allow that many bytes to be read before the mark position gets
     * invalidated.
     *
     * <p> The general contract of <code>mark</code> is that, if the method
     * <code>markSupported</code> returns <code>true</code>, the stream somehow
     * remembers all the bytes read after the call to <code>mark</code> and
     * stands ready to supply those same bytes again if and whenever the method
     * <code>reset</code> is called.  However, the stream is not required to
     * remember any data at all if more than <code>readlimit</code> bytes are
     * read from the stream before <code>reset</code> is called.
     *
     * <p> Marking a closed stream should not have any effect on the stream.
     *
     * <p> The <code>mark</code> method of <code>InputStream</code> does
     * nothing.
     */
    public synchronized void mark(int readlimit) {}

    /**
     * Repositions this stream to the position at the time the
     * <code>mark</code> method was last called on this input stream.
     *
     * <p> The general contract of <code>reset</code> is:
     *
     * <ul>
     * <li> If the method <code>markSupported</code> returns
     * <code>true</code>, then:
     *
     *     <ul><li> If the method <code>mark</code> has not been called since
     *     the stream was created, or the number of bytes read from the stream
     *     since <code>mark</code> was last called is larger than the argument
     *     to <code>mark</code> at that last call, then an
     *     <code>IOException</code> might be thrown.
     *
     *     <li> If such an <code>IOException</code> is not thrown, then the
     *     stream is reset to a state such that all the bytes read since the
     *     most recent call to <code>mark</code> (or since the start of the
     *     file, if <code>mark</code> has not been called) will be resupplied
     *     to subsequent callers of the <code>read</code> method, followed by
     *     any bytes that otherwise would have been the next input data as of
     *     the time of the call to <code>reset</code>. </ul>
     *
     * <li> If the method <code>markSupported</code> returns
     * <code>false</code>, then:
     *
     *     <ul><li> The call to <code>reset</code> may throw an
     *     <code>IOException</code>.
     *
     *     <li> If an <code>IOException</code> is not thrown, then the stream
     *     is reset to a fixed state that depends on the particular type of the
     *     input stream and how it was created. The bytes that will be supplied
     *     to subsequent callers of the <code>read</code> method depend on the
     *     particular type of the input stream. </ul></ul>
     *
     * <p>The method <code>reset</code> for class <code>InputStream</code>
     * does nothing except throw an <code>IOException</code>.
     */
    public synchronized void reset() throws IOException {
        throw new IOException("mark/reset not supported");
    }

    /**
     * Tests if this input stream supports the <code>mark</code> and
     * <code>reset</code> methods. Whether or not <code>mark</code> and
     * <code>reset</code> are supported is an invariant property of a
     * particular input stream instance. The <code>markSupported</code> method
     * of <code>InputStream</code> returns <code>false</code>.
     */
    public boolean markSupported() {
        return false;
    }

}
```
他的子类使用这些方法从特定的某种介质中读取数据，例如FileInputStream从文件中读取数据。TelnetInputStream从网络连接中读取数据。ByteArrayInputStream从字节数组中读取数据。没有参数的read()方法，从输入流中读取1字节数据作为0-255的int值返回，流的结束通过返回-1来表示。read()方法会等待并阻塞其后任何代码的执行，需要注意的是需要判断读取为-1的时候标识该结束退出了，因为输入和输出可能很慢，建议将I/O放在单独的线程当中。

read(byte[] input)和read(byte[] input, int offset, int length)用于更高效的读取数据，他们将数据读入指定的字节数组中，当读取的数据结束时，返回的int值为-1。
```
int bytesRead = 0;
int bytesToRead = 1024;
byte[] input = new byte[bytesToRead];
while(bytesRead < bytesToRead){
    int result = in.read(input, bytesRead, bytesToRead - bytesRead);
    fi(result == -1)break;
    bytesRead += result; 
}
```
# 4. 过滤器流
InputSream和OutputStream是非常原始的类，只是提供了非常基础的字节读取，在之后还需要根据字节的含义使用相应协议解码为文本或别的内容，有很多常用的协议，比如多字节UTF-8，ASCII等等，Java提供了很多常用的过滤器类，可以附加到原始流中，在原始字节和各种格式之间来回转换。过滤器有两个版本：过滤器流以及阅读器和书写器。
过滤器以链的形式进行组织，链中每个环节都接收前一个过滤器或流的数据，并把数据传递给链中的下一个环节。
## 4.1 将过滤器串联在一起
过滤器通过其构造方法与流连接，例如将缓冲文件data.txt输入
```
FileInputStream fileIn = new FileInputStream("data.txt");
BufferedInputStream bufferedIn = new BufferedInputStream(fileIn);
```
底层的实例作为上层的构造参数传入，当然我们也应该尽量调用上层的同名方法而不是既调用上层又调用底层这样可能会破坏一些隐含约定，当然可以通过覆盖引用的方式完全避免
```
FileInputStream in = new FileInputStream("data.txt");
in = new BufferedInputStream(in);
```
还有另一种方式就是
```
BufferedInputStream bufferedIn = new BufferedInputStream(
    FileInputStream("data.txt")
);
```
无论串联了几个过滤器流，我们需要记住的是除了链中最后一个过滤器之外，无论如何不应该使用其他的过滤器读取数据或向其写入任何内容。
## 4.2 缓冲流
BufferedOutputStream类将写入的数据存储在缓冲区中(一个名为buf的保护字节数组数组字段)，直到缓冲区满或刷新输出流，然后他将数据一次全部写入底层输出流。一般来说，同样数量的字节，一次全部写入要比多次写入少量字节快的多，因为TCP/UDP包都有一定开销，大约为40字节，假设有1k需要发送的字节，单独发送总共需要消耗40k，而一次性发送则会快很多，所以缓冲区对网络传输带来了很大对性能提升。

BufferedInputStream也有一个缓冲区的保护字节数组，名为buf，当调用read时会首先检查从缓冲区获取请求数据，只有当缓冲区没有数据时，流才从底层读取数据。这时他会从源中读取尽可能多的数据存入缓冲区，而不管是否马上需要所有这些数据。

缓冲流并未添加新的方法，需要注意的是每次写入会把数据放在缓冲区，而不是直接放入底层的输出流，需要发送数据时需要刷新一下，这一点非常重要。
## 4.3 数据流
DataInputStream和DataOutPutStream类提供一些方法，可以用二进制格式读/写Java 的基本数据类型和字符串。
# 5. 阅读器和书写器
java.io.Reader和java.io.Writer是处理读/写字符的两个抽象超类，他们最重要的具体子类是InputStreamReader和OutputStreamWriter类，InputStreamReader类包含了一个底层输入流，可以从中读取原始字节，并根据指定的编码方式，将这些字节转化为Unicode字符。
OutputStreamWriter将Unicode字符使用指定的编码方式转换为字节，再将这些字节写入底层输出流。
## 5.1 书写器
OutputStreamWriter是Writer最重要的具体子类，他的构造函数制定了要写入的输出流和使用的编码方式
```
//指定文件位置和编码方式
OutputStreamWriter outputStreamWriter = new OutputStreamWriter(
        new FileOutputStream("test.txt", true),
        "UTF-8"
    );
    outputStreamWriter.write("hello world 中文哈喽");
    outputStreamWriter.flush();

```


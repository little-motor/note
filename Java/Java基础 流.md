[toc]
# 1. 引言
Java的I/O建立与流(stream)之上，输入流读取字节，输出流写入字节。过滤器(filter)流可以串联到输入/出流上，可以用于加密、压缩或只是提供额外到方法。阅读器（reader）和书写器（writer）可以串联到输入/出流上，允许程序读/写文本（即字符）而不是字节。
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
持续更新中
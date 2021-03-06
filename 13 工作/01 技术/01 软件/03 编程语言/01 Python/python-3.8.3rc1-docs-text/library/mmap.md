"mmap" --- 内存映射文件支持
***************************

======================================================================

内存映射文件对象的行为既像 "bytearray" 又像 *文件对象*。 你可以在大部
分接受 "bytearray" 的地方使用 mmap 对象；例如，你可以使用 "re" 模块来
搜索一个内存映射文件。 你也可以通过执行 "obj[index] = 97" 来修改单个字
节，或者通过对切片赋值来修改一个子序列: "obj[i1:i2] = b'...'"。 你还可
以在文件的当前位置开始读取和写入数据，并使用 "seek()" 前往另一个位置。

内存映射文件是由 "mmap" 构造函数创建的，其在 Unix 和 Windows 上是不同
的。 无论哪种情况，你都必须为一个打开的文件提供文件描述符以进行更新。
如果你希望映射一个已有的 Python 文件对象，请使用该对象的 "fileno()" 方
法来获取 *fileno* 参数的正确值。 否则，你可以使用 "os.open()" 函数来打
开这个文件，这会直接返回一个文件描述符（结束时仍然需要关闭该文件）。

注解:

  如果要为可写的缓冲文件创建内存映射，则应当首先 "flush()" 该文件。 这
  确保了对缓冲区的本地修改在内存映射中可用。

对于 Unix 和 Windows 版本的构造函数，可以将 *access* 指定为可选的关键
字参数。 *access* 接受以下四个值之一： "ACCESS_READ" ， "ACCESS_WRITE"
或 "ACCESS_COPY" 分别指定只读，直写或写时复制内存，或 "ACCESS_DEFAULT"
推迟到 *prot* 。 *access* 可以在 Unix 和 Windows 上使用。如果未指定
*access* ，则 Windows mmap 返回直写映射。这三种访问类型的初始内存值均
取自指定的文件。向 "ACCESS_READ" 内存映射赋值会引发 "TypeError" 异常。
向 "ACCESS_WRITE" 内存映射赋值会影响内存和底层的文件。 向
"ACCESS_COPY" 内存映射赋值会影响内存，但不会更新底层的文件。

在 3.7 版更改: 添加了 "ACCESS_DEFAULT" 常量。

要映射匿名内存，应将 -1 作为 fileno 和 length 一起传递。

class mmap.mmap(fileno, length, tagname=None, access=ACCESS_DEFAULT[, offset])

   **（ Windows 版本）** 映射被文件句柄 *fileno* 指定的文件的 *length*
   个字节，并创建一个 mmap 对象。如果 *length* 大于当前文件大小，则文
   件将扩展为包含 *length* 个字节。如果 *length* 为 "0"，则映射的最大
   长度为当前文件大小。如果文件为空， Windows 会引发异常（你无法在
   Windows上创建空映射）。

   如果 *tagname* 被指定且不是 "None" ，则是为映射提供标签名称的字符串
   。 Windows 允许你对同一文件拥有许多不同的映射。如果指定现有标签的名
   称，则会打开该标签，否则将创建该名称的新标签。如果省略此参数或设置
   为 "None" ，则创建的映射不带名称。避免使用 tag 参数将有助于使代码在
   Unix和Windows之间可移植。

   *offset* 可以被指定为非负整数偏移量。 mmap 引用将相对于从文件开头的
   偏移。 *offset* 默认为0。 *offeset* 必须是 "ALLOCATIONGRANULARITY"
   的倍数。

   引发一个 审计事件 "mmap.__new__" 附带参数 "fileno", "length",
   "access", "offset"。

class mmap.mmap(fileno, length, flags=MAP_SHARED, prot=PROT_WRITE|PROT_READ, access=ACCESS_DEFAULT[, offset])

   **(Unix 版本)** 映射文件描述符 *fileno* 指定的文件的 *length* 个字
   节，并返回一个 mmap 对象。如果 *length* 为 "0" ，则当调用 "mmap" 时
   ，映射的最大长度将为文件的当前大小。

   *flags* specifies the nature of the mapping. "MAP_PRIVATE" creates
   a private copy-on-write mapping, so changes to the contents of the
   mmap object will be private to this process, and "MAP_SHARED"
   creates a mapping that's shared with all other processes mapping
   the same areas of the file.  The default value is "MAP_SHARED".

   *prot*, if specified, gives the desired memory protection; the two
   most useful values are "PROT_READ" and "PROT_WRITE", to specify
   that the pages may be read or written.  *prot* defaults to
   "PROT_READ | PROT_WRITE".

   *access* may be specified in lieu of *flags* and *prot* as an
   optional keyword parameter.  It is an error to specify both
   *flags*, *prot* and *access*.  See the description of *access*
   above for information on how to use this parameter.

   *offset* may be specified as a non-negative integer offset. mmap
   references will be relative to the offset from the beginning of the
   file. *offset* defaults to 0. *offset* must be a multiple of
   "ALLOCATIONGRANULARITY" which is equal to "PAGESIZE" on Unix
   systems.

   To ensure validity of the created memory mapping the file specified
   by the descriptor *fileno* is internally automatically synchronized
   with physical backing store on Mac OS X and OpenVMS.

   This example shows a simple way of using "mmap":

      import mmap

      # write a simple example file
      with open("hello.txt", "wb") as f:
          f.write(b"Hello Python!\n")

      with open("hello.txt", "r+b") as f:
          # memory-map the file, size 0 means whole file
          mm = mmap.mmap(f.fileno(), 0)
          # read content via standard file methods
          print(mm.readline())  # prints b"Hello Python!\n"
          # read content via slice notation
          print(mm[:5])  # prints b"Hello"
          # update content using slice notation;
          # note that new content must have same size
          mm[6:] = b" world!\n"
          # ... and read again using standard file methods
          mm.seek(0)
          print(mm.readline())  # prints b"Hello  world!\n"
          # close the map
          mm.close()

   "mmap" can also be used as a context manager in a "with" statement:

      import mmap

      with mmap.mmap(-1, 13) as mm:
          mm.write(b"Hello world!")

   3.2 新版功能: Context manager support.

   The next example demonstrates how to create an anonymous map and
   exchange data between the parent and child processes:

      import mmap
      import os

      mm = mmap.mmap(-1, 13)
      mm.write(b"Hello world!")

      pid = os.fork()

      if pid == 0:  # In a child process
          mm.seek(0)
          print(mm.readline())

          mm.close()

   引发一个 审计事件 "mmap.__new__" 附带参数 "fileno", "length",
   "access", "offset"。

   Memory-mapped file objects support the following methods:

   close()

      Closes the mmap. Subsequent calls to other methods of the object
      will result in a ValueError exception being raised. This will
      not close the open file.

   closed

      "True" if the file is closed.

      3.2 新版功能.

   find(sub[, start[, end]])

      Returns the lowest index in the object where the subsequence
      *sub* is found, such that *sub* is contained in the range
      [*start*, *end*]. Optional arguments *start* and *end* are
      interpreted as in slice notation. Returns "-1" on failure.

      在 3.5 版更改: 现在支持可写的 *字节类对象*。

   flush([offset[, size]])

      Flushes changes made to the in-memory copy of a file back to
      disk. Without use of this call there is no guarantee that
      changes are written back before the object is destroyed.  If
      *offset* and *size* are specified, only changes to the given
      range of bytes will be flushed to disk; otherwise, the whole
      extent of the mapping is flushed.  *offset* must be a multiple
      of the "PAGESIZE" or "ALLOCATIONGRANULARITY".

      "None" is returned to indicate success.  An exception is raised
      when the call failed.

      在 3.8 版更改: Previously, a nonzero value was returned on
      success; zero was returned on error under Windows.  A zero value
      was returned on success; an exception was raised on error under
      Unix.

   madvise(option[, start[, length]])

      Send advice *option* to the kernel about the memory region
      beginning at *start* and extending *length* bytes.  *option*
      must be one of the MADV_* constants available on the system.  If
      *start* and *length* are omitted, the entire mapping is spanned.
      On some systems (including Linux), *start* must be a multiple of
      the "PAGESIZE".

      Availability: Systems with the "madvise()" system call.

      3.8 新版功能.

   move(dest, src, count)

      Copy the *count* bytes starting at offset *src* to the
      destination index *dest*.  If the mmap was created with
      "ACCESS_READ", then calls to move will raise a "TypeError"
      exception.

   read([n])

      Return a "bytes" containing up to *n* bytes starting from the
      current file position. If the argument is omitted, "None" or
      negative, return all bytes from the current file position to the
      end of the mapping. The file position is updated to point after
      the bytes that were returned.

      在 3.3 版更改: Argument can be omitted or "None".

   read_byte()

      Returns a byte at the current file position as an integer, and
      advances the file position by 1.

   readline()

      Returns a single line, starting at the current file position and
      up to the next newline.

   resize(newsize)

      Resizes the map and the underlying file, if any. If the mmap was
      created with "ACCESS_READ" or "ACCESS_COPY", resizing the map
      will raise a "TypeError" exception.

   rfind(sub[, start[, end]])

      Returns the highest index in the object where the subsequence
      *sub* is found, such that *sub* is contained in the range
      [*start*, *end*]. Optional arguments *start* and *end* are
      interpreted as in slice notation. Returns "-1" on failure.

      在 3.5 版更改: 现在支持可写的 *字节类对象*。

   seek(pos[, whence])

      Set the file's current position.  *whence* argument is optional
      and defaults to "os.SEEK_SET" or "0" (absolute file
      positioning); other values are "os.SEEK_CUR" or "1" (seek
      relative to the current position) and "os.SEEK_END" or "2" (seek
      relative to the file's end).

   size()

      Return the length of the file, which can be larger than the size
      of the memory-mapped area.

   tell()

      Returns the current position of the file pointer.

   write(bytes)

      Write the bytes in *bytes* into memory at the current position
      of the file pointer and return the number of bytes written
      (never less than "len(bytes)", since if the write fails, a
      "ValueError" will be raised).  The file position is updated to
      point after the bytes that were written.  If the mmap was
      created with "ACCESS_READ", then writing to it will raise a
      "TypeError" exception.

      在 3.5 版更改: 现在支持可写的 *字节类对象*。

      在 3.6 版更改: The number of bytes written is now returned.

   write_byte(byte)

      Write the integer *byte* into memory at the current position of
      the file pointer; the file position is advanced by "1". If the
      mmap was created with "ACCESS_READ", then writing to it will
      raise a "TypeError" exception.


MADV_* Constants
================

mmap.MADV_NORMAL
mmap.MADV_RANDOM
mmap.MADV_SEQUENTIAL
mmap.MADV_WILLNEED
mmap.MADV_DONTNEED
mmap.MADV_REMOVE
mmap.MADV_DONTFORK
mmap.MADV_DOFORK
mmap.MADV_HWPOISON
mmap.MADV_MERGEABLE
mmap.MADV_UNMERGEABLE
mmap.MADV_SOFT_OFFLINE
mmap.MADV_HUGEPAGE
mmap.MADV_NOHUGEPAGE
mmap.MADV_DONTDUMP
mmap.MADV_DODUMP
mmap.MADV_FREE
mmap.MADV_NOSYNC
mmap.MADV_AUTOSYNC
mmap.MADV_NOCORE
mmap.MADV_CORE
mmap.MADV_PROTECT

   These options can be passed to "mmap.madvise()".  Not every option
   will be present on every system.

   Availability: Systems with the madvise() system call.

   3.8 新版功能.

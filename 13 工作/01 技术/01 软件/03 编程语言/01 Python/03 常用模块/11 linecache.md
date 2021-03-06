

# linecache


作用：

- 该模块允许从任何文件里得到任何的行，并且使用缓存进行优化，常见的情况是从单个文件读取多行。

函数：

- **linecache.getlines(filename)** 从名为 filename 的文件中得到全部内容，输出为列表格式，以文件每行为列表中的一个元素，并以 linenum-1为元素在列表中的位置存储
- **linecache.getline(filename,lineno)** 从名为 filename 的文件中得到第 lineno 行。这个函数从不会抛出一个异常–产生错误时它将返回”（换行符将包含在找到的行里）。
如果文件没有找到，这个函数将会在 sys.path搜索。
- **linecache.clearcache()** 清除缓存。如果你不再需要先前从 getline()中得到的行
- **linecache.checkcache(filename)** 检查缓存的有效性。如果在缓存中的文件在硬盘上发生了变化，并且你需要更新版本，使用这个函数。如果省略 filename，将检查缓存里的所有条目。
- **linecache.updatecache(filename)** 更新文件名为 filename 的缓存。如果 filename 文件更新了，使用这个函数可以更新 linecache.getlines(filename)返回的列表。



举例：


```py
import linecache

print(linecache.getlines('test.txt'))
print(linecache.getlines('test.txt')[0:4])
print(linecache.getline('test.txt', 4))
```

文件内容：test.txt

```txt
1a
2b
3c
4d
5e
6f
7g
```

输出：

```txt
['1a\n', '2b\n', '3c\n', '4d\n', '5e\n', '6f\n', '7g\n']
['1a\n', '2b\n', '3c\n', '4d\n']
4d

```


注意：

- 使用 linecache.getlines('a.txt')打开文件的内容之后，如果 a.txt 文件发生了改变，如果要再次用 linecache.getlines 获取的内容，不是文件的最新内容，还是之前的内容，此时有两种方法：
  - 使用 linecache.checkcache(filename)来更新文件在硬盘上的缓存，然后在执行 linecache.getlines('a.txt')就可以获取到 a.txt的最新内容；
  - 直接使用 linecache.updatecache('a.txt')，即可获取最新的 a.txt的罪行内容
- 读取文件之后，如果不需要使用文件的缓存时，需要在最后清理一下缓存，使 linecache.clearcache()清理缓存，释放缓存。
- 此模块使用内存来缓存文件内容，所以需要耗费内存，打开文件的大小和打开速度和你的内存大小有关系。


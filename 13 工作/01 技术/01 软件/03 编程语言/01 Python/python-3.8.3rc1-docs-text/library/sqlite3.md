"sqlite3" --- SQLite 数据库 DB-API 2.0 接口模块
***********************************************

**源代码：** Lib/sqlite3/

======================================================================

SQLite 是一个C语言库，它可以提供一种轻量级的基于磁盘的数据库，这种数据
库不需要独立的服务器进程，也允许需要使用一种非标准的 SQL 查询语言来访
问它。一些应用程序可以使用 SQLite 作为内部数据存储。可以用它来创建一个
应用程序原型，然后再迁移到更大的数据库，比如 PostgreSQL 或 Oracle。

sqlite3 模块由  Gerhard Häring 编写。它提供了符合 DB-API 2.0 规范的接
口，这个规范是 **PEP 249**。

要使用这个模块，必须先创建一个  "Connection" 对象，它代表数据库。下面
例子中，数据将存储在 "example.db" 文件中：

   import sqlite3
   conn = sqlite3.connect('example.db')

你也可以使用 ":memory:" 来创建一个内存中的数据库

当有了 "Connection" 对象后，你可以创建一个 "Cursor" 游标对象，然后调用
它的 "execute()" 方法来执行 SQL 语句：

   c = conn.cursor()

   # Create table
   c.execute('''CREATE TABLE stocks
                (date text, trans text, symbol text, qty real, price real)''')

   # Insert a row of data
   c.execute("INSERT INTO stocks VALUES ('2006-01-05','BUY','RHAT',100,35.14)")

   # Save (commit) the changes
   conn.commit()

   # We can also close the connection if we are done with it.
   # Just be sure any changes have been committed or they will be lost.
   conn.close()

这些数据被持久化保存了，而且可以在之后的会话中使用它们：

   import sqlite3
   conn = sqlite3.connect('example.db')
   c = conn.cursor()

通常你的 SQL 操作需要使用一些 Python 变量的值。你不应该使用 Python 的
字符串操作来创建你的查询语句，因为那样做不安全；它会使你的程序容易受到
SQL 注入攻击（在 https://xkcd.com/327/ 上有一个搞笑的例子，看看有什么
后果）

推荐另外一种方法：使用 DB-API 的参数替换。在你的 SQL 语句中，使用 "?"
占位符来代替值，然后把对应的值组成的元组做为 "execute()" 方法的第二个
参数。（其他数据库可能会使用不同的占位符，比如 "%s" 或者 ":1"）例如：

   # Never do this -- insecure!
   symbol = 'RHAT'
   c.execute("SELECT * FROM stocks WHERE symbol = '%s'" % symbol)

   # Do this instead
   t = ('RHAT',)
   c.execute('SELECT * FROM stocks WHERE symbol=?', t)
   print(c.fetchone())

   # Larger example that inserts many records at a time
   purchases = [('2006-03-28', 'BUY', 'IBM', 1000, 45.00),
                ('2006-04-05', 'BUY', 'MSFT', 1000, 72.00),
                ('2006-04-06', 'SELL', 'IBM', 500, 53.00),
               ]
   c.executemany('INSERT INTO stocks VALUES (?,?,?,?,?)', purchases)

要在执行 SELECT 语句后获取数据，你可以把游标作为 *iterator*，然后调用
它的 "fetchone()" 方法来获取一条匹配的行，也可以调用 "fetchall()" 来得
到包含多个匹配行的列表。

下面是一个使用迭代器形式的例子：

   >>> for row in c.execute('SELECT * FROM stocks ORDER BY price'):
           print(row)

   ('2006-01-05', 'BUY', 'RHAT', 100, 35.14)
   ('2006-03-28', 'BUY', 'IBM', 1000, 45.0)
   ('2006-04-06', 'SELL', 'IBM', 500, 53.0)
   ('2006-04-05', 'BUY', 'MSFT', 1000, 72.0)

参见:

  https://github.com/ghaering/pysqlite
     pysqlite的主页 -- sqlite3 在外部使用 “pysqlite” 名字进行开发。

  https://www.sqlite.org
     SQLite的主页；它的文档详细描述了它所支持的 SQL 方言的语法和可用的
     数据类型。

  https://www.w3schools.com/sql/
     学习 SQL 语法的教程、参考和例子。

  **PEP 249** - DB-API 2.0 规范
     Marc-André Lemburg 写的 PEP。


模块函数和常量
==============

sqlite3.version

   这个模块的版本号，是一个字符串。不是 SQLite 库的版本号。

sqlite3.version_info

   这个模块的版本号，是一个由整数组成的元组。不是 SQLite 库的版本号。

sqlite3.sqlite_version

   使用中的 SQLite 库的版本号，是一个字符串。

sqlite3.sqlite_version_info

   使用中的 SQLite 库的版本号，是一个整数组成的元组。

sqlite3.PARSE_DECLTYPES

   这个常量可以作为 "connect()" 函数的 *detect_types* 参数。

   设置这个参数后，"sqlite3" 模块将解析它返回的每一列申明的类型。它会
   申明的类型的第一个单词，比如“integer primary key”，它会解析出
   “integer”，再比如“number(10)”，它会解析出“number”。然后，它会在转换
   器字典里查找那个类型注册的转换器函数，并调用它。

sqlite3.PARSE_COLNAMES

   这个常量可以作为 "connect()" 函数的 *detect_types* 参数。

   设置此参数可使得 SQLite 接口解析它所返回的每一列的列名。 它将在其中
   查找形式为 [mytype] 的字符串，然后将 'mytype' 确定为列的类型。 它将
   尝试在转换器字典中查找 'mytype' 条目，然后用找到的转换器函数来返回
   值。 在 "Cursor.description" 中找到的列名并不包括类型，举例来说，如
   果你在你的 SQL 中使用了像 "'as "Expiration date [datetime]"'" 这样
   的写法，那么我们将解析出在第一个 then we will parse out everything
   until the first "'['" 之前的所有内容并去除前导空格作为列名：即列名
   将为 "Expiration date"。

sqlite3.connect(database[, timeout, detect_types, isolation_level, check_same_thread, factory, cached_statements, uri])

   连接 SQLite 数据库 *database*。默认返回 "Connection" 对象，除非使用
   了自定义的 *factory* 参数。

   *database* 是准备打开的数据库文件的路径（绝对路径或相对于当前目录的
   相对路径），它是 *path-like object*。你也可以用 "":memory:"" 在内存
   中打开一个数据库。

   当一个数据库被多个连接访问的时候，如果其中一个进程修改这个数据库，
   在这个事务提交之前，这个 SQLite 数据库将会被一直锁定。*timeout* 参
   数指定了这个连接等待锁释放的超时时间，超时之后会引发一个异常。这个
   超时时间默认是 5.0（5秒）。

   *isolation_level* 参数，请查看 "Connection" 对象的
   "isolation_level" 属性。

   SQLite 原生只支持5种类型：TEXT，INTEGER，REAL，BLOB 和 NULL。如果你
   想用其它类型，你必须自己添加相应的支持。使用 *detect_types* 参数和
   模块级别的 "register_converter()" 函数注册**转换器** 可以简单的实现
   。

   *detect_types* 默认为0（即关闭，没有类型检测）。你也可以组合
   "PARSE_DECLTYPES" 和 "PARSE_COLNAMES" 来开启类型检测。

   默认情况下，*check_same_thread* 为 "True"，只有当前的线程可以使用该
   连接。 如果设置为 "False"，则多个线程可以共享返回的连接。 当多个线
   程使用同一个连接的时候，用户应该把写操作进行序列化，以避免数据损坏
   。

   默认情况下，当调用 connect 方法的时候，"sqlite3" 模块使用了它的
   "Connection" 类。当然，你也可以创建 "Connection" 类的子类，然后创建
   提供了 *factory* 参数的 "connect()" 方法。

   详情请查阅当前手册的 SQLite 与 Python 类型 部分。

   "sqlite3" 模块在内部使用语句缓存来避免 SQL 解析开销。 如果要显式设
   置当前连接可以缓存的语句数，可以设置 *cached_statements* 参数。 当
   前实现的默认值是缓存100条语句。

   如果 *uri* 为真，则 *database* 被解释为 URI。 它允许您指定选项。 例
   如，以只读模式打开数据库：

      db = sqlite3.connect('file:path/to/database?mode=ro', uri=True)

   有关此功能的更多信息，包括已知选项的列表，可以在 ` SQLite URI 文档
   <https://www.sqlite.org/uri.html>`_ 中找到。

   引发一个 审计事件 "sqlite3.connect"，附带参数 "database"。

   在 3.4 版更改: 增加了 *uri* 参数。

   在 3.7 版更改: *database* 现在可以是一个 *path-like object* 对象了
   ，不仅仅是字符串。

sqlite3.register_converter(typename, callable)

   注册一个回调对象 *callable*, 用来转换数据库中的字节串为自定的
   Python 类型。所有类型为 *typename* 的数据库的值在转换时，都会调用这
   个回调对象。通过指定 "connect()" 函数的 *detect-types* 参数来设置类
   型检测的方式。注意，*typename* 与查询语句中的类型名进行匹配时不区分
   大小写。

sqlite3.register_adapter(type, callable)

   注册一个回调对象 *callable*，用来转换自定义Python类型为一个 SQLite
   支持的类型。 这个回调对象 *callable* 仅接受一个 Python 值作为参数，
   而且必须返回以下某个类型的值：int，float，str 或 bytes。

sqlite3.complete_statement(sql)

   如果字符串 *sql* 包含一个或多个完整的 SQL 语句（以分号结束）则返回
   "True"。它不会验证 SQL 语法是否正确，仅会验证字符串字面上是否完整，
   以及是否以分号结束。

   它可以用来构建一个 SQLite shell，下面是一个例子：

      # A minimal SQLite shell for experiments

      import sqlite3

      con = sqlite3.connect(":memory:")
      con.isolation_level = None
      cur = con.cursor()

      buffer = ""

      print("Enter your SQL commands to execute in sqlite3.")
      print("Enter a blank line to exit.")

      while True:
          line = input()
          if line == "":
              break
          buffer += line
          if sqlite3.complete_statement(buffer):
              try:
                  buffer = buffer.strip()
                  cur.execute(buffer)

                  if buffer.lstrip().upper().startswith("SELECT"):
                      print(cur.fetchall())
              except sqlite3.Error as e:
                  print("An error occurred:", e.args[0])
              buffer = ""

      con.close()

sqlite3.enable_callback_tracebacks(flag)

   默认情况下，您不会获得任何用户定义函数中的回溯消息，比如聚合，转换
   器，授权器回调等。如果要调试它们，可以设置 *flag* 参数为 "True" 并
   调用此函数。 之后，回调中的回溯信息将会输出到 "sys.stderr"。 再次使
   用 "False" 来禁用该功能。


连接对象（Connection）
======================

class sqlite3.Connection

   SQLite 数据库连接对象有如下的属性和方法：

   isolation_level

      获取或设置当前默认的隔离级别。 表示自动提交模式的 "None" 以及
      "DEFERRED", "IMMEDIATE" 或 "EXCLUSIVE" 其中之一。 详细描述请参阅
      控制事务。

   in_transaction

      如果是在活动事务中（还没有提交改变），返回 "True"，否则，返回
      "False"。它是一个只读属性。

      3.2 新版功能.

   cursor(factory=Cursor)

      这个方法接受一个可选参数 *factory*，如果要指定这个参数，它必须是
      一个可调用对象，而且必须返回 "Cursor" 类的一个实例或者子类。

   commit()

      这个方法提交当前事务。如果没有调用这个方法，那么从上一次提交
      "commit()" 以来所有的变化在其他数据库连接上都是不可见的。如果你
      往数据库里写了数据，但是又查询不到，请检查是否忘记了调用这个方法
      。

   rollback()

      这个方法回滚从上一次调用 "commit()" 以来所有数据库的改变。

   close()

      关闭数据库连接。注意，它不会自动调用 "commit()" 方法。如果在关闭
      数据库连接之前没有调用 "commit()"，那么你的修改将会丢失！

   execute(sql[, parameters])

      这是一个非标准的快捷方法，它会调用 "cursor()" 方法来创建一个游标
      对象，并使用给定的 *parameters* 参数来调用游标对象的 "execute()"
      方法，最后返回这个游标对象。

   executemany(sql[, parameters])

      这是一个非标准的快捷方法，它会调用 "cursor()" 方法来创建一个游标
      对象，并使用给定的 *parameters* 参数来调用游标对象的
      "executemany()" 方法，最后返回这个游标对象。

   executescript(sql_script)

      这是一个非标准的快捷方法，它会调用 "cursor()" 方法来创建一个游标
      对象，并使用给定的 *sql_script* 参数来调用游标对象的
      "executescript()" 方法，最后返回这个游标对象。

   create_function(name, num_params, func, *, deterministic=False)

      创建一个可以在 SQL 语句中使用的用户自定义函数，函数名为 *name*。
      *num_params* 为该函数所接受的形参个数（如果 *num_params* 为 -1，
      则该函数可接受任意数量的参数）， *func* 是一个 Python 可调用对象
      ，它将作为 SQL 函数被调用。 如果 *deterministic* 为真值，则所创
      建的函数将被标记为 deterministic，这允许 SQLite 执行额外的优化。
      此旗标在 SQLite 3.8.3 或更高版本中受到支持，如果在旧版本中使用将
      引发 "NotSupportedError"。

      此函数可返回任何 SQLite 所支持的类型: bytes, str, int, float 和
      "None"。

      在 3.8 版更改: 增加了 *deterministic* 形参。

      示例:

         import sqlite3
         import hashlib

         def md5sum(t):
             return hashlib.md5(t).hexdigest()

         con = sqlite3.connect(":memory:")
         con.create_function("md5", 1, md5sum)
         cur = con.cursor()
         cur.execute("select md5(?)", (b"foo",))
         print(cur.fetchone()[0])

         con.close()

   create_aggregate(name, num_params, aggregate_class)

      创建一个自定义的聚合函数。

      参数中 *aggregate_class* 类必须实现两个方法："step" 和
      "finalize"。"step" 方法接受 *num_params* 个参数（如果
      *num_params* 为 -1，那么这个函数可以接受任意数量的参数）；
      "finalize" 方法返回最终的聚合结果。

      "finalize" 方法可以返回任何 SQLite 支持的类型：bytes，str，int，
      float 和 "None"。

      示例:

         import sqlite3

         class MySum:
             def __init__(self):
                 self.count = 0

             def step(self, value):
                 self.count += value

             def finalize(self):
                 return self.count

         con = sqlite3.connect(":memory:")
         con.create_aggregate("mysum", 1, MySum)
         cur = con.cursor()
         cur.execute("create table test(i)")
         cur.execute("insert into test(i) values (1)")
         cur.execute("insert into test(i) values (2)")
         cur.execute("select mysum(i) from test")
         print(cur.fetchone()[0])

         con.close()

   create_collation(name, callable)

      使用 *name* 和 *callable* 创建排序规则。这个 *callable* 接受两个
      字符串对象，如果第一个小于第二个则返回 -1， 如果两个相等则返回 0
      ，如果第一个大于第二个则返回 1。注意，这是用来控制排序的（SQL 中
      的 ORDER BY），所以它不会影响其它的 SQL 操作。

      注意，这个 *callable* 可调用对象会把它的参数作为 Python 字节串，
      通常会以 UTF-8 编码格式对它进行编码。

      以下示例显示了使用“错误方式”进行排序的自定义排序规则：

         import sqlite3

         def collate_reverse(string1, string2):
             if string1 == string2:
                 return 0
             elif string1 < string2:
                 return 1
             else:
                 return -1

         con = sqlite3.connect(":memory:")
         con.create_collation("reverse", collate_reverse)

         cur = con.cursor()
         cur.execute("create table test(x)")
         cur.executemany("insert into test(x) values (?)", [("a",), ("b",)])
         cur.execute("select x from test order by x collate reverse")
         for row in cur:
             print(row)
         con.close()

      要移除一个排序规则，需要调用 "create_collation" 并设置 callable
      参数为 "None"。

         con.create_collation("reverse", None)

   interrupt()

      可以从不同的线程调用这个方法来终止所有查询操作，这些查询操作可能
      正在连接上执行。此方法调用之后， 查询将会终止，而且查询的调用者
      会获得一个异常。

   set_authorizer(authorizer_callback)

      此方法注册一个授权回调对象。每次在访问数据库中某个表的某一列的时
      候，这个回调对象将会被调用。如果要允许访问，则返回 "SQLITE_OK"，
      如果要终止整个 SQL 语句，则返回 "SQLITE_DENY"，如果这一列需要当
      做 NULL 值处理，则返回 "SQLITE_IGNORE"。这些常量可以在
      "sqlite3" 模块中找到。

      回调的第一个参数表示要授权的操作类型。 第二个和第三个参数将是参
      数或 "None"，具体取决于第一个参数的值。 第 4 个参数是数据库的名
      称（“main”，“temp”等），如果需要的话。 第 5 个参数是负责访问尝试
      的最内层触发器或视图的名称，或者如果此访问尝试直接来自输入 SQL
      代码，则为 "None"。

      请参阅 SQLite 文档，了解第一个参数的可能值以及第二个和第三个参数
      的含义，具体取决于第一个参数。 所有必需的常量都可以在 "sqlite3"
      模块中找到。

   set_progress_handler(handler, n)

      此例程注册回调。 对SQLite虚拟机的每个多指令调用回调。 如果要在长
      时间运行的操作期间从SQLite调用（例如更新用户界面），这非常有用。

      如果要清除以前安装的任何进度处理程序，调用该方法时请将 *handler*
      参数设置为 "None"。

      从处理函数返回非零值将终止当前正在执行的查询并导致它引发
      "OperationalError" 异常。

   set_trace_callback(trace_callback)

      为每个 SQLite 后端实际执行的 SQL 语句注册要调用的
      *trace_callback*。

      传递给回调的唯一参数是正在执行的语句（作为字符串）。 回调的返回
      值将被忽略。 请注意，后端不仅运行传递给 "Cursor.execute()" 方法
      的语句。 其他来源包括 Python 模块的事务管理和当前数据库中定义的
      触发器的执行。

      将传入的 *trace_callback* 设为 "None" 将禁用跟踪回调。

      3.3 新版功能.

   enable_load_extension(enabled)

      此例程允许/禁止SQLite引擎从共享库加载SQLite扩展。 SQLite扩展可以
      定义新功能，聚合或全新的虚拟表实现。 一个众所周知的扩展是与
      SQLite一起分发的全文搜索扩展。

      默认情况下禁用可加载扩展。 见 [1].

      3.2 新版功能.

         import sqlite3

         con = sqlite3.connect(":memory:")

         # enable extension loading
         con.enable_load_extension(True)

         # Load the fulltext search extension
         con.execute("select load_extension('./fts3.so')")

         # alternatively you can load the extension using an API call:
         # con.load_extension("./fts3.so")

         # disable extension loading again
         con.enable_load_extension(False)

         # example from SQLite wiki
         con.execute("create virtual table recipe using fts3(name, ingredients)")
         con.executescript("""
             insert into recipe (name, ingredients) values ('broccoli stew', 'broccoli peppers cheese tomatoes');
             insert into recipe (name, ingredients) values ('pumpkin stew', 'pumpkin onions garlic celery');
             insert into recipe (name, ingredients) values ('broccoli pie', 'broccoli cheese onions flour');
             insert into recipe (name, ingredients) values ('pumpkin pie', 'pumpkin sugar flour butter');
             """)
         for row in con.execute("select rowid, name, ingredients from recipe where name match 'pie'"):
             print(row)

         con.close()

   load_extension(path)

      此例程从共享库加载SQLite扩展。 在使用此例程之前，必须使用
      "enable_load_extension()" 启用扩展加载。

      默认情况下禁用可加载扩展。 见 [1].

      3.2 新版功能.

   row_factory

      您可以将此属性更改为可接受游标和原始行作为元组的可调用对象，并将
      返回实际结果行。 这样，您可以实现更高级的返回结果的方法，例如返
      回一个可以按名称访问列的对象。

      示例:

         import sqlite3

         def dict_factory(cursor, row):
             d = {}
             for idx, col in enumerate(cursor.description):
                 d[col[0]] = row[idx]
             return d

         con = sqlite3.connect(":memory:")
         con.row_factory = dict_factory
         cur = con.cursor()
         cur.execute("select 1 as a")
         print(cur.fetchone()["a"])

         con.close()

      如果返回一个元组是不够的，并且你想要对列进行基于名称的访问，你应
      该考虑将 "row_factory" 设置为高度优化的 "sqlite3.Row" 类型。
      "Row" 提供基于索引和不区分大小写的基于名称的访问，几乎没有内存开
      销。 它可能比您自己的基于字典的自定义方法甚至基于 db_row 的解决
      方案更好。

   text_factory

      使用此属性可以控制为 "TEXT" 数据类型返回的对象。 默认情况下，此
      属性设置为 "str" 和 "sqlite3" 模块将返回 "TEXT" 的 Unicode 对象
      。 如果要返回字节串，可以将其设置为 "bytes"。

      您还可以将其设置为接受单个 bytestring 参数的任何其他可调用对象，
      并返回结果对象。

      请参阅以下示例代码以进行说明：

         import sqlite3

         con = sqlite3.connect(":memory:")
         cur = con.cursor()

         AUSTRIA = "\xd6sterreich"

         # by default, rows are returned as Unicode
         cur.execute("select ?", (AUSTRIA,))
         row = cur.fetchone()
         assert row[0] == AUSTRIA

         # but we can make sqlite3 always return bytestrings ...
         con.text_factory = bytes
         cur.execute("select ?", (AUSTRIA,))
         row = cur.fetchone()
         assert type(row[0]) is bytes
         # the bytestrings will be encoded in UTF-8, unless you stored garbage in the
         # database ...
         assert row[0] == AUSTRIA.encode("utf-8")

         # we can also implement a custom text_factory ...
         # here we implement one that appends "foo" to all strings
         con.text_factory = lambda x: x.decode("utf-8") + "foo"
         cur.execute("select ?", ("bar",))
         row = cur.fetchone()
         assert row[0] == "barfoo"

         con.close()

   total_changes

      返回自打开数据库连接以来已修改，插入或删除的数据库行的总数。

   iterdump()

      返回以SQL文本格式转储数据库的迭代器。 保存内存数据库以便以后恢复
      时很有用。 此函数提供与 **sqlite3** shell 中的 ".dump" 命令相同
      的功能。

      示例:

         # Convert file existing_db.db to SQL dump file dump.sql
         import sqlite3

         con = sqlite3.connect('existing_db.db')
         with open('dump.sql', 'w') as f:
             for line in con.iterdump():
                 f.write('%s\n' % line)
         con.close()

   backup(target, *, pages=0, progress=None, name="main", sleep=0.250)

      即使在 SQLite 数据库被其他客户端访问时，或者同时由同一连接访问，
      该方法也会对其进行备份。 该副本将写入强制参数 *target*，该参数必
      须是另一个 "Connection" 实例。

      默认情况下，或者当 *pages* 为 "0" 或负整数时，整个数据库将在一个
      步骤中复制；否则该方法一次循环复制 *pages* 规定数量的页面。

      如果指定了 *progress*，则它必须为 "None" 或一个将在每次迭代时附
      带三个整数参数执行的可调用对象，这三个参数分别是前一次迭代的状态
      *status*，将要拷贝的剩余页数 *remaining* 以及总页数 *total*。

      *name* 参数指定将被拷贝的数据库名称：它必须是一个字符串，其内容
      为表示主数据库的默认值 ""main""，表示临时数据库的 ""temp"" 或是
      在 "ATTACH DATABASE" 语句的 "AS" 关键字之后指定表示附加数据库的
      名称。

      *sleep* 参数指定在备份剩余页的连续尝试之间要休眠的秒数，可以指定
      为一个整数或一个浮点数值。

      示例一，将现有数据库复制到另一个数据库中：

         import sqlite3

         def progress(status, remaining, total):
             print(f'Copied {total-remaining} of {total} pages...')

         con = sqlite3.connect('existing_db.db')
         bck = sqlite3.connect('backup.db')
         with bck:
             con.backup(bck, pages=1, progress=progress)
         bck.close()
         con.close()

      示例二，将现有数据库复制到临时副本中：

         import sqlite3

         source = sqlite3.connect('existing_db.db')
         dest = sqlite3.connect(':memory:')
         source.backup(dest)

      可用性：SQLite 3.6.11 或以上版本

      3.7 新版功能.


Cursor 对象
===========

class sqlite3.Cursor

   "Cursor" 游标实例具有以下属性和方法。

   execute(sql[, parameters])

      执行SQL语句。 可以是参数化 SQL 语句（即，在 SQL 语句中使用占位符
      ）。"sqlite3" 模块支持两种占位符：问号（qmark风格）和命名占位符
      （命名风格）。

      以下是两种风格的示例：

         import sqlite3

         con = sqlite3.connect(":memory:")
         cur = con.cursor()
         cur.execute("create table people (name_last, age)")

         who = "Yeltsin"
         age = 72

         # This is the qmark style:
         cur.execute("insert into people values (?, ?)", (who, age))

         # And this is the named style:
         cur.execute("select * from people where name_last=:who and age=:age", {"who": who, "age": age})

         print(cur.fetchone())

         con.close()

      "execute()" 将只执行一条单独的 SQL 语句。 如果你尝试用它执行超过
      一条语句，将会引发 "Warning"。 如果你想要用一次调用执行多条 SQL
      语句请使用 "executescript()"。

   executemany(sql, seq_of_parameters)

      基于在序列 *seq_of_parameters* 中找到的所有形参序列或映射执行一
      条 SQL 命令。 "sqlite3" 模块还允许使用 *iterator* 代替序列来产生
      形参。

         import sqlite3

         class IterChars:
             def __init__(self):
                 self.count = ord('a')

             def __iter__(self):
                 return self

             def __next__(self):
                 if self.count > ord('z'):
                     raise StopIteration
                 self.count += 1
                 return (chr(self.count - 1),) # this is a 1-tuple

         con = sqlite3.connect(":memory:")
         cur = con.cursor()
         cur.execute("create table characters(c)")

         theIter = IterChars()
         cur.executemany("insert into characters(c) values (?)", theIter)

         cur.execute("select c from characters")
         print(cur.fetchall())

         con.close()

      这是一个使用生成器 *generator* 的简短示例：

         import sqlite3
         import string

         def char_generator():
             for c in string.ascii_lowercase:
                 yield (c,)

         con = sqlite3.connect(":memory:")
         cur = con.cursor()
         cur.execute("create table characters(c)")

         cur.executemany("insert into characters(c) values (?)", char_generator())

         cur.execute("select c from characters")
         print(cur.fetchall())

         con.close()

   executescript(sql_script)

      这是一个非标准的便捷方法，可用于一次执行多条 SQL 语句。 它会首先
      执行一条 "COMMIT" 语句，再执行以形参方式获取的 SQL 脚本。

      *sql_script* 可以是一个 "str" 类的实例。

      示例:

         import sqlite3

         con = sqlite3.connect(":memory:")
         cur = con.cursor()
         cur.executescript("""
             create table person(
                 firstname,
                 lastname,
                 age
             );

             create table book(
                 title,
                 author,
                 published
             );

             insert into book(title, author, published)
             values (
                 'Dirk Gently''s Holistic Detective Agency',
                 'Douglas Adams',
                 1987
             );
             """)
         con.close()

   fetchone()

      获取一个查询结果集的下一行，返回一个单独序列，或是在没有更多可用
      数据时返回 "None"。

   fetchmany(size=cursor.arraysize)

      获取下一个多行查询结果集，返回一个列表。 当没有更多可用行时将返
      回一个空列表。

      每次调用获取的行数由 *size* 形参指定。 如果没有给出该形参，则由
      cursor 的 arraysize 决定要获取的行数。 此方法将基于 size 形参值
      尝试获取指定数量的行。 如果获取不到指定的行数，则可能返回较少的
      行。

      请注意 *size* 形参会涉及到性能方面的考虑。为了获得优化的性能，通
      常最好是使用 arraysize 属性。 如果使用 *size* 形参，则最好在从一
      个 "fetchmany()" 调用到下一个调用之间保持相同的值。

   fetchall()

      获取一个查询结果的所有（剩余）行，返回一个列表。 请注意 cursor
      的 arraysize 属性会影响此操作的执行效率。 当没有可用行时将返回一
      个空列表。

   close()

      立即关闭 cursor（而不是在当 "__del__" 被调用的时候）。

      从这一时刻起该 cursor 将不再可用，如果再尝试用该 cursor 执行任何
      操作将引发 "ProgrammingError" 异常。

   rowcount

      虽然 "sqlite3" 模块的 "Cursor" 类实现了此属性，但数据库引擎本身
      对于确定 "受影响行"/"已选择行" 的支持并不完善。

      对于 "executemany()" 语句，修改行数会被汇总至 "rowcount"。

      根据 Python DB API 规格描述的要求，"rowcount" 属性 "当未在
      cursor 上执行 "executeXX()" 或者上一次操作的 rowcount 不是由接口
      确定时为 -1"。 这包括 "SELECT" 语句，因为我们无法确定一次查询将
      产生的行计数，而要等获取了所有行时才会知道。

      在 SQLite 的 3.6.5 版之前，如果你执行 "DELETE FROM table" 时不附
      带任何条件，则 "rowcount" 将被设为 0。

   lastrowid

      这个只读属性会提供最近修改行的 rowid。 它只在你使用 "execute()"
      方法执行 "INSERT" 或 "REPLACE" 语句时会被设置。 对于 "INSERT" 或
      "REPLACE" 以外的操作或者当 "executemany()" 被调用时，"lastrowid"
      会被设为 "None"。

      如果 "INSERT" 或 "REPLACE" 语句操作失败则将返回上一次成功操作的
      rowid。

      在 3.6 版更改: 增加了 "REPLACE" 语句的支持。

   arraysize

      用于控制 "fetchmany()" 返回行数的可读取/写入属性。 该属性的默认
      值为 1，表示每次调用将获取单独一行。

   description

      这个只读属性将提供上一次查询的列名称。 为了与 Python DB API 保持
      兼容，它会为每个列返回一个 7 元组，每个元组的最后六个条目均为
      "None"。

      对于没有任何匹配行的 "SELECT" 语句同样会设置该属性。

   connection

      这个只读属性将提供 "Cursor" 对象所使用的 SQLite 数据库
      "Connection"。 通过调用 "con.cursor()" 创建的 "Cursor" 对象所包
      含的 "connection" 属性将指向 *con*:

         >>> con = sqlite3.connect(":memory:")
         >>> cur = con.cursor()
         >>> cur.connection == con
         True


行对象
======

class sqlite3.Row

   一个 "Row" 实例，该实例将作为用于 "Connection" 对象的高度优化的
   "row_factory"。 它的大部分行为都会模仿元组的特性。

   它支持使用列名称的映射访问以及索引、迭代、文本表示、相等检测和
   "len()" 等操作。

   如果两个 "Row" 对象具有完全相同的列并且其成员均相等，则它们的比较结
   果为相等。

   keys()

      此方法会在一次查询之后立即返回一个列名称的列表，它是
      "Cursor.description" 中每个元组的第一个成员。

   在 3.5 版更改: 添加了对切片操作的支持。

让我们假设我们如上面的例子所示初始化一个表:

   conn = sqlite3.connect(":memory:")
   c = conn.cursor()
   c.execute('''create table stocks
   (date text, trans text, symbol text,
    qty real, price real)''')
   c.execute("""insert into stocks
             values ('2006-01-05','BUY','RHAT',100,35.14)""")
   conn.commit()
   c.close()

现在我们将 "Row" 插入:

   >>> conn.row_factory = sqlite3.Row
   >>> c = conn.cursor()
   >>> c.execute('select * from stocks')
   <sqlite3.Cursor object at 0x7f4e7dd8fa80>
   >>> r = c.fetchone()
   >>> type(r)
   <class 'sqlite3.Row'>
   >>> tuple(r)
   ('2006-01-05', 'BUY', 'RHAT', 100.0, 35.14)
   >>> len(r)
   5
   >>> r[2]
   'RHAT'
   >>> r.keys()
   ['date', 'trans', 'symbol', 'qty', 'price']
   >>> r['qty']
   100.0
   >>> for member in r:
   ...     print(member)
   ...
   2006-01-05
   BUY
   RHAT
   100.0
   35.14


异常
====

exception sqlite3.Warning

   "Exception" 的一个子类。

exception sqlite3.Error

   此模块中其他异常的基类。 它是 "Exception" 的一个子类。

exception sqlite3.DatabaseError

   针对数据库相关错误引发的异常。

exception sqlite3.IntegrityError

   当数据库的关系一致性受到影响时引发的异常。 例如外键检查失败等。 它
   是 "DatabaseError" 的子类。

exception sqlite3.ProgrammingError

   编程错误引发的异常，例如表未找到或已存在，SQL 语句存在语法错误，指
   定的形参数量错误等。 它是 "DatabaseError" 的子类。

exception sqlite3.OperationalError

   与数据库操作相关而不一定能受程序员掌控的错误引发的异常，例如发生非
   预期的连接中断，数据源名称未找到，事务无法被执行等。 它是
   "DatabaseError" 的子类。

exception sqlite3.NotSupportedError

   在使用了某个数据库不支持的方法或数据库 API 时引发的异常，例如在一个
   不支持事务或禁用了事务的连接上调用 "rollback()" 方法等。 它是
   "DatabaseError" 的子类。


SQLite 与 Python 类型
=====================


概述
----

SQLite 原生支持如下的类型： "NULL"，"INTEGER"，"REAL"，"TEXT"，"BLOB"
。

因此可以将以下Python类型发送到SQLite而不会出现任何问题：

+---------------------------------+---------------+
| Python 类型                     | SQLite 类型   |
|=================================|===============|
| "None"                          | "NULL"        |
+---------------------------------+---------------+
| "int"                           | "INTEGER"     |
+---------------------------------+---------------+
| "float"                         | "REAL"        |
+---------------------------------+---------------+
| "str"                           | "TEXT"        |
+---------------------------------+---------------+
| "bytes"                         | "BLOB"        |
+---------------------------------+---------------+

这是SQLite类型默认转换为Python类型的方式：

+---------------+------------------------------------------------+
| SQLite 类型   | Python 类型                                    |
|===============|================================================|
| "NULL"        | "None"                                         |
+---------------+------------------------------------------------+
| "INTEGER"     | "int"                                          |
+---------------+------------------------------------------------+
| "REAL"        | "float"                                        |
+---------------+------------------------------------------------+
| "TEXT"        | 取决于 "text_factory" , 默认为 "str"           |
+---------------+------------------------------------------------+
| "BLOB"        | "bytes"                                        |
+---------------+------------------------------------------------+

"sqlite3" 模块的类型系统可通过两种方式来扩展：你可以通过对象适配将额外
的 Python 类型保存在 SQLite 数据库中，你也可以让 "sqlite3" 模块通过转
换器将 SQLite 类型转换为不同的 Python 类型。


使用适配器将额外的 Python 类型保存在 SQLite 数据库中。
------------------------------------------------------

如上文所述，SQLite 只包含对有限类型集的原生支持。 要让 SQLite 能使用其
他 Python 类型，你必须将它们 **适配** 至 sqlite3 模块所支持的 SQLite
类型中的一种：NoneType, int, float, str, bytes。

有两种方式能让 "sqlite3" 模块将某个定制的 Python 类型适配为受支持的类
型。


让对象自行适配
~~~~~~~~~~~~~~

如果类是你自己编写的，这将是一个很好的方式。 假设你有这样一个类:

   class Point:
       def __init__(self, x, y):
           self.x, self.y = x, y

现在你想将这种点对象保存在一个 SQLite 列中。 首先你必须选择一种受支持
的类型用来表示点对象。 让我们就用 str 并使用一个分号来分隔坐标值。 然
后你需要给你的类加一个方法 "__conform__(self, protocol)"，它必须返回转
换后的值。 形参 *protocol* 将为 "PrepareProtocol"。

   import sqlite3

   class Point:
       def __init__(self, x, y):
           self.x, self.y = x, y

       def __conform__(self, protocol):
           if protocol is sqlite3.PrepareProtocol:
               return "%f;%f" % (self.x, self.y)

   con = sqlite3.connect(":memory:")
   cur = con.cursor()

   p = Point(4.0, -3.2)
   cur.execute("select ?", (p,))
   print(cur.fetchone()[0])

   con.close()


注册可调用的适配器
~~~~~~~~~~~~~~~~~~

另一种可能的做法是创建一个将该类型转换为字符串表示的函数并使用
"register_adapter()" 注册该函数。

   import sqlite3

   class Point:
       def __init__(self, x, y):
           self.x, self.y = x, y

   def adapt_point(point):
       return "%f;%f" % (point.x, point.y)

   sqlite3.register_adapter(Point, adapt_point)

   con = sqlite3.connect(":memory:")
   cur = con.cursor()

   p = Point(4.0, -3.2)
   cur.execute("select ?", (p,))
   print(cur.fetchone()[0])

   con.close()

"sqlite3" 模块有两个适配器可用于 Python 的内置 "datetime.date" 和
"datetime.datetime" 类型。 现在假设我们想要存储 "datetime.datetime" 对
象，但不是表示为 ISO 格式，而是表示为 Unix 时间戳。

   import sqlite3
   import datetime
   import time

   def adapt_datetime(ts):
       return time.mktime(ts.timetuple())

   sqlite3.register_adapter(datetime.datetime, adapt_datetime)

   con = sqlite3.connect(":memory:")
   cur = con.cursor()

   now = datetime.datetime.now()
   cur.execute("select ?", (now,))
   print(cur.fetchone()[0])

   con.close()


将SQLite 值转换为自定义Python 类型
----------------------------------

编写适配器让你可以将定制的 Python 类型发送给 SQLite。 但要令它真正有用
，我们需要实现从 Python 到 SQLite 再回到 Python 的双向转换。

输入转换器。

让我们回到 "Point" 类。 我们以字符串形式在 SQLite 中存储了 x 和 y 坐标
值。

首先，我们将定义一个转换器函数，它接受这样的字符串作为形参并根据该参数
构造一个 "Point" 对象。

注解:

  转换器函数在调用时 **总是** 会附带一个 "bytes" 对象，无论你将何种数
  据类型的值发给 SQLite。

   def convert_point(s):
       x, y = map(float, s.split(b";"))
       return Point(x, y)

现在你需要让 "sqlite3" 模块知道你从数据库中选取的其实是一个点对象。 有
两种方式都可以做到这件事：

* 隐式的声明类型

* 显式的通过列名

这两种方式会在 模块函数和常量 一节中描述，相应条目为 "PARSE_DECLTYPES"
和 "PARSE_COLNAMES" 常量。

下面的示例说明了这两种方法。

   import sqlite3

   class Point:
       def __init__(self, x, y):
           self.x, self.y = x, y

       def __repr__(self):
           return "(%f;%f)" % (self.x, self.y)

   def adapt_point(point):
       return ("%f;%f" % (point.x, point.y)).encode('ascii')

   def convert_point(s):
       x, y = list(map(float, s.split(b";")))
       return Point(x, y)

   # Register the adapter
   sqlite3.register_adapter(Point, adapt_point)

   # Register the converter
   sqlite3.register_converter("point", convert_point)

   p = Point(4.0, -3.2)

   #########################
   # 1) Using declared types
   con = sqlite3.connect(":memory:", detect_types=sqlite3.PARSE_DECLTYPES)
   cur = con.cursor()
   cur.execute("create table test(p point)")

   cur.execute("insert into test(p) values (?)", (p,))
   cur.execute("select p from test")
   print("with declared types:", cur.fetchone()[0])
   cur.close()
   con.close()

   #######################
   # 1) Using column names
   con = sqlite3.connect(":memory:", detect_types=sqlite3.PARSE_COLNAMES)
   cur = con.cursor()
   cur.execute("create table test(p)")

   cur.execute("insert into test(p) values (?)", (p,))
   cur.execute('select p as "p [point]" from test')
   print("with column names:", cur.fetchone()[0])
   cur.close()
   con.close()


默认适配器和转换器
------------------

对于 datetime 模块中的 date 和 datetime 类型已提供了默认的适配器。 它
们将会以 ISO 日期/ISO 时间戳的形式发给 SQLite。

默认转换器使用的注册名称是针对 "datetime.date" 的 "date" 和针对
"datetime.datetime" 的 "timestamp"。

通过这种方式，你可以在大多数情况下使用 Python 的 date/timestamp 对象而
无须任何额外处理。 适配器的格式还与实验性的 SQLite date/time 函数兼容
。

下面的示例演示了这一点。

   import sqlite3
   import datetime

   con = sqlite3.connect(":memory:", detect_types=sqlite3.PARSE_DECLTYPES|sqlite3.PARSE_COLNAMES)
   cur = con.cursor()
   cur.execute("create table test(d date, ts timestamp)")

   today = datetime.date.today()
   now = datetime.datetime.now()

   cur.execute("insert into test(d, ts) values (?, ?)", (today, now))
   cur.execute("select d, ts from test")
   row = cur.fetchone()
   print(today, "=>", row[0], type(row[0]))
   print(now, "=>", row[1], type(row[1]))

   cur.execute('select current_date as "d [date]", current_timestamp as "ts [timestamp]"')
   row = cur.fetchone()
   print("current_date", row[0], type(row[0]))
   print("current_timestamp", row[1], type(row[1]))

   con.close()

如果存储在 SQLite 中的时间戳的小数位多于 6 个数字，则时间戳转换器会将
该值截断至微秒精度。


控制事务
========

底层的 "sqlite3" 库默认会以 "autocommit" 模式运行，但 Python 的
"sqlite3" 模块默认则不使用此模式。

"autocommit" 模式意味着修改数据库的操作会立即生效。 "BEGIN" 或
"SAVEPOINT" 语句会禁用 "autocommit" 模式，而用于结束外层事务的
"COMMIT", "ROLLBACK" 或 "RELEASE" 则会恢复 "autocommit" 模式。

Python 的 "sqlite3" 模块默认会在数据修改语言 (DML) 类语句 (即
"INSERT"/"UPDATE"/"DELETE"/"REPLACE") 之前隐式地执行一条 "BEGIN" 语句
。

你可以控制 "sqlite3" 隐式执行的 "BEGIN" 语句的种类，具体做法是通过将
*isolation_level* 形参传给 "connect()" 调用，或者通过指定连接的
"isolation_level" 属性。 如果你没有指定 *isolation_level*，将使用基本
的 "BEGIN"，它等价于指定 "DEFERRED"。 其他可能的值为 "IMMEDIATE" 和
"EXCLUSIVE"。

你可以禁用 "sqlite3" 模块的隐式事务管理，具体做法是将
"isolation_level" 设为 "None"。 这将使得下层的 "sqlite3" 库采用
"autocommit" 模式。 随后你可以通过在代码中显式地使用 "BEGIN",
"ROLLBACK", "SAVEPOINT" 和 "RELEASE" 语句来完全控制事务状态。

在 3.6 版更改: 以前 "sqlite3" 会在 DDL 语句之前隐式地提交未完成事务。
现在则不会再这样做。


有效使用 "sqlite3"
==================


使用快捷方式
------------

使用 "Connection" 对象的非标准 "execute()", "executemany()" 和
"executescript()" 方法，可以更简洁地编写代码，因为不必显式创建（通常是
多余的） "Cursor" 对象。相反， "Cursor" 对象是隐式创建的，这些快捷方法
返回游标对象。这样，只需对 "Connection" 对象调用一次，就能直接执行
"SELECT" 语句并遍历对象。

   import sqlite3

   persons = [
       ("Hugo", "Boss"),
       ("Calvin", "Klein")
       ]

   con = sqlite3.connect(":memory:")

   # Create the table
   con.execute("create table person(firstname, lastname)")

   # Fill the table
   con.executemany("insert into person(firstname, lastname) values (?, ?)", persons)

   # Print the table contents
   for row in con.execute("select firstname, lastname from person"):
       print(row)

   print("I just deleted", con.execute("delete from person").rowcount, "rows")

   # close is not a shortcut method and it's not called automatically,
   # so the connection object should be closed manually
   con.close()


通过名称而不是索引访问索引
--------------------------

"sqlite3" 模块的一个有用功能是内置的 "sqlite3.Row" 类，该类旨在用作行
工厂。

该类的行装饰器可以用索引（如元组）和不区分大小写的名称访问：

   import sqlite3

   con = sqlite3.connect(":memory:")
   con.row_factory = sqlite3.Row

   cur = con.cursor()
   cur.execute("select 'John' as name, 42 as age")
   for row in cur:
       assert row[0] == row["name"]
       assert row["name"] == row["nAmE"]
       assert row[1] == row["age"]
       assert row[1] == row["AgE"]

   con.close()


使用连接作为上下文管理器
------------------------

连接对象可以用来作为上下文管理器，它可以自动提交或者回滚事务。如果出现
异常，事务会被回滚；否则，事务会被提交。

   import sqlite3

   con = sqlite3.connect(":memory:")
   con.execute("create table person (id integer primary key, firstname varchar unique)")

   # Successful, con.commit() is called automatically afterwards
   with con:
       con.execute("insert into person(firstname) values (?)", ("Joe",))

   # con.rollback() is called after the with block finishes with an exception, the
   # exception is still raised and must be caught
   try:
       with con:
           con.execute("insert into person(firstname) values (?)", ("Joe",))
   except sqlite3.IntegrityError:
       print("couldn't add Joe twice")

   # Connection object used as context manager only commits or rollbacks transactions,
   # so the connection object should be closed manually
   con.close()


常见问题
========


多线程
------

较老版本的 SQLite 在共享线程之间存在连接问题。这就是Python模块不允许线
程之间共享连接和游标的原因。如果仍然尝试这样做，则在运行时会出现异常。

唯一的例外是调用 "interrupt()" 方法，该方法仅在从其他线程进行调用时才
有意义。

-[ 脚注 ]-

[1] sqlite3 模块默认没有构建可加载扩展支持，因为有一些平台带有不支持这
    个特性的 SQLite 库（特别是 Mac OS X）。要获得可加载扩展的支持，那
    么在编译配置的时候必须指定 --enable-loadable-sqlite-extensions 选
    项。

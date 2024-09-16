article::

- 考虑到这是第一次搞这样（相对）系统的东西在logseq上，单独分个page。
- 学习[[Python]]和一些常用第三方库，尝试以后使用Python去编写脚本。学习过程中不断感叹python标准库的丰富……头疼的是做一件事通常会有多种方式，谁说python一种事只有一种做法的？？？
- ## 字符串格式化
  collapsed:: true
	- Python 提供了三种格式化方式——`%`，`str.format` 和 `f"{}"`。`%` 使用方式类似C的printf，形如` "My name is %s, I'm %d years old" % (name, age) `（单个参数可以不用元组），这里不表。
	- `str.format`和`f"{}"`则是更现代的方式，两者使用同样的语法去进行格式化，**并允许被格式化的对象自己决定用来格式化的微语言和结果**，两者区别在于，**`f"{}"` 直接把`{}`中的内容作为表达式，将计算后的结果去进行插值**，而`str.format`从传入参数中查找变量去插值：
	- ```python
	  print(str.format("my name is {0:>10s}, my age is {age}".format('haruka', age=17)))
	  
	  name = 'haruka'
	  age = 17
	  print(f"my name is {name:>10s}, my age is {age}")
	  ```
	- 因为`str.format`是从参数中查找的，所以既可以用参数下标索引，也可以用关键字索引，而`f"{}"`直接处理表达式，后面均使用`f"{}"`（因为它更美观且功能更强大，甚至支持嵌套的插值，只有在必须的情况下，比如需要重用模板，才使用`str.format`甚至`%`），并将之称为字符串格式化。
	- 字符串格式化的插值格式形如`{ any_expr [=][:<spec_lang>]}`，其中：
		- `str.format`在`:`前面只能是下标或参数名，不能有任何其他东西，而`f"{}"`则更花哨：
		  logseq.order-list-type:: number
		- 表达式前后能包含任意数量空白字符，其在插值时会被带过去
		  logseq.order-list-type:: number
		- 表达式后能包含一个`=`，其前后同样可以包含任意数量空白字符，当`=`存在时，整个表达式包括`=`会被**原样**打印，以方便debug，如：
		  logseq.order-list-type:: number
			- ```python
			  print(f"{  1 + 1   = }") # '  1 + 1   = 2'
			  ```
		- 在`:` 右侧是一个可选的格式化语言，其整个字符串会被传递给要格式化的对象的`__format__`方法，因此**任何类型都可以定制自己的格式化方法**。参考 ((66c6ab3d-91f7-464f-a9fb-d99c54882f8d))
		  logseq.order-list-type:: number
	- #cheetsheet Python 格式化微语言
		- ```python
		  from datetime import datetime
		  
		  some_float = 1234.5678
		  print(f"{some_float:f}")  # 默认6位小数，输出: 12345.678900
		  print(f"{some_float:.2f}")  # 输出: 1234.57
		  print(f"{some_float:10.2f}")  # 输出: '   1234.57'
		  print(f"{some_float:e}")  # 输出: 1.234568e+03
		  print(f"{some_float:.2e}")  # 保留2位小数，输出: 1.23e+03
		  
		  some_int = 12348
		  print(f"{some_int:d}")  # 输出: 12348
		  print(f"{some_int:x}")  # 小写十六进制，输出: 303c
		  print(f"{some_int:X}")  # 大写十六进制，输出: 303C
		  print(f"{some_float:%}")  # 默认6位小数，输出: 123456.780000%
		  print(f"{some_float:.1%}")  # 保留1位小数，输出: 123456.8%
		  print(f"{1234567890:,}")  # 输出: 1,234,567,890
		  
		  name = "Alice"
		  print(f"{name:<10}")  # 左对齐，输出: 'Alice     '
		  print(f"{name:>10}")  # 右对齐，输出: '     Alice'
		  print(f"{name:^10}")  # 居中对齐，输出: '  Alice   '
		  print(f"{name:_^10}")  # 居中对齐，使用_去填充，输出: '__Alice___'
		  
		  print(f"{42:0>10}")   # 右对齐，使用0而非空格去填充，0000000042
		  
		  now = datetime.now()
		  print(f"{now :%Y-%m-%d %H:%M:%S}")  # 输出: '2024-08-26 12:34:56'
		  ```
- ## 正则表达式
  collapsed:: true
	- 注意——`str.split`只能按纯文本分割，要用正则分割得使用`re.split`。
	- 直接贴上GPT给的示例，注意其中的`\b`是零宽断言，匹配单词边界：
	- ```python
	  import re
	  
	  # 示例文本
	  text = "The quick brown fox jumps over the lazy dog."
	  
	  # 1. 匹配：使用 re.search() 查找第一个匹配项
	  match = re.search(r'\b\w{5}\b', text)  # 匹配五个字母的单词
	  if match:
	      print(f"Match found: {match.group()}")  # 输出: Match found: quick
	  
	  # 多个捕获组可以直接用groups全部拿到（注意groups不返回整个匹配字符串，即不包含group(0)）
	  FILE_NAME_REGEX = re.compile(r'(.*?)_(.*?)_(.*?)_p(.*?)\.(png|jpg|jpeg)')
	  # fullmatch 检查是否整个匹配
	  print(FILE_NAME_REGEX.fullmatch('Nnnn_真_100844395_p0.jpg').groups()) # ('Nnnn', '真', '100844395', '0', 'jpg')
	      
	  # 2. 流式查找匹配结果：使用 re.finditer()
	  pattern = re.compile(r'\b\w{5}\b')  # 匹配五个字母的单词
	  for match in pattern.finditer(text):
	      print(f"Match found: {match.group()} at position {match.start()}-{match.end()}")
	      
	  # 输出:
	  # Match found: quick at position 4-9
	  # Match found: brown at position 10-15
	  # Match found: jumps at position 20-25
	      
	  # 3. 查找：使用 re.findall() 查找所有匹配项
	  matches = re.findall(r'\b\w{5}\b', text)
	  print(f"All matches: {matches}")  # 输出: All matches: ['quick', 'brown', 'jumps']
	  
	  # 4. 替换：使用 re.sub() 进行简单替换
	  # 替换所有长度为4的单词为 '****'
	  pattern = re.compile(r'\b\w{4}\b')
	  replaced_text = pattern.sub('****', text)
	  print(f"Regex replacement: {replaced_text}")
	  # 输出: Regex replacement: The quick brown fox jumps **** the **** dog.
	  
	  # 5. 替换：使用 re.sub()，替换时引用捕获组
	  # 给所有长度为4的单词前后添加 #
	  pattern = re.compile(r'\b(\w{4})\b')
	  replaced_text = pattern.sub(r'#\1#', text)
	  print(f"Regex replacement: {replaced_text}")
	  # 输出: Regex replacement: The quick brown fox jumps #over# the #lazy# dog.
	  
	  
	  # 6. 替换：使用 re.sub() ，使用函数进行替换
	  # 替换函数示例：将匹配到的单词转换为大写
	  def replace_function(match):
	      return match.group().upper()
	  
	  replaced_text = re.sub(r'\b\w{5}\b', replace_function, text)
	  print(f"Replaced text with function: {replaced_text}")
	  # 输出: Replaced text with function: The QUICK BROWN fox JUMPS over the lazy dog.
	  
	  
	  ```
- ## 文件/路径操作
  collapsed:: true
	- 考虑进行文件/路径操作时优先使用`pathlib`。
	- 文件操作主要使用如下标准库和方法：
		- `os`：提供一切POSIX提供的玩意儿，在这里包括但不限于删除，重命名，创建/读取文件夹，删除非空文件夹，修改工作目录等操作。
		  logseq.order-list-type:: number
			- `getcwd/chdir`：获取/设置工作目录
			  logseq.order-list-type:: number
			- `listdir`：列举文件夹内容
			  logseq.order-list-type:: number
			- `mkdir/makedirs`：创建文件夹，递归创建文件夹
			  logseq.order-list-type:: number
			- `remove/unlink`：删除文件
			  logseq.order-list-type:: number
			- `rmdir/removedirs`：删除空目录，递归删除空目录
			  logseq.order-list-type:: number
			- `rename`：重命名文件/文件夹
			  logseq.order-list-type:: number
			- `stat`：获取文件/文件夹状态（获取文件类型使用`os.path.isXXX`)
			  logseq.order-list-type:: number
			- `walk`：递归遍历文件夹，返回一个迭代器，迭代值为 `(当前目录，子目录，子文件)`
			  logseq.order-list-type:: number
		- `os.path`：路径操作，但也包含一些和`os`交叉的部分
		  logseq.order-list-type:: number
			- `join`：路径拼接（同nodejs的join，不会主动转换成绝对路径）
			  logseq.order-list-type:: number
			- `abspath`：相对路径到绝对路径
			  logseq.order-list-type:: number
			- `expanduser`，把路径中的`~`转换为用户目录
			  logseq.order-list-type:: number
			- `isabs`：检查是否是绝对路径
			  logseq.order-list-type:: number
			- `basename`：获取路径中文件名部分
			  logseq.order-list-type:: number
			- `dirname`：获取路径中目录部分，即上级目录
			  logseq.order-list-type:: number
			- `exists`：文件是否存在
			  logseq.order-list-type:: number
			- `isfile/isdir/islink`：路径是文件/目录/链接，文件不存在时返回 False
			  logseq.order-list-type:: number
			- `getsize`：获取文件大小（字节）
			  logseq.order-list-type:: number
			- `split`：获取`(目录, 文件名)`**（注意这不是通常意义的split！）**
			  logseq.order-list-type:: number
			- `splitext`：获取`(其他部分, 扩展名（带dot）)`
			  logseq.order-list-type:: number
		- `pathlib.Path`：似乎是Python当前更推荐的，一个面向对象的路径操作库，似乎应当用来替代`os.path`，**一个好玩的地方是`pathlib`允许用`/`去连接路径**。功能极其强大，覆盖文件IO，路径操作乃至glob，**考虑始终用它**。
		  logseq.order-list-type:: number
			- `open`：打开文件
			  logseq.order-list-type:: number
			- `glob`：本层`glob`
			  logseq.order-list-type:: number
			- `rglob`：递归`glob`
			  logseq.order-list-type:: number
			- `iterdir`：本层文件迭代器
			  logseq.order-list-type:: number
			- `is_xxx`：是 xxx
			  logseq.order-list-type:: number
			- `mkdir`：创建文件夹
			  logseq.order-list-type:: number
			- `parent`：父路径
			  logseq.order-list-type:: number
			- `parents`：祖先路径序列，最后一个路径是根路径
			  logseq.order-list-type:: number
			- `name`：整个文件名
			  logseq.order-list-type:: number
			- `stem`：除去最后一个扩展名的文件名，如`wtf.txt.gz`得到`wtf.txt`
			  logseq.order-list-type:: number
			- `suffix`：最后一个扩展名，带dot（`.tar.gz`得到`.gz`）
			  logseq.order-list-type:: number
			- `suffixes`：所有扩展名
			  logseq.order-list-type:: number
			- `with_name`：替换整个文件名后的新路径
			  logseq.order-list-type:: number
			- `with_stem`：替换stem后的新路径
			  logseq.order-list-type:: number
			- `with_suffix`：替换最后一个扩展名后的新路径
			  logseq.order-list-type:: number
		- `shutil`：提供文件批量拷贝，移动，删除操作；提供`which`操作，提供压缩/解压缩操作（ zip和tar.gz 均支持，wtf在标准库里提供这个）
		  logseq.order-list-type:: number
			- `copy/copytree`：复制文件，递归复制目录
			  logseq.order-list-type:: number
			- `move`：移动文件
			  logseq.order-list-type:: number
			- `rmtree`：递归删除目录
			  logseq.order-list-type:: number
		- `tempfile`：创建临时文件、文件夹，考虑优先使用`TemporaryXXX`。
		  logseq.order-list-type:: number
			- `NamedTemporaryFile/TemporaryFile`：临时文件对象，可使用上下文管理器，关闭后自动删除，但该行为可配置，以及可以指定存储路径
			  logseq.order-list-type:: number
				- `NamedTemporaryFile`会保证文件有可见的文件名，这玩意儿在Win上行为和 `TemporaryFile` 是完全一样的，考虑始终使用`NamedTemporaryFile`
			- `TemporaryDirectory`：临时文件夹对象，可使用上下文管理器
			  logseq.order-list-type:: number
			- `mkstemp`：创建临时文件，返回文件描述符（似乎可以无视？）和路径
			  logseq.order-list-type:: number
			- `mkdtemp`：创建临时文件夹
			  logseq.order-list-type:: number
		- `glob`：使用`glob`语法筛选、遍历目录树
		  logseq.order-list-type:: number
			- `glob`：根据glob模式查找文件，注意**默认是非递归的**！
			  logseq.order-list-type:: number
			- `iglob`：同`glob`，但返回迭代器
			  logseq.order-list-type:: number
				- `escape`：转义特殊字符，需要匹配字面量时使用，如文件中包含`[]`时
				  logseq.order-list-type:: number
		- `fnmatch`：文件名匹配库，看上去功能是glob的子集……？不表
		  logseq.order-list-type:: number
	- glob专用来匹配文件路径，它支持：
		- `*`：零个或多个任意字符，文件分隔符除外
		- `**`：零个或多个**路径**（这使得 `**/abc` 能够匹配文件 `abc`）
		- `?`：单个任意字符
		- `[]`：同正则，匹配特定字符集
	- 此外，关于URL路径操作，使用`from urllib.parse import urlparse`
- ## 文件IO
  collapsed:: true
	- **注意！文件读写默认字符集用的是系统字符集，对中国地区的Windows，是GBK！记得指定utf-8**。
	- 文件读写主要涉及如下的标准库：
		- `io`：提供流处理功能，内置函数中的`open`，以及其返回值各种`XXXIOWrapper`均来自于此包；`open`可以同时处理文本文件和二进制文件的读写，只需要它了。
		  logseq.order-list-type:: number
		- `os`：提供更底层的，基于文件描述符的文件读写功能，没有利用上下文处理器
		  logseq.order-list-type:: number
		- `pathlib`：本身是路径操作库，但也提供了功能和内置`open`一样的方法用来打开文件
		  logseq.order-list-type:: number
	- 仅仅是要完整读取文件内容的话，可以使用`pathlib`的`read_text`，如果要流式读取等，才使用`open`。
- ## 时间、日期处理
  collapsed:: true
	- 注意要安装`tzdata`包，否则无法在win系统上使用时区！
	- Python的时间处理主要来自包`datetime`，其提供了下面的类以抽象各种时间概念：
		- `date/time/datetime`：日期，时间和日期+时间，精确到**微秒**， **不可变**，**未给定时区参数时默认不包含时区信息（称为Naive的）**。
		  logseq.order-list-type:: number
		- `timedelta`：时间差，同时也是偏移量，精确到微秒，可为负数，两个日期的距离即为时间差，日期可以应用上一个偏移量。
		  logseq.order-list-type:: number
	- `date`和`time`包下同样提供了时间操作，但是很原始，这里不表。
	- `datetime`可调用`astimezone`方法去不改变时间去修改时区（无参为系统本地时区），**修改后仍为同一时间**，如中国的8点改到`Etc/GMT0`会得到伦敦的0点；时区相关类在`zoneinfo`包下。倘若要从中国的8点得到伦敦的8点，可调用`replace`方法。
	- 问题：
		- 时间格式化和解析
		  logseq.order-list-type:: number
		- 时间差计算，时间偏移
		  logseq.order-list-type:: number
		- 本小时开始，本日开始，本周开始……
		  logseq.order-list-type:: number
	- ### 格式化和解析
		- 格式化和解析分别使用`strftime`（没必要用它，直接用`f"{}"`就行了）和`strptime`。
		- #cheetsheet Python（C标准库`strftime`，`strptime`）时间格式化占位符
			- `%Y`：四位数的年份（例如：1999, 2020）
			- `%m`：月份（01-12）
			- `%d`：月份中的一天（01-31）
			- `%H`：小时（24小时制，00-23）
			- `%M`：分钟（00-59）
			- `%S`：秒（00-59）
			- `%f`：微秒（000000-999999）
			- `%y`：两位数年份
			- `%I`：小时（12小时制，01-12）
			- `%p`：AM 或 PM
			- `%w`：星期几，0是星期天
			- `%z`：UTC偏移
			- `%Z`：时区名称（考虑到有歧义，尽量别用）
			- `%j`：年中的第几天
			- `%U/%W`：年中的第几周，分别以星期日和星期一作为每周第一天
		- 注意，对naive时间，它是没有时区信息的，格式化时`%z`和`%Z`会得到空。
		- **解析时间时，如果带上时区信息，得到的时间便包含时区信息**，否则为naive时间。
	- ### 时间差，偏移量
		- 两个时间相减便得到时间差，时间可以和时间差进行加减。
		- 时间差（从用户的角度看）分为三段：天，天中的秒，秒中的微秒
		- ```python
		  from datetime import datetime, timedelta
		  
		  a: datetime = datetime.now()
		  b: datetime = datetime.now() + timedelta(days=1, minutes=60 * 23, seconds=40, microseconds=50)
		  delta: timedelta = b - a
		  
		  print(timedelta.resolution) # 0:00:00.000001，时间戳的精度
		  print(delta) # 1 day, 23:00:40.000050
		  print(delta.days) # 1，时间间隔中的天数
		  print(delta.seconds) # 82840，时间间隔中的秒数，小于1天
		  print(delta.microseconds) # 50，时间间隔中的微秒
		  print(delta.total_seconds()) # 169240.00005，使用秒表示的整个时间间隔
		  
		  # timedelta的构造器参数是可以溢出的
		  new_delta = timedelta(seconds=delta.total_seconds())
		  print(new_delta) # 1 day, 23:00:40.000050
		  ```
	- ### trunc，以及本周开始时间
		- 遗憾的是，Python没有内置trunc（按某个时间粒度去截断时间，用来获取本小时，本日，本月等的开始时间）
		- 这点小需求也没意义用第三方库，直接手动处理：
		- ```python
		  from datetime import datetime
		  
		  now = datetime.now()
		  
		  def trunc(field: str, date: datetime) -> datetime:
		      EMPTY = dict(year=1, month=1,day=1,hour=0,minute=0,second=0,microsecond=0)
		      keys = list(EMPTY.keys())
		      field_idx = keys.index(field)
		      if field_idx == len(keys) - 1:
		          return date
		      
		      to_zero_fielts = keys[field_idx + 1:]
		      return date.replace(**{field: EMPTY[field] for field in to_zero_fielts})
		  
		  print(trunc('day', now))
		  
		  # 但本周开始时间就得用到timedelta了，得得到当前是星期几，然后减去相应时间
		  def start_of_week(date: datetime) -> datetime:
		      start_of_day = trunc('day', date)
		      return start_of_day - timedelta(days=start_of_day.weekday())
		  
		  print(start_of_week(now))
		  ```
- ## 解析命令行参数
  collapsed:: true
	- 参考 <https://blog.csdn.net/craftsman2020/article/details/129237425>。考虑到这个用的不会很多，用的时候现查文档就是了。
	- `sys.argv`列表第一个参数是脚本名称，剩余是命令行参数，但Python提供了内置库`argparse`去解析命令行参数。
	- `argparse`功能强大，允许声明式地添加参数，选项，提供完善的帮助消息，以及通过字典形式去获取用户输入。
	- 下面以一个转换工具来做示例，用户输入两个文件名，脚本把输入文件转换为输出文件，其中根据输出文件的扩展名判断输出文件类型，但也可以通过可选参数主动去传递：
	- ```python
	  import argparse
	  
	  parser = argparse.ArgumentParser(
	      description="A comprehensive argparse example.", # 描述，在帮助命令中显示
	      epilog="Example usage: script.py input.txt --log-level DEBUG" # 例子，在帮助命令中显示
	  )
	  
	  # 添加位置参数，这两个是必填的
	  parser.add_argument(
	      'input_file', type=str, help='Path to input file',
	  )
	  
	  parser.add_argument(
	      'output_file', type=str, help='Path to output file'
	  )
	  
	  # 添加无参选填参数
	  parser.add_argument(
	      '-q', '--quiet',
	      action='store_true', # 倘若用户给定，存储一个 True，否则存储一个 False
	      help='suppress all output'
	  )
	  
	  # 添加有参选填参数
	  parser.add_argument(
	      '--output-type', # 这里会保存到 output_file 变量
	      type=str,
	      # required=False,  # 这是默认行为
	      help='output type (would ignore output file extension if given)'
	  )
	  
	  # 添加枚举参数
	  parser.add_argument(
	      '--log-level', # 这里会保存到 output_file 变量
	      choices=['INFO', 'DEBUG'],
	      default='INFO',
	      help='output type (would ignore output file extension if given)'
	  )
	  
	  # 添加可反复给定参数
	  parser.add_argument(
	      '--add-filter',
	      type=str,
	      action='append'
	  )
	  
	  args = parser.parse_args()
	  print(args, args.input_file)
	  ```
	- 可选参数中的action有如下类型：
		- `store`：存储参数，多次出现取最后一个，默认行为
		  logseq.order-list-type:: number
		- `store_true`：存储一个true如果给定，否则存储一个False
		  logseq.order-list-type:: number
		- `store_const`：存储一个给定值，用户通过`const`参数传入该给定值
		  logseq.order-list-type:: number
		- `append`：参数存储为列表，每次给定后append
		  logseq.order-list-type:: number
		- `append_const`：参数存储为列表，用户通过`const`参数传入该给定值
		  logseq.order-list-type:: number
- ## 系统调用
  collapsed:: true
	- 系统调用使用`subprocess`包下的函数，Python在本进程中执行其他程序似乎有限制，所以建议使用子进程的形式。
	- 这玩意儿酷炫之处在于它能自动处理不同命令行的转义，nodejs的标准库中即使使用数组传递命令，仍旧需要进行转义。
	- ```python
	  import subprocess
	  
	  result = subprocess.run(['magick', 'identify', '-format',  '{"width":%w, "height":%h}', 'some.png'], text=True, capture_output=True)
	  print('Return code:', result.returncode) # 0
	  print('Standard output:', result.stdout) # {"width":700, "height":411}
	  print('Standard error:', result.stderr) # 
	  
	  # 如果只要标准输出，可以直接使用 check_output，其返回值如果不是0会抛出异常
	  result = subprocess.check_output(['magick', 'identify', '-format',  '{"width":%w, "height":%h}', 'D:/DESKTOP/F511298C3004B5F2260B10AA5A29F2D2.png'], text=True)
	  print(result)
	  ```
	- 注意：`subprocess.run`在返回值非0时不抛异常，除非给定`check=True`；`subprocess.check_output`则会抛出异常。
- ## Excel操作
  collapsed:: true
	- 关于Excel操作，我只需要进行简单读写就行了。使用`openpyxl`库。
	- openpyxl基本上和Excel一致，提供3个抽象——Workbook，Worksheet和Cell。
	- 测试数据：
	  |名称|年龄|所属|更新日期|
	  |:--:|:--:|:--:|:--:|
	  |Haruka|17|765|2020-12-31 00:00:00|
	  |Chihaya|17|765||
	  |Miki||961||
	  |总和|34（计算值）|||
	- ### 对格式规范的表，按行导出实体
		- 使用下标获取表头，使用迭代器获取剩下每一行即可。
		- ```python
		  from openpyxl import load_workbook
		  
		  excel_path = r'D:\DESKTOP\TMP\a.xlsx'
		  
		  wb = load_workbook(excel_path, data_only=True) # data_only=True 使得公式格自动计算出结果
		  
		  # 获取第一个sheet
		  ws = wb.active
		  
		  # 表头，注意索引从1开始
		  print(tuple(cell.value for cell in ws[1]))
		  for row in ws.iter_rows(2, values_only=True):
		      print(row) # 返回类型为 None, int, float, datetime
		  ```
	- ### 对格式固定的表，能查询特定Cell的数据（如果是公式Cell，得到计算后的值）
		- 同上，使用Cell地址直接访问Cell。
		- ```python
		  from openpyxl import load_workbook
		  excel_path = r'D:\DESKTOP\TMP\a.xlsx'
		  wb = load_workbook(excel_path, data_only=True)
		  ws = wb.active
		  # 同上
		  
		  print(ws['B5'].value)
		  ```
	- ### 能把实体集合转换成格式规范的Excel去导出
		- 假设实体是dataclass或NamedTuple定义的
		- ```python
		  from openpyxl import load_workbook
		  
		  from dataclasses import dataclass, fields, is_dataclass
		  from typing import NamedTuple
		  
		  import openpyxl
		  
		  @dataclass
		  class IdolDataClass:
		      name: str
		      age: int
		      cls: str
		  
		  @dataclass
		  class IdolNamedTuple:
		      name: str
		      age: int
		      cls: str
		  
		  def get_fields(obj) -> list[str]:
		      if is_dataclass(obj):
		          return [field.name for field in fields(obj)]
		      return list(obj._fields)
		  
		  idols = [
		      IdolNamedTuple('Haruka', 17, '765'),
		      IdolNamedTuple('Chihaya', 17, '765'),
		      IdolNamedTuple('Miki', 14, '961'),
		  ]
		  
		  headers = get_fields(idols[0])
		  
		  wb = openpyxl.Workbook()
		  ws = wb.active
		  
		  # append 表头（有必要的话可以做个映射到中文）
		  ws.append(headers)
		  
		  for idol in idols:
		      ws.append([getattr(idol, field) for field in headers])
		  
		  wb.save(r'D:\DESKTOP\TMP\dest.xlsx')
		  ```
- ## xml/html解析
  :LOGBOOK:
  CLOCK: [2024-08-27 Tue 16:06:22]
  :END:
	- Python提供了内置库`http.parser`去解析html，但这里选择使用第三方库`beautifulsoup4`（简称bs4），它能同时解析`html`和`xml`，同时使用`lxml`作为它的“后端”。
	- bs包装的html片段是有状态的，可以自由修改属性，内容，子元素等。
	- ```sh
	  pip install beautifulsoup4 lxml # lxml 作为bs4的“后端”，性能更高 
	  ```
	- 直接上示例，包括查询，操作和导出html。
	- ```python
	  from bs4 import BeautifulSoup, Tag
	  
	  html_doc = """
	  <html>
	  <head><title>Example Page</title></head>
	  <body>
	  <h1 id="main-heading" class="header" style="font-size: 100px">Welcome to the Example Page</h1>
	  <p class="description">This is a sample page with <a href="http://example.com" class="link">a link</a>.</p>
	  <p class="description">Another paragraph.</p>
	  </body>
	  </html>
	  """
	  
	  # 构建bs4实例，这里也可以直接传一个文件对象
	  soup = BeautifulSoup(html_doc, 'lxml')  # 也可以使用 'lxml'
	  
	  # 根据CSS选择器去查询元素
	  print(soup.select('html > body > p'))
	  print(soup.select('.description'))
	  
	  # 根据tag，id和class去查询元素
	  print(soup.find('body'))
	  print(soup.find(id='main-heading'))
	  print(soup.find_all(class_='description'))
	  
	  # 获取元素的属性，文本内容
	  print(soup.find(id='main-heading')['style']) # 注意获取的 class 是数组
	  print(soup.find(id='main-heading').string) # 该方法在有多个子元素时返回None！
	  print(soup.find_all(class_='description')[0].get_text()) # 获取整个标签包括子元素的纯文本表示，可指定分隔符
	  
	  # 获取子元素
	  for child in soup.find('body').children:
	      # child 可能是字符串！
	      print(child)
	  # 递归获取子元素
	  for child in soup.find('body').descendants:
	      # child 可能是字符串！
	      print(child)
	  
	  # 修改元素属性，内容
	  last_p: Tag = soup.find_all(class_='description')[-1]
	  last_p['style'] = 'color: white' # 设置属性
	  last_p.string = '替换整个内容！' # 替换子元素为文本内容
	  
	  # 替换所有子元素
	  last_p.clear() # 清空所有子元素，然后添加
	  
	  # 删除自己
	  last_p.parent.decompose(last_p)
	  # 从文档树中删除并获得自己
	  me = last_p.parent.extract(last_p)
	  
	  # 创建，添加新元素
	  some_a = soup.new_tag('a')
	  some_a['href']='https://baidu.com'
	  some_a.string = '一个link'
	  
	  another_a = soup.new_tag('a')
	  another_a['href']='https://google.com'
	  another_a.string = '又一个link'
	  
	  last_p.append(some_a)
	  last_p.append(another_a)
	  
	  # 导出元素为字符串
	  print(str(last_p)) # 直接转换成字符串
	  print(last_p.prettify()) # 带着缩进
	  ```
	- #cheetsheet CSS选择器示例，仅用这玩意儿去选择html元素就够啦！
		- ```
		  *                            # 全部元素
		  div                          # 全部 div 元素
		  .some-class                  # 类名含有 some-class 的元素
		  .class1.class2               # 类名含有 class1，class2 的元素
		  div.some-class               # div且类名含有 some-class 的元素
		  [attr]                       # 含有attr属性的元素
		  [attr='abc']                 # attr 属性值为 'abc' 的元素
		  div.some-class[attr='abc']   # div且类名含有some-class且attr为'abc'的元素
		  #someId                      # id为'someId'的元素
		  
		  # 注意下面的XXX，YYY均可以为 *
		  XXX > *                      # 满足XXX选择器元素的直接子元素
		  XXX *                        # 满足XXX选择器元素的递归子元素
		  XXX + YYY                    # 上一个元素为满足XXX选择器的元素，自己满足YYY选择器的元素
		  XXX ~ YYY                    # 之前的元素满足XXX选择器，自己满足YYY选择器的元素
		  XXX, YYY                     # 满足XXX选择器或YYY选择器的元素
		  :not(XXX)                    # 不满足XXX选择器的元素
		  
		  html > body > a              # html下的body下的所有p子元素
		  .content p                   # class为content的元素递归下的所有p元素
		  
		  # :开头的选择器是html自动标注的，称为伪类选择器
		  body :first-child            # body 子孙中每一个第一个子元素
		  body > :first-child          # body 的第一个子元素
		  body > p:first-of-type       # body 的第一个 p 子元素
		  body > :last-child           # body 的最后一个子元素
		  body > p:last-of-type        # body 的最后一个 p 子元素
		  ```
- ## 浏览器自动化
  collapsed:: true
  :LOGBOOK:
  CLOCK: [2024-08-27 Tue 16:06:25]
  :END:
	- 加入这一节主要目的是为了让写爬虫时在登陆难以处理时直接打开浏览器让用户手动登陆。因此只提供一个示例——打开网易云音乐登录界面、用户手动登陆、拿到cookie，关闭页面；另一个情况可能就得是操作被ban，要打开页面让用户手动验证自己是人类。
	- 需要先`pip install selenium`，然后安装一个浏览器driver，可以直接装[Edge的driver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver)，毕竟谁电脑里都有Edge，
	- 实现大概是如此逻辑：
		- 首先定义判断是否登录逻辑，可以找一个需要登陆才能访问的接口去验证
		  logseq.order-list-type:: number
		- 打开浏览器，打开登录页面
		  logseq.order-list-type:: number
		- 主进程轮询是否登录或浏览器窗口关闭；当登录成功时，主动关闭浏览器并返回（这里提供了一个把selenium形式的cookie转换成CookieJar的函数），当浏览器关闭时，抛出异常
		  logseq.order-list-type:: number
	- ```python
	  from typing import Callable, Dict
	  import requests
	  from selenium import webdriver
	  from selenium.webdriver.edge.options import Options
	  import time
	  from urllib.parse import urlparse
	  
	  LOGIN_URL = 'XXX'
	  CHECK_LOGIN_URL = 'XXX'
	  NOT_LOGIN_TEXT = 'XXX'
	  
	  def is_login(cookie: Dict[str, str]):
	      res = requests.get(CHECK_LOGIN_URL, cookies=cookie)
	      return NOT_LOGIN_TEXT not in res.text
	  
	  def save_selenium_cookie(selenium_cookies: List[dict], cookiejar: CookieJar):
	      for cookie in selenium_cookies:
	          cookiejar.set_cookie(Cookie(
	              version=0,
	              name=cookie['name'],
	              value=cookie['value'],
	              port=None,
	              port_specified=False,
	              domain=cookie['domain'],
	              domain_specified=True,
	              domain_initial_dot=cookie['domain'].startswith('.'),
	              path=cookie['path'],
	              path_specified=True,
	              secure=cookie.get('secure', False),
	              expires=cookie.get('expiry', None),
	              discard=False,
	              comment=None,
	              comment_url=None,
	              rest={'HttpOnly': cookie.get('httpOnly', False)},
	              rfc2109=False
	          ))
	    
	  def user_login(check_login: Callable[[Dict[str, str]], bool], login_url: str):
	      """用户手动登陆，返回cookie"""
	      options = Options()
	      options.add_argument(f"--app={login_url}")  # 只显示页面内容
	      options.add_argument("--disable-blink-features=AutomationControlled") 
	      options.add_argument("--ignore-certificate-errors")  
	      options.add_argument("--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0") 
	  
	      driver = webdriver.Edge(options=options)
	      while True:
	          time.sleep(0.5)
	          try:
	              cookie = driver.get_cookies()
	          except:
	              raise RuntimeError("Window closed without login")
	          
	          filtered_cookie = {i['name']:i['value'] for i in cookie if urlparse(login_url).netloc in i['domain']}
	          if check_login(filtered_cookie):
	              driver.quit()
	              return filtered_cookie
	          
	  print(user_login(is_login, LOGIN_URL))
	  ```
- ## 数据可视化
  collapsed:: true
  :LOGBOOK:
  CLOCK: [2024-08-27 Tue 16:06:31]--[2024-08-28 Wed 12:20:25] =>  20:13:54
  :END:
	- 用`matplotlib`，它的绘图功能似乎是类似MATLAB。
	- `matplotlib` 本身包含下面的概念：
		- `Figure`：图表的顶层容器，`Figure`包含一个或多个`Axes`对象，一个`Figure`对应一个系统窗口；`add_subplot`方法可以添加`Axes`到`Figure`中
		  logseq.order-list-type:: number
		- `Axes`：Axes是通常意义上的图表，每个图表包含一个或多个数据series，并包含两个`Axis`对象，即坐标轴
		  logseq.order-list-type:: number
		- `Axis`：坐标轴对象，每个`Axes`包含两个坐标轴（3d绘图内置库没有直接提供）
		  logseq.order-list-type:: number
		- `Plot`：Plot并非是独立存在的类，它是`axes`的一系列方法，用于修改`axes`的状态，绘制数据点和线条，`Plot`包括plot（折线图），bar（柱状图，以及barh，水平柱状图），scatter（散点图），hist（直方图），pie（饼图）等
		  logseq.order-list-type:: number
		- `Artist`：所有可绘制对象的基类，用户可以自定义子类去实现它，绘制自定义图像（如3D图像）
		  logseq.order-list-type:: number
		- `Figure Canvas`和`Backend`：绘图后端，和GUI后端相关，配置此可以将所绘制图表集成到其他GUI框架中，如PyQt5。
		  logseq.order-list-type:: number
	- `matplotlib`有两种调用方式——基于命令式和基于面向对象式的，**前者假设只有一个figure，一个axes，然后把所有操作该figure和axes的函数都暴露在顶层**。
	- 命令式的示例：
		- ```python
		  import matplotlib.pyplot as plt
		  
		  # 绘制折线图
		  plt.plot([1, 2, 3], [4, 5, 6], label='Line Plot')
		  
		  # 接着绘制散点图
		  plt.scatter([1, 2, 3], [6, 5, 4], color='red', label='Scatter Plot')
		  
		  # 坐标轴标签
		  plt.xlabel('x')
		  plt.ylabel('y')
		  # 添加图例
		  plt.legend()
		  
		  # 显示图形
		  plt.show()
		  ```
	- 声明式的示例：
		- ```python
		  import matplotlib.pyplot as plt
		  
		  # 创建一个 1x2 的子图布局，即axs会有两个axes
		  # 有多行时，使用二维矩阵去访问
		  fig, axs = plt.subplots(1, 2)
		  
		  # 在第一个子图中绘制折线图
		  axs[0].plot([1, 2, 3], [4, 5, 6], label='Line Plot')
		  axs[0].legend()
		  axs[0].set_title('Line Plot')
		  
		  # 在第二个子图中绘制散点图
		  axs[1].scatter([1, 2, 3], [6, 5, 4], color='red', label='Scatter Plot')
		  axs[1].legend()
		  axs[1].set_title('Scatter Plot')
		  
		  # 显示图形
		  plt.show()
		  ```
	- 关于axes的坐标，axes的坐标根据三个参数去确定——行数，列数，第几个图表，三个参数均从1开始；python用3位整数去表示坐标。不同的axes可以有不同的行数、列数，此时会出现重叠，所以一般都保持行数列数固定。
- ## 数据库操作
  collapsed:: true
  :LOGBOOK:
  CLOCK: [2024-08-27 Tue 16:06:34]
  :END:
	- Python有一个关系型数据库标准协议，DB-API，如MySQL，Oracle和SQLite（内置标准库）均遵循该协议。这里只去学习它。但注意到**Pandas支持利用它去直接得到DataFrame**，或许后面值得一学。
	- 注意DB-API只是协议，它不对应任何具体的抽象类，只是一个规范。
	- 同样，这里直接给示例，反正这玩意儿和直接执行sql本来也没啥区别。注意DB-API的关键概念是cursor游标，一切操作都必须用它去执行。
	- **注意写入操作后必须要提交事务！！！**
	- ```python
	  import mysql.connector 
	  
	  # 创建MySQL连接，sqlite连接则是指定文件名
	  conn = mysql.connector.connect(
	      user = 'root',
	      password = 'root',
	      host = '172.1.1.1',
	      port = 3306,
	      database = 'hello-world',
	  )
	  # 创建游标
	  cur = conn.cursor()
	  
	  # 查询
	  cur.execute('select * from abc')
	  for row in cur.fetchall():
	      print(row)
	  
	  # 游标也可以执行 DDL
	  cur.execute('create table wtf(a int, b int)')
	  
	  # 写入时可能会创建事务，也可能不会，但总之自己提交事务准没错
	  # 如果没有提交事务，修改会自动回滚！！！
	  cur.execute('insert into wtf values (1, 2)')
	  
	  # 提交事务
	  conn.commit()
	  ```
	- 关于MySQL类型到Python的映射：
		- TINYINT, SMALLINT, MEDIUMINT, INT, INTEGER, BIGINT -> `int`
		  logseq.order-list-type:: number
		- DOUBLE, FLOAT -> `float`
		  logseq.order-list-type:: number
		- DECIMAL -> `decimal.Decimal`
		  logseq.order-list-type:: number
		- DATE -> `datetime.date`
		  logseq.order-list-type:: number
		- TIME -> `datetime.time`
		  logseq.order-list-type:: number
		- DATETIME, TIMESTAMP -> `datetime.datetime`
		  logseq.order-list-type:: number
		- CHAR, VARCHAR, TEXT...... -> `str`
		  logseq.order-list-type:: number
		- XXXBLOB -> `bytes`
		  logseq.order-list-type:: number
- ## HTTP请求
  collapsed:: true
  :LOGBOOK:
  CLOCK: [2024-08-27 Tue 16:06:18]
  :END:
	- 关于HTTP请求，有如下需求：
		- 异步，使得能直接地高并发
		  logseq.order-list-type:: number
		- 但在简单情况下也能同步执行
		  logseq.order-list-type:: number
		- 能持久化和共享cookie
		  logseq.order-list-type:: number
	- 这里使用`httpx`库。`pip install httpx 'httpx[cli]'`，这里同时安装了httpx的cli，能够替代curl，虽然没啥必要。详见[官方文档](https://www.python-httpx.org)。
	- **同步**发送HTTP请求（同步，但异步代码是类似的）：
	- ```python
	  import httpx
	  
	  ### GET请求
	  # 这里的headers和params可以传字典也可以传这种键值对集合
	  # 因为params本身就是可以重复的，而headers重复时会被其按逗号拼接
	  r = httpx.get('https://httpbin.org/get', 
	                headers=[('Accept', 'text/html'), ('Accept', 'text/plain')], 
	                cookies={'hello': 'world'}, # cookie直接提供了字段
	                params=[('a', 1), ('a', 2)], # URL 查询参数 
	                timeout=10.0)  # 超时时间
	  print(r.encoding) # utf-8，可以赋值
	  print(r) # <Response [200 OK]>
	  print(r.status_code)  # 200    
	  print(r.text) # 获取响应体，以str形式，似乎会有缓存 
	  print(r.content) # 获取响应体，以bytes形式
	  r.raise_for_status() # 抛出异常如果响应码是4xx，5xx
	  
	  ### POST application/json
	  r = httpx.post('https://httpbin.org/post', 
	                params=[('a', 1), ('b', 2)],
	                json={'hello': 'world'}) 
	  
	  ### POST，application/x-www-form-urlencoded
	  # 上面的json换成data即可
	  
	  ### POST，multipart/form-data
	  # 上面的json换成file即可
	  ```
	- **异步**的话需要使用AsyncClient，它自带连接池和cookie持久化，**异步的返回结果的类型和同步的是相同的**……不太优雅：
	- ```python
	  import httpx
	  import asyncio
	  async def go():
	      # client自带连接池和cookie持久化（在程序的生命周期内），创建是比较昂贵的，不要多次创建
	      async with httpx.AsyncClient() as f: 
	          # 虽然是异步，但是在这里已经得到了整个响应体了
	          r = await f.post('https://httpbin.org/post', 
	                      params=[('a', 1), ('a', 2)],
	                      json={'hello': 'world'}) 
	          print(r.text)
	  
	          # 但要连响应体也异步，流式地读的话，使用stream方法
	          async with f.stream('POST', 'https://httpbin.org/post', files={'hello': open('image.png', 'rb')}) as response:
	              async for line in response.aiter_lines():
	                  print(line)
	  
	  asyncio.run(go())
	  ```
	- 关于**Cookie持久化**，Client的cookies属性能获取存储的cookies，该对象可以使用pickle去进行持久化：
	- ```python
	  import httpx
	  
	  async def go():
	      cookie_file = Path('my_cookies.dat')
	      cookies = None
	      if cookie_file.exists():
	          cookies = pickle.loads(cookie_file.read_bytes())
	  
	      async with httpx.AsyncClient(cookies=cookies) as f: 
	          r = await f.post('https://blog.csdn.net/wayne_youlu/article/details/135596307', 
	                      params=[('a', 1), ('a', 2)],
	                      json={'hello': 'world'}) 
	          print(r.text)
	          cookie_file.write_bytes(pickle.dumps({**f.cookies}))
	  asyncio.run(go())
	  ```
	- {{embed ((66e38f96-e83b-4fdc-a1d3-9b429e1aa8ce))}}
- ## 图像处理
	- Python内置了图像处理库`PIL`（年久失修，现在用的都是它的fork `Pillow`，两者认为是同义词），如果没有内置，就`pip install Pillow`安装。
	- PIL中有如下实体/类/模块可能会常用：
		- Image模块：PIL的门面，提供最基础和常用的函数以及`Image`类，包括打开文件为Image，多个Image的组合操作等，很多时候只导入它就行了
		  logseq.order-list-type:: number
		- Image类：**代表实际图像**，能够获取图像各种信息，包括大小，类型，色彩模式（通道），EXIF，乃至实际像素，直方图，同时也能对图像进行操作，如裁剪，缩放，设置像素颜色，应用滤镜等。滤镜需要结合`ImageFilter`模块去使用，同时`ImageOps`提供了更多操作。注意有的操作是返回新`Image`，而有的操作是原地的，比如在图像上添加图像的`paste`方法，还有设置像素信息的`putXXX`。
		  logseq.order-list-type:: number
		- ImageColor模块，提供`getrgb`和`getcolor`方法，**把各种描述的颜色（似乎所有css表示都支持，如`#ffffff`，`rgb(1,2,3)`，`red`）转换成相应色彩模式的元组**，`getrgb`等价于`getcolor(mode='RGB')`
		  logseq.order-list-type:: number
		- ImageDraw模块：**提供绘制操作**
		  logseq.order-list-type:: number
		- ImageFilter模块：**提供滤镜操作**，滤镜以`Filter`类的实例去存在
		  logseq.order-list-type:: number
		- ImageOps模块：提供**更多图像操作**
		  logseq.order-list-type:: number
	- PIL太大，只给定可能常用的示例，后面有需要就现查。
	- ### 水平拼接两张图片，放大高度更小的那一张使得高度一致
	-
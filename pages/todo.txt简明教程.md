- [todo.txt](https://github.com/todotxt/todo.txt) 是一种使用纯文本形式来表示 TODO 清单的一个格式，考虑对其进行一些了解。然后也学习一下 [todo.txt-cli](https://github.com/todotxt/todo.txt-cli) 的使用，它是一个使用 shell 去实现的，控制台的 todo.txt 文件管理脚本。
- # todo.txt
- todo.txt 使用纯文本形式来保存 TODO 清单，其最重要的规则是，**一行一个 TODO**，它的语法如下。
- ![image.png](../assets/image_1729439429363_0.png){:height 392, :width 766}
- `todo.txt`的格式是十分自由的，其设计初衷之一就是要求人类和机器均可读。对每一个 TODO，其有三个维度：
	- **优先级**：最重要的 TODO 要能够先出现，其通过`(A-Z)`的语法表示。
	- **项目**：TODO 应该能够从属于特定项目——人类解决复杂问题的方式就是将其分解为一个个更容易解决的子问题并着手解决。
	- **上下文**：一个 TODO 要在哪里，在怎样的情况下去完成，换句话说，TODO 的上下文，是需要能够表示的，这样就能让人去关注在当前能够去做的事情。（这似乎是某种称为 [GTD，Getting Things Done](https://en.wikipedia.org/wiki/Getting_Things_Done) 的哲学，之后去了解一番）
- 项目使用`+`语法去表示，上下文通过`@`语法去表示，两者都接受非空字符串作为名称，两者都可以出现任意次数。
- 比如，下面指定一个最高优先级的，处在学习项目的，上下文为下班后的 TODO：
- ```
  (A) @下班后 +学习 Spring MVC
  ```
- 然后是 KV tag，这是**供开发者**去使用的 metadata，语法是`key:value`。
- `todo.txt`就这些东西…简单，但优雅。
- # todo.txt-cli
- `todo.txt-cli`是官方提供的命令行下管理`todo.txt`的脚本，这里介绍它的使用。
- MacOS 和 Linux 可直接使用，Win 用户可使用 Git For Bash 等终端。
- ## 安装
- 在 [官方仓库](https://github.com/todotxt/todo.txt-cli) 下载`todo.sh`和`todo.cfg`，其中将`todo.sh`放置到 PATH 中（下面的示例将其改名为`todo`），将`todo.cfg`重命名为`~/.todo.cfg`，然后按需求编辑`.todo.cfg`，默认todo存储位置为todo脚本位置。
- 然后进行测试，输入`todo add 'Hello, Happy World!'`，并检查`todo存储位置/todo.txt`是否有数据。
- ## 示例
- 先创建测试“数据”，直接编辑`todo.txt`或者通过`todo add`命令：
- ```
  (A) +健身 @早晨 @晚上 @楼下 各跳绳 500 个
  (C) +英语 @晚上 完成扇贝阅读一篇文章
  (A) +吉他 yousician 30 分钟
  (B) +吉他 +辨识 200 个 note
  +吉他 完成练习二
  ```

> 小工具：[Bookmarklet Crunchinator](http://ted.mielczarek.org/code/mozilla/bookmarklet.html)

新建后缀为`url`的文件，这是一个书签文件。使用vscode或其他文本编辑器打开。书签文件的简单模板如下：

```
[InternetShortcut]
URL=https://www.google.com
```

当我们需要执行js代码时，只需要修改为下面这样

```
[InternetShortcut]
URL=javascript:javascript:(function(){alert('111');})();
```

文件保存后，将文件拖放到浏览器的书签栏，点击书签，即可触发这段代码。

Bookmarklet是一个复合词，由Bookmark（书签）和-let（小的）构成，中文可以译成"书签工具"。

### Bookmarklet编写规则

使用书签的方式编写js脚本，需要注意一下规则：

1. 必须以`javascript:`开头

   浏览器把`javascript:`当做协议看待。有了它，浏览器才知道要用`javascript`解释后面的代码。它的作用等同于将代码放在`<script></script>`之间运行。

2. 所有代码必须在同一行

   因为浏览器把Bookmarklet当做网址保存，而网址是不能分行的，所以Bookmarklet也不能分行。另一方面，网址是有长度限制的。IE的最长网址不能超过2083个字符。

3. 使用单引号

   根据Javascript的语法，单引号（'xxx'）和双引号（"xxx"）都能使用。但是由于html语言主要使用双引号，所以Bookmarklet优先使用单引号。万一遇到必须使用双引号的情况，就采用它的URL编码形式"%22"。

4. 不要污染全局变量

   Bookmarklet最好不要生成新的全局变量，可以采用直接运行匿名函数的方式。

5. 对文本和URL进行编码

   为了防止出现非法字符，代码以外的文本都应该使用encodeURIComponent()函数进行编码，比如把空格变成%20。


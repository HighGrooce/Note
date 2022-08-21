# RE-NOTE

#### 尽量使用原始字符串表示法 r'...'

https://docs.python.org/zh-cn/3/howto/regex.html#regex-howto

#### 常用特殊字符(元字符)

- .      (点) 在默认模式，匹配除了换行的任意字符。如果指定了标签 [`DOTALL`](https://docs.python.org/zh-cn/3/library/re.html?highlight=re#re.DOTALL) ，它将匹配包括换行符的任意字符。
- ^      (插入符号) 匹配字符串的开头， 并且在 MULTILINE 模式也匹配换行后的首个符号。
- $      匹配字符串尾或者在字符串尾的换行符的前一个字符，在 MULTILINE 模式下也会匹配换行符之前的文本。
- \*      匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \\\*。
- \+      匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \\\+。
- ?      匹配前面的子表达式零次或一次，或指明一个非贪婪限定符。要匹配 ? 字符，请使用 \\?。

#### 函数

##### re.compile(pattern, flags=0)

将正则表达式的样式编译为一个 正则表达式对象，通过 re.compile() 编译后的样式，和模块级的函数会被缓存，如果需要多次使用这个正则表达式的话，使用 re.compile() 和保存这个正则对象以便复用，可以让程序更加高效。

```python
# 序列
prog = re.compile(pattern)
result = prog.match(string)
# 等价于
result = re.match(pattern, string)
```

##### re.findall(pattern, string, flags=0)

返回 pattern 在 string 中的所有非重叠匹配，以字符串列表或字符串元组列表的形式。对 string 的扫描从左至右，匹配结果按照找到的顺序返回。 空匹配也包括在结果中。

返回结果取决于模式中捕获组的数量。如果没有组，返回与整个模式匹配的字符串列表。如果有且仅有一个组，返回与该组匹配的字符串列表。如果有多个组，返回与这些组匹配的字符串元组列表。非捕获组不影响结果。

```python
re.findall(r'\bf[a-z]*', 'which foot or hand fell fastest')
# ['foot', 'fell', 'fastest']
re.findall(r'(\w+)=(\d+)', 'set width=20 and height=10')
# [('width', '20'), ('height', '10')]
```

##### re.sub(pattern, repl, string, count=0, flags=0)

返回通过使用 repl 替换在 string 最左边非重叠出现的 pattern 而获得的字符串。 如果样式没有找到，则不加改变地返回 string。 repl 可以是字符串或函数；如为字符串，则其中任何反斜杠转义序列都会被处理。 也就是说，\n 会被转换为一个换行符，\r 会被转换为一个回车符，依此类推。 未知的 ASCII 字符转义序列保留在未来使用，会被当作错误来处理。 其他未知转义序列例如 \& 会保持原样。 向后引用像是 \6 会用样式中第 6 组所匹配到的子字符串来替换。 例如:

```python
re.sub(r'def\s+([a-zA-Z_][a-zA-Z_0-9]*)\s*\(\s*\):',
        r'static PyObject*\npy_\1(void)\n{',
        'def myfunc():')
# 'static PyObject*\npy_myfunc(void)\n{'
```

如果 repl 是一个函数，那它会对每个非重复的 pattern 的情况调用。这个函数只能有一个 匹配对象 参数，并返回一个替换后的字符串。比如

```python
def dashrepl(matchobj):
     if matchobj.group(0) == '-': return ' '
     else: return '-'
re.sub('-{1,2}', dashrepl, 'pro----gram-files')
# 'pro--gram files'
re.sub(r'\sAND\s', ' & ', 'Baked Beans And Spam',flags=re.IGNORECASE)
# 'Baked Beans & Spam'
```

##### re.search(pattern, string, flags=0)

扫描整个 字符串 找到匹配样式的第一个位置，并返回一个相应的 匹配对象。如果没有匹配，就返回一个 None ； 注意这和找到一个零长度匹配是不同的。

##### re.match(pattern, string, flags=0)
如果 string 开始的0或者多个字符匹配到了正则表达式样式，就返回一个相应的 匹配对象 。 如果没有匹配，就返回 None ；注意它跟零长度匹配是不同的。
注意即便是 MULTILINE 多行模式， re.match() 也只匹配字符串的开始位置，而不匹配每行开始。
如果你想定位 string 的任何位置，使用 search() 来替代

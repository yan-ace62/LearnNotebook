# VIM学习教程笔记

本教程记录使用vim编码过程中常用命令。
## vim模式

vim编辑器主要包括三种模式：**normal**、**insert**、**visual**、**command**. 默认模式是**normal**,通过按esc键从其他模式切换到normal模式.  
normal模式下主要使用**操作符-动作**组合命令实现各种编辑.  
#### 操作符类型与动作

|操作符类别 | |动作类别 |主要包括移动动作 |
|:--:|--|:--:|--|
| d |删除动作| 行内移动 |如w, e, 0 |
| c |修改操作| 行间移动 |如j, k|
| y |复制操作|
| p |黏贴操作|

### **insert**模式
insert模式与我们常用文本编辑使用方式一样，可以通过键盘进行输入、删除等等.

-  
    |进入insert模式命令 | |
    |:--: |--|
    |  i  |插入到光标前|
    |  a  |插入到光标后|
    |  I  |插入到行的第一个字符前|
    |  A  |插入到行的最后一个字符后面|
    |  o  |在光标下一行插入空行|
    |  O  |光标上一行插入空行|
    |  s  |删除当前光标字符|
    |  S  |清除当前行的内容|
    |c + motion|删除motion移动范围字符|

### **normal**模式
1. 移动功能  

    |行内移动   |                  |词移动     |    |
    |:---:|:---|:---:|---|
    | l       |移动到光标的后一个字符| w |移动到后一个单词首部|
    | h       |移动到光标的前一个字符| e |移动到后一个单词的尾部|
    |shift + ^|移动到行第一个字符   |**括号匹配**|          |
    |shift + $|移动到行最后一个字符 | % |移动到匹配的括号另一半的位置|
    | 0       |移动到行首部|

    |行位置移动|number表示一个数字  | 滚屏移动| |
    | :---:  |:---|:---:|---|
    |[number] + k  |移动到光标的上number行|ctrl + d|向下移动半个显示屏幕|
    |[number] + j  |移动到光标的下number行|ctrl + u|向上移动半个显示屏幕
    |    gg        |移动到文档的第一行    |**标记跳转**|使用G移动时，记录为标记位置|
    |    G         |移动到文档的最后一行   |ctrl + o|跳转到较老标记位置|
    |[number] + G  |移动到第number行     |ctrl + i|跳转到较新标记位置|

2. 编辑功能
    | 删除动作d|motion表示移动范围命令 |复制黏贴动作| |
    |:---:|---|:---:|---|
    |[number] + x |删除光标后number个字符| yy |将光标行抽取到寄存器|
    |[number] + X |删除光标前number个字符| p  |将寄存器的内容黏贴到光标下一行|
    |dd  |删除当前行     | P | 将寄存器的内容黏贴到光标的上一行|
    |[number] + dd|删除当前行开始的后number行|y + motion|将motion移动的字符抽取到寄存器|
    |d + motion|删除motion移动范围的字符|
    |d + w|删除光标当前位置到单词结尾|
    | J |删除行尾换行符(将当前行与下一行连接)|

3. 折叠功能与撤销、重做功能
    | 折叠命令 | |撤销与重做命令||
    | :---: | --- |:---:|---|
    |z + o |打开当前折叠| u |撤销最后一个修改动作|
    |z + c |关闭当前折叠| U |撤销在行中最后一个修改动作|
    |z + M |关闭所有折叠|ctrl + r|撤销上一个撤销|
    |z + R |打开所有折叠|

4. 搜索(查找)：默认为正则匹配查找
    |查找　|正则匹配元字符.*[]^%/\>~$|
    |:--: | -- |
    |/string|(从上往下)搜索符合string的内容|
    |   n   |跳转到下一个匹配的string|
    |　　N  |跳转到上一个匹配的string|
    |  ^ $ |定位元字符:行首与行尾|
    |   .  |匹配任意单个字符|

### **visual**模式

visual模式能够高亮显示光标移动范围的字符.  
三种进入visual模式的命令:
1. v键

    可使用**操作符-动作**命令操作行

2. V键

    可方便的操作多行

3. ctrl + v

    从光标开始操作块文本
## 1、RAR简介

> **`wikipedia:`**  
> 1、[https://en.wikipedia.org/wiki/RAR_(file_format](https://en.wikipedia.org/wiki/RAR_(file_format))  
> 2、[https://www.loc.gov/preservation/digital/formats/fdd/fdd000450.shtml](https://www.loc.gov/preservation/digital/formats/fdd/fdd000450.shtml)

  RAR是一种专利文件格式，用于`数据压缩`与`归档打包`，开发者为`尤金·罗谢尔`（俄语：Евгений Лазаревич Рошал，拉丁转写：Yevgeny Lazarevich Roshal），RAR的全名是“`Roshal ARchive`”，即“罗谢尔的归档”之意。首个公开版本`RAR 1.3`发布于1993年。  
  尤金·罗谢尔，1972年3月10日生于`俄罗斯`。毕业于俄罗斯车里雅宾斯克工业大学（Chelyabinsk Technical University，今南乌拉州立大学），也是`FAR文件管理器`的作者。他开发程序压缩或解压RAR文件，最初用于DOS，后来移植到其它平台。主要的Windows版本编码器，称为WinRAR，以共享软件的形式发行。不过罗谢尔公开了解码器源码，UnRAR解码器许可证以不许发布编译RAR兼容编码器为条件下允许有条件自由发布与修改，而RAR编码器一直是有专利的。  
  最近的开发者是尤金·罗谢尔的胞兄`亚历山大·罗谢尔`。虽然其解码器有专利，编译好的解压程序仍然存在于若干平台，例如开源的7-Zip。  
  RAR同时也拥有成熟的加密算法，`2.0 版本前`加密算法未公开，`2.0 版本后`使用AES算法加密，在没有密码情况下目前只有`暴力破解`。

**`文件特点：`**

- RAR通常情况比ZIP压缩比高，但压缩／解压缩速度较慢。
- **分卷压缩：**压缩后分割为多个文件。
- **固实压缩：**将要压缩的文件视为同一个文件以加大压缩比，代价是取用压缩包中任何文件需要解压整个压缩包。
- **恢复记录：**加入冗余数据用于修复，在压缩包本身损坏但恢复记录够多时可对损坏压缩包进行恢复。
- **加密：**RAR 2.0使用AES-128-CBC，RAR5.0以后为AES-256-CBC。之前RAR的加密算法为私有。当前除了暴力破解之外不存在（至少没有公开）有效的破解方法。

### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#1-1%E3%80%81RAR%E7%9A%84%E5%8E%86%E5%8F%B2%E7%89%88%E6%9C%AC "1.1、RAR的历史版本")1.1、RAR的历史版本

RAR文件格式的`修订历史记录`：

- **`1.3`** 第一个公开版本，没有“ Rar！” 签名。
- **`1.5`** 更改未知。
- **`2.0`** 与WinRAR 2.0和Rar一起发布，用于MS-DOS 2.0。
- **`2.9`** 在WinRAR版本3.00中发布。
- **`5.0`** WinRAR 5.0及更高版本支持。

WinRAR 5.0和Android的RAR将`2.9版本`称为`RAR4`。

---

## [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2%E3%80%81RAR%E5%8E%8B%E7%BC%A9%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90 "2、RAR压缩文件格式分析")2、RAR压缩文件格式分析

### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-1%E3%80%81RAR-1-5-4-0%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90 "2.1、RAR 1.5 - 4.0文件格式分析")2.1、RAR 1.5 - 4.0文件格式分析

> **`参考资料：`**  
> 1、[http://acritum.com/winrar/rar-format](http://acritum.com/winrar/rar-format)  
> 2、[https://forensicswiki.xyz/page/RAR](https://forensicswiki.xyz/page/RAR)  
> 3、[https://blog.csdn.net/howeverpf/article/details/8909362](https://blog.csdn.net/howeverpf/article/details/8909362)  
> 4、[https://codedread.github.io/bitjs/docs/unrar.html](https://codedread.github.io/bitjs/docs/unrar.html)

**`实例环境：`**

[![例子RAR文件环境0](https://sp4n9x.github.io/resources/2020/RAR_Format/SimpleRAR_env0.jpg "例子RAR文件环境0")](https://sp4n9x.github.io/resources/2020/RAR_Format/SimpleRAR_env0.jpg "例子RAR文件环境0")

  

[![例子RAR文件环境1](https://sp4n9x.github.io/resources/2020/RAR_Format/SimpleRAR_env1.jpg "例子RAR文件环境1")](https://sp4n9x.github.io/resources/2020/RAR_Format/SimpleRAR_env1.jpg "例子RAR文件环境1")

  压缩文件由`可变长度`的块组成。这些`块的顺序`可以变化，但是第一个块必须是`标记块`，然后是`压缩文件头块`。

  现在公开的块类型有：`标记块`，`压缩文件头块`，`文件头块`，`注释头块`，`用户身份信息块`，`子块`和`恢复记录块`等。每一块的开始是由通用字段开始，且每一个不同的块的通用字段结构都是一样的。

`通用字段结构`如下表：

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|所有块或块部分的CRC|
|HEAD_TYPE|1|块类型|
|HEAD_FLAGS|2|块标记|
|HEAD_SIZE|2|块大小|

如果块标记的第一位被置1的话(`(HEAD_FLAGS & 0x8000)!= 0`)还存在：

|字段名称|长度(byte)|说明|
|---|---|---|
|ADD_SIZE|4|可选结构 - 增加块大小|

  所以文件大小的计算分两种情况，当块标记`HEAD_FLAGS`首位未置1，则总块大小就是`HEAD_SIZE`，当块标记`HEAD_FLAGS`首位置1，可选结构`ADD_SIZE`存在，则总块大小为`HEAD_SIZE`+`ADD_SIZE`。

  在每个块中，`HEAD_FLAGS`中的以下位具有相同的含义：

> `0x4000` - 如果设置，则较旧的RAR版本将忽略该块，并在更新存档时将其删除。如果清除，则在更新存档时将此块复制到新的存档文件；  
> `0x8000` - 如果设置，则存在ADD_SIZE字段，且完整块大小为HEAD_SIZE + ADD_SIZE。

**`块类型：`**

> - `HEAD_TYPE = 0x72` - MARK_HEAD(标记块)
> - `HEAD_TYPE = 0x73` - MAIN_HEAD(压缩文件头)
> - `HEAD_TYPE = 0x74` - FILE_HEAD(文件头)
> - `HEAD_TYPE = 0x75` - COMM_HEAD(旧风格的注释头)
> - `HEAD_TYPE = 0x76` - AV_HEAD(旧风格的授权信息块/用户身份信息块)
> - `HEAD_TYPE = 0x77` - SUB_HEAD(旧风格的子块)
> - `HEAD_TYPE = 0x78` - PROTECT_HEAD(旧风格的恢复记录)
> - `HEAD_TYPE = 0x79` - SIGN_HEAD(旧风格的授权信息块/用户身份信息块)
> - `HEAD_TYPE = 0x7A` - NEWSUB_HEAD(子块)
> - `HEAD_TYPE = 0x7B` - ENDARC_HEAD(结束块)

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-1-1%E3%80%81%E6%A0%87%E8%AE%B0%E5%9D%97-MARK-HEAD "2.1.1、标记块( MARK_HEAD )")2.1.1、标记块( MARK_HEAD )

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|总是0x6152|
|HEAD_TYPE|1|0x72|
|HEAD_FLAGS|2|总是0x1A21|
|HEAD_SIZE|2|块大小 = 0x0007,即7个字节|

  所以这里标记块的大小固定是7 个字节，且是一个固定的字节序列。`标记块`也称为`Magic number`。

**`实例：`**  
[![标记块](https://sp4n9x.github.io/resources/2020/RAR_Format/MARK_HEAD.jpg "标记块")](https://sp4n9x.github.io/resources/2020/RAR_Format/MARK_HEAD.jpg "标记块")

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-1-2%E3%80%81%E5%8E%8B%E7%BC%A9%E6%96%87%E4%BB%B6%E5%A4%B4-MAIN-HEAD "2.1.2、压缩文件头( MAIN_HEAD )")2.1.2、压缩文件头( MAIN_HEAD )

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|HEAD_TYPE到RESERVED2的CRC|
|HEAD_TYPE|1|0x73|
|HEAD_FLAGS|2|位标记：  <br>`0x0001` - MHD_VOLUME: 卷属性(压缩文件卷)  <br>`0x0002` - MHD_COMMENT: 压缩文件注释存在，RAR 3.x使用单独的注释块，不设置这个标记。  <br>`0x0004` - MHD_LOCK: 压缩文件锁定属性  <br>`0x0008` - MHD_SOLID: 固实属性(固实压缩文件)  <br>`0x0010` - MHD_NEWNUMBERING: 新的卷命名法则(‘volname.partN.rar’)  <br>`0x0020` - MHD_AV: 用户身份信息存在，RAR 3.x使用单独的用户身份信息块，不设置这个标记。  <br>`0x0040` - MHD_PROTECT: 恢复记录存在  <br>`0x0080` - HD_PASSWORD: 块头被加密  <br>`0x0100` - MHD_FIRSTVOLUME: 第一卷(只有RAR3.0及以后版本设置)  <br>`0x0200` - MHD_ENCRYPTVER:  <br>HEAD_FLAGS中的其他位保留用于内部使用。|
|HEAD_SIZE|2|压缩文件头总大小（包括压缩文件注释）|
|RESERVED1|2|保留|
|RESERVED2|4|保留|

  对于`压缩文件头`里的`位标记`，如果它的第9位(从左到右)被置1，块头被加密，也就是通常所说的`加密文件名`，打开这样加密的RAR文件时，需要先输入密码才能看到压缩包内的文件列表。

**`实例：`**  
[![压缩文件头](https://sp4n9x.github.io/resources/2020/RAR_Format/MAIN_HEAD.jpg "压缩文件头")](https://sp4n9x.github.io/resources/2020/RAR_Format/MAIN_HEAD.jpg "压缩文件头")

  这里头类型是0x73表示是`压缩文件头块`，位标记为0x0000未有位被置1，如果`块头被加密`则位标记应为0x0080，`文件头大小`为0x000D，所以这个`压缩文件头块`占用13 个字节，保留字节用0x00 填充。

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-1-3%E3%80%81%E6%96%87%E4%BB%B6%E5%A4%B4-FILE-HEAD "2.1.3、文件头( FILE_HEAD )")2.1.3、文件头( FILE_HEAD )

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|从HEAD_TYPE到FILEATTR的CRC|
|HEAD_TYPE|1|0x74|
|HEAD_FLAGS|2|位标记:  <br>`0x0001` - LHD_SPLIT_BEFORE: 文件从上一卷继续  <br>`0x0002` - LHD_SPLIT_AFTER: 文件从后一卷继续  <br>`0x0004` - LHD_PASSWORD: 文件使用密码加密  <br>`0x0008` - LHD_COMMENT: 文件注释存在，RAR 3.x使用独立的注释块，不设置这个标记。  <br>`0x0010` - LHD_SOLID: 前一文件信息被使用(固实标记) (对于RAR 2.0 和以后版本)  <br>  7 6 5 位(对于RAR 2.0和以后版本)  <br>  0 0 0 - 字典大小 64KB  <br>  0 0 1 - 字典大小 128KB  <br>  0 1 0 - 字典大小 256KB  <br>  0 1 1 - 字典大小 512KB  <br>  1 0 0 - 字典大小 1024KB  <br>  1 0 1 - 字典大小 2048KB  <br>  1 1 0 - 字典大小 4096KB  <br>  1 1 1 - 文件作为字典  <br>`0x0100` - LHD_LARGE: HIGH_PACK_SIZE和HIGH_UNP_SIZE结构存在。这些结构仅用在非常大(大于2GB)的文档，对于小文件这些结构不存在。  <br>`0x0200` - LHD_UNICODE: FILE_NAME是用0隔开的包含普通的和Unicode编码的文件名。所以NAME_SIZE字段的值等于普通文件名的长度加Unicode编码文件名的长度再加1。如果此标记存在，但是FILE_NAME不包含0字节，它意味文件名使用UTF-8编码。  <br>`0x0400` - LHD_SALT: 文件头在文件名后包含附加的8byte，它对于增加加密的安全性是必需的。(所谓的’Salt’)。  <br>`0x0800` - LHD_VERSION: 版本标记。它是旧文件版本，版本号作为’;n’附加到文件名后。  <br>`0x1000` - LHD_EXTTIME: 扩展时间区域存在。  <br>`0x2000` - LHD_EXTFLAGS:  <br>`0x8000` - 此位总被设置，所以完整的块的大小是HEAD_SIZE + PACK_SIZE(如果`0x100`位被设置，再加上HIGH_PACK_SIZE)|
|HEAD_SIZE|2|文件头的全部大小(包含文件名和注释)|
|PACK_SIZE|4|已压缩文件大小|
|UNP_SIZE|4|未压缩文件大小|
|HOST_OS|1|保存压缩文件使用的操作系统  <br>`0x00` - MS DOS  <br>`0x01` - OS/2  <br>`0x02` - Win32  <br>`0x03` - Unix  <br>`0x04` - Mac OS  <br>`0x05` - BeOS|
|FILE_CRC|4|文件CRC|
|FTIME|4|MS DOS标准格式的日期和时间|
|UNP_VER|1|解压文件所需要最低RAR版本，版本编码方法：10 * 主版本 + 副版本。|
|METHOD|1|压缩方式：  <br>`0x30` - 存储  <br>`0x31` - 最快压缩  <br>`0x32` - 较快压缩  <br>`0x33` - 标准压缩  <br>`0x34` - 较好压缩  <br>`0x35` - 最好压缩|
|NAME_SIZE|2|文件名大小|
|ATTR|4|文件属性|
|HIGH_PACK_SIZE|4|可选值，`已压缩文件大小`64位值的高4字节。只HEAD_FLAGS中的`0x100`位被设置才存在。|
|HIGH_UNP_SIZE|4|可选值，`未压缩文件大小`64位值的高4字节。只有HEAD_FLAGS中的`0x100`位被设置才存在。|
|FILE_NAME|NAME_SIZE|文件名 - NAME_SIZE字节大小字符串|
|SALT|8|可选值，如果(HEAD_FLAGS & 0x400)!= 0，则存在|
|EXT_TIME|可变大小|可选值，扩展时间区域，如果(HEAD_FLAGS & 0x1000)!= 0，则存在|

**`实例1：`**  
[![文件头1](https://sp4n9x.github.io/resources/2020/RAR_Format/FILE_HEAD.jpg "文件头1")](https://sp4n9x.github.io/resources/2020/RAR_Format/FILE_HEAD.jpg "文件头1")

**`实例2：`**  
[![文件头2](https://sp4n9x.github.io/resources/2020/RAR_Format/FILE_HEAD1.jpg "文件头2")](https://sp4n9x.github.io/resources/2020/RAR_Format/FILE_HEAD1.jpg "文件头2")

  在这个块中，存在`两个CRC值`，一个是文件头块中从`块类型`到`文件名`这38(实例1)或40(实例2)个字节的校验，后一个则是压缩包中所含`文件的CRC校验`，解压时，会计算解压后生成文件的CRC值，如果等于这里的CRC，则解压完成，如果不同，则报错中断。  
  位标记`0x9020`(10010000 00100000B) = 0x0020 || 0x1000 || 0x8000，`0x0020`(字典大小 128KB)、`0x1000`(扩展时间区域存在)、`0x8000`(此位总被设置)。

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-1-4%E3%80%81%E7%BB%93%E5%B0%BE%E5%9D%97-ENDARC-HEAD "2.1.4、结尾块( ENDARC_HEAD )")2.1.4、结尾块( ENDARC_HEAD )

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|从HEAD_TYPE 到HEAD_SIZE 的CRC校验值|
|HEAD_TYPE|1|0x7B|
|HEAD_FLAGS|2|位标记|
|HEAD_SIZE|2|结尾块大小|

  与标记块类似的是，`结尾块`也是一个固定字节串的块，依次是`C4 3D 7B 00 40 07 00`。

**`实例：`**  
[![结尾块](https://sp4n9x.github.io/resources/2020/RAR_Format/END_HEAD.jpg "结尾块")](https://sp4n9x.github.io/resources/2020/RAR_Format/END_HEAD.jpg "结尾块")

  `前一个文件块起始位置偏移`为81，而`前一个文件块的大小`是HEAD_SIZE + PACK_SIZE(0x002F + 0x153A = 0x1569) = 5481byte。所以`结尾块的偏移`为5481 + 81 = 5562。

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-1-5%E3%80%81%E6%97%A7%E9%A3%8E%E6%A0%BC%E7%9A%84%E5%9D%97%E7%B1%BB%E5%9E%8B "2.1.5、旧风格的块类型")2.1.5、旧风格的块类型

   除以上格式块以外，还存在一些旧风格的块类型，不过在新的版本中已经不存在了。

**`旧风格的注释头块：`**

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|从HEAD_TYPE 到COMM_CRC 的CRC校验值|
|HEAD_TYPE|1|0x75|
|HEAD_FLAGS|2|位标记|
|HEAD_SIZE|2|注释头大小|
|UNP_SIZE|2|未压缩注释大小|
|UNP_VER|1|提取注释的RAR最低版本|
|METHOD|1|压缩方法|
|COMM_CRC|2|注释CRC|
|COMMENT|N|注释正文|

**`旧风格的授权信息块/用户身份信息块：`**

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|从HEAD_TYPE 到HEAD_SIZE的CRC校验值|
|HEAD_TYPE|1|0x76|
|HEAD_FLAGS|2|位标记|
|HEAD_SIZE|2|块大小|
|INFO|N|授权信息/用户身份信息正文|

**`旧风格的子块：`**  
   在压缩文件中`任意文件头块`后面都可以附加一个`子块`。这个子块依赖于它前面的这个`主块`。当更新时新版本的RAR压缩包可能会删除或者移动这个子块。

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|块CRC|
|HEAD_TYPE|1|0x77|
|HEAD_FLAGS|2|位标记|
|HEAD_SIZE|2|总块大小|
|DATA_SIZE|4|总数据块大小|
|SUB_TYPE|2|子块类型|
|RESERVED|1|保留字段，必须为0|
|其余字段||由SUB_TYPE 决定其余字段类型|

   以`SUB_TYPE`为0x100为例，0x100定义子块类型为`扩展属性类型`，一般用于压缩一些文件属性信息较详细的文件。

|字段名称|长度(byte)|说明|
|---|---|---|
|HEAD_CRC|2|块CRC|
|HEAD_TYPE|1|0x77|
|HEAD_FLAGS|2|位标记|
|HEAD_SIZE|2|总块大小|
|DATA_SIZE|4|总数据块大小|
|SUB_TYPE|2|子块类型，定义子块为扩展属性类型|
|RESERVED|1|保留字段，必须为0。以上为子块中固定格式|
|UNP_SIZE|4|未压缩扩展属性大小。以下为扩展属性附加字段。|
|UNP_VER|1|提取扩展属性的RAR最低版本|
|METHOD|1|压缩方式|
|EA_CRC|4|扩展属性CRC|

### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2%E3%80%81RAR-5-%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90 "2.2、RAR 5+文件格式分析")2.2、RAR 5+文件格式分析

> **`参考资料：`**  
> 1、[https://www.rarlab.com/technote.htm](https://www.rarlab.com/technote.htm)

  Here we describe basic data structures of archive format introduced in RAR 5.0. If you need information about algorithms or more detailed information on data structures, please use UnRAR source code.  
  在这里，我们描述了`RAR 5.0`中引入的存档格式的`基本数据结构`。如果您需要有关算法的信息或有关数据结构的更多详细信息，请使用`UnRAR`源代码。

**`实例环境：`**

[![例子RAR文件环境0](https://sp4n9x.github.io/resources/2020/RAR_Format/env0.jpg "例子RAR文件环境0")](https://sp4n9x.github.io/resources/2020/RAR_Format/env0.jpg "例子RAR文件环境0")[![例子RAR文件环境1](https://sp4n9x.github.io/resources/2020/RAR_Format/env5.jpg "例子RAR文件环境1")](https://sp4n9x.github.io/resources/2020/RAR_Format/env5.jpg "例子RAR文件环境1")

  

[![例子RAR文件环境2](https://sp4n9x.github.io/resources/2020/RAR_Format/env1.jpg "例子RAR文件环境2")](https://sp4n9x.github.io/resources/2020/RAR_Format/env1.jpg "例子RAR文件环境2")[![例子RAR文件环境3](https://sp4n9x.github.io/resources/2020/RAR_Format/env2.jpg "例子RAR文件环境3")](https://sp4n9x.github.io/resources/2020/RAR_Format/env2.jpg "例子RAR文件环境3")

  

[![例子RAR文件环境4](https://sp4n9x.github.io/resources/2020/RAR_Format/env3.jpg "例子RAR文件环境4")](https://sp4n9x.github.io/resources/2020/RAR_Format/env3.jpg "例子RAR文件环境4")

  

[![例子RAR文件环境5](https://sp4n9x.github.io/resources/2020/RAR_Format/env4.jpg "例子RAR文件环境5")](https://sp4n9x.github.io/resources/2020/RAR_Format/env4.jpg "例子RAR文件环境5")

设置的密码`未加密文件名`。

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-1%E3%80%81Data-types-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B "2.2.1、Data types(数据类型)")2.2.1、Data types(数据类型)

**`vint`**  
  Variable length integer. Can include one or more bytes, where lower 7 bits of every byte contain integer data and highest bit in every byte is the continuation flag. If highest bit is 0, this is the last byte in sequence. So first byte contains 7 least significant bits of integer and continuation flag. Second byte, if present, contains next 7 bits and so on.  
  `可变长度`整数。可以包含`一个`或`多个`字节，其中每个字节的`低7位`包含整数数据，而每个字节中的`最高位`是连续标志。如果`最高位为0`，则这是序列中的最后一个字节。因此，第一个字节包含`低位7个有效位的整数`和`连续标志`。第二个字节（如果存在）包含下一个7位，依此类推。  
  Currently RAR format uses vint to store up to 64 bit integers, resulting in 10 bytes maximum. This value may be increased in the future if necessary for some reason.  
  当前的RAR格式使用`vint`最多存储`64位`整数，所以`vint`最大可以包含`10个字节`。如果出于某种原因，将来可能会增加此值。  
  Sometimes RAR needs to pre-allocate space for vint before knowing its exact value. In such situation it can allocate more space than really necessary and then fill several leading bytes with 0x80 hexadecimal, which means 0 with continuation flag set.  
  有时，RAR需要在知道确切值之前为`vint`预先分配空间。在这种情况下，它可以分配`超出`实际需要的空间，然后用`0x80`十六进制填充几个前导字节，这意味着设置了`连续标志`。

**`byte`**, **`uint16`**, **`uint32`**, **`uint64`**  
Byte, 16-, 32-, 64- bit unsigned integer in little endian format.  
`Byte`，`16bit`、`32bit`、`64bit`的无符号整型使用的都是`小端存储格式`。

**`Variable length data(可变长度的数据)`**  
We use ellipsis … to denote variable length data areas.  
我们使用`省略号(...)`表示`可变长度`的数据区域。

**`Hexadecimal values(十六进制值)`**  
We use 0x prefix to define hexadecimal values, such as 0xf000.  
我们使用0x前缀定义十六进制值，例如`0xf000`。

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-2%E3%80%81General-archive-structure-%E9%80%9A%E7%94%A8%E5%8E%8B%E7%BC%A9%E6%96%87%E6%A1%A3%E7%BB%93%E6%9E%84 "2.2.2、General archive structure(通用压缩文档结构)")2.2.2、General archive structure(通用压缩文档结构)

**`General archive block format(通用压缩文档块格式)`**

|字段名称|大小(类型)|说明|
|---|---|---|
|Header CRC32|uint32|头数据的CRC32，从“`Header size`”字段开始，直至“`Extra area`”字段。|
|Header size|vint|头数据的大小，从“`Header type`”字段开始，直至“`Extra area`”字段。在当前的实现中，该字段不得超过`3个字节`，从而导致最大的头大小为`2MB`。|
|Header type|vint|压缩文档头的类型。可能的值为：  <br>  `0x01` - 压缩文档头。  <br>  `0x02` - 文件头。  <br>  `0x03` - 服务头。  <br>  `0x04` - 压缩文档加密头。  <br>  `0x05` - 结尾块。|
|Header flags|vint|所有头共有的标志：  <br>`0x01` - 头末尾存在扩展区域。  <br>`0x02` - 头末尾存在数据区域。  <br>`0x04` - 当更新压缩文档时，必须跳过类型未知并拥有该标志的块。  <br>`0x08` - 数据区域从上一卷继续。  <br>`0x10` - 数据区域从下一卷继续。  <br>`0x20` - 块取决于前一个文件块。  <br>`0x40` - 如果修改了主块，则保留一个子块。|
|Extra area size|vint|`可选字段`，扩展区域的大小。仅当设置了`0x01`头标志时才存在。|
|Data area size|vint|`可选字段`，数据区域的大小。仅当设置了`0x02`头标志时才存在。|
|…|…|当前块类型专用的字段。有关详细信息，请参见具体块类型描述。|
|Extra area|…|`可选字段`，扩展区域。仅当设置了`0x01`头标志时才存在。|
|Data area|vint|`可选字段`，数据区域。仅当设置了`0x02`头标志时才存在。用于存储大量数据，例如压缩文件数据。不包括“`Header CRC`”和“`Header size`”字段。|

**`General extra area format(通用扩展区域格式)`**  
Extra area can include one or more records having the following format:  
扩展区域可以包含一个或多个具有以下格式的记录：

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`记录类型`。不同的压缩文档块具有不同的关联的扩展区域记录类型。阅读具体的压缩文档块描述以获取详细信息。将来可以添加新的记录类型，因此需要跳过未知的记录类型而不中断操作。|
|Data|…|`记录相关数据`。如果记录仅由Size和Type组成，则可能会丢失。|

**`General archive layout(通用压缩文档布局)`**  

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14|Self-extracting module(optional)：自解压模块(可选的)  <br>RAR 5.0 signature：RAR5.0签名  <br>Archive encryption header(optional)：压缩文档加密头(可选的)  <br>Main archive header：压缩文档头  <br>Archive comment service header(optional)：压缩文档注释服务头(可选的)  <br>  <br>File header 1：文件头1  <br>Service headers(NTFS ACL, streams, etc.)for preceding file(optional)：前一个文件的服务头(可选的)  <br>...  <br>File header N：文件头N  <br>Service headers(NTFS ACL, streams, etc.)for preceding file(optional)：前一个文件的服务头(可选的)  <br>  <br>Recovery record(optional)：恢复记录(可选的)  <br>End of archive header：结尾块|

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3%E3%80%81Archive-blocks-%E5%8E%8B%E7%BC%A9%E6%96%87%E6%A1%A3%E5%9D%97 "2.2.3、Archive blocks(压缩文档块)")2.2.3、Archive blocks(压缩文档块)

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3-1%E3%80%81Self-extracting-module-SFX-%E8%87%AA%E8%A7%A3%E5%8E%8B%E6%A8%A1%E5%9D%97 "2.2.3.1、Self-extracting module (SFX)(自解压模块)")2.2.3.1、**`Self-extracting module (SFX)(自解压模块)`**

  Any data preceding the archive signature. Self-extracting module size and contents is not defined. At the moment of writing this documentation RAR assumes the maximum SFX module size to not exceed 1 MB, but this value can be increased in the future.  
  `压缩文档签名`之前的所有数据。`自解压模块`的大小和内容未定义。在撰写本文档时，RAR假定`SFX模块`的最大大小不超过1 MB，但是以后可以增加该值。

**`实例：`**  
[![自解压模块头](https://sp4n9x.github.io/resources/2020/RAR_Format/SFX1.jpg "自解压模块头")](https://sp4n9x.github.io/resources/2020/RAR_Format/SFX1.jpg "自解压模块头")  
[![自解压模块尾](https://sp4n9x.github.io/resources/2020/RAR_Format/SFX2.jpg "自解压模块尾")](https://sp4n9x.github.io/resources/2020/RAR_Format/SFX2.jpg "自解压模块尾")

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3-2%E3%80%81RAR-5-0-signature-RAR5-0%E7%AD%BE%E5%90%8D "2.2.3.2、RAR 5.0 signature(RAR5.0签名)")2.2.3.2、**`RAR 5.0 signature(RAR5.0签名)`**

  RAR 5.0 signature consists of 8 bytes: 0x52 0x61 0x72 0x21 0x1A 0x07 0x01 0x00. You need to search for this signature in supposed archive from beginning and up to maximum SFX module size. Just for comparison this is RAR 4.x 7 byte length signature: 0x52 0x61 0x72 0x21 0x1A 0x07 0x00.  
  `RAR 5.0签名`由8个字节组成：`0x52 0x61 0x72 0x21 0x1A 0x07 0x01 0x00`。您需要从假想的压缩文档中搜索该签名，从开始到最大SFX模块大小。只是为了比较，这是`RAR 4.x` 7字节长度的签名：`0x52 0x61 0x72 0x21 0x1A 0x07 0x00`。

**`实例：`**  
[![RAR 5.0签名](https://sp4n9x.github.io/resources/2020/RAR_Format/RAR%205.0%20signature.jpg "RAR 5.0签名")](https://sp4n9x.github.io/resources/2020/RAR_Format/RAR%205.0%20signature.jpg "RAR 5.0签名")

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3-3%E3%80%81Archive-encryption-header-%E5%8E%8B%E7%BC%A9%E6%96%87%E6%A1%A3%E5%8A%A0%E5%AF%86%E5%9D%97 "2.2.3.3、Archive encryption header(压缩文档加密块)")2.2.3.3、**`Archive encryption header(压缩文档加密块)`**

|字段名称|大小(类型)|说明|
|---|---|---|
|Header CRC32|uint32|头数据的CRC32，从“`Header size`”字段开始，直至“`Check value`”字段的CRC32校验码。|
|Header size|vint|头数据的大小，从“`Header type`”字段开始，直至“`Check value`”字段。|
|Header type|vint|`0x04`|
|Header flags|vint|所有头共有的标志：  <br>`0x01` - 头末尾存在扩展区域。  <br>`0x02` - 头末尾存在数据区域。  <br>`0x04` - 当更新压缩文档时，必须跳过类型未知并拥有该标志的块。  <br>`0x08` - 数据区域从上一卷继续。  <br>`0x10` - 数据区域从下一卷继续。  <br>`0x20` - 块取决于前一个文件块。  <br>`0x40` - 如果修改了主块，则保留一个子块。|
|Encryption version|vint|`加密算法的版本`。现在仅支持0版本(AES-256)。|
|Encryption flags|vint|`0x00` - 不存在密码检查数据(Check value字段)。  <br>`0x01` - 存在密码检查数据(Check value字段)。|
|KDF count|1 byte|`PBKDF2函数`的`迭代数`的`二进制对数`。 RAR可以拒绝处理`超过`某个阈值的KDF计数。阈值的具体值取决于`版本`。|
|Salt|16 byte|全局地用于`所有加密`的压缩文档头的盐值。|
|Check value|12 byte|用于`验证密码有效性`的值。仅当设置了`0x01`加密标志时才存在。`前8个字节`是使用额外的PBKDF2轮次计算的，`最后4个字节`是额外的校验和。与标准`Header CRC32`一起，我们具有`64位校验和`，以可靠地`验证此字段的完整性`并区分`无效的密码`和`损坏的数据`。可以在UnRAR源代码中找到更多详细信息。|

  This header is present only in archives with encrypted headers. Every next header after this one is started from 16 byte AES-256 initialization vector followed by encrypted header data. Size of encrypted header data block is aligned to 16 byte boundary.  
  该头仅存在于`带有加密头`的压缩文档中。此后的`每下一个头`均从16字节`AES-256初始化向量`开始，然后是`加密的头数据`。加密头数据块的大小与16字节边界对齐。

**`实例：`**  
这是设置密码时勾选了`加密文件名`后，出现的块。  
[![压缩文档加密块](https://sp4n9x.github.io/resources/2020/RAR_Format/Archive%20encryption%20header.jpg "压缩文档加密块")](https://sp4n9x.github.io/resources/2020/RAR_Format/Archive%20encryption%20header.jpg "压缩文档加密块")

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15|00000000 - 00310783 SFX(自解压模块)  <br>  <br>00310784 - 00310791  <br>52 61 72 21 1A 07 01 00  <br>  <br>00310792 - 00310829  <br>49 23 54 91 - Header CRC32( Header size -> Check value )  <br>21 - Header size( Header type -> Check value ) 21H = 33D  <br>04 - Header type( 压缩文档加密块 )  <br>00 - Header flags  <br>00 - Encryption version  <br>01 - Encryption flags  <br>0F - KDF count  <br>89 54 B1 34 4A 94 3D 09 FA 1A CB 8A C8 A9 F3 A3 - Salt  <br>21 43 42 86 61 24 8A 16 E4 1B C8 8E - Check value|

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3-4%E3%80%81Main-archive-header-%E5%8E%8B%E7%BC%A9%E6%96%87%E4%BB%B6%E5%A4%B4 "2.2.3.4、Main archive header(压缩文件头)")2.2.3.4、**`Main archive header(压缩文件头)`**

|字段名称|大小(类型)|说明|
|---|---|---|
|Header CRC32|uint32|头数据的CRC32，从“`Header size`”字段开始，直至“`Extra area`”字段。|
|Header size|vint|头数据的大小，从“`Header type`”字段开始，直至“`Extra area`”字段。|
|Header type|vint|`0x01`|
|Header flags|vint|所有头共有的标志：  <br>`0x01` - 头末尾存在扩展区域。  <br>`0x02` - 头末尾存在数据区域。  <br>`0x04` - 当更新压缩文档时，必须跳过类型未知并拥有该标志的块。  <br>`0x08` - 数据区域从上一卷继续。  <br>`0x10` - 数据区域从下一卷继续。  <br>`0x20` - 块取决于前一个文件块。  <br>`0x40` - 如果修改了主块，则保留一个子块。|
|Extra area size|vint|`可选字段`，扩展区域的大小。仅当设置了`0x01`头标志时才存在。|
|Archive flags|vint|`0x01` - 卷。压缩文档是多卷集的一部分。  <br>`0x02` - “Volume number”字段存在。除第一卷外，所有的卷都存在此标志。  <br>`0x04` - 可靠的压缩文档。  <br>`0x08` - 存在恢复记录。  <br>`0x10` - 压缩文档已锁定。|
|Volume number|vint|`可选字段`，仅当设置了Archive flags的`0x02`位时才显示。第一卷不存在，第二卷为1，第三卷为2，依此类推。|
|Extra area|…|`可选字段`，扩展区域。仅当设置了`0x01`头标志时才存在。|

Extra area of main archive header can contain following record types：  
`压缩文件头`的`扩展区域`可以包含以下记录类型：

|类型编号|类型名称|说明|
|---|---|---|
|0x01|Locator|包含`不同服务块`的位置，因此可以`快速访问`它们，而无需扫描整个压缩文档。该记录是`可选的`。如果缺少它，仍然有必要扫描`整个压缩文件`以验证服务块的存在。|

**`Locator record(定位记录)`**

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x01`|
|Flags|vint|`0x01` - “Quick open record offset”字段存在。  <br>`0x02` - “Recovery record offset”字段存在。|
|Quick open offset|vint|`可选字段`，从`“Quick open”服务块的开始`到`Main archive header的开始`的距离。仅当设置了`0x01`标志时才存在。如果等于0，则应忽略。如果预分配的空间不足以存储产生的偏移量，则可以将其设置为零。|
|Recovery record offset|vint|`可选字段`，从`“Recovery record”服务块的开始`到`Main archive header的开始`的距离。仅当设置了`0x02`标志时才存在。如果等于0，则应忽略。如果预分配的空间不足以存储产生的偏移量，则可以将其设置为零。|

**`实例：`**  
[![压缩文件头](https://sp4n9x.github.io/resources/2020/RAR_Format/Main%20archive%20header.jpg "压缩文件头")](https://sp4n9x.github.io/resources/2020/RAR_Format/Main%20archive%20header.jpg "压缩文件头")

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26|00310792 - 00310811  <br>49 64 D1 32 - Header CRC32( Header size -> Extra area )  <br>0F - Header size( Header type -> Extra area ) 0FH = 15D  <br>01 - Header type( 压缩文件头 )  <br>05 - Header flags( 0x05 = 0x01 + 0x04 )  <br>0B - Extra area size( 0BH = 11D )  <br>18 - Archive flags( 0x18 = 0x10 + 0x08 )  <br>0A - Size[ Extra area ] 0AH = 10D  <br>01 - Type( Locator record )  <br>03 - Flags( 0x03 = 0x01 + 0x02 )  <br>A3 C7 81 00 - Quick open offset  <br>DB C8 81 00 - Recovery record offset  <br>---------------------------------------------------------------  <br>Note: vint类型的数据，如果是多字节的，前几个字节的最高位为连续标志  <br>(设为1)，低7位为数据，最后一个字节的最高位为0，低7位依旧是数据。所  <br>以我们需要提取出数据，进行重新组合，才能得到“Quick open header”  <br>和“Recovery record”真实偏移。  <br>1、Quick open offset  <br>0x0081C7A3( 00000000 10000001 ‭11000111‬ ‭10100011 ‬)  <br>0000001 1000111‬ 0100011‬B = 01100011 1‬0100011‬B = 63A3H = ‭25507‬D  <br>310792 + 25507 = 336299  <br>2、Recovery record offset  <br>0x0081C8DB( ‭00000000 10000001 11001000 11011011 ‬)  <br>0000001 1001000 1011011‬B = 01100100 01011011‬B = 645BH = ‭25691‬D  <br>310792 + ‭25691‬ = ‭336483‬  <br>---------------------------------------------------------------|

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3-5%E3%80%81File-header-and-service-header-%E6%96%87%E4%BB%B6%E5%A4%B4%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%A4%B4 "2.2.3.5、File header and service header(文件头和服务头)")2.2.3.5、**`File header and service header(文件头和服务头)`**

  These two header types use the similar data structure, so we describe them both here.  
  这两种头类型使用相似的数据结构，因此我们在这里对它们一起进行描述。

|字段名称|大小(类型)|说明|
|---|---|---|
|Header CRC32|uint32|头数据的CRC32，从“`Header size`”字段开始，直至“`Name`”字段。|
|Header size|vint|头数据的大小，从“`Header type`”字段开始，直至“`Name`”字段。|
|Header type|vint|`0x02` - 文件头  <br>`0x03` - 服务头|
|Header flags|vint|所有头共有的标志：  <br>`0x01` - 头末尾存在扩展区域。  <br>`0x02` - 头末尾存在数据区域。  <br>`0x04` - 当更新压缩文档时，必须跳过类型未知并拥有该标志的块。  <br>`0x08` - 数据区域从上一卷继续。  <br>`0x10` - 数据区域从下一卷继续。  <br>`0x20` - 块取决于前一个文件块。  <br>`0x40` - 如果修改了主块，则保留一个子块。|
|Extra area size|vint|`可选字段`，扩展区域的大小。仅当设置了`0x01`头标志时才存在。|
|Data area size|vint|`可选字段`，数据区域的大小。仅当设置了`0x02`头标志时才存在。|
|File flags|vint|这些头类型专用的标志：  <br>  `0x01` - 目录文件系统对象(文件头)。  <br>  `0x02` - 存在Unix格式的时间字段。  <br>  `0x04` - 存在CRC32字段。  <br>  `0x08` - 解压后大小未知。  <br>如果设置了标志`0x08`，则“`Unpacked size`”字段仍然存在，但是必须忽略，并且必须执行提取，直到到达压缩流的末尾。如果实际文件大小大于操作系统报告的大小，或者文件大小未知（例如，除从stdin到multivolume archive归档时的最后一个卷以外的所有卷），则可以设置此标志。|
|Unpacked size|vint|解压缩的`文件`或`服务数据`大小。|
|Attributes|vint|如果是`文件头`，则为操作系统特定的文件属性。如果是`服务头`，可以用于特定数据需求，也可以保留，并可以设置为0。|
|mtime|uint32|`可选字段`，Unix时间格式的`文件修改时间`。如果设置了`0x02`文件标志，则存在。|
|Data CRC32|uint32|`可选字段`，已解压缩的`文件`或`服务数据`的CRC32。对于在卷之间分割的文件，它包含当前卷中除最后文件部分以外的所有文件部分所包含的文件打包数据的CRC32。如果设置了`0x04`文件标志，则存在。|
|Compression information|vint|`低6位(0x003f)`：压缩算法的版本，可能有0-63个值。当前版本是0。  <br>`第7位(0x0040)`：定义solid flag。如果已设置，RAR将在处理之前的文件后继续使用剩下的压缩字典。只能为文件头设置，而不能为服务头设置。  <br>`第8-10位(0x0380)`：定义压缩方法。当前仅使用值0-5。  <br>  10 9 8 位  <br>  `0 0 0` - 存储  <br>  `0 0 1` - 最快压缩  <br>  `0 1 0` - 较快压缩  <br>  `0 1 1` - 标准压缩  <br>  `1 0 0` - 较好压缩  <br>  `1 0 1` - 最好压缩  <br>`第11-14位(0x3c00)`：定义提取数据所需的字典大小的最小大小。  <br>  14 13 12 11 位  <br>  `0 0 0 0` - 字典大小 128KB  <br>  `0 0 0 1` - 字典大小 256KB  <br>  `0 0 1 0` - 字典大小 512KB  <br>  `0 0 1 1` - 字典大小 1MB  <br>  `0 1 0 0` - 字典大小 2MB  <br>  `0 1 0 1` - 字典大小 4MB  <br>  `0 1 1 0` - 字典大小 8MB  <br>  `0 1 1 1` - 字典大小 16MB  <br>  `1 0 0 0` - 字典大小 32MB  <br>  `1 0 0 1` - 字典大小 64MB  <br>  `1 0 1 0` - 字典大小 128MB  <br>  `1 0 1 1` - 字典大小 256MB  <br>  `1 1 0 0` - 字典大小 512MB  <br>  `1 1 0 1` - 字典大小 1024MB  <br>  `1 1 1 0` - 字典大小 2048MB  <br>  `1 1 1 1` - 字典大小 4096MB|
|Host OS|vint|用于创建压缩文档的操作系统的类型：  <br>`0x00` - Windows  <br>`0x01` - Unix|
|Name length|vint|`文件`或`服务头`名称的长度。|
|Name|N bytes|1、可变长度字段，包含UTF-8格式的名称，不以0结尾。  <br>2、对于`文件头`，这是已压缩文件的名称。正斜杠字符用作Unix和Windows名称的路径分隔符。对于Unix文件名称，反斜杠被视为名称的一部分，对于Windows文件名称，反斜杠被视为无效字符。名称类型由“Host OS”字段定义。  <br>3、如果Unix文件名包含无法正确转换为Unicode和UTF-8的任何高级ASCII字符，我们会将这些字符映射到0xE080-0xE0FF私有Unicode区域，并在结果字符串中插入Unicode非字符0xFFFE以表明它包含已映射字符，提取时需要转换回去。没有定义0xFFFE的具体位置，我们需要搜索整个字符串。此类映射的名称不可移植，并且只能在创建它们的同一系统上正确解压缩。  <br>4、对于`服务头`，此字段包含服务头的名称。现在使用以下名称：  <br>`CMT` - 压缩文档注释  <br>`QO` - 压缩文档Quick open数据  <br>`ACL` - NTFS文件权限  <br>`STM` - NTFS交换数据流  <br>`RR` - 恢复记录|
|Extra area|…|`可选字段`，扩展区域，包含附加的头字段。仅当设置了`0x01`头标志时才存在。|
|Data area|vint|`可选字段`，数据区域，仅当设置了`0x02`头标志时才存在。如果是`文件头`，则存储文件数据；对于`服务头`，则存储服务数据。根据压缩方法中的值，可以对压缩信息进行未压缩（压缩方法0）或压缩。|

File and service headers use the same types of extra area records:  
`文件头`和`服务头`使用相同类型的扩展区域记录类型：

|类型|类型名称|说明|
|---|---|---|
|0x01|File encryption|文件加密信息。|
|0x02|File hash|文件数据哈希。|
|0x03|File time|高精度文件时间。|
|0x04|File version|文件版本号。|
|0x05|Redirection|文件系统重定向。|
|0x06|Unix owner|Unix所有者和组信息。|
|0x07|Service data|服务头数据数组。|

**`File encryption record(文件加密记录)`**  
This record is present if file data is encrypted.  
如果`文件数据被加密`，则存在该记录。

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x01`|
|Version|vint|加密算法的版本。现在仅支持`0版本`(AES-256)。|
|Flags|vint|`0x01` - 存在密码验证数据。  <br>`0x02` - 使用调整过的校验和而不是普通校验和。  <br>如果存在标志`0x02`，则RAR转换保留的校验和文件或服务数据的完整性，因此它取决于加密密钥。它使得不可能根据校验和猜测文件内容。它会影响文件头中的数据CRC32和扩展区域中文件哈希记录中的校验和。|
|KDF count|1 byte|`PBKDF2函数`的迭代数的`二进制对数`。 RAR可以拒绝处理超过`某个阈值`的KDF计数。阈值的具体值取决于`版本`。|
|Salt|16 byte|设置加密文件`解密密钥`的盐值。|
|IV|16 byte|AES-256`初始化向量`。|
|Check value|12 byte|`可选字段`，用于验证密码有效性的值。仅当设置了`0x01`加密标志时才存在。`前8个字节`是使用额外的PBKDF2轮次计算的，`最后4个字节`是额外的校验和。与标准`Header CRC32`一起，我们具有`64位校验和`，以可靠地`验证此字段的完整性`并区分`无效的密码`和`损坏的数据`。可以在UnRAR源代码中找到更多详细信息。|

**`File hash record(文件哈希记录)`**  
  Only the standard CRC32 checksum can be stored directly in file header. If other hash is used, it is stored in this extra area record:  
  仅`标准CRC32校验和`可以直接存储在`文件头`中。如果使用`其他哈希`，它将存储在此`扩展区域记录`中：

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x02`|
|Hash type|vint|`0x00` - BLAKE2sp哈希函数。|
|Hash data|N bytes|`0x00哈希类型`的32个字节的BLAKE2sp哈希值。|

  For files split between volumes it contains a hash of file packed data contained in current volume for all file parts except the last. For files not split between volumes and for last parts of split files it contains an unpacked data hash.  
  对于`在卷之间分割`的文件，它包含当前卷中除最后文件部分以外的所有文件部分所包含的文件压缩数据的哈希。对于`未在卷之间拆分`的文件以及拆分文件的最后部分，它包含未压缩的数据哈希。

**`File time record(文件时间记录)`**  
  This record is used if it is necessary to store creation and last access time or if 1 second precision of Unix mtime stored in file header is not enough:  
  如果需要存储`创建`和`上次访问`时间，或者`文件头`中存储的Unix mtime的1秒`精度不够`，则使用此记录：

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x03`|
|Flags|vint|`0x01` - 如果设置了此标志，则时间以Unix time_t格式存储，否则以Windows FILETIME格式存储。  <br>`0x02` - 存在“mtime”字段。  <br>`0x04` - 存在“ctime”字段。  <br>`0x08` - 存在“atime”字段。  <br>`0x10` - Unix时间格式，具有纳秒级精度。|
|mtime|uint32 or uint64|`修改时间`。如果设置了`0x02`标志，则存在。根据`0x01`，标志可以采用Unix time_t或Windows FILETIME格式。|
|ctime|uint32 or uint64|`创建时间`。如果设置了`0x04`标志，则存在。根据`0x01`，标志可以采用Unix time_t或Windows FILETIME格式。|
|atime|uint32 or uint64|`上次访问时间`。如果设置了`0x08`标志，则存在。根据`0x01`，标志可以采用Unix time_t或Windows FILETIME格式。|
|mtime nanoseconds|uint32|添加到`mtime`的纳秒值。如果`0x01`、`0x02`和`0x10`标志都设置了，则存在。|
|ctime nanoseconds|uint32|添加到`ctime`的纳秒值。如果`0x01`、`0x04`和`0x10`标志都设置了，则存在。|
|atime nanoseconds|uint32|添加到`atime`的纳秒值。如果`0x01`、`0x08`和`0x10`标志都设置了，则存在。|

**`File version record(文件版本记录)`**  
This record is used in archives created with -ver switch.  
该记录用于通过`-ver开关`创建的压缩文档中。

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x04`|
|Flags|vint|尚未定义文件版本标志，因此将其设置为0。|
|Version number|vint|文件版本号。|

**`File system redirection record(文件系统重定向记录)`**

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x05`|
|Redirection type|vint|`0x01` - Unix符号链接  <br>`0x02` - Windows符号链接  <br>`0x03` - Windows交接点  <br>`0x04` - 硬链接  <br>`0x05` - 文件复制|
|Flags|vint|`0x01` - 链接目标为目录|
|Name length|vint|链接目标名称的长度|
|Name|vint|`UTF-8`格式的链接目标名称，不以0结尾|

**`Unix owner record(Unix所有者记录)`**

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x06`|
|Flags|vint|`0x01` - “User name”符串存在  <br>`0x02` - “Group name”字符串存在  <br>`0x04` - 存在数字“User ID”  <br>`0x08` - 存在数字“Group ID”|
|User name length|vint|`可选字段`，所有者用户名的长度。如果设置了`0x01`标志，则存在。|
|User name|N bytes|`可选字段`，所有者用户名（采用本机编码）。非零终止。如果设置了`0x01`标志，则存在。|
|Group name length|vint|`可选字段`，所有者组名称的长度。如果设置了`0x02`标志，则存在。|
|Group name|N bytes|`可选字段`，所有者组名称（采用本机编码）。非零终止。如果设置了`0x02`标志，则存在。|
|User ID|vint|`可选字段`，数字所有者用户ID。如果设置了`0x04`标志，则存在。|
|Group ID|vint|`可选字段`，数字所有者组ID。如果设置了`0x08`标志，则存在。|

**`Service data record(服务数据记录)`**  
This record is used only by service headers to store additional parameters.  
该记录仅由`服务头`使用，以存储其他参数。

|字段名称|大小(类型)|说明|
|---|---|---|
|Size|vint|从Type开始的记录数据的大小。|
|Type|vint|`0x07`|
|Data|N bytes|服务数据的具体内容取决于`服务头类型`。|

**`实例：`**  
[![文件头](https://sp4n9x.github.io/resources/2020/RAR_Format/File%20header.jpg "文件头")](https://sp4n9x.github.io/resources/2020/RAR_Format/File%20header.jpg "文件头")

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31  <br>32  <br>33  <br>34  <br>35  <br>36  <br>37  <br>38  <br>39  <br>40  <br>41  <br>42  <br>43  <br>44  <br>45  <br>46  <br>47  <br>48  <br>49  <br>50  <br>51  <br>52  <br>53  <br>54  <br>55  <br>56  <br>57  <br>58  <br>59  <br>60  <br>61  <br>62  <br>63  <br>64  <br>65|00310850 - 00311002  <br>FF A6 73 44 - Header CRC32( Header size -> Extra area )  <br>93 01 - Header size( Header type -> Extra area )  <br>02 - Header type( 文件头 )  <br>03 - Header flags( 0x03 = 0x01 + 0x02 )  <br>6F - Extra area size[ 0x6F = (0x30+0x01) + (0x22+0x01) + (0x1A+0x01) ]  <br>D0 C5 01 - Data area size  <br>00 - File flags                       ‭    <br>B3 E9 - Unpacked size   <br>01 20 - Attributes      <br>80 03 - Compression information  <br>00 - Host OS  <br>15 - Name length( 15H = 21D )  <br>32 30 32 30 30 32 30 35 31 31 34 36 31 34 39 34 35 2E 70 6E 67 - Name  <br>30 - Size( Extra area ) 30H = 48D  <br>01 - Type( 文件加密记录 )  <br>00 - Version  <br>03 - Flags( 0x03 = 0x02 + 0x01 )  <br>0F - KDF count  <br>48 2F 76 C7 97 E7 8A 77 D5 55 F8 1F 5F 31 D8 26 - Salt  <br>5C 4C 4F B0 4F 94 01 90 95 95 2E E9 E6 BF 8F 75 - IV  <br>4D 87 A2 57 6C EA BA B2 EB D6 4E A8 - Check value  <br>22 - Size( Extra area ) 22H = 34D  <br>02 - Type( 文件哈希记录 )  <br>00 - Hash type  <br>16 87 E7 C4 D1 8C 13 AA BD B5 18 4A A9 C7 3E D1 85 85 95 B4 6F 12 3E 55 10 C6 86 7C 5A 2E 80 B9 - Hash data  <br>1A - Size( Extra area ) 1AH = 26D  <br>03 - Type( 文件时间记录 )  <br>0E - Flags( 0x0E = 0x02 + 0x04 + 0x08 )  <br>8C E5 A3 8C 2A 01 D6 01 - mtime  <br>7D A3 8E 8E 2A 01 D6 01 - ctime  <br>87 27 7A 12 2D 2D D6 01 - atime                             <br>Data area: 00311003 - 00336298  25296B(压缩后大小)  <br>----------------------------------------------------------------------  <br>Note: vint类型的数据，如果是多字节的，前几个字节的最高位为连续标志  <br>(设为1)，低7位为数据，最后一个字节的最高位为0，低7位依旧是数据。所  <br>以我们需要提取出数据，进行重新组合，才能得到“Quick open header”  <br>和“Recovery record”真实偏移。  <br>1、Header size  <br>0x0193( 00000001 10010011 )  <br>0000001 0010011B = 10010011B = 93H = 147D  <br>2、Data area size  <br>0x01C5D0( 00000001 ‭11000101 11010000‬ )  <br>0000001 1000101 1010000‬B = 01100010 11010000‬B = 62D0H = 25296D  <br>3、Unpacked size  <br>0xE9B3( ‭11101001 10110011‬ )  <br>11101001 0110011B = ‬01110100 10110011B = 74B3H = ‭29875‬D  <br>4、Compression information  <br>0x0380( 00000011 10000000 )  <br>0000011 0000000B = 0000 011 0 000000B  <br>( 0000 - 字典大小128KB、011 - 标准压缩、0 - solid flag、000000 - 压缩算法版本 )  <br>5、FILETIME  <br>(1)https://support.microsoft.com/zh-cn/help/188768/info-working-with-the-filetime-structure  <br>(2)https://www.pressc.cn/123.html  <br>(3)http://www.beijing-time.org/riqi.htm  <br>ctime  <br>7D A3 8E 8E 2A 01 D6 01  2020/05/19 00:24:42  <br>dwLowDateTime dwHighDateTime     <br>0x8E8EA37D    0x01D6012A  <br>0x01D6012A8E8EA37D = ‭132294521345975165‬  <br>153175.0039699074*24*60*60*10000000 = 132343203429999993.6‬  <br>42*10000000 + 24*60*10000000 = 14820000000‬  <br>132343203429999993.6‬ + 14820000000‬ = 132343218249999993.6  <br>                                     132294521345975165‬  <br>----------------------------------------------------------------------|

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3-6%E3%80%81Recovery-record-%E6%81%A2%E5%A4%8D%E8%AE%B0%E5%BD%95 "2.2.3.6、Recovery record(恢复记录)")2.2.3.6、**`Recovery record(恢复记录)`**

  加入冗余数据用于修复，在`压缩包本身损坏`但`恢复记录够多`时可对损坏压缩包进行恢复。  
**`实例：`**  
我们可以从例子RAR环境的图片中看到,恢复记录的大小约为19.6KB。  

|   |   |
|---|---|
|1  <br>2|336483 - 356643  (20161B 恢复记录)  <br>20161B/1024 = 19.6KB|

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-3-7%E3%80%81End-of-archive-header-%E7%BB%93%E5%B0%BE%E5%9D%97 "2.2.3.7、End of archive header(结尾块)")2.2.3.7、**`End of archive header(结尾块)`**

  End of archive marker. RAR does not read anything after this header letting to use third party tools to add extra information such as a digital signature to archive.  
  标记`压缩文档结束`。 RAR在此头之后不读取任何内容，从而允许使用`第三方工具`来添加`扩展的信息`，例如`压缩文档的数字签名`。

|字段名称|大小(类型)|说明|
|---|---|---|
|Header CRC32|uint32|头数据的CRC32，从“`Header size`”字段开始，直至“`End of archive flags`”字段。|
|Header size|vint|头数据的大小，从“`Header type`”字段开始，直至“`End of archive flags`”字段。|
|Header type|vint|`0x05`|
|Header flags|vint|所有头共有的标志：  <br>`0x01` - 头末尾存在扩展区域。  <br>`0x02` - 头末尾存在数据区域。  <br>`0x04` - 当更新压缩文档时，必须跳过类型未知并拥有该标志的块。  <br>`0x08` - 数据区域从上一卷继续。  <br>`0x10` - 数据区域从下一卷继续。  <br>`0x20` - 块取决于前一个文件块。  <br>`0x40` - 如果修改了主块，则保留一个子块。|
|End of archive flags|vint|`0x00` - 压缩文档不使用卷分割。或是集合中的最后一个卷。  <br>`0x01` - 压缩文档是卷，不是集合中的最后一个卷。|

**`实例：`**  
[![结尾块](https://sp4n9x.github.io/resources/2020/RAR_Format/End%20of%20archive%20header.jpg "结尾块")](https://sp4n9x.github.io/resources/2020/RAR_Format/End%20of%20archive%20header.jpg "结尾块")

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6|00356644 - 00356651  <br>1D 77 56 51 - Header CRC32( Header size -> End of archive flags )  <br>03 - Header size( Header size -> End of archive flags ) 03H = 3D  <br>05 - Header type( 结尾块 )  <br>04 - Header flags( 0x04 )  <br>00 - End of archive flags|

#### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-4%E3%80%81Service-headers-%E6%9C%8D%E5%8A%A1%E5%A4%B4 "2.2.4、Service headers(服务头)")2.2.4、Service headers(服务头)

  RAR uses service headers based on the file header data structure to store different supplementary information.  
  RAR使用基于`文件头数据结构`的`服务头`来存储不同的补充信息。

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-4-1%E3%80%81Archive-comment-header-%E5%8E%8B%E7%BC%A9%E6%96%87%E6%A1%A3%E6%B3%A8%E9%87%8A%E6%9C%8D%E5%8A%A1%E5%A4%B4 "2.2.4.1、Archive comment header(压缩文档注释服务头)")2.2.4.1、**`Archive comment header(压缩文档注释服务头)`**

  Optional header storing the main archive comment. Contains CMT identifier in file name field. Placed before any file headers and after the main archive header. Comment data is stored in UTF-8 immediately after the archive comment header. Now RAR does not use compression for archive comments, so packed and unpacked data sizes in header are equal and they both define the comment data size. Compression method in header is set to 0.  
  可选的头，用于存储`压缩文档注释`。在`文件名字段`中包含`CMT标识符`。放在`任何文件头`之前和`压缩文档头`之后。`注释数据`在`压缩文档注释头`之后用UTF-8存储。现在，RAR不对`压缩文件注释`使用压缩，因此头中的`压缩数据大小`和`未压缩数据大小`相等，并且它们都定义了`注释数据大小`。头中的`压缩方法`设置为0。

**`实例：`**  
[![压缩文档注释服务头](https://sp4n9x.github.io/resources/2020/RAR_Format/Archive%20comment%20header.jpg "压缩文档注释服务头")](https://sp4n9x.github.io/resources/2020/RAR_Format/Archive%20comment%20header.jpg "压缩文档注释服务头")

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28|00310812 - 00310849  <br>EF 0D 46 57 - Header CRC32( Header size -> Name )  <br>13 - Header size( Header type -> Name ) 13H = 19B  <br>03 - Header type( 服务头 )  <br>02 - Header flags( 0x02 )  <br>8E 00 - Data area size  <br>04 - File flags( 0x04 )  <br>8E 00 - Unpacked size  <br>00 - Attributes  <br>F5 88 A4 AC - Data CRC32( 77 77 77 2E 62 61 69 64 75 2E 63 6F 6D 00 )  <br>80 00 - Compression information  <br>00 - Host OS	  <br>03 - Name length( 03H = 3D )  <br>43 4D 54 - Name( CMT - 压缩文档注释 )                                                            <br>77 77 77 2E 62 61 69 64 75 2E 63 6F 6D 00 - Data area( www.baidu.com )  <br>----------------------------------------------------------------------  <br>Note: vint类型的数据，如果是多字节的，前几个字节的最高位为连续标志  <br>(设为1)，低7位为数据，最后一个字节的最高位为0，低7位依旧是数据。所  <br>以我们需要提取出数据，进行重新组合，才能得到“Quick open header”  <br>和“Recovery record”真实偏移。  <br>1、Data size && Unpacked size  <br>0x008E( 00000000 10001110 )  <br>0001110B = 00001110B = EH = 14D  <br>2、Compression information  <br>0x0080( 00000000 10000000 )  <br>0000000 0000000B = 0000 000 0 000000B  <br>( 0000 - 字典大小128KB、000 - 存储、0 - solid flag、000000 - 压缩算法版本 )  <br>----------------------------------------------------------------------|

##### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#2-2-4-2%E3%80%81Quick-open-header-%E5%BF%AB%E9%80%9F%E6%89%93%E5%BC%80%E6%9C%8D%E5%8A%A1%E5%A4%B4 "2.2.4.2、Quick open header(快速打开服务头)")2.2.4.2、**`Quick open header(快速打开服务头)`**

  Optional header storing the quick open record. Contains QO identifier in file name field. Placed after all file headers, but before the recovery record and end of archive header. It is possible to locate the quick open header with locator record in main archive header.  
  可选的头，用于存储`快速打开记录`。在`文件名`字段中包含`QO标识符`。置于`所有文件头`之后，但在`恢复记录`和`压缩文档结尾块`之前。可以在`压缩文档头`中找到带有定位记录的`快速打开头`。  
  Quick open record data is stored immediately after the quick open header. RAR does not use compression for quick open data, so packed and unpacked data sizes in header are equal and they both define the quick open data size. Compression method in header is set to 0.  
  `快速打开记录数据`将立即存储在`快速打开头`之后。 RAR不对`快速打开数据`使用压缩，因此头中的`压缩数据大小`和`未压缩数据大小`相等，并且它们都定义了`快速打开数据大小`。头中的压缩方法设置为0。  
  Quick open data is the array consisting of data cache structures. Every data cache structure stores a portion of archived data and has the following format:  
  `快速打开数据`是由`数据缓存结构`组成的数组。`每个数据缓存结构`都存储`一部分压缩数据`，并具有以下格式：

|字段名称|大小(类型)|说明|
|---|---|---|
|Structure CRC32|uint32|从“`Structure size`”字段开始的结构数据的`CRC32`。|
|Structure size|vint|从`标志字段`开始的结构数据的大小。在当前实现中，该字段不得超过`3个字节`，因此最大大小为`2 MB`。|
|Flags|vint|当前设置为0。|
|Offset|vint|从`快速打开头的开始`到当前结构中缓存的`压缩文档数据的开始`的偏移量。我们可以使用该值来计算存储当前结构的`压缩数据的绝对位置`。从结构数组的开头到结尾，可以保证数据缓存结构引用的绝对存档位置始终在增长。|
|Data size|vint|当前结构中存储的`压缩文档数据的大小`。|
|Data|N bytes|存储在当前结构中的`压缩文档数据`。|

  Normally RAR uses the quick open data to store copies of file and service headers. It can store either all headers or only a part of them. If required header is missing in quick open data or if structure CRC32 is invalid, data are read from its original archive position.  
  通常，RAR使用`快速打开数据`来存储`文件的副本`和`服务头`。它可以存储`所有头`，也可以只存储`其中的一部分`。如果快速打开数据中`缺少必需的标头`，或者结构`CRC32无效`，则从其`原始存档位置`读取数据。  
  Using the quick open data is optional. You can skip it completely and read only standard archive headers. But it is important to use the same access pattern when reading file names to display them to user and to extract files. Otherwise it would be possible to see one file name and extract another in case the quick open data and real archive data are intentionally created different. It could introduce a security threat. So if you use the quick open data when displaying the archive contents, use it when extracting. If you do not use it when displaying the archive contents, do not use it when extracting.  
  使用`快速打开数据`是可选的。您可以完全跳过它，而仅读取`标准压缩文档头`。但是，重要的是在`读取文件名`时使用相同的访问模式以将其`显示给用户`并`提取文件`。否则，如果有意创建的`快速打开数据`和`实际存档数据`不同，则可能会看到`一个文件名`并提取`另一个文件名`。它可能会带来安全威胁。因此，如果在`显示存档内容时`使用快速打开的数据，请在`提取时`使用它。如果在`显示存档内容时`不使用它，则在`提取时`不要使用它。

**`实例：`**  
[![快速打开服务头](https://sp4n9x.github.io/resources/2020/RAR_Format/Quick%20open%20header.jpg "快速打开服务头")](https://sp4n9x.github.io/resources/2020/RAR_Format/Quick%20open%20header.jpg "快速打开服务头")

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31  <br>32  <br>33  <br>34  <br>35  <br>36  <br>37  <br>38  <br>39  <br>40  <br>41  <br>42  <br>43  <br>44  <br>45  <br>46  <br>47  <br>48  <br>49  <br>50  <br>51  <br>52  <br>53  <br>54  <br>55  <br>56  <br>57  <br>58  <br>59  <br>60  <br>61  <br>62  <br>63  <br>64  <br>65  <br>66  <br>67  <br>68  <br>69  <br>70  <br>71  <br>72|00336299 - 00336482    <br>C3 CA 91 FD - Header CRC32( Header size -> Name )  <br>0E - Header size( Header size -> Name ) 0EH = 14D  <br>03 - Header type( 服务头 )   <br>06 - Header flags( 0x06 = 0x02 + 0x04 )  <br>A5 01 - Data area size  <br>00 - File flags  <br>A5 01 - Unpacked size  <br>00 - Attributes  <br>80 00 - Compression information  <br>00 - Host OS  <br>02 - Name length( 02H = 2D )  <br>51 4F - Name( QO - 快速打开头 )     <br>16 E8 D1 20 - Structure CRC32( Structure size -> Data ) Data area(快速打开记录数据)  <br>9F 01 - Structure size( Structure size -> Data )  <br>00 - Flags  <br>E9 C6 01 - Offset  <br>99 01 - Data size  <br>FF A6 73 44 - Header CRC32( Header size -> Extra area ) Data  <br>93 01 - Header size( Header type -> Extra area )  <br>02 - Header type( 文件头 )  <br>03 - Header flags( 0x03 = 0x01 + 0x02 )  <br>6F - Extra area size[ 0x6F = (0x30+0x01) + (0x22+0x01) + (0x1A+0x01) ]  <br>D0 C5 01 - Data area size  <br>00 - File flags                       ‭    <br>B3 E9 - Unpacked size   <br>01 20 - Attributes      <br>80 03 - Compression information  <br>00 - Host OS  <br>15 - Name length( 15H = 21D )  <br>32 30 32 30 30 32 30 35 31 31 34 36 31 34 39 34 35 2E 70 6E 67 - Name  <br>30 - Size( Extra area ) 30H = 48D  <br>01 - Type( 文件加密记录 )  <br>00 - Version  <br>03 - Flags( 0x03 = 0x02 + 0x01 )  <br>0F - KDF count  <br>48 2F 76 C7 97 E7 8A 77 D5 55 F8 1F 5F 31 D8 26 - Salt  <br>5C 4C 4F B0 4F 94 01 90 95 95 2E E9 E6 BF 8F 75 - IV  <br>4D 87 A2 57 6C EA BA B2 EB D6 4E A8 - Check value  <br>22 - Size( Extra area ) 22H = 34D  <br>02 - Type( 文件哈希记录 )  <br>00 - Hash type  <br>16 87 E7 C4 D1 8C 13 AA BD B5 18 4A A9 C7 3E D1 85 85 95 B4 6F 12 3E 55 10 C6 86 7C 5A 2E 80 B9 - Hash data  <br>1A - Size( Extra area ) 1AH = 26D  <br>03 - Type( 文件时间记录 )  <br>0E - Flags( 0x0E = 0x02 + 0x04 + 0x08 )  <br>8C E5 A3 8C 2A 01 D6 01 - mtime  <br>7D A3 8E 8E 2A 01 D6 01 - ctime  <br>87 27 7A 12 2D 2D D6 01 - atime 	  <br>----------------------------------------------------------------------  <br>Note: vint类型的数据，如果是多字节的，前几个字节的最高位为连续标志  <br>(设为1)，低7位为数据，最后一个字节的最高位为0，低7位依旧是数据。所  <br>以我们需要提取出数据，进行重新组合，才能得到“Quick open header”  <br>和“Recovery record”真实偏移。  <br>1、Data area size && Unpacked size  <br>0x01A5( 00000001 10100101 )  <br>0000001 0100101B = 10100101B = A5H = 165D  <br>2、Compression information	  <br>0x0080( 00000000 10000000 )  <br>0000000 0000000B = 0000 000 0 000000B  <br>( 0000 - 字典大小128KB、000 - 存储、0 - solid flag、000000 - 压缩算法版本 )  <br>3、Structure size  <br>0x019F( 00000001 10011111 )  <br>0000001 0011111B = 10011111B = 9FH = 159D  <br>4、Offset  <br>0x01C6E9( 00000001 11000110 11101001 )  <br>0000001 1000110 1101001B = 01100011 01101001B = 6369H = ‭25449‬D  <br>336299 - 25449 = 310850(文件头起始偏移)  <br>5、Data size  <br>0x0199( 00000001 10011001 )  <br>0000001 0011001B = 10011001B = 99H = 153D  <br>----------------------------------------------------------------------|

---

## [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#3%E3%80%81%E9%92%88%E5%AF%B9RAR%E7%9A%84%E4%B8%BB%E8%A6%81%E6%94%BB%E5%87%BB%E6%96%B9%E5%BC%8F "3、针对RAR的主要攻击方式")3、针对RAR的主要攻击方式

### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#3-1%E3%80%81%E7%88%86%E7%A0%B4 "3.1、爆破")3.1、爆破

RAR 5.0 密码破解  
[https://blog.xugr.me/post/rar-crack/](https://blog.xugr.me/post/rar-crack/)

### [](https://sp4n9x.github.io/2020/04/10/RAR%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%88%86%E6%9E%90/#3-2%E3%80%81%E4%BC%AA%E5%8A%A0%E5%AF%86 "3.2、伪加密")3.2、伪加密

  `伪加密`只发生在`RAR5.0以前的版本`中，我们只需要修改`FILE_HEAD`中的`HEAD_FLAGS`的`0x0004`标记为1，就可以造成`RAR伪加密`。
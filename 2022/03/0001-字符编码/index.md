# 字符编码


## 字符编码基本概念
### 字符
字符（Character）：在计算机和电信技术中，一个字符是一个单位的字形、类字形单位或符号的基本信息。说的简单点字符是各种文字和符号的总称。一个字符可以是一个中文汉字、一个英文字母、一个阿拉伯数字、一个标点符号、一个图形符号或者控制符号等。

### 字符集
字符集（Character Set）：是指多个字符的集合。不同的字符集包含的字符个数不一样、包含的字符不一样、对字符的编码方式也不一样。例如GB2312是中国国家标准的简体中文字符集，GB2312收录简化汉字（6763个）及一般符号、序号、数字、拉丁字母、日文假名、希腊字母、俄文字母、汉语拼音符号、汉语注音字母，共 7445 个图形字符。而ASCII字符集只包含了128字符，这个字符集收录的主要字符是英文字母、阿拉伯字母和一些简单的控制字符。

另外，还有其他常用的字符集有 GBK字符集、GB18030字符集、Big5字符集、Unicode字符集等。

### 字符编码
字符编码（Character Encoding）：字符编码是指一种映射规则，根据这个映射规则可以将某个字符映射成其他形式的数据以便在计算机中存储和传输。例如ASCII字符编码规定使用单字节中低位的7个比特去编码所有的字符，在这个编码规则下字母A的编号是65（ASCII码），用单字节表示就是0x41，因此写入存储设备的时候就是二进制的 01000001。每种字符集都有自己的字符编码规则，常用的字符集编码规则还有 UTF-8编码、GBK编码、Big5编码等。

### 码点
码点（Code Point）：有些地方翻译为码值或内码。是指在某个字符集中，根据某种编码规则将字符编码后得到的值。比如在ASCII字符集中，字母A经过ASCII编码得到的值是65，那么65就是字符A在ASCII字符集中的码点。

> 总结：通俗解释字符集就是把字符放到一起的一个集合。而这个集合的每一个字符都对应一个数字，叫做码点。那么，这样就建立起来数字和字符之间的索引关系。那么，某个字符在计算机中怎么表示，具体占用几个字节等等，这些就需要编码规则来解决了。这个就是字符编码，他来解决根据某个规则来将字符映射到相应的码点上面。

## 字符编码

### ASCII 码

ASCII 编码于 1967 年第一次发布，最后一次更新是在 1986 年，迄今为止共收录了 128  个字符，包含了基本的拉丁字母（英文字母）、阿拉伯数字（也就是  1234567890）、标点符号（,.!等）、特殊符号（@#$%^&等）以及一些具有控制功能的字符（往往不会显示出来）。

ASCII  编码是美国人给自己设计的，他们并没有考虑欧洲那些扩展的拉丁字母，也没有考虑韩语和日语，我大中华几万个汉字更是不可能被重视。计算机也是美国人发明的，起初使用的就是 ASCII 码，只能显示英文字符。各个国家为了让本国公民也能正常使用计算机，开始效仿 ASCII 开发自己的字符编码，例如 ISO/IEC  8859（欧洲字符集）、shift_Jis（日语字符集）、GBK（中文字符集）等。


| 二进制   | 十进制 | 十六进制 | 字符/缩写                                     | 解释                               |
| :------- | :----- | :------- | :-------------------------------------------- | :--------------------------------- |
| 00000000 | 0      | 00       | NUL (NULL)                                    | 空字符                             |
| 00000001 | 1      | 01       | SOH (Start Of Headling)                       | 标题开始                           |
| 00000010 | 2      | 02       | STX (Start Of Text)                           | 正文开始                           |
| 00000011 | 3      | 03       | ETX (End Of Text)                             | 正文结束                           |
| 00000100 | 4      | 04       | EOT (End Of Transmission)                     | 传输结束                           |
| 00000101 | 5      | 05       | ENQ (Enquiry)                                 | 请求                               |
| 00000110 | 6      | 06       | ACK (Acknowledge)                             | 回应/响应/收到通知                 |
| 00000111 | 7      | 07       | BEL (Bell)                                    | 响铃                               |
| 00001000 | 8      | 08       | BS (Backspace)                                | 退格                               |
| 00001001 | 9      | 09       | HT (Horizontal Tab)                           | 水平制表符                         |
| 00001010 | 10     | 0A       | LF/NL(Line Feed/New Line)                     | 换行键                             |
| 00001011 | 11     | 0B       | VT (Vertical Tab)                             | 垂直制表符                         |
| 00001100 | 12     | 0C       | FF/NP (Form Feed/New Page)                    | 换页键                             |
| 00001101 | 13     | 0D       | CR (Carriage Return)                          | 回车键                             |
| 00001110 | 14     | 0E       | SO (Shift Out)                                | 不用切换                           |
| 00001111 | 15     | 0F       | SI (Shift In)                                 | 启用切换                           |
| 00010000 | 16     | 10       | DLE (Data Link Escape)                        | 数据链路转义                       |
| 00010001 | 17     | 11       | DC1/XON  (Device Control 1/Transmission On)   | 设备控制1/传输开始                 |
| 00010010 | 18     | 12       | DC2 (Device Control 2)                        | 设备控制2                          |
| 00010011 | 19     | 13       | DC3/XOFF  (Device Control 3/Transmission Off) | 设备控制3/传输中断                 |
| 00010100 | 20     | 14       | DC4 (Device Control 4)                        | 设备控制4                          |
| 00010101 | 21     | 15       | NAK (Negative Acknowledge)                    | 无响应/非正常响应/拒绝接收         |
| 00010110 | 22     | 16       | SYN (Synchronous Idle)                        | 同步空闲                           |
| 00010111 | 23     | 17       | ETB (End of Transmission Block)               | 传输块结束/块传输终止              |
| 00011000 | 24     | 18       | CAN (Cancel)                                  | 取消                               |
| 00011001 | 25     | 19       | EM (End of Medium)                            | 已到介质末端/介质存储已满/介质中断 |
| 00011010 | 26     | 1A       | SUB (Substitute)                              | 替补/替换                          |
| 00011011 | 27     | 1B       | ESC (Escape)                                  | 逃离/取消                          |
| 00011100 | 28     | 1C       | FS (File Separator)                           | 文件分割符                         |
| 00011101 | 29     | 1D       | GS (Group Separator)                          | 组分隔符/分组符                    |
| 00011110 | 30     | 1E       | RS (Record Separator)                         | 记录分离符                         |
| 00011111 | 31     | 1F       | US (Unit Separator)                           | 单元分隔符                         |
| 00100000 | 32     | 20       | (Space)                                       | 空格                               |
| 00100001 | 33     | 21       | !                                             |                                    |
| 00100010 | 34     | 22       | "                                             |                                    |
| 00100011 | 35     | 23       | #                                             |                                    |
| 00100100 | 36     | 24       | $                                             |                                    |
| 00100101 | 37     | 25       | %                                             |                                    |
| 00100110 | 38     | 26       | &                                             |                                    |
| 00100111 | 39     | 27       | '                                             |                                    |
| 00101000 | 40     | 28       | (                                             |                                    |
| 00101001 | 41     | 29       | )                                             |                                    |
| 00101010 | 42     | 2A       | *                                             |                                    |
| 00101011 | 43     | 2B       | +                                             |                                    |
| 00101100 | 44     | 2C       | ,                                             |                                    |
| 00101101 | 45     | 2D       | -                                             |                                    |
| 00101110 | 46     | 2E       | .                                             |                                    |
| 00101111 | 47     | 2F       | /                                             |                                    |
| 00110000 | 48     | 30       | 0                                             |                                    |
| 00110001 | 49     | 31       | 1                                             |                                    |
| 00110010 | 50     | 32       | 2                                             |                                    |
| 00110011 | 51     | 33       | 3                                             |                                    |
| 00110100 | 52     | 34       | 4                                             |                                    |
| 00110101 | 53     | 35       | 5                                             |                                    |
| 00110110 | 54     | 36       | 6                                             |                                    |
| 00110111 | 55     | 37       | 7                                             |                                    |
| 00111000 | 56     | 38       | 8                                             |                                    |
| 00111001 | 57     | 39       | 9                                             |                                    |
| 00111010 | 58     | 3A       | :                                             |                                    |
| 00111011 | 59     | 3B       | ;                                             |                                    |
| 00111100 | 60     | 3C       | <                                             |                                    |
| 00111101 | 61     | 3D       | =                                             |                                    |
| 00111110 | 62     | 3E       | >                                             |                                    |
| 00111111 | 63     | 3F       | ?                                             |                                    |
| 01000000 | 64     | 40       | @                                             |                                    |
| 01000001 | 65     | 41       | A                                             |                                    |
| 01000010 | 66     | 42       | B                                             |                                    |
| 01000011 | 67     | 43       | C                                             |                                    |
| 01000100 | 68     | 44       | D                                             |                                    |
| 01000101 | 69     | 45       | E                                             |                                    |
| 01000110 | 70     | 46       | F                                             |                                    |
| 01000111 | 71     | 47       | G                                             |                                    |
| 01001000 | 72     | 48       | H                                             |                                    |
| 01001001 | 73     | 49       | I                                             |                                    |
| 01001010 | 74     | 4A       | J                                             |                                    |
| 01001011 | 75     | 4B       | K                                             |                                    |
| 01001100 | 76     | 4C       | L                                             |                                    |
| 01001101 | 77     | 4D       | M                                             |                                    |
| 01001110 | 78     | 4E       | N                                             |                                    |
| 01001111 | 79     | 4F       | O                                             |                                    |
| 01010000 | 80     | 50       | P                                             |                                    |
| 01010001 | 81     | 51       | Q                                             |                                    |
| 01010010 | 82     | 52       | R                                             |                                    |
| 01010011 | 83     | 53       | S                                             |                                    |
| 01010100 | 84     | 54       | T                                             |                                    |
| 01010101 | 85     | 55       | U                                             |                                    |
| 01010110 | 86     | 56       | V                                             |                                    |
| 01010111 | 87     | 57       | W                                             |                                    |
| 01011000 | 88     | 58       | X                                             |                                    |
| 01011001 | 89     | 59       | Y                                             |                                    |
| 01011010 | 90     | 5A       | Z                                             |                                    |
| 01011011 | 91     | 5B       | [                                             |                                    |
| 01011100 | 92     | 5C       | \                                             |                                    |
| 01011101 | 93     | 5D       | ]                                             |                                    |
| 01011110 | 94     | 5E       | ^                                             |                                    |
| 01011111 | 95     | 5F       | _                                             |                                    |
| 01100000 | 96     | 60       | `                                             |                                    |
| 01100001 | 97     | 61       | a                                             |                                    |
| 01100010 | 98     | 62       | b                                             |                                    |
| 01100011 | 99     | 63       | c                                             |                                    |
| 01100100 | 100    | 64       | d                                             |                                    |
| 01100101 | 101    | 65       | e                                             |                                    |
| 01100110 | 102    | 66       | f                                             |                                    |
| 01100111 | 103    | 67       | g                                             |                                    |
| 01101000 | 104    | 68       | h                                             |                                    |
| 01101001 | 105    | 69       | i                                             |                                    |
| 01101010 | 106    | 6A       | j                                             |                                    |
| 01101011 | 107    | 6B       | k                                             |                                    |
| 01101100 | 108    | 6C       | l                                             |                                    |
| 01101101 | 109    | 6D       | m                                             |                                    |
| 01101110 | 110    | 6E       | n                                             |                                    |
| 01101111 | 111    | 6F       | o                                             |                                    |
| 01110000 | 112    | 70       | p                                             |                                    |
| 01110001 | 113    | 71       | q                                             |                                    |
| 01110010 | 114    | 72       | r                                             |                                    |
| 01110011 | 115    | 73       | s                                             |                                    |
| 01110100 | 116    | 74       | t                                             |                                    |
| 01110101 | 117    | 75       | u                                             |                                    |
| 01110110 | 118    | 76       | v                                             |                                    |
| 01110111 | 119    | 77       | w                                             |                                    |
| 01111000 | 120    | 78       | x                                             |                                    |
| 01111001 | 121    | 79       | y                                             |                                    |
| 01111010 | 122    | 7A       | z                                             |                                    |
| 01111011 | 123    | 7B       | {                                             |                                    |
| 01111100 | 124    | 7C       | \|                                            |                                    |
| 01111101 | 125    | 7D       | }                                             |                                    |
| 01111110 | 126    | 7E       | ~                                             |                                    |
| 01111111 | 127    | 7F       | DEL (Delete)                                  | 删除                               |

### GB2312(别名:EUC-CN)
《信息交换用汉字编码字符集》是由中国国家标准总局1980年发布，1981年5月1日开始实施的一套国家标准，标准号是GB 2312—1980。

GB2312的出现，基本满足了汉字的计算机处理需要，它所收录的汉字已经覆盖中国大陆99.75%的使用频率。对于人名、古汉语等方面出现的罕用字，GB 2312不能处理，这导致了后来GBK及GB 18030汉字字符集的出现。

GB2312(1980年)共收录 7445 个字符 --> GBK1.0(1995年)共收录 21886 个字符 --> GB18030(2000年)取代GBK1.0共收录 27484 个汉字包含了少数民族语言。

> 从 ASCII -> GB2312 -> GBK -> GB18030，这些编码都是向下兼容的。区分中文编码的方法是最高字节的最高位不为 0。这些都属于双字节字符集。
#### GB2312 编码规则
- GB2312 规定每个英文字符占1字节(只有这一个特例，为了兼容 ASCII)，中文字符采用两个字节表示
- GB2312 整体分为 94 个区，每个区 94 位，01-09 特殊符号区；16-55一级汉字，按拼音排序；56-87二级汉字，部首/笔划排序；10-15与88-94未定义
- GB2312 是区位编码，编码范围是 0XA1A1-0XFEFE，其中汉字编码范围是 0XB0A1-0XF7FE。
- GB2312 01-09 符号和数字区，16-87是汉字区
- GB2312 编码计算：[区号+0xA0][位号+0xA0]

#### GB2312 编码识别
保存为`demo-gb2312.c`
```c
//
// Created by dingjing on 3/28/22.
//
#include <stdio.h>
#include <string.h>
#include <stdint.h>
#include <stdbool.h>

/**
 * 总共 94 个区 94 个位
 */
bool is_gb2312 (const char* code)
{
    int len = strlen (code);
    if (len != 2 && len != 1) {
        return false;
    }

    uint8_t codeH = code[0];
    uint8_t codeL = code[1];

    uint8_t area = codeH - 0xA0;
    uint8_t pos  = codeL - 0xA0;

    if ((len == 1) && (0 == (codeL & 0x80))) {
        printf ("ascii code: '%-3c' hex code: 0x%-2X\n", *code, *code);
        return true;
    }

    if (area >= 1 && area <= 94 && pos >= 1 && pos <= 94) {
        printf ("gb2312 code: '%-3s' -- area: %-2u, pos: %-2u hex code: 0x%-2X 0x%-2X\n", code, area, pos, codeH, codeL);
        return true;
    }
}


int main (int argc, char* argv[])
{
    /**
     * https://uic.io/en/charset/show_raw/gb2312
     * http://tools.jb51.net/table/gb2312
     * 1. '残'   十六进制: B2D0
     * 2. '怖'   十六进制: B2C0
     * 3. '惭'   十六进制: B2D1
     *
     * 4. ascii 里显示的特殊符号 32(space) - 47(/)
     * 5. ascii 里数字 30(0) - 39(9)
     * 6. ascii 里显示的特殊符号 58(:) - 64(@)
     * 7. ascii 里显示的大写字母 65(A) - 90(Z)
     * 8. ascii 里显示的标号字符 91([) - 96(`)
     * 9. ascii 里显示的大写字母 97(a) - 122(z)
     * 10. ascii 其它可显示字符 123({) - 126(~)
     *
     * uint16_t 就足够了
     */
    const char* arr[] = {
            "、", "∩", "〓",                              /* 第一区 */
            "残", "怖", "惭", "庄", "丽", "君", "齄",
            " ", "/",
            "0", "9",
            ":", "@",
            "A", "Z",
            "[", "`",
            "a", "z",
            "{", "~"
    };

    printf ("array: %lu/%lu\n\n", sizeof arr[0], sizeof arr);

    int len = sizeof arr / sizeof arr[0];
    for (int i = 0; i < len; ++i) {
        const char* code = arr[i];

        if (!is_gb2312 (code)) {
            printf ("index: %d -- code '%-3s' is not gb2312\n", i, code);
        }
    }

    return 0;
}
```
编译命令:
```
gcc -O0 -finput-charset=utf8 -fexec-charset=gb2312 demo-gb2312.run demo-gb2312.c
```
#### GB2312识别思路(后续补充)

### Unicode 编码
Unicode 的使命就是为了统一世界上所有语言的编码，它包含了世界上所有字符的编码，规定了每个字符对应的码点值。

> 这里需要注意：Unicode 只是一个符号集，它只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。<br/>
> 问题1: 如何才能区别unicode和ascii <br/>
> 问题2: 我们已经知道，英文字母只用一个字节表示就够了，如果 unicode 统一规定，每个符号用三个或四个字节表示，那么每个英文字母前都必然有二到三个字节是 0， 这对于存储来说是极大的浪费，文本文件的大小会因此大出 2-3 倍，这是无法接受的。

#### 另附unicode中汉字编码

| 字符集       | 字数    | unicode 编码                                         |
| ------------ | ------- | ---------------------------------------------------- |
| 基本汉字     | 20902字 | 4E00(一) - 9FA5(龥)                                  |
| 基本汉字扩充 | 90字    | 9FA6(龦) - 9FFF(<无法显示>) 这些看着不像汉字，先不管 |
| 扩展A        | 6592字  | 3400 - 4DBF，看着不像汉字                            |
| 扩展B        | 42720字 | 20000-2A6DF                                          |
| 扩展C        | 4153字  | 2A700-2B738                                          |
| 扩展D        | 222字   | 2B740-2B81D                                          |
| 扩展E        | 5762字  | 2B820-2CEA1                                          |
| 扩展F        | 7473字  | 2CEB0-2EBE0                                          |
| 扩展G        | 4939字  | 30000-3134A                                          |
| 康熙部首     | 214字   | 2F00-2FD5                                            |
| 部首扩展     | 115字   | 2E80-2EF3                                            |
| 兼容汉字     | 477字   | F900-FAD9                                            |
| 兼容扩展     | 542字   | 2F800-2FA1D                                          |
| PUA(GBK)部件 | 81字    | E815-E86F                                            |
| 部件扩展     | 452字   | E400-E5E8                                            |
| PUA增补      | 207字   | E600-E6CF                                            |
| 汉字笔画     | 36字    | 31C0-31E3                                            |
| 汉字结构     | 12字    | 2FF0-2FFB                                            |
| 汉语注音     | 43字    | 3105-312F                                            |
| 注音扩展     | 22字    | 31A0-31BA                                            |
| 〇           | 1字     | 3007                                                 |

### utf8 编码(unicode编码的一种实现)

UTF-8 是一种变长字节编码方式，是 Unicode 的一种实现方式，使用 1-4 个字节表示一个符号， 对于某一个字符的 UTF-8 编码，如果只有一个字节则其最高二进制位为 0； 如果是多字节，其第一个字节从最高位开始，连续的二进制位值为 1 的个数决定了其编码的位数，其余各字节均以 10 开头。UTF-8 最多可用到 6 个字节。

> 注意：这里是utf8编码的核心，它规定了utf8最多可以使用多少字节编码<br/>
> 思考：为什么utf8最多只能使用6字节参与编码？<br/>
> 实际上 utf-8 编码没用到 6 字节那么多，只用到了 4 字节<br/>
> 另外补充：另外补充 utf16，介于 utf8 与 utf32 之间，部分编码定长、部分编码不定长(只有两种) utf32 是定长的，用 4 字节表示一个字符<br/>
> utf8中文汉字占3字节(这种结论不是一直不变的)

以下展示 utf8 分别为 1、2、3、4、5、6字节时候，理论上其它位的编码范围。

```
1字节 0xxxxxxx
2字节 110xxxxx 10xxxxxx
3字节 1110xxxx 10xxxxxx 10xxxxxx
4字节 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
5字节 111110xx 10xxxxxx 10xxxxxx 10xxxxxx 10xxxxxx
6字节 1111110x 10xxxxxx 10xxxxxx 10xxxxxx 10xxxxxx 10xxxxxx
```
与上述等价
```
1字节, utf8-1 0x00-0x7F
2字节, utf8-2 0xC2-0XDF   0X80-0XBF
3字节, utf8-3 0xE0        0xA0-0XBF   0x80-0xBF
             0xE1-0XEC   0x80-0XBF   0x80-0xBF
             0xED        0x80-0x9F   0x80-0xBF
             0xEE-0xEF   0x80-0xBF   0x80-0xBF
4字节, utf8-4 0xF0        0x90-0xBF   0x80-0xBF   0x80-0xBF
             0xF1-0xF3   0x80-0xBF   0x80-0xBF   0x80-0xBF
             0xF4        0x80-0x8F   0x80-0xBF   0x80-0xBF
```
#### utf8 编码识别
保存为`demo-utf8.c`
```c
//
// Created by dingjing on 3/28/22.
//

#include <stdio.h>
#include <stdint.h>
#include <string.h>
#include <stdbool.h>

bool is_utf8_1 (const char* code)
{
    int len = strlen(code);
    if (len < 1) {
        return false;
    }
    const uint8_t code1 = code[0];

    if (!(0x80 & code1)) {
        return true;
    }

    return false;
}

bool is_utf8_2 (const char* code)
{
    int len = strlen(code);
    if (len < 2) {
        return false;
    }
    const uint8_t code1 = code[0];
    const uint8_t code2 = code[1];

//    printf ("debug2: 0x%-4X\n", code);

    if (!is_utf8_1 (code) && (0xC0 == (0xE0 & code1)) && (0x80 == (0xE0 & code2))) {
//        printf ("debug: is utf8-2\n");
        if (code1 >= 0xC0 && code1 <= 0xDF && code2 >= 0x80 && code2 <= 0xBF) {
            return true;
        }
    }

    return false;
}

bool is_utf8_3 (const char* code)
{
    /**
     * 0xE0        0xA0-0XBF   0x80-0xBF
     * 0xE1-0XEC   0x80-0XBF   0x80-0xBF
     * 0xED        0x80-0x9F   0x80-0xBF
     * 0xEE-0xEF   0x80-0xBF   0x80-0xBF
     */
    int len = strlen(code);
    if (len < 3) {
        return false;
    }

    const uint8_t code1 = code[0];
    const uint8_t code2 = code[1];
    const uint8_t code3 = code[2];

    if ((0xE0 == code1) && ((code2 >= 0xA0) && (code2 <= 0XBF)) && ((code3 >= 0x80) && (code3 <= 0xBF))) {
        return true;
    } else if (((code1 >= 0xE1) && (code1 <= 0xEC)) && ((code2 >= 0x80) && (code2 <= 0xBF)) && ((code3 >= 0x80) && (code3 <= 0xBF))) {
        return true;
    } else if ((0xED == code1) && ((code2 >= 0x80) && (code2 <= 0x9F)) && ((code3 >= 0x80) && (code3 <= 0xBF))) {
        return true;
    } else if (((code1 >= 0xEE) && (code1 <= 0xEF)) && ((code2 >= 0x80) && (code2 <= 0xBF)) && ((code3 >= 0x80) && (code3 <= 0xBF))) {
        return true;
    }

    return false;
}

bool is_utf8_4 (const char* code)
{
    /**
     * 0xF0        0x90-0xBF   0x80-0xBF   0x80-0xBF
     * 0xF1-0xF3   0x80-0xBF   0x80-0xBF   0x80-0xBF
     * 0xF4        0x80-0x8F   0x80-0xBF   0x80-0xBF
     */
    int len = strlen(code);
    if (len < 4) {
        return false;
    }

    const uint8_t code1 = code[0];
    const uint8_t code2 = code[1];
    const uint8_t code3 = code[2];
    const uint8_t code4 = code[3];

//    printf ("debug4: 0x%-4X %-4X\n", code1H, code1L);

    if (!is_utf8_1 (code) && !is_utf8_2 (code) && !is_utf8_3 (code)) {
        if ((0xF0 == code1)
            && ((code2 >= 0x90) && (code2 <= 0xBF))
            && ((code3 >= 0x80) && (code3 <= 0xBF))
            && ((code4 >= 0x80) && (code4 <= 0xBF))) {
            return true;
        } else if (((code1 >= 0xF1) && (code1 <= 0xF3))
                   && ((code2 >= 0x80) && (code2 <= 0xBF))
                   && ((code3 >= 0x80) && (code3 <= 0xBF))
                   && ((code4 >= 0x80) && (code4 <= 0xBF))) {
            return true;
        } else if ((0xF4 == code1)
                   && ((code2 >= 0x80) && (code2 <= 0x8F))
                   && ((code3 >= 0x80) && (code3 <= 0xBF))
                   && ((code4 >= 0x80) && (code4 <= 0xBF))) {
            return true;
        }
    }

    return false;
}

int main (int argc, char* argv[])
{
    // utf-8 1 - 4 字节
    const char* arr[] = {
            " ", "/", "0", "9", ":", "@", "A", "Z", "[", "`", "a", "z", "{", "~",       /* 以下都是 ASCII */
            "§", "©", "®", "Ą", "Ď", "Đ", "Ő",                                          /* 2 字节 utf8 编码 */
            "ᥕ", "一", "龥", "庄", "丽", "君",                                            /* 3 字节 */
            "契", "い", "龜"
    };

    printf ("array: %lu/%lu\n\n", sizeof arr[0], sizeof arr);

    char buf[128] = {0};
    int len = sizeof arr / sizeof arr[0];
    for (int i = 0; i < len; ++i) {
        const char* code = arr[i];

        if (is_utf8_1 (code)) {
            printf ("index: %-2d --> utf8-1 code: '%c'  --  hex code: 0x%-02X <==> dec: %d\n", i, code[0], code[0] & 0xFF, code[0] & 0xFF);
            continue;
        } else if (is_utf8_2 (code)) {
            printf ("index: %-2d --> utf8-2 code: '%s'  --  hex code: 0x%-02X 0x%-02X\n", i, code, code[0] & 0xFF, code[1] & 0xFF);
            continue;
        } else if (is_utf8_3 (code)) {
            printf ("index: %-2d --> utf8-3 code: '%s'  --  hex code: 0x%-02X 0x%-02X 0x%-02X\n", i, code, code[0] & 0xFF, code[1] & 0xFF, code[2] & 0xFF);
        } else if (is_utf8_4 (code)) {
            printf ("index: %-2d --> utf8-4 code: '%s'  --  hex code: 0x%-02X 0x%-02X 0x%-02X 0x%-02X\n", i, code, code[0] & 0xFF, code[1] & 0xFF, code[2] & 0xFF, code[3] & 0xFF);
        }
    }

    return 0;
}
```
编译方法
```shell
gcc -O0 -finput-charset=utf8 -fexec-charset=utf8 demo-utf8.run demo-utf8.c
```
#### utf8识别思路(后续补充)

### utf16 编码(unicode编码的一种实现)
> 没用到，后续补充。<br/>
> 部分编码定长、部分编码不定长(只有两种)

### utf32 编码(unicode编码的一种实现)
> utf32 是定长的，用 4 字节表示一个字符<br/>


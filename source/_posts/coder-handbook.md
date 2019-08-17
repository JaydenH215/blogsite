---
title: Coder Handbook
date: 2019-08-17 17:45:04
tags:
- 代码
- 整洁


categories:
- 软件工程

---

大佬有云：
> Talk is cheap. Show me the code.       --Linus Torvalds

官方来说，代码好坏体现一个程序员的职业素养，在笔者看来代码是一个程序员的门面担当，在这个颜值即正义的时代，如何码的一手好代码，是门必修课中的必修课。
<!-- more --> 


前言
===
- 写该文档为了能拥有一套属于自己的代码规范，没有对错，只有适合与否。
- 本文大部分的理论经验参考《Clean Code》——Robert C.Martin，如有理解不对，还请斧正。
- 最后，会附上笔者对于一些``名词``、``动词对``和``形容词``的理解和使用场景。

目录
===

<!-- TOC -->

- [命名](#命名)
    - [代码简洁不代表模糊](#代码简洁不代表模糊)
    - [有意义的变量名](#有意义的变量名)
    - [避免使用编码或者前缀](#避免使用编码或者前缀)
    - [类名应该是清晰的名词或者名词短语，尽量不要用单一抽象名词](#类名应该是清晰的名词或者名词短语尽量不要用单一抽象名词)
- [函数](#函数)
    - [只在函数名的同一抽象层做一件事情，且函数应该自顶向下，不窜层。](#只在函数名的同一抽象层做一件事情且函数应该自顶向下不窜层)
    - [使用描述性名称](#使用描述性名称)
    - [命名方式要保持一致](#命名方式要保持一致)
    - [函数参数](#函数参数)
    - [动词与关键词](#动词与关键词)
    - [输出参数](#输出参数)
    - [分隔指令与询问](#分隔指令与询问)
    - [结构化编程](#结构化编程)
    - [如何写出心目中的函数](#如何写出心目中的函数)
- [附录](#附录)
    - [名词](#名词)
    - [动词](#动词)
    - [形容词](#形容词)

<!-- /TOC -->

# 命名
## 代码简洁不代表模糊
```C
/*
 * 不推荐
 */
uint8_t array[10];

/*
 * 推荐
 */
uint8_t students_id[10];
```
## 有意义的变量名
不要写a1，a2，a3，a，b，这样的变量名，除非在特定应用场景。
## 避免使用编码或者前缀
以前变量名带编码是因为编译器不会帮忙检查类型，要人工检查，现在编译器会帮忙检查类型，没必要增加冗余的东西。
```C
/*
 * 不推荐
 */
uint32_t  u32_student_id;
uint8_t   u8_student_weight;
uint8_t  *g_student_weight;
uint8_t  *g_student_height;

/*
 * 推荐
 */
uint32_t student_id;
uint8_t  student_weight;
```

## 类名应该是清晰的名词或者名词短语，尽量不要用单一抽象名词
我觉得在嵌入式行业会经常使用manager，controller等词，而每个人都会对它们有不同的定义，在不同场景它们也确实有不同的意义，所以就是会产生歧义和模糊性，除非对于这些名词有比较好的区分说明和规范，否则不太推荐使用，推荐使用大家能不产生歧义的名词。
```C
/*
 * 不推荐
 */
struct manager {
    ...
};

struct processor {
    ...
};

struct data {
    ...
};

struct info {
    ...
};

/*
 * 推荐
 */
struct student_info {
    uint32_t    weight;
    uint32_t    height;
    ...
};

enum customer_hobbies {
    FOOTBALL,
    BASKETBALL,
    ...
};

uint8_t dev_addr[6];
```

# 函数
## 只在函数名的同一抽象层做一件事情，且函数应该自顶向下，不窜层。

```C
/*
 * 一个抽象层（发送数据），做一件事情（发数据），推荐
 */
void send_fifo_data(void *fifo)
{
    if (!is_fifo_empty(fifo)) {
        send(fifo);
    }
}

/*
 * 两个抽象层（发送数据、fifo为空），做一件事情（发数据），不推荐
 */
void send_fifo_data(void *fifo)
{
    if (!is_fifo_empty(fifo)) {
        if (first_elem_in_fifo(fifo) == 0xa5) {
            send(fifo);
        }
    }
}

/*
 * 两个函数，两个抽象层，做了两件事情，函数自顶向下，不窜层，推荐
 */
void send_fifo_data(void *fifo)
{
    if (is_fifo_valid(fifo)) {
        send(fifo);
    }
}

uint8_t is_fifo_valid(void *fifo)
{
    if (is_fifo_empty(fifo)) {
        return 0;
    }    

    if (first_elem_in_fifo(fifo) == 0xa5) {
        return 1;
    }
}
```

## 使用描述性名称

函数内容应该尽可能短，但是函数名称没限制，只要能把事情描述清楚，长一点没问题。

## 命名方式要保持一致

```C

/*
 * 推荐
 */
uint32_t get_year();
uint32_t get_month();
uint32_t get_day();
uint32_t get_hour();

/*
 * 不推荐
 */
uint32_t get_year();
uint32_t month_get();
uint32_t fetch_day();
uint32_t read_hour();
```

## 函数参数

个数越少越好，最好没有。
- 参数多会导致单元测试出现很多分支，覆盖率复杂。
- 如果参数多于3个，说明需要定义一个结构体了。

## 动词与关键词

```C
/*
 * 一元函数，应该遵循：动词/名词结构比较好。
 */
void write(void *name)
void write_field(void *name)

/*
 * 二元函数，参数包含在函数名内，可以防止我们对于参数输入顺序出错，也是不错的选择
 */
void is_a_larger_b(void *a, void *b);
```

## 输出参数
尽量避免输出过多参数，可以用``对象参数``或者``返回值``来替代。
```C
/*
 * len是希望读到buf的个数，p_len是实际读到buf的个数
 */
void read_bytes(void *fifo, void *buf, uint8_t len, void *p_len);

/*
 * len是希望读到buf的个数，返回值是实际读到buf的个数
 */
uint32_t read_bytes(void *fifo, void *buf, uint8_t len);
```
## 分隔指令与询问
函数要么做什么事，要么回答什么事，不要两者兼得。
```C
/*
 * 设置名字属性，如果name已经存在，则返回1，否则返回0。（不推荐）
 */
uint8_t set_attribute(void *name, void *val)
{
    if (name) {
        return 1;
    }

    name = val;

    return 0;
}

/*
 * 分开两个函数比较合理，然后用以下形式实现。（推荐）
 */
if (!is_attribute_exist(name)) {
    set_attribute(name, val);
}
```

## 结构化编程
Edsger Dijkstra提出结构化编程，即每个函数，只能有一个入口和一个出口。
- 永远不能出现goto
- 循环中不能出现break和continue
- 每个函数只有一个return

这个原则用在复杂的大函数效果显著，在小函数可以适当出现return和break，但是goto还是不要出现。

## 如何写出心目中的函数

尝试 + 单元测试 + 提炼，循环上述步骤。

# 附录

## 名词
data
buf:buffer
fifo
status
cb:callback
res:result
str:string
suite
transmitter
receiver
msg:message
evt:event
ctx:context
field
number
amount
id
len:length
size
report:
rsp:response
req:request
cmd:command
ack:acknowledgement
this
self
hdr:header
hdl:handle
hdlr:handler
val:value
attribute
db:database
addr:address
sector
block
page
pkt:packet
create
update
delete
reserve
vers:version
type
kind
driver
manager
info:information
processor
master
slave
controller
dev:device

## 动词
req
get
read
write
fetch
store

trans
tx
rx
start
stop
begin
end
enter
exit
open
close
input
output

append
obj:object
erase
up
down
set
clr:clear
reset:rst
notify
contains
put
pull
remove
wait
add
sub
calc:calculate
peek
process
analyze
inquire
find
search
pend
post
imp:implement
indicate

## 形容词
equal
diff
expected
tmp
actual
real
virtual
larger
bigger
smaller
err:error
ok
success
fail
valid
invalid
busy
idle

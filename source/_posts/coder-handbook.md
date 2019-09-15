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

官方来说，代码好坏体现一个程序员的职业素养，在笔者看来代码是一个程序员的门面担当，在这个颜值即正义的时代，作为程序员如何码的一手好代码，是门必修课中的必修课。
<!-- more --> 


前言
===
- 写该文档为目的是：形成一套平时监督自己代码是否合理的规范。
- 本文大部分的理论经验参考《Clean Code》——Robert C.Martin，如有理解不对，还请斧正。
- 最后，会附上笔者对于一些``名词``、``动词对``和``形容词``的理解。

目录
===

<!-- TOC -->

- [1. 命名](#1-命名)
    - [1.1. 代码简洁不代表模糊](#11-代码简洁不代表模糊)
    - [1.2. 有意义的变量名](#12-有意义的变量名)
    - [1.3. 避免使用编码或者前缀](#13-避免使用编码或者前缀)
    - [1.4. 类名应该是清晰的名词或者名词短语，尽量不要用单一抽象名词](#14-类名应该是清晰的名词或者名词短语尽量不要用单一抽象名词)
- [2. 函数](#2-函数)
    - [2.1. 只在函数名的同一抽象层做一件事情，且函数应该自顶向下，不窜层。](#21-只在函数名的同一抽象层做一件事情且函数应该自顶向下不窜层)
    - [2.2. 使用描述性名称](#22-使用描述性名称)
    - [2.3. 命名方式要保持一致](#23-命名方式要保持一致)
    - [2.4. 函数参数](#24-函数参数)
    - [2.5. 动词与关键词](#25-动词与关键词)
    - [2.6. 输出参数](#26-输出参数)
    - [2.7. 分隔指令与询问](#27-分隔指令与询问)
    - [2.8. 结构化编程](#28-结构化编程)
    - [2.9. 如何写出心目中的函数](#29-如何写出心目中的函数)
- [3. 注释](#3-注释)
    - [3.1. 好注释](#31-好注释)
- [4. 对象和数据结构](#4-对象和数据结构)
    - [4.1. 数据抽象](#41-数据抽象)
    - [4.2. 数据结构、对象的反对称性](#42-数据结构对象的反对称性)
    - [4.3. LoD](#43-lod)
- [5. 异常处理](#5-异常处理)
    - [5.1. 别传递null值](#51-别传递null值)
- [6. 附录](#6-附录)
    - [6.1. 名词](#61-名词)
    - [6.2. 动词](#62-动词)
    - [6.3. 形容词](#63-形容词)

<!-- /TOC -->

# 1. 命名
## 1.1. 代码简洁不代表模糊

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

## 1.2. 有意义的变量名
不要写a1，a2，a3，a，b，这样的变量名，除非在特定应用场景。
## 1.3. 避免使用编码或者前缀
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

## 1.4. 类名应该是清晰的名词或者名词短语，尽量不要用单一抽象名词
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

# 2. 函数
## 2.1. 只在函数名的同一抽象层做一件事情，且函数应该自顶向下，不窜层。

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

## 2.2. 使用描述性名称

函数内容应该尽可能短，但是函数名称没限制，只要能把事情描述清楚，长一点没问题。

## 2.3. 命名方式要保持一致

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

## 2.4. 函数参数

个数越少越好，最好没有。
- 参数多会导致单元测试出现很多分支，覆盖率复杂。
- 如果参数多于3个，说明需要定义一个结构体了。

## 2.5. 动词与关键词

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

## 2.6. 输出参数
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

## 2.7. 分隔指令与询问
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

## 2.8. 结构化编程
Edsger Dijkstra提出结构化编程，即每个函数，只能有一个入口和一个出口。
- 永远不能出现goto
- 循环中不能出现break和continue
- 每个函数只有一个return

这个原则用在复杂的大函数效果显著，在小函数可以适当出现return和break，但是goto还是不要出现。

## 2.9. 如何写出心目中的函数

尝试 + 单元测试 + 提炼，循环上述步骤。

# 3. 注释

## 3.1. 好注释

- 对意图的注释

```C
void uart_receiver_fsm()
{
    uint8_t chr;

    if (is_fifo_not_empty(&fifo)) {
        chr = get_elem(&fifo);
        /*
         * 当收到第一个字节时候便打开接收超时定时器，
         * 用于计算接收超时
         */
        timer_start(0);
    }
}
```

- 阐释

```C
/*
 * 当a==b的时候，compare返回0
 */
if (comapre(a, b) == 0) {
    ...
}

/*
 * 当a > b(asc2)的时候，compare返回1
 */
if (comapre(a, b) == 1) {
    ...
}
```

# 4. 对象和数据结构

## 4.1. 数据抽象

- 抽象接口不是简单的取值器和赋值器

```C
/*
 * 不推荐，battery对象的接口只是将能量数值以函数形式暴露，
 * 这里的几个接口函数就是简单取值器，没有太多抽象作用。   
 */
typedef struct {
    uint32_t remained_energy_in_mw;
    ...
} battery_t;
uint32_t get_remained_energy_in_mw(battery_t *self);

/*
 * 推荐，battery对象的接口将能量抽象成百分比形式，
 * 没有暴露具体数值和单位。
 */
typedef struct {
    uint32_t remained_energy_in_mw;
    ...
} battery_t;
uint32_t get_remained_percent_energy(battery_t *self);
```

要以最好的形式呈现对象的数据，不能随意加取值期和赋值器。


## 4.2. 数据结构、对象的反对称性
- 过程式代码：暴露数据结构细节，便于在不改动数据结构基础上增添函数。
- 面向对象式代码：隐藏数据结构细节，便于在不改动函数的基础上增添新类。

过程式代码难修改数据结构，因为这样需要修改全部函数。面向对向式代码难修改函数，因为这样需要修改全部类。所以需要根据实际情况权衡什么时候使用哪种代码。

- 混杂对象和数据结构

混杂增加了添加新函数的难度，也增加了添加数据结构的难度，两面都不讨好。

```C
/*
 * 推荐，过程式代码，数据结构暴露数据，不提供有意义的函数
 * 优势：添加/修改新的函数不会影响数据结构（比如增加计算形状周长函数）
 * 缺陷：修改数据结构影响所有函数（比如增加一个新的形状）
 */
enum {
    SQUARE,
    RECTANGLE,
    CIRCLE,
};

typedef struct {
    uint8_t type;

    union {
        ...
    }
    ...
} shape_info_t;

uint32_t get_area(shape_info_t *shape_info)
{
    switch (shape_info->type) {
        case SQUARE:
            ...
            break;
        case RECTANGLE:
            ...
            break;
        case CIRCLE:
            ...
            break; 
    }
}

/*
 * 推荐，面向对象式代码，隐藏数据，暴露操作数据的函数（shape_get_area）
 * 优势：添加/修改新的数据结构很简单（比如增加一个长方形），对原来的函数没有任何影响。
 * 缺陷：如果需要增加一个获取形状周长的函数，那么全部形状都需要修改。
 * NOTE：这里的关系是is a，如果是has a关系，那么结构体变量类型shape_t应该是shape_t *
 */

#define PI 3.1415926

typedef uint32_t (*pfn_get_area_t)(void);

typedef struct {
    pfn_get_area_t get_area;
} shape_t;

typedef struct {
    shape_t shape_ops;
    uint32_t r;    
} circle_t;

typedef struct {
    shape_t shape_ops;
    uint32_t side;    
} square_t;

static uint32_t __cicrle_get_area(void)
{
    return PI * r * r;
}

static uint32_t __square_get_area(void)
{
    return side * side;
}

circle_t c1 = {
    .shape_ops = __cicrle_get_area,
};

square_t s1 = {
    .shape_ops = __square_get_area,
};

uint32_t shape_get_area(shape_t *shape)
{
    return shape->get_area();
}

void main()
{
    shape_get_area((shape_t *)&c1);
    shape_get_area((shape_t *)&s1);   
}

/*
 * 不推荐，一半对象，一半数据结构，既拥有函数，又拥有公共变量。
 * 缺陷：添加新函数难，添加新数据结构难。
 * 如果需要添加形状（即公共变量改变），则访问公共变量的函数都要修改
 * 如果需要新增一个获取周长的抽象方法（即增加执行函数），则所有形状对象都要修改
 */
enum {
    SQUARE,
    RECTANGLE,
    CIRCLE,
};

typedef struct {
    uint8_t type;
    uint8_t color_var1;
    uint8_t color_var2;
} shape_info_t;

typedef uint32_t (*pfn_get_color_t)(void);

typedef struct {
    shape_info_t shape_info;    //公共变量
    pfn_get_color_t get_color;  //执行函数
} shape_t;

typedef struct {
    shape_t shape;
    uint32_t r;    
} circle_t;

typedef struct {
    shape_t shape;
    uint32_t side;    
} square_t;

static uint32_t __circle_get_color(void)
{
    return color_val1 * color_val2;
}

static uint32_t __square_get_color(void)
{
    return color_val1 + color_val2;
}

uint8_t shape_get_color(shape_t *shape)
{
    return shape->get_color();
}

uint32_t shape_get_area(shape_t *shape)
{
    /*
     * 访问公共变量 
     */
    switch (shape->shape_info.type) {
        case SQUARE:
            ...
            break;
        case RECTANGLE:
            ...
            break;
        case CIRCLE:
            ...
            break; 
    }
}

circle_t c1 = {
    .shape_info.type = CIRCLE,
    ...
};

square_t s1 = {
    .shape_info.type = SQUARE,
    ...
};

void main()
{
    shape_get_color((shape_t *)&c1))
    shape_get_area((shape_t *)&c1))
}
```

## 4.3. LoD

Law of demeter认为，模块不应该了解它所操作对象的内部情形。

- 火车失事（只关乎于编码风格，像火车一样的代码）

```C
/*
 * 不推荐，像火车一样的代码
 */
output_dir = ctxt.get_options().get_scratch_dir().get_abs_path();

/*
 * 推荐（对象）
 */
ops = ctxt.get_options();
scratch_dir = ops.get_scratch_dir();
abs_path = scratch_dir.get_abs_path();

/*
 * 如果ctxt、ops等只是单纯数据结构，而不是对象则没有违反LoD
 */
output_dir = ctxt.ops.scratch_dir.abs_path
```

# 5. 异常处理
## 5.1. 别传递null值
调用者可能会意外传入null，自己写代码时候应该不写可以传入null的函数，这样只要发现函数列表中有null就是出问题了。




# 6. 附录

概念解释的来源主要有
- 百度翻译中的牛津词典
- google翻译
- stackoverflow高赞答案
- wikidiff.com
- 笔者理解
- 其他地方

## 6.1. 名词

- acknowledge
    - 解释：确认帧/应答
    - 缩写：ack
    - 关联词：nack

- address
    - 解释：地址
    - 缩写：addr

- amount
    - 解释：合计总量，表示量
    - 关联词：number

- argument
    - 解释：Argument is the actual value of this variable that gets passed to function.
    - 缩写：arg
    - 关联词：parameter
```C
void foo(void *param);

...
uint8_t arg1[] = "this is my argument";
foo(arg1);
```

- attribute
    - 解释：强调事物固有的属性，或区别其他事物的特征。比如车是红色，某某品牌
    - 缩写：attr
    - 关联词：property

- backup
    - 解释：备份
    - 缩写：bk
    
- buffer
    - 解释：短暂数据存储的地方
    - 缩写：buf
    - 关联词：fifo

- callback
    - 解释：邀请返回做某事，带点因果关系的意思。
    - 缩写：cb
    - 关联词：handler

- case
    - 解释：具体情况/事例/特殊情况
    
- command
    - 解释：命令/指令
    - 缩写：cmd
    - 关联词：event

- content
    - 解释：所含之物/内容

- context
    - 解释：事情发生的背景/上下文环境

- data
    - 解释：存储在计算机中的资料/原始数据/调查资料/材料
    - 同义词：information/results/statistics

- device
    - 解释：用于做某项工作的对象或者仪器
    - 缩写：dev
    
- driver
    - 解释：software that controls the sending of data between a computer and a piece of equipment that is attached to it, such as a printer

- event
    - 解释：发生的事情
    - 缩写：evt

- fail
    - 解释：失败

- field
    - 解释：字段/信息组

- fifo
    - 解释：先进先出算法/先进先出缓冲区
    - 关联词：lifo

- frame
    - 解释：帧有格式，包括帧头+数据部分+帧尾
    - 关联词：pakcet

- header
    - 解释：帧头
    - 缩写：hdr

- handle
    - 解释：事物控制的部分
    - 缩写：hdl

- handler
    - 解释：处理某些事物的人
    - 缩写：hdlr

- identity
    - 解释：身份/本体
    - 缩写：id

- information
    - 解释：事实/某些事物的细节
    - 缩写：info

- kind
    - 解释：a group of people or things that are the same in some way; a particular variety or type，如某个种类的音乐。
    - 关联词：type
    
- length
    - 解释：长度/距离/持续时间的长短
    - 缩写：len     

- message
    - 解释：（书面或口头的）信息，消息，音信
    - 缩写：msg

- nack
    - 解释：表示报文有错误，要求重发。
    - 关联词：ack/syn

- master
    - 解释：具有控制力的角色
    - 关联词：slave

- number
    - 解释：一个符号，代表数字/序号/数，表示数
    - 缩写：num
    - 关联词：amount

- packet
    - 解释：a piece of information that forms part of a message sent through a computer network
    - 缩写：pkt
    - 关联词：frame/payload

- parameter
    - 解释：Parameter is variable in the declaration of function
    - 缩写：param

- payload
    - 解释：有效载荷，是frame中的一部分

- property
    - 解释：强调“拥有”的参数，比如车有颜色属性，有尺寸大小属性。
    - 关联词：attribute

- receiver
    - 解释：接收者
    - 缩写：rx
    
- report
    - 解释：新闻/报告

- request
    - 解释：(正式礼貌的)请求和要求的事
    - 缩写：req

- response
    - 解释：(口头或书面的)回复/响应
    - 缩写：rsp

- result
    - 解释：后果/结果
    - 缩写：res

- self
    - 解释：指向当前类的指针
    - 关联词：this

- slave
    - 解释：受控制的角色
    - 关联词：master

- sender
    - 解释：邮寄人 

- size
    - 解释：抽象大小概念/标定大小尺寸

- status
    - 解释：(进展中的)状况/情形

- string
    - 解释：字符串，带'\0'结尾。
    - 缩写：str
    - 关联词：bytes

- succeess
    - 解释：成功

- syn
    - 解释：在发消息之前，需要先同步。（tcp）
    - 关联词：ack/nack

- this
    - 解释：指向当前对象的指针
    - 关联词：self

- transmitter
    - 解释：发射机/发射者
    - 缩写：tx
    - 关联词：sender/receiver

- type
    - 解释：a class or group of people or things that share particular qualities or features and are part of a larger group，如不同的人种
    - 关联词：kind

- value
    - 解释：由代数项表示的数值、数量
    - 缩写：val
    - 关联词：data

- variable
    - 解释：可变因素
    - 缩写：var

- version
    - 解释：版本
    - 缩写：vers
 
- manager
    - 解释：待定
- processor
    - 解释：待定
- controller
    - 解释：待定

- block
    - 解释：由sector组成
- sector
    - 解释：由page组成
- page
    - 解释：SPI FLASH最小写操作单元

- 关于setting/option/preference/property/configuration的理解：

    Someone style:

    - Settings: "View or modify the list of things that can     be set"
    - Options: "We have set some things already, and give   you the option to enable or disable them"
    - Preferences: "Tell us how you prefer this to work"
    - Properties: "Change one or more properties of this    item"
    - Configuration: "We have defaults, but they're so  barebones you probably want to configure it yourself"

    Following an approximate lead from Visual Studio and    other Microsoft products:

    - Properties represent the characteristics of a single  component or object in the application.
    - Options alter global ways that the application works.     Microsoft products use this to customise the UI toolbar,     for example. There's an implication here that you can  disable UI elements altogether (e.g. a "Simple" user     interface or an "Advanced" user interface).
    - Settings and Preferences change qualities of how the  application works. The implication here is to change,    not disable: for example, "Metric measurements" or     "British Imperial measurements".
    - Configuration is often where an application is    customised for each user or group.

    知乎：

        Configure some options in the settings.

    - 程序所有的可变项叫做Settings。中文译作设置。所有的设置都是    “可选项”，Option（选项，不是期权），因为在程序世界里没有真的    开放式问题只有选择题（Option）。改变可选项的过程叫做    Configure配置（动词）。
    - 附赠：已确定的Configure结果叫做Configuration配置（名词）  。Configure和Configuration都经常缩写为Config。一套既成的可    迁移的实现特定目的的Configuration叫做Profile，例如手机里的  静音Profile、仅震动Profile、蓝牙各种profile。


## 6.2. 动词

- 持久层/数据库操作
    - create
        - 创建新的记录
    - read
        - 读已存在的记录
    - update
        - 更新已存在的记录
    - delete
        - 删除已存在的记录

- 整体和个体的访问操作
    - put
        - 将对象或者任意数据存进去（对象大小由sizeof决定）
    - get
        - 将对象或者任意数据读出来（对象大小由sizeof决定）
    - write
        - 将n个字节写进去
    - read
        - 将n个字节读出来

- 恢复/复原/重置
    - recover
        - 解释：从备份中获取部分文件

    - restore
        - 解释：从备份中获取整个系统

    - refresh
        - 解释：重装系统，但是保留应用程序和个人设置文件

    - reset
        - 解释：重装系统，删除所有东西
        - 缩写：rst

- 查询/寻找/搜索
    - inquire
        - 解释：在某个范围内查询某指定事物（ask sth）
    - find
        - 解释：偶然遇到或者发现特定事物
    - search
        - 解释：I searched on the internet. I found what I was looking for.（指定性没那么强）

- 过滤/拦截/排列
    - filter
        - 解释：pass (a liquid, gas, light, or sound) through a device to remove unwanted material.
        - 关联词：filter（名词）
    - intercept
        - 解释：to stop sb/sth that is going from one place to another from arriving
        - 关联词：interceptor（名词） 
    - sort
        - 解释：按顺序排列


- 学习/研究/分析
    - study
    - learn
    - research
        - 解释：不断寻找和检查来研究某事物
    - analyze
        - 解释：对当前主题进行分析

- 覆盖
    - cover
        - 解释：把某事物放在某事物上面，如隐藏和保护
    - overlap
        - 解释：延伸并部分覆盖或者替代掉某物

- 固定搭配
    - fetch
        - 解释：获取东西
        - 关联词：store   
    - store
        - 解释：将东西保存起来以备以后使用
        - 关联词：fetch
    
    - start
        - 解释：较随意的开始/创立
    - stop
        - 解释：较随意的结束

    - begin
        - 解释：从头开始
    - end
        - 解释：终点结束

    - enter
        - 解释：加入/进入/开始从事/开始活动
    - exit
        - 解释：退出（计算机程序）

    - open
        - 解释：参考文件系统使用方法
    - close
        - 解释：参考文件系统使用方法

    - input
        - 解释：第三人称描述某个事物的输入
        - 关联词：receive（第一人称）
    - output
        - 解释：第三人称描述某个事物的输出
        - 关联词：transmit（第一人称）
   
    - upload
        - 解释：上传到服务器或者别的电脑
    - download
        - 解释：从服务器或者别的电脑下载
        
    - set
        - 解释：设置
    - clear
        - 解释：清除
        - 缩写：clr

    - init
        - 解释：是一个实例的初始化方法
    - deinit
        - 解释：释放内存

    - plus
        - 解释：加法
    - minus
        - 解释：减法

    - subscribe
        - 解释：订阅
        - 缩写：subs
    - publish
        - 解释：发布
        - 缩写：pub

    - add
        - 解释：增加某东西
    - sub
        - 解释：减去某东西

    - pend
        - 参考操作系统
    - post
        - 参考操作系统

    - error
        - 解释：err
        - 缩写：
    - ok
        - 解释：
        - 缩写：



- append
    - 解释：在结尾插入内容


- calculate
    - 解释：计算
    - 缩写：calc

- contain
    - 解释：包含/含有/容纳

- erase
    - 解释：flash专用

- indicate
    - 解释：指示
    - 关联词：confirm

- initialize
    - 解释：类的方法，在所有实例方法和类方法执行前运行

- notify
    - 解释：正式通报，通知

- implement
    - 解释：实现
    - 缩写：impl

- remove
    - 解释：拿走，但内存还在
    - 关联词：delete（删除释放内存）、recover

- transfer
    - 解释：移交/转移

- translate
    - 解释：转变/变为

- overflow
    - 解释：溢出某个容器
    
- peek
    - 解释：偷看一眼

- process
    - 解释：对data作一些的操作或处理


## 6.3. 形容词

- 固定搭配
    - valid
        - 解释：有效的
    - invalid
        - 解释：无效的

    - busy
        - 解释：忙
    - idle
        - 解释：空闲

    - used
        - 解释：已使用
    - unused
        - 解释：没用着的/空闲的



- 描述真实/真的
    - actual
        - 解释：形容客观存在的事实或者行为
    - real
        - 解释：真实的/实际存在的，对真理的描述

- 描述大小
    - larger
        - 解释：强调体积/能力/数量
    - bigger
        - 解释：表示由“重”的意思，重要/重量。
    - smaller
        - 解释：无关紧要的/小的数量
    - little
        - 解释：小的尺寸/只有一点点
    - tiny
        - 解释：极小的/微小的/微量的

- different
    - 解释：不同
    - 缩写：diff

- equal
    - 解释：大小、价值、数量相等或相同

- expected
    - 解释：预料的/预期的

- initial
    - 解释：初始化的

- remain
    - 解释：仍然存在/可以使用/还没被处理

- temp
    - 解释：临时
    - 缩写：tmp

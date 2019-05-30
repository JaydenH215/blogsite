---
title: cm_backtrace学习笔记
date: 2019-05-24 15:31:52

tags:
- hardfault
- cortex
- debug
- 栈帧回溯

categories:
- 术业专攻

---


[龙神](https://github.com/armink/CmBacktrace)的debug组件学习笔记。
<!-- more --> 


前言
===
最近了解cortexM的hardfault回溯，记得之前有前辈写过一个组件，趁这次机会拜读一下，并且记录下学习过程。

# cmb_fault.s汇编文件

组件支持IAR/KEIL/GNU，因为最近用到KEIL，所以就直接看KEIL的汇编文件了，下面摘取了一部分的代码。之前对这块没有好好的总结过，所以这次简单总结一下。

        AREA |.text|, CODE, READONLY, ALIGN=2
        THUMB
        REQUIRE8
        PRESERVE8

    ; NOTE: If use this file's HardFault_Handler, please comments the HardFault_Handler code on other file.
        IMPORT cm_backtrace_fault
        EXPORT HardFault_Handler

    HardFault_Handler    PROC
        MOV     r0, lr                  ; get lr
        MOV     r1, sp                  ; get stack pointer (current is MSP)
        BL      cm_backtrace_fault

    Fault_Loop
        BL      Fault_Loop              ;while(1)
        ENDP

        END

源码实现的功能：

1. 当hardfault发生时，将lr和sp作为参数传递给cm_backtrace_fault；
2. cm_backtrace_fault处理完之后，进入死循环。

## QA：

**Q：AREA |.text|, CODE, READONLY, ALIGN=2的意义？**

- AREA：asm指令，参考DUI0379G_02_mdk_armasm_user_guide文档
- |.text|：|.text| is used for code sections produced by
the C compiler, or for code sections otherwise associated with the C library.
- CODE：Contains machine instructions. READONLY is the default
- READONLY：Indicates that this section must not be written to. This is the default for Code areas.
- ALIGN=2：By default, ELF sections are aligned on a four-byte boundary. expression can have
any integer value from 0 to 31. The section is aligned on a 2expression-byte boundary. For
example, if expression is 10, the section is aligned on a 1KB boundary.
This is not the same as the way that the ALIGN directive is specified.

> Q：为什么需要4字节对齐？我也不知道




**Q：THUMB的意义？**

参考《DUI0379G_02_mdk_armasm_user_guide》有还怎么一些话

>The THUMB directive instructs the assembler to interpret subsequent instructions as Thumb instructions,using the UAL syntax. In files that contain code using different instruction sets, THUMB must precede Thumb code written in
UAL syntax.

《The Definitive Guide to theARM Cortex-M0》有这么一段话：

>Be careful with legacy Thumb programs that use the CODE16 directive. When the CODE16
directive is used, the instructions are interpreted as traditional Thumb syntax. For example,
data processing op-codes without S suffixes are converted to instructions that update
APSR when the CODE16 directive is used. However, you can reuse assembly files with the
CODE16 directive because it is still supported by existing ARM development tools. For
new assembly code, the THUMB directive is recommended, which indicates to the assembly
that the Unified Assembly Language (UAL) is used. With UAL syntax, data processing
instructions updating the APSR require the S suffix.


**Q：REQUIRE8和PRESERVE8的意义？**

《The Definitive Guide to theARM Cortex-M0》有这么一段话：

>In ARM/Keil development tools, the assembler provides the REQUIRE8 directive to indicate if
the function requires double-word-stack alignment and the PRESERVE8 directive to indicate
that a function preserves the double-word alignment. This directive can help the assembler
to analyze your code and generate warnings if a function that requires a double-word-aligned
stack frame is called by another function that does not guarantee double-word-stack alignment.
Depending on your application, these directives might not be required, especially for projects
built entirely with assembly code.

# 栈帧回溯功能分析

```c
/**
 * backtrace function call stack
 *
 * @param buffer call stack buffer
 * @param size buffer size
 * @param sp stack pointer
 *
 * @return depth
 */
size_t cm_backtrace_call_stack(uint32_t *buffer, size_t size, uint32_t sp) {
    
    ...

    /* copy called function address */
    for (; sp < stack_start_addr + stack_size; sp += sizeof(size_t)) {
        /* the *sp value may be LR, so need decrease a word to PC */
        /* 
         *   假设用户调用了这么一段断言代码：
         * 
         *  void usr_assert(uin8_t zoo)
         *  {
         *       uint32_t sp = cmb_get_msp();
         *       cm_backtrace_assert(sp);
         *       foo(zoo);
         *   }
         * 
         *   那么在进入cm_backtrace_assert的时候，会将foo的地址写进LR，然后将
         *   LR进栈，所以SP指向LR，LR-4刚好就是cm_backtrace_assert的地址 
         *   (因为BL跳转指令刚好占4个字节）。
         *   然后从该地址开始轮询已经入栈的数据，看看被压栈里的地址哪个是在代码区
         *   间内的，找到就将他们保存进buffer。
         * 
         *   这里的回溯原理：找到调用栈中在代码区间内的地址
         *   需要注意：code_start_addr和code_size是否定义正确，决定了是否能正
         *   确找到调用的函数地址，这个和分散加载（链接脚本）有密切关系。
         */
        pc = *((uint32_t *) sp) - sizeof(size_t);
        /* the Cortex-M using thumb instruction, so the pc must be an odd number */
        if (pc % 2 == 0) {
            continue;
        }
        if ((pc >= code_start_addr) && (pc <= code_start_addr + code_size) && (depth < CMB_CALL_STACK_MAX_DEPTH)
                && (depth < size)) {
            /* the second depth function may be already saved, so need ignore repeat */
            if ((depth == 2) && regs_saved_lr_is_valid && (pc == buffer[1])) {
                continue;
            }
            buffer[depth++] = pc;
        }
    }

    return depth;
}
```






# 参考资料
- 《The Definitive Guide to theARM Cortex-M0》
- 《DUI0379G_02_mdk_armasm_user_guide》
- 《DUI0497A_cortex_m0_r0p0_generic_ug》
- 《DDI0419C_arm_architecture_v6m_reference_manual》
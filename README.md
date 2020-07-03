# study-os

week 1: finish lab0 and establish ucore environment, read the first and second lecture.  
列出本实验各练习中对应的OS原理的知识点，并说明本实验中的实现部分如何对应和体现了原理中的基本概念和关键知识点。  
      
      lab1：  
      练习1：  
      运行make "V="
      先生成bootloader，再生成kernel，
      为了生成bootblock，首先需要生成bootasm.o、bootmain.o、sign
      为了生成kernel，首先需要 kernel.ld init.o readline.o stdio.o kdebug.o
      从sign.c的代码来看，一个磁盘主引导扇区只有512字节。且第510个（倒数第二个）字节是0x55，第511个（倒数第一个）字节是0xAA。
      练习2：  
      bootloader做的事：
      切换到保护模式，启用分段机制 （修改A20地址线），读磁盘中ELF执行文件格式的ucore操作系统到内存，显示字符串信息，把控制权交给ucore操作系统
      练习3：  
      重要的段寄存器清零，开启A20，初始化GDT
      从实模式切换至保护模式：通过将cr0寄存器PE位（第0位）置1便开启了保护模式；通过长跳转更新cs的基地址；设置段寄存器，并建立堆栈（BIOS数据区：0x0000-0x7c00）
      使用引导程序GDT和段转换，使虚拟地址与物理地址相同，从而从实模式切换到保护模式
      练习4：  
      bootloader如何读取硬盘扇区的？：
      在reasect函数中，读一个扇区：insl(0x1F0, dst, SECTSIZE / 4); 
      bootloader是如何加载ELF格式的OS？：
      从硬盘读了8个扇区数据到内存0x10000处，并把这里强制转换成elfhdr使用。
      校验e_magic字段。
      根据偏移量分别把程序段的数据读取到内存中。
      练习5：  
     
      uint32_t eip = read_eip();
      int m,n;    
        for(m=0; m<STACKFRAME_DEPTH&&ebp!=0;m++) {
        cprintf("ebp:0x%08x ",ebp);
        cprintf("eip:0x%08x ",eip);
        uint32_t* args = (uint32_t*)ebp + 2;
        cprintf("args:");
        for(n=0;n<4;n++){
            cprintf("0x%08x ",args[n]);
        }     
        cprintf("\n");
        print_debuginfo(eip-1);
        eip = ((uint32_t*)ebp)[1];
        ebp = ((uint32_t*)ebp)[0];      
        }

      练习6：中断描述符表一个表项占用8字节，其中2-3字节是段选择子，0-1字节和6-7字节拼成位移，两者联合便是中断处理程序的入口地址
       void idt_init(void) {
    extern uintptr_t __vectors[]; 
    int i;
    for(i=0; i<sizeof(idt)/sizeof(struct gatedesc);i++){
        SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
    } 
    SETGATE(idt[T_SWITCH_TOK], 0, GD_KTEXT, __vectors[T_SWITCH_TOK], DPL_USER);
    lidt(&idt_pd);
}

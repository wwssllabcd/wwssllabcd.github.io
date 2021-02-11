---
title: 從零開始的 Gameboy 模擬器開發 -- Step 2
date: 2021-02-10 00:24:46
tags: [How-To, c, emulator]
toc: true
type: adsense
---

建立 opcode map 
========

在開始之前讀取第一道指令之前，我們要做一件很不有趣的事情，就是建立所有 op code 的 map

首先我們建立 opcode_map.c，裡面的內容如下
```
void not_support_cb_code() { ASSERT_CODE(0, "not support CB code"); }
void not_support_op_code() { ASSERT_CODE(0, "not support op code"); }

void op_00() { not_support_op_code(); }
.......
void op_FF() { not_support_op_code(); }

void op_cb_00() { not_support_cb_code(); }
.......
void op_cb_FF() { not_support_cb_code(); }

opcode_fun g_opcode_fun_map[0x100] = {
    GEN_FUN_MAP(, 0), GEN_FUN_MAP(, 1), GEN_FUN_MAP(, 2), GEN_FUN_MAP(, 3),
    GEN_FUN_MAP(, 4), GEN_FUN_MAP(, 5), GEN_FUN_MAP(, 6), GEN_FUN_MAP(, 7),
    GEN_FUN_MAP(, 8), GEN_FUN_MAP(, 9), GEN_FUN_MAP(, A), GEN_FUN_MAP(, B),
    GEN_FUN_MAP(, C), GEN_FUN_MAP(, D), GEN_FUN_MAP(, E), GEN_FUN_MAP(, F)
};

opcode_fun g_opcode_cb_fun_map[0x100] = {
    GEN_FUN_MAP(cb_, 0), GEN_FUN_MAP(cb_, 1), GEN_FUN_MAP(cb_, 2), GEN_FUN_MAP(cb_, 3),
    GEN_FUN_MAP(cb_, 4), GEN_FUN_MAP(cb_, 5), GEN_FUN_MAP(cb_, 6), GEN_FUN_MAP(cb_, 7),
    GEN_FUN_MAP(cb_, 8), GEN_FUN_MAP(cb_, 9), GEN_FUN_MAP(cb_, A), GEN_FUN_MAP(cb_, B),
    GEN_FUN_MAP(cb_, C), GEN_FUN_MAP(cb_, D), GEN_FUN_MAP(cb_, E), GEN_FUN_MAP(cb_, F)
};
```
其中 GEN_FUN_MAP() macro 是幫你產生 op_00(), op_01 ... op_FF 的格式，因為一個一個打實在太累了，當然你有毅力我也不反對

<!--more-->

這邊的重點放在兩個 array, 分別是 `opcode_fun g_opcode_fun_map[0x100]` 與 `opcode_fun g_opcode_cb_fun_map[0x100]`，也就是說，我們使用一個 function pointer 的 array，然後使用該 array 的 index 當作 op code 呼叫號碼，這樣就不用寫落落長的 switch case 了，有了 opcode 了，我們就可以開始 fetch 指令了。新增以下的 code 到 cpu.c 中

```
eu8 fetch(void) {
    eu8 val = get_ram(REG_PC);
    INC_REG(pc);
    return val;
}

eu8 execute_opcode() {
    eu8 opcode = fetch();
    opcode_fun_usp op_map = g_opcode_fun_map;
    bool is_cb_cmd = false;

    if (opcode == PREFIX_CMD) {
        opcode = fetch();
        op_map = g_opcode_cb_fun_map;
        is_cb_cmd = true;
    }

    op_map[opcode]();
    return 1;
}

eu8 cpu_tick() {
    return execute_opcode();
}

void tick() {
    cpu_tick();
}

void run_cpu() {
    while (g_cpu.running) {
        tick();
    }
}
```

- fetch() 其實就是以 pc 當作記憶體位置，然後把該記憶體位置的值取出來，取出來後 pc 就要步進1
- Gameboy 的 op code 有兩種，一種是 normal 的，一種是老任新增的 -- 以 0xCB 做前綴的 external op code ，所以我們這邊一旦遇到 cb cmd 的時候，就使用 cb cmd 的 function array，反之則用正常的 function array

----

boot code 的選擇
========

好，在開始之前，還有一件事很重要就是，這顆 cpu 的 pc 是從 0 開始讀，然後我們又把 game rom 放到 0 ~ 32k 的地方，理論上 pc = 0 開始讀的位置是 game rom 的第 0 byte，但是呢，如果你有注意到的話，其實 address 0 ~ 255 這個位置會跟 boot rom 重疊，也就是說 boot rom 與 game com 在 address 0 ~ 255 byte 的地方產生了 overlay

那既然 overlay 了，那我去讀 address 0，到底是讀到的是 game rom 的第一個 byte，還是 boot rom 的第一個 byte 呢? 為了讓 fw 可以選擇使用哪個 code，所以有個 0xFF50 這個 boot_rom_disable 的 register  會做切換的功能，當我們要存取 0 ~ 255 的位置時，就要先去看一下這個 reg，當 boot_rom_disable 設為 0 時，我們的 mmu 就要送出 boot rom 的值，反之就會讀到 game rom 的位置，這個 reg 預設是讀 boot rom， 而一旦當要開始執行遊戲畫面時，fw 就會把這個 register 切到 1，這樣 mmu 就必須把這塊記憶體 mapping 到 game rom 去

那這個 isp 的功能要實作也很簡單，就是在額外宣告一個 eu8 boot_rom_dmg[0x100] ，並且在 get_ram_ptr 動手腳就可以了

```
eu8_p get_ram_ptr(RamAddr address) {
    if (address < BOOT_ROM_LENGTH) {
        if (g_zero_page->boot_rom_disable == 0) {
            return (eu8_p)&boot_rom_dmg[address];
        }
    }
    return (eu8_p)&g_ram[address];
}
```

如果 ok 之後，我們就可以正式的 fatch 第一個指令了，此時順利的話，你會 fetch 第一個 op code ，這個是從 boot rom 提出來的，號碼為 0x31，此時如果繼續執行的話會碰到 error ，因為我們在所有沒有實作的 cmd 裡面都加上了 not_support_op_code()，進而會引發 assert，所以我們的目標就是把所有的 cmd 給做出來

----

建立起 256 + 256 個 opcode
========

這邊雖然一共有 normal op + external op 最多 512 道指令，但你如果仔細觀察的話，其實很多指令都相同，只是來源 register 或是目標 register 不同，我的建議是按照手冊一個一個做下去，這樣在實作的時候也比較不會亂

回到教學這邊，雖然每個 cmd 都是短短的，即便如此，若講解每道指令的話會使用很多篇幅，所以這邊只會挑重點講

首先我們先做點前置作業，在 cpu.c 中加入等下會用到的 function，如下所示

```
void disable_halt() {
    g_cpu.halt = false;
}

void enable_halt() {
    g_cpu.halt = true;
}

void stack_push(WordReg_p reg) {
    DEC_REG(sp);
    set_ram(REG_SP, REG_VAL(REG_HIGH(reg)));
    DEC_REG(sp);
    set_ram(REG_SP, REG_VAL(REG_LOW(reg)));
}

void stack_pop(WordReg_p reg) {
    REG_VAL(REG_LOW(reg)) = get_ram(REG_SP);
    INC_REG(sp);
    REG_VAL(REG_HIGH(reg)) = get_ram(REG_SP);
    INC_REG(sp);
}

eu16 fetch_word() {
    eu16 val = 0;
    val |= fetch();
    val |= fetch() << 8;
    return val;
}

eu8 cpu_tick() {
    if (g_cpu.halt) {
        return 1;
    }
    return execute_opcode();
}
```

你可以看到其實實作的方式都很簡單， fetch_word 就是使用 fetch 兩次，而 stack_push 就是先把 sp 減一後，在把 pc 所指向的 ram 的值讀出來放到 sp 所指的位置，stack 的操作大多是以 WordRegister 的方式
而 stack_pop 就是反過來做，此外，我們也增加了 halt 的 function ，並在原本的 cpu_tick() 中，加入 check g_cpu.halt 來模擬 halt 的指令

-----

LD, r1 r2 指令
-----------

這個指令是最多使用的，從 op code 0x40~0x7F 都是用它做出來的，所以一下子 64 個指令就完成了，
他的實作方式也很簡單，就是把 r2 這個 byte ，設給 r1 這個 byte register

而有時候設值目標不是 register ，而是記憶體位置，所以這邊就又弄一個 opcode_ld_r1_r2_addr() ，C 不支援同名異式的 Polymorphism ，所以抽象表現力就比較差一點，不過對於這個小專案來說還是可以忍受的範圍內就是了，當然 reg 與記憶體位置你也可以看成都是這隻模擬器的記憶體位置，真的發狠起來要搞合併也不是不行，只是這樣 code 會變成更難理解，這樣就本末倒置了

```
void opcode_ld_r1_r2(ByteReg_p r1, eu8 r2) {
    REG_VAL(r1) = r2;
}
void opcode_ld_r1_r2_addr(RamAddr address, eu8 r2) {
    set_ram(address, r2);
}
```

這邊也稍微貼一下他在 opcode_map.c 中的樣子
```
void op_40() { opcode_ld_r1_r2(b, REG_B); }
void op_41() { opcode_ld_r1_r2(b, REG_C); }
void op_42() { opcode_ld_r1_r2(b, REG_D); }
void op_43() { opcode_ld_r1_r2(b, REG_E); }
void op_44() { opcode_ld_r1_r2(b, REG_H); }
void op_45() { opcode_ld_r1_r2(b, REG_L); }
void op_46() { opcode_ld_r1_r2(b, RAM_VAL_HL); }
....
void op_48() { opcode_ld_r1_r2(c, REG_B); }
void op_49() { opcode_ld_r1_r2(c, REG_C); }
void op_4A() { opcode_ld_r1_r2(c, REG_D); }
void op_4B() { opcode_ld_r1_r2(c, REG_E); }
void op_4C() { opcode_ld_r1_r2(c, REG_H); }
void op_4D() { opcode_ld_r1_r2(c, REG_L); }
void op_4E() { opcode_ld_r1_r2(c, RAM_VAL_HL); }
```

-------

bit b, r，res, b r 與 set, b r 指令
---------

在 cb cmd 中，也是有跟 opcode_ld_r1_r2 一樣廣泛使用的 cmd ，那就是 opcode_cb_bit_b_r() ， opcode_cb_res_b_r()與 opcode_cb_set_b_r()，他們也是各占 64 個 opcode，算是很補的 cmd， 作法如下
```
void opcode_cb_bit_b_r(eu8 val, eu8 bit) {
    set_z(CHECK_BIT(val, bit) == 0);
    set_n(false);
    set_h(true);
}

void opcode_cb_res_b_r(ByteReg_p reg, eu8 bit) {
    CLEAR_BIT(REG_VAL(reg), bit);
}

void opcode_cb_res_b_r_addr(RamAddr addr, eu8 bit) {
    eu8 res = get_ram(addr);
    CLEAR_BIT(res, bit);
    set_ram(addr, res);
}

void opcode_cb_set_b_r(ByteReg_p reg, eu8 bit) {
    SET_BIT(REG_VAL(reg), bit);
}
```

這3個 function 只有  opcode_cb_bit_b_r 需要注意的就是，你必須要開始設定 flags，不過這3個 function 的 flag 比較沒那麼複雜，我挑一個比較複雜的運算來討論一下

-------

add_a_n 與 adc_a_n 指令
---------

以下是 opcode 0x80 ~ 0x87 的 add_a_n 與 opcode 0x88 ~ 0x8F 的 adc_a_n 的實作，順帶一提的是，這邊 function 的命名規則是，如果是對應到 op code 的話就會以 "opcode_ " 為開頭，如果沒這開頭的就都是 base function 了

```
bool check_hc_add(eu32 summand, eu32 addend, eu32 mask) {
    return ((summand & mask) + (addend & mask) > mask);
}

eu32 add_a_b(eu32 summand, eu32 addend, eu32 mask) {
    eu32 result = summand + addend;

    if (mask == 0xFF) {
        set_z((result & mask) == 0);
    }
    
    set_n(false);
    set_h(check_hc_add(summand, addend, mask >> 4));
    set_c(result > mask);

    return result;
}

void opcode_add_a_n(eu8 n) {
    REG_VAL(a) = (eu8)add_a_b(REG_VAL(a), n, 0xFF);
}

void opcode_adc_a_n(eu8 n) {
    opcode_add_a_n(n + flags->c);
}
```

opcode_adc_a_n 與 opcode_add_a_n 其實是由 add_a_b 組成，這邊可看到 add 指令對 flag 的變化，在大部分的狀況規則整理如下

* flag->z: 是用來判斷運算結果是不是為0
* flag->n: 代表是不是減法
* flag->h: 代表 bit 4 有沒有產生變化，也就是 halt carry 的意思
* flag->c: 就是代表 carry，也就是運算結果本身有無溢位

對於 flag->c 來說，要判斷兩個值相加是否 overflow，所以就直接設了一個 eu32 result 的變數來存放相加過的值，並且判斷相加後是否超過 0x100 即可，反正我們是 32 位元的 cpu，要觀察兩個 8 bit regsiter 相加後是否溢位其實是一塊小蛋糕

這邊要注意的是, add u8 與 add u16 對於 flag 的設定有一點點不同

-------

rlc 指令
---------

除了加/減法會影響到 flag 外，它們也會拿去支援一些 cmd，可說是多才多藝，例如 cb cmd 的 opcde 0x00 ~ 0x07 的 rlc，
他的內容是把輸入值左移一位，那這樣最高位就會消失，不過這個指令會把最高位的 bit，保存在 flag->c 中，不過這個指令有趣的地方就在於它其實也把消失的 bit，又加回到 bit 0 的地方

```
eu8 rlx_n(eu8 value, eu8 plus) {
    eu8 result = (value << 1) | plus;

    set_z(result == 0);
    set_n(false);
    set_h(false);
    set_c(CHECK_BIT(value, 7));

    return result;
}

void opcode_cb_rlc_n(ByteReg_p reg) {
    eu8 plus = CHECK_BIT(REG_VAL(reg), 7);
    REG_VAL(reg) = rlx_n(REG_VAL(reg), plus);
}

void opcode_cb_rlc_n_addr(RamAddr addr) {
    eu8 plus = CHECK_BIT(get_ram(addr), 7);
    set_ram(addr, rlx_n(get_ram(addr), plus));
}
```

-------

jr cc, n 指令
---------

最後講一下 opcode_jr_cc_n()，他其實是用 jr_n 組合出來的，特別要提的是，如果判斷是不需要 jump 的話，PC 也必須要往前移動一個，所以這邊就使用 INC_REG(pc) 來達到這個目的

而跳轉命令的實作也很簡單，就是改變 pc 值就可以了，所以 jr_n 指令的作法就是把現在的 pc 加上一個 offset，而這個 offset 的是從 fetch 而來，不過 jr_n 這邊是有順序性的，你一定要先 fetch 才可以做 offset 的動作，因為 fetch 會改變 pc 的值
```
void opcode_jr_n() {
    es8 offset = fetch();
    REG_PC = REG_PC + offset;
}

void opcode_jr_cc_n(bool condition) {
    if (condition) {
        opcode_jr_n();
    } else {
        INC_REG(pc);
    }
}
```

-------

di 與 ei 指令
---------

再來比較特別的就是 di 與 ei，這個是打開 interrupt，我的感覺有點像是 8051 中的 EA=1 的感覺
```
void opcode_di() {
    g_cpu.enable_interrupt = false;
}

void opcode_ei() {
    g_cpu.enable_interrupt = true;
}
```


其實 op code 大概就是這樣，真正需要認真實作的部分約100行左右，除了一個叫 daa 指令之外, 剩下的大部分都很簡單，就是組合再組合就搞定了，接下來下一篇就是要寫 gb 的 video 與 tile 系統了


如何 Debug 
========

其實剩下的指令都很短，也不複雜，但就是多，東西一多起來就很難不出錯，把每個步驟執行後的每個 register 都印出來是一個好方法，不過即便如此，光是開機到開頭動畫，可能就會執行 30~40 萬筆指令，若把所有的東西都寫進 log 中，log 會很龐大，所以我們會需要一些有效率的 debug 方式，新增一個 debug.c 並加入以下的 code

```
#define START_CNT (0x0)
#define END_CNT   (0x0)

void print_ram(RamAddr ptr, eu32 len) {
   print_ram_base(ptr, len, get_ram);
}

void set_debug_flag() {
    if ((START_CNT <= g_cmdCnt) && (g_cmdCnt < END_CNT)) {
        g_debug = true;
    } else {
        g_debug = false;
        }

    if (g_cmdCnt < START_CNT) {
        if ((g_cmdCnt % 0x1000) == 0) {
            g_debug = true;
    } else {
            g_debug = false;
        }
    }
}

void show_reg_ram(bool is_cb_cmd, eu8 opcode, eu8 clock) {
    if (is_cb_cmd) {
        printf("\n\nccnt=%X, cbc=%02X, ", g_cmdCnt, opcode);
    } else {
        printf("\n\nccnt=%X, opc=%02X, ", g_cmdCnt, opcode);
    }
    printf("af=%04X, bc=%04X, de=%04X, hl=%04X, pc=%04X, sp=%04X, clk=%X\n", af->all, bc->all, de->all, hl->all, pc->all, sp->all, clock);

    print_ram(0x0, 0x10);
    print_ram(0x100, 0x10);
    print_ram(0x2000, 0x10);
    print_ram(0x4000, 0x10);
    print_ram(0x8000, 0x10);
    print_ram(0x9800, 0x10);
    print_ram(0xFE00, 0x10);
    print_ram(0xFF00, 0x10);
    print_ram(0xFF40, 0x11);
    print_ram(0xFFA0, 0x10);
    printf("\n");
}

void debug_show_reg_ram(bool is_cb_cmd, eu8 opcode, eu8 clock) {
    set_debug_flag();
    if (g_debug) {
        show_reg_ram(is_cb_cmd, opcode, clock);
    }
}
```

這邊會使用兩種印指令的方式，一種是每隔 0x1000 就印一次，一種是指定區域印，這有個好處就是，你每次比較的時候，都可以知道錯誤大概在哪個區間，然後知道區間之後，再用詳細印的方式，就可以知道是哪一筆出錯了， log 也部會太大

把印 register 的指令插入到 execute_opcode 內，這樣你就可以得到每次執行指令後，register 的變化了
```
eu8 execute_opcode() {
    eu8 opcode = fetch();
    
	....
	....

    opcodeFunMap[opcode]();
    debug_show_reg_ram(is_cb_cmd, opcode, clock);

    ....

    return clock;
}
```


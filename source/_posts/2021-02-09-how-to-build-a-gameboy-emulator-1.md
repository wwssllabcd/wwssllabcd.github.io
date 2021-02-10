---
title: 從零開始的 Gameboy 模擬器開發 -- Step 1
date: 2021-02-08 00:24:46
tags: [How-To, c, emulator]
toc: true
type: adsense
---

建立硬體描述
==========

我們首先對 cpu 做描述， 先介紹 LR35902 這顆 cpu 其中的 4 個 word Register(Register，以後均簡稱 Reg) 分別為 `AF，BC，DE，HL`，這 4 個 word reg，比較特別的是，它們個別又可以拆成 byte reg ，例如 AF 就可以拆成兩個 byte reg `A(accumulator)`與 `F(Flags)` 來使用的，或者是合併讀取，例如 HL 常常當作 ram address 使用

到這邊如果你覺得陌生的話，建議你可以去惡補一下 cpu 暫存器的知識


考量 reg 可以分開讀取，或是合併讀取的特性，所以我們需要建立某種的描述，是可以分開，也可以合併的讀寫，翻翻 C 的手冊，發現使用 union 就可以達到這種目的了，所以我們使用 typedef 建立起對 byte reg 與 wordd reg 的描述

<!--more-->


```
typedef struct {
    eu8 all;
}ByteReg;
ERIC_GEN_POINTER_TYPE(ByteReg);

typedef union {
    eu16 all;
    struct {
        ByteReg low;
        ByteReg high;
    };
}WordReg;
ERIC_GEN_POINTER_TYPE(WordReg);

typedef union {
    eu8 all;
    struct {
        eu8 : 4;
        eu8 c : 1;
        eu8 h : 1;
        eu8 n : 1;
        eu8 z : 1;
    };
}FlagReg;
ERIC_GEN_POINTER_TYPE(FlagReg);

typedef struct {
    union {
        WordReg af;
        struct {
            FlagReg f;
            ByteReg a;
        };
    };
    union {
        WordReg bc;
        struct {
            ByteReg c;
            ByteReg b;
        };
    };
    union {
        WordReg de;
        struct {
            ByteReg e;
            ByteReg d;
        };
    };
    union {
        WordReg hl;
        struct {
            ByteReg l;
            ByteReg h;
        };
    };
    WordReg sp;
    WordReg pc;
	
    bool halt;
    bool running;
    bool enable_interrupt;
    eu32 clock_cnt;
	
}Cpu;
ERIC_GEN_POINTER_TYPE(Cpu);

```
可以看到 word reg 是由兩個 byte reg 來組成的，而她又有一個 all 的屬性，是可以存取本身的 word 的值，剛好符合我們的需求，而我們把 word reg 與 byte reg 的取值變數全部都命名為 all，因為這樣會比較有一致性，當你需要存取不管是 ByteReg 或者是 WordReg，一律都使用 all 就可以存取了，這樣一來我們就能得到一個通用的取值的方法

其中的 ERIC_GEN_POINTER_TYPE 的 Macro 他會幫你產生 xxx_p, xxx_sp, xxx_usp 的 type，例如 `Cpu_usp cpu` 就會等同於 `Cpu* cpu` 的意思，另外，cpu struct 方面，我們也建立了 SP, PC, 的 Word register ，此外也建立一些變數，例如 

- halt 是給 halt 指令使用,
- running 是開/關模擬器使用
- enable_interrupt 是用來支援 ei 與 di 命令，它給我的感覺很像是 8051 中的 EA
- clock_cnt 是用來模擬 cpu 內部的 clock，這個 clock 會跟產生畫面有關係

這邊有個比較特別的 FlagReg_p，他其實就 AF 裡面的 F，又稱為 flags，他只有 4 個 bit 是有用的，所以我們使用分號的方式把那幾個 flag 都限定為 1 bit

---

有了一個 Cpu type 後，我們就可以建立一個全域變數 g_cpu 了，這邊我們全域變數一律使用 `g_` 開頭，我們只有針對全域變數使用匈牙利命名法外，其他情況則不使用，方便我們對全域變數的管制

```
Cpu g_cpu = {
    .af = 0,
    .bc = 0,
    .de = 0,
    .hl = 0,
    .sp = 0,
    .pc = 0,
    .halt = false,
    .running = true,
    .enable_interrupt= false,
    .clock_cnt = 0,
};

```

---

為了方便起見，我們也對比較常用的 register 建立對應的全域變數，可以讓我們之後在寫 opcode 的時候比較直覺一點，也可以少打幾個字，這邊的全域變數又都沒有加 `g_` ，直接光速打臉上面的原則，原因是加上去真的很醜，所以這邊會有個 trade off，反正我們都知道 af 是全域的 Register 就好

```
WordReg_p af = &g_cpu.af;
WordReg_p bc = &g_cpu.bc;
WordReg_p de = &g_cpu.de;
WordReg_p hl = &g_cpu.hl;
WordReg_p sp = &g_cpu.sp;
WordReg_p pc = &g_cpu.pc;

ByteReg_p a = &g_cpu.a;
FlagReg_p flags = &g_cpu.f;
ByteReg_p b = &g_cpu.b;
ByteReg_p c = &g_cpu.c;
ByteReg_p d = &g_cpu.d;
ByteReg_p e = &g_cpu.e;
ByteReg_p h = &g_cpu.h;
ByteReg_p l = &g_cpu.l;
```

我們故意地把所有的 register 都變成了 pointer 型態，原因是要統一存取的格式，例如我今天要取一個 byre reg 與 word reg 的方式都是使用 reg->all

---

所以我們也建立一些對 flags 設定的方式，如下所示，而其中的 SET_FLAG 只是幫你把 bool 轉成 1 or 0 而已，如果是 true，他就會回傳1，反之則0
```
void set_z(bool val) {
    flags->z = SET_FLAG(val);
}
void set_h(bool val) {
    flags->h = SET_FLAG(val);
}
void set_n(bool val) {
    flags->n = SET_FLAG(val);
}
void set_c(bool val) {
    flags->c = SET_FLAG(val);
}
```

接下來建立一些 macro 也是讓我們可以使用比較直覺的方式去存取 Register 的值，而不是依賴 ->all 或是 ->high 這種特定的方式讀取，我們可以利用 macro 去隱藏底層的實作

```
#define REG_VAL(REG)                              ((REG)->all)
#define INC_REG(REG)                              (REG_VAL(REG)++)
#define DEC_REG(REG)                              (REG_VAL(REG)--)

#define REG_HIGH(REG)                             (&(REG)->high)
#define REG_LOW(REG)                              (&(REG)->low)

#define REG_A                       REG_VAL(a)
#define REG_B                       REG_VAL(b)
#define REG_C                       REG_VAL(c)
#define REG_D                       REG_VAL(d)
#define REG_E                       REG_VAL(e)
#define REG_F                       REG_VAL(f)
#define REG_H                       REG_VAL(h)
#define REG_L                       REG_VAL(l)
#define REG_AF                      REG_VAL(af)
#define REG_BC                      REG_VAL(bc)
#define REG_DE                      REG_VAL(de)
#define REG_HL                      REG_VAL(hl)
#define REG_SP                      REG_VAL(sp)
#define REG_PC                      REG_VAL(pc)

#define FLAG_Z                      (flags->z)
#define FLAG_NZ                    (!flags->z)
#define FLAG_C                      (flags->c)
#define FLAG_NC                    (!flags->c)

```

這邊的規則大概是
- 如果你想要讀取變數 Register 的值，你就使用 REG_VAL()，例如 ByteReg reg 的讀值方式就是 REG_VAL(reg)
- 如果你想要讀取特定 Register 的值，你就是使用 REG_A，REG_B ... 等方式


---

建立記憶體管理
==========

接著建立 mmu(memory management unit) ，這個是負責管理記憶體的地方，所有的記憶體的存取都會經過這隻程式 -- 沒有例外，也就是我們會建立一個 64k 的 byte array，而這個 array 會假裝自己是 cpu 的 ram，其實這種說法並不正確，我們的目的應該是 mmu 會把自己假裝成是一種可以存取的裝置，但是本身實作的內容是什麼並不重要，就像是上面的 reg 的 macro，我們一直推遲 reg 取值的方式，直到最後一刻的 REG_VAL 才暴露出來原來是一個叫 all 的變數，在這之前我們都一直在玩文字遊戲，換句話說，只是 mmu 在實作的方式剛好是使用 byte array，有天你不高興，把 array 換成一個檔案，或是雲端某個可以存放資料的地方也是可以的

回到記憶體的話題，當然真實世界的 cpu 根據 sram 的位置，會有 internal sram 與 external sram 之分，像是 LR35902 這顆 cpu 的 0xFF00 的位置就是所謂的 Zero page，我猜它跟 8051 一樣 -- 藉由鎖定一個 register (也許是 P2)，達到快速讀取記憶體的方式，不過對我們來說是不太重要的，就通通看成 ram 就可以了

------

開始實作 mmu 吧，新增檔案 mmu.c 並加入以下的 code 

```
eu8 g_ram[GB_RAM_SIZE] = { 0 };

eu8_p get_ram_ptr(RamAddr address) {
    return (eu8_p)&g_ram[address];
}

eu8 get_ram(RamAddr address) {
    return (*get_ram_ptr(address));
}

void set_ram(RamAddr address, eu8 val) {
    (*get_ram_ptr(address)) = val;
}

void init_mmu(eu8_p rom) {
    mem_cpy(g_ram + GAME_ROM_START_ADDR, rom, GAME_ROM_LENGTH);
    set_ram(0xFF00, 0x3F);
    set_ram(0xFF02, 0xFF);
}
```

對應到 cpu 的 rom 就是 g_ram[_64K] 了，然後提供 gettter / setter，還有 init_mmu()

g_ram 利用 `eu8 g_ram[GB_RAM_SIZE] = {0}` 的方式來達到 init buffer 的功能，不過我們還需要一個 init_mmu() 的 function，它會把 game-rom copy 到 rom 0 ~ 32k 的位置，addres 0的位置有點特別，蠻重要的，晚一點會在說明，這邊還有一點比較特別的是 `0xFF00` 與 `0xFF02` 的初始值是 0x3F 與 0xFF，這邊就先照填吧

cpu.c 這邊也增加 run code ，其中 tick 就是執行 opcode 的地方，基本上 cpu 進到 run_cpu() 會在這邊無限循環直到關機為止，不過這邊我們先讓他強制停止，等晚一點再來處理 opcode 
```
void tick() {
    g_cpu.running = false;
}

void run_cpu() {
    while (g_cpu.running) {
        tick();
    }
}

void power_on_cpu(eu8_p rom) {
    init_mmu(rom);
    run_cpu();
}
```

我們也順便把 game rom 讀進來，game rom 的檔案請自己想辦法嚕，我們在 main 中把 rom 讀進來，並且傳入 power_on_cpu，這段 code 你可以想像成把遊戲 rom 插入主機後，然後 power on 的樣子

`注意: 目前我們只支援 32k 的 rom，像是 MBC 格式的我們目前是不支援的`

```
#include <stdio.h>
#include <Windows.h>
#include <SDL.h>
#include "header.h"
#include "adapter_sdl.h"
#include "cpu.h"

int main(int argc, char* argv[]){
    printf("start\n");
    if (argc == 1) {
        return 0;
    }

    init_sdl("My Gameboy Emulator", SCREEN_WIDTH * SDL_PIXEL_SIZE, SCREEN_HEIGHT * SDL_PIXEL_SIZE);

    echar_p romFileName = argv[1];
    eu8 rom[GAME_ROM_LENGTH];
    file_read(romFileName, rom, _32K);

    power_on_cpu(rom);
	
    SDL_Quit();
    return 0;
}
```

到目前為止，我們基本上已經完成了初步的架構，一個 cpu，一個記憶體，然後我們也把 rom 讀進來了，接下來就可以開始化身成 cpu ，去執行一道道 op code 了
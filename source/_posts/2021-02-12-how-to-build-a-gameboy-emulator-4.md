---
title: 從零開始的 Gameboy 模擬器開發 -- Step 4
date: 2021-02-12 00:24:46
tags: [How-To, C, Emulator]
toc: true
type: adsense
---

Gameboy 的 tile 系統與繪圖
===========

接著要來做有關繪圖的部分，這部分說簡單也不簡單，說難也不會難，難的部分是因為我們沒有碰過 tile 系統，所以要花點時間去理解，當理解後，實作部分其實很簡單一點也不難，如果你還沒去看最前面介紹的影片，我建議你趕快去看，因為看完之後會比較好理解

首先我再幫大家整理一下概念
- 一個 tile 由 8*8 個 pixel 所組成
- GB 整個顯示的部分是 256 * 256 個 pixel ，換算成 tile 就是 32 * 32 個 tile, 在code 裡面我們會以 scroll_px_x, scroll_px_y 來表示
	- 好像有人稱作虛擬螢幕
- 但是  GB 能顯示的螢幕(screen)卻只有 160 * 144 個 pixel ，也就是說他只會顯示上述畫面的一部分
	- 他的用意在於，他可以利用這種窗口，去實現類似鏡頭平移的的功能，這邊的 demo 你可以在影片中看到
	- 對於螢幕的 x, y ，在code 裡面會以 screen_px_x 與 screen_px_y 來表示
- 總結就是，cpu 內設計的螢幕結構是 256*256 的，但是你只能看到這 256*256  的 160*144 ，
	- 這樣設計的原因就是他可以做出一些視覺的效果
	- 例如網球比賽的鏡頭，馬力歐跳躍時的畫面移動，或是橫向捲軸遊戲的效果
	

-----

像我對於這東西就困擾很久，想說這什麼鬼東西，為什麼任天堂的螢幕不弄得跟他內部一樣都是  256*256 等等，
反正你不要想得太複雜，以下做個比喻

* 地上有個由 32 *32 個巧拼所組成的正方形背景拚圖，每個巧拼為 8 * 8 cm，所以整個拼背景圖大小為 256 cm * 256 cm
* 然後你手上有個 160 cm * 144 cm 的白色方框，這個代表你所能看到的區域
* 然後你試著把方框在那個拼圖上任意移動，然後觀察方框內的畫面，這個就像是GB 的畫面移動方式
* 然後你在方框內平放一個 8 * 8 cm 的巧拼，上面畫有一個馬力歐，讓他在方框內任意移動，
當紙娃娃向右移動快移出方框的時候，你也同步的把方框往右移
* GB 的顯示系統大概就是這個樣子，拼圖代表 256 * 256 pixel 的虛擬螢幕(scroll)，巧拼代表 8 * 8  pixel 的 tile，方框代表 144* 160 pixel 的顯示螢幕(screen)，而馬力歐代表一個 sprite ，他由1個 or 2 個 tile 組成

<!--more-->

-------

我們再來講一下 screen 的部分
- screen 會由三種東西組成，分別是 BG, Windw, sprite
- 而 sprite 在任天堂內部稱作 Object ，而他的屬性就稱作 OAM(Object Attribute Memory)，sprite 與 object 他們是一樣的東西 ，只是一個是美國稱作 sprite ，任天堂叫做 Object
- 所以畫面的產生方式就是
	- 先畫出 BG 
	- 再背景上面畫 window 
	- 最後再把 sprite 覆蓋上去
	- 以這種順序來畫的話，他的圖層就會是 sprite 是最高，最低的就是 bg

你拿之前我們那個拼圖的比喻去理解就可以了

PS: 順帶一提的是這邊可能有錯，理論上 window 應該是最高，不過我看的那個 project  的順序是這樣，所以我就先 fallow 了

其實作的方式就是我們會有一個二維陣列叫 frame_buffer[144][160] , 負責把畫面上的每個 pixel 收集起來後，最後再輸出到 sdl2 上面
那我們的 psudo code 會長成這樣 


````
eu8 frame_buffer[144][160];
init_frame_buffer(frame_buffer)
get_bg_data(frame_buffer)
get_window_data(frame_buffer)
get_sprite_data(frame_buffer)
draw_by_sdl2(frame_buffer)
````

frame_buffer 會去收集整個畫面的 pixel 後，window 與 sprite 又會把跟著把剛剛收集到的點蓋過去，用這種方式可以做出不同的圖層的效果


開始建置顯示系統
============

產生 cpu clock 
----------


要先做顯示系統之前，我們必須加入 cpu clock 的計算，也就是我們要模擬 cpu 經過的時間，每個 op code 都會花不同的 cycle 時間，
所以我們只要根據 op code 去查表就可以得知這次所花的 cycle 數

新增一個檔案叫 cpu_cycle.c ，對外提供一個 function 叫做 get_op_cycle()

````
const eu8 g_opcodeCycles[256] = {
    1, 3, 2, 2, 1, 1, 2, 1, 5, 2, 2, 2, 1, 1, 2, 1, 
    1, 3, 2, 2, 1, 1, 2, 1, 3, 2, 2, 2, 1, 1, 2, 1, 
    2, 3, 2, 2, 1, 1, 2, 1, 2, 2, 2, 2, 1, 1, 2, 1, 
    2, 3, 2, 2, 3, 3, 3, 1, 2, 2, 2, 2, 1, 1, 2, 1, 
    1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 2, 1, 
    1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 2, 1, 
    1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 2, 1, 
    2, 2, 2, 2, 2, 2, 1, 2, 1, 1, 1, 1, 1, 1, 2, 1, 
    1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 2, 1, 
    1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 2, 1, 
    1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 2, 1, 
    1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 2, 1, 
    2, 3, 3, 4, 3, 4, 2, 4, 2, 4, 3, 0, 3, 6, 2, 4,
    2, 3, 3, 0, 3, 4, 2, 4, 2, 4, 3, 0, 3, 0, 2, 4, 
    3, 3, 2, 0, 0, 4, 2, 4, 4, 1, 4, 0, 0, 0, 2, 4, 
    3, 3, 2, 1, 0, 4, 2, 4, 3, 2, 4, 1, 0, 0, 2, 4
};

const eu8 g_opcodeCycles_cb[256] = {
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 3, 2, 2, 2, 2, 2, 2, 2, 3, 2, 
    2, 2, 2, 2, 2, 2, 3, 2, 2, 2, 2, 2, 2, 2, 3, 2, 
    2, 2, 2, 2, 2, 2, 3, 2, 2, 2, 2, 2, 2, 2, 3, 2, 
    2, 2, 2, 2, 2, 2, 3, 2, 2, 2, 2, 2, 2, 2, 3, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2,
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 
    2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2
};

eu8 get_cycle(eu8 opcode) {
    return g_opcodeCycles[opcode];
}

eu8 get_cb_cycle(eu8 opcode) {
    return g_opcodeCycles_cb[opcode];
}

eu8 get_op_cycle(bool is_cb_cmd, eu8 opcode) {
    if (is_cb_cmd) {
        return get_cb_cycle(opcode);
    }
    return get_cycle(opcode);
}
````

並修改cpu.c 中的 execute_opcode() 與 void tick()，加入回傳 clock 後，在把 clock 傳入到  video_tick() 中，這個 function 是放在 video.c 中

````
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

    eu8 clock = get_op_cycle(is_cb_cmd, opcode);
    return clock;
}

void tick() { 
    eu8 clock = cpu_tick(); 
    video_tick(clock); 
}
````


新增 video.c ，並加入以下的 code 


`````
#define CLOCKS_PER_HBLANK (204)         // Mode 0  
#define CLOCKS_PER_SCANLINE_OAM (80)    // Mode 2  
#define CLOCKS_PER_SCANLINE_VRAM (172)  // Mode 3  
#define CLOCKS_PER_SCANLINE (CLOCKS_PER_HBLANK + CLOCKS_PER_SCANLINE_OAM + CLOCKS_PER_SCANLINE_VRAM) 

#define STAT_HBLANK_PERIOD (0) 
#define STAT_VBLANK_PERIOD (1) 
#define STAT_SCAN_OAM_RAM (2) 
#define STAT_TRANSFER_DATA_TO_LCD_DRIVE (3) 

eu8 g_curMode = 0; 
eu32 g_videoClock = 0; 
eu8 g_screenFrameBuffer[SCREEN_HEIGHT][SCREEN_WIDTH];

void video_tick(eu8 clock) { 
    g_videoClock += clock; 
}
`````

這邊分別說明一下 code 的用途

* 全域變數 g_videoClock: 紀錄目前 video 所使用的 clock
* g_curMode: 紀錄目前 video 是處在什麼模式
* g_screenFrameBuffer: 對應到 144*160 的顯示畫面

video 顯示的方式是一條一條的 row (或稱 line) 在畫，畫 row 的時候，動作會依序著做 SCAN_OAM -> TRANSFER_DATA_TO_LCD -> HBLANK ，而 HBLANK 畫完一條之後，就會又回到 SCAN_OAM，當畫到 row > 144 的時候，此時就不會跳回 SCAN_OAM，而是跳到 VBLANK，以下我會詳細說明


---------

我先把完整個 code 貼出來，再分別的解釋每個區塊

```
switch (g_curMode) { 
        case ACCESS_OAM: 
            if (g_videoClock >= CLOCKS_PER_SCANLINE_OAM) { 
                g_videoClock = g_videoClock % CLOCKS_PER_SCANLINE_OAM; 
                LCD_STAT.mode_flag = STAT_TRANSFER_DATA_TO_LCD_DRIVE; 
                g_curMode = ACCESS_VRAM; 
            } 
            break; 
        case ACCESS_VRAM: 
            if (g_videoClock >= CLOCKS_PER_SCANLINE_VRAM) { 
                g_videoClock = g_videoClock % CLOCKS_PER_SCANLINE_VRAM; 
                // if it is H-bank period 
                if (LCD_STAT.mode_00) { 
                    IE.lcdc = 1; 
                } 
                check_and_set_lyc_flag(); 
                LCD_STAT.mode_flag = STAT_HBLANK_PERIOD; 
                g_curMode = HBLANK; 
            } 
            break; 
        case HBLANK: 
            if (g_videoClock >= CLOCKS_PER_HBLANK) { 
                g_videoClock = g_videoClock % CLOCKS_PER_HBLANK; 
                write_scan_line(LCD.ly); 
                LCD.ly++; 
                 
                if (LCD.ly == 144) { 
                    LCD_STAT.mode_flag = STAT_VBLANK_PERIOD; 
                    IF.vblank = 1; 
                    g_curMode = VBLANK; 
                } else { 
                    LCD_STAT.mode_flag = STAT_SCAN_OAM_RAM; 
                    g_curMode = ACCESS_OAM; 
                } 
            } 
            break; 
        case VBLANK: 
            if (g_videoClock >= CLOCKS_PER_SCANLINE) { 
                g_videoClock = g_videoClock % CLOCKS_PER_SCANLINE; 
                LCD.ly++; 
                if (LCD.ly >= 154) { 
                    write_sprites(); 
                    draw(); 
                    LCD.ly = 0; 
                    LCD_STAT.mode_flag = STAT_SCAN_OAM_RAM; 
                    g_curMode = ACCESS_OAM; 
                } 
            } 
            break; 
    };
```

ACCESS_OAM:
------

在開始畫線之前，vidoe hw 第一個動作是做 scan 線上面的 OAM物件(也就是sprite)，他會花 80 個 clock 做這件事，對於我們來說，我們只要在超過 80 clock之後，把流程導到 ACCESS_VRAM 即可，順便要把 LCD_STA(0xFF41) 的 mode_flag 設成 3，這個舉動可以通知 fw 目前是在 xfer data to lcd，不能 access VRAM 

ACCESS_VRAM 
----------

這個時期是 hw transfer data 到 lcd 顯示器的時候，  fw 是不能 access  video ram 的，當經過 172  個 cycle 之後，就要進入到  HBLANK，在進入 之前一樣要把  LCD_STA(0xFF41) 的 mode_flag 設成 0，此時還要檢查目前的要畫的 line (ly) 是否與lyc(0xFF45) 相同，fw 會想要在hw 畫到某條 line 的時候通知 fw，以便 fw 能介入，所以 fw 會去填 0xFF45 register ，hw 若畫到這條 line 的時候會觸發一個中斷，此時 fw 便可介入去做一些事情，所以若畫的 line 與 lyc 相同的話，我們就要把 lcd  state 中的 bit 2 給設起來，這邊請去看手冊的 0xFF41 與 0xFF45 的解釋

```
LCD_STAT.coincidence_flag = (LCD.ly == LCD.lyc);
if (LCD_STAT.ly_c && LCD_STAT.coincidence_flag) {
    IE.lcdc = 1;
}
```


HBLANK
----------

經過 204 個 cycle 之後，就會離開 HBLANK，離開前會收集本次 line 的 144個 pixel 的值，若是 ly < 144，則就是繼續回去 SCAN_OAM，若超過則會進入到 VBLANK

```
write_scan_line(LCD.ly);
LCD.ly++;
                
if (LCD.ly < 144) { 
    LCD_STAT.mode_flag = STAT_SCAN_OAM_RAM; 
    g_curMode = ACCESS_OAM; 
} else { 
    LCD_STAT.mode_flag = STAT_VBLANK_PERIOD; 
    IF.vblank = 1; 
    g_curMode = VBLANK; 
}
```



VBLANK
----------

VBLANK 當 ly 超過 154 時，就會把整個畫面畫上去，並且 reset lcd.ly , 並且回到 SCAN_OAM 去

```
LCD.ly++;
if (LCD.ly >= 154) {
    write_sprites();
    draw();

    LCD.ly = 0;
    LCD_STAT.mode_flag = STAT_SCAN_OAM_RAM;
    g_curMode = ACCESS_OAM;
}
```


準備畫出一條線
========

write_scan_line 如下，沒什麼特別的，這邊有作一些顯示的控制，fw 可以決定要不要顯示 bg or window，或是關掉整個 lcd ，所以我們必須要支援這些功能

```
void write_scan_line(eu8 curLine) {
    if (!LCD_CTRL.lcd_control_operation) {
        return;
    }

    if (LCD_CTRL.bg_and_window_display) {
        get_bg_line_data(curLine);
    }

    if (LCD_CTRL.window_dispaly) {
        draw_window_line(curLine);
    }
}
```


比較複雜的就是怎樣畫出一條 line 的，我相信這也是很多人想了解的地方，在我們講解之前，我們要先做一些前置作業

Sprite 相關 type
------

sprite(OAM) 就是對應到手冊的說明，而最多只會有 40 個 sprite，所以 type SpriteMap 只有40個 sprite
```
 typedef struct { 
    eu8 y; 
    eu8 x; 
     
    //Byte2: Pattern number 0-255，(Unlike some tile numbers, sprite pattern numbers are unsigned. LSB is ignored(treated as 0) in 8x16 mode.) 
    eu8 pattern_num; 
    union { 
        eu8 all; 
        struct { 
            eu8 : 4; 
            //Bit 4, Sprite colors are taken from OBJ1PAL if this bit is set to 1 and from OBJ0PAL otherwise 
            eu8 palette_number : 1; 
            //Bit 5, Sprite pattern is flipped horizontally if this bit is set to 1. 
            eu8 flap_x : 1; 
            //Bit 6, Sprite pattern is flipped vertically   if this bit is set to 1. 
            eu8 flap_y : 1; 
            //Bit 7, 0: sprite is displayed on top of background & window 
            //1: sprite will be hidden behind colors 1, 2, and 3 of the background & window 
            eu8 priority : 1; 
        }; 
    }flags; 
}Sprite; 

#define TOTAL_SPRITE_CNT (40) 
typedef struct { 
    Sprite spriteMap[TOTAL_SPRITE_CNT]; 
}SpriteMap; 
```

---------

Tile 相關 type
------

一個 tile 的定義為 8 * 8 個 pixel，而一個 pixel 會花兩個 bit 來儲存他的四種顏色
所以根據定義， tile 的 row(或稱 line)  共有 8 個 pixel ，會需要花 8*2 = 16 bit = 2 byte 來儲存

- TileLine:  一個 tile 的 row 會花兩個 byte 儲存，所以 TileLine 就是 代表那兩個 byte 
- TilePattern: 一個 tile 會有 8 個 TileLine，這個結構代表著整個 tile 
- TilePatternMap: 存放 tile pattern data 的地方，這個會根據 0xFF40的 bit4，來決定本次的pattern map 是使用 0x8000 or 0x8800 的地方

```
typedef struct {
    eu8 byte0;
    eu8 byte1;
}TileLine;

#define TILE_LINE_CNT     (0x8)
#define TILE_PATTERN_SIZE (TILE_LINE_CNT*2)
typedef struct {
    TileLine line_map[TILE_LINE_CNT];
}TilePattern;

#define TOTAL_TILE_PATTERN_CNT (256)
typedef struct {
    TilePattern pattern_map[TOTAL_TILE_PATTERN_CNT];
}TilePatternMap;
```

最後一個是 TileIdMap，這個結構就是對應到之前講的 GB 內部那個 256 *256 pixel(32 * 32個 tile) 的完整螢幕的 tileMap，所以 tileMap 你可以看到他有兩種存取的方式，一種是線性一維的 map，一種是你可以用 `[y][x]` 的二維陣列存取，利用這種方式，可以讓我們等下的 code 簡化很多

```
#define TILE_CNT_OF_SCREEN (32)
#define TOTAL_TILE_CNT_OF_SCREEN (TILE_CNT_OF_SCREEN*TILE_CNT_OF_SCREEN)
typedef struct {
    union {
        eu8 map[TOTAL_TILE_CNT_OF_SCREEN];
        eu8 map_xy[TILE_CNT_OF_SCREEN][TILE_CNT_OF_SCREEN];
    };
}TileIdMap;
ERIC_GEN_POINTER_TYPE(TileIdMap);
```

其他 type 
----

最後則是 ScrollPx 與 Palette，ScrollPx  這邊有個有趣的地方，我們利用它來取商數與餘數，也就是當你把 .all 設成 10 的時候，你會的到 tile_id = 1 與 tile_px =2
而 Palette 就是照手冊設計

```
typedef union {
    eu8 all;
    struct {
        eu8 tile_px : 3;
        eu8 tile_id : 5;
    };
}ScrollPx;

typedef union {
    eu8 all;
    struct {
        //This selects the shade of grays to use for the background(BG)& window pixels.
        //Since each pixel uses 2 bits, the corresponding shade will be selected from here.
        eu8 data_for_dot_data_00 : 2;
        eu8 data_for_dot_data_01 : 2;
        eu8 data_for_dot_data_10 : 2;
        eu8 data_for_dot_data_11 : 2;
    };
}Palette;
```

tile pattern map 與 tile id map 存放的位置
------

剛有提到 tile pattern map 與 tile id map 選擇的問題，是靠LCD_CTRL 中的 bg_and_window_tile_data_select 與 bg_tile_map_display_select 所指定，所以這邊也把 selection 的 function 做出來

```
 TilePatternMap_p get_tile_data_table(eu8 select) {
    if (select) {
        return (TilePatternMap_p)(get_ram_ptr(TILE_PATTERN_MAP_0));
    }
    return (TilePatternMap_p)(get_ram_ptr(TILE_PATTERN_MAP_1));
}

TileIdMap_p get_tile_id_map(eu8 select) {
    if (select) {
        return (TileIdMap_p)(g_ram + TILE_MAP_1);
    }
    return (TileIdMap_p)(g_ram + TILE_MAP_0);
}

TilePatternMap_p get_tile_data_table_bg_window() {
    return get_tile_data_table(LCD_CTRL.bg_and_window_tile_data_select);
}

TileIdMap_p get_screen_bg_tile_id_table() {
    return get_tile_id_map(LCD_CTRL.bg_tile_map_display_select);
}
```

上面各做了一個 select funciton 再搭配特化 function 以減少參數輸入

---------

GB 的色彩選擇 -- palette
-------

接下來要新增取特定 pixel 所對應的顏色，我們知道顏色有四種，編號 0 ~ 3，get_pixel_data() 代表是怎樣從一個 tile 中的兩個byte 中，取出該對應到的 pixel 所代表的 color 編號，比較特別的是他是倒過來取的，而取出來的 color 編號也不是最終的顏色編號，他還要去 LCD.bgp 那邊查表一次，才會知道最後顯示的 color 編號

write_screen_frame_buffer() 會收集輸出到 144 * 160 的每個 pixel 的值，蠻直覺的 function 
```
void write_screen_frame_buffer(eu8 x, eu8 y, eu8 color) { 
    if (x >= SCREEN_WIDTH) { 
        ASSERT_CODE(0, "screen_x ofb =%X", x); 
    } 
    if (y >= SCREEN_HEIGHT) { 
        ASSERT_CODE(0, "screen_y ofb =%X", y); 
    } 
    g_screenFrameBuffer[y][x] = color; 
}

eu8 get_palette(Palette_sp p, eu8 num) {
    if (num == 0) {
        return p->data_for_dot_data_00;
    }
    if (num == 1) {
        return p->data_for_dot_data_01;
    }
    if (num == 2) {
        return p->data_for_dot_data_10;
    }
    if (num == 3) {
        return p->data_for_dot_data_11;
    }
    ASSERT_CODE(0, "wrong palette num=%X", num);
    return NULL_8;
}

eu8 get_bg_palette(eu8 num) {
    return get_palette(&LCD.bgp, num);
}

eu8 get_pixel_data(eu8 byte_0, eu8 byte_1, eu8 px_offset) {
    return (CHECK_BIT(byte_1, 7 - px_offset) << 1) | (CHECK_BIT(byte_0, 7 - px_offset));
}
```

到此準備工作就告個段落了

-------

組合所有的 function
======


畫出 window 
-------

一切準備就緒後，就可以把一切的東西組合起來嚕，開始寫 get_bg_line_data()，如下所示

```
void draw_bg_window_line(eu8 screen_x, eu8 screen_y, ScrollPx scroll_x, ScrollPx scroll_y) {
    TilePatternMap_sp tileDataTable = get_tile_data_table_bg_window();
    TileIdMap_sp screenTileIdTable = get_screen_window_tile_id_table();
    eu8 tile_id = screenTileIdTable->map_xy[scroll_y.tile_id][scroll_x.tile_id];

    if (!LCD_CTRL.bg_and_window_tile_data_select) {
        tile_id += 128;
    }

    TileLine_sp tileLine = &tileDataTable->pattern_map[tile_id].line_map[scroll_y.tile_px];
    write_screen_frame_buffer(screen_x, screen_y, get_bg_palette(get_pixel_data(tileLine->byte0, tileLine->byte1, scroll_x.tile_px)));
}

void get_bg_line_data(eu8 screen_y) {
    eu8 scrolled_x = LCD.scx;
    eu8 scrolled_y = screen_y + LCD.scy;

    for (eu8 screen_x = 0; screen_x < SCREEN_WIDTH; screen_x++) {
        ScrollPx scroll_x = {.all = (scrolled_x + screen_x) % BG_MAP_PIXEL_SIZE};
        ScrollPx scroll_y = {.all = (scrolled_y) % BG_MAP_PIXEL_SIZE};

        draw_bg_window_line(screen_x, screen_y, scroll_x, scroll_y);
    }
}
```

如果你有看過其他人的 code ，應該會對這邊簡短的 code 感到訝異，事實上 tile 的概念就是這樣，沒有很複雜，讓我解釋一下裡面用到的變數，這邊為了避免混淆虛擬螢幕與顯示螢幕，這邊命名 scroll 的是代表虛體螢幕(也就是256 * 256 的拼圖)，而 screen 代表顯示的螢幕(也就是 144 * 160 白色方框)

- scrolled_x 與 scrolled_y : 代表 BG 要從 256 *256 的實體螢幕上的哪一點 x,y 開始畫，還記得我們之前拼圖，白色方框的比喻嗎? 這個就是決定白色方框的 x, y 
- scroll_x 與 scroll_y: 代表每一次要取的 scroll x, y ，這邊有做一個取餘數 % 的動作，可以讓你畫到超出界外的時候，roll back 到最前面

接下來講解 draw_bg_window_line ()

取出目前對應的 tileDataTable  與 screenTileIdTable，screenTileIdTable->map 代表著整個虛擬螢幕所對應的 tile id ，不過我們這邊是使用他的 map_xy 來取值，因為我們有他的 x,y 座標


取到正確 id 後，就根據 tile id 去 tilePatternMap 去取出 tileline 的兩個 byte了，然後再利用我們剛剛提到的function 把他們通通組合起來就可以了

這邊所有定位與計算的動作都是使用之前定義的 type 巧妙地閃過

---------

畫出 window 
-------
也是利用類似方法，先取出 patternMap 與 screenTileIdMap之後，再查表即可，比較特別的是， 如果 bg_and_window_tile_data_select == 0 的話，tileId 要再加上 128

```
void draw_bg_window_line(eu8 screen_x, eu8 screen_y, ScrollPx scroll_x, ScrollPx scroll_y) {
    TilePatternMap_sp tileDataTable = get_tile_data_table_bg_window();
    TileIdMap_sp screenTileIdTable = get_screen_window_tile_id_table();
    eu8 tile_id = screenTileIdTable->map_xy[scroll_y.tile_id][scroll_x.tile_id];

    if (!LCD_CTRL.bg_and_window_tile_data_select) {
        // In the second case, patterns have signed numbers from - 128 to 127
        //(i.e. pattern #0 lies at address $9000).
        tile_id += 128;
    }

    TileLine_sp tileLine = &tileDataTable->pattern_map[tile_id].line_map[scroll_y.tile_px];
    write_screen_frame_buffer(screen_x, screen_y, get_bg_palette(get_pixel_data(tileLine->byte0, tileLine->byte1, scroll_x.tile_px)));
}
```

畫出 sprite 
-------

sprite 的畫法就單純很多，去 oam 把 40 個 sprite 取出來後，再根據每個所指定 tile id 畫到指定的 x，y 即可，這邊有幾點說明

- sprite 也有開關讓fw 決定要不要顯示它
- 一共最多要畫 40 個 sprite
- sprite 所對應的 tilePatternMap 必須為 0x8000
- OAM 有時候會要求畫兩個 tile 上去，他是以垂直的方式來畫
- 當pixel color 取出來的值為0的時候，就不去畫他，這樣可以做出 sprite 透明的效果
- sprite x 與 sprite 要分別減掉固定值 8 與 16，代蓋是他們紀錄的點是 sprite 右下角那個點

PS: 你可以試試看加上 if (px_data == 0) { continue; } 還有移掉她，對於畫面的改變為何

```
void draw_sprite(eu8 spriteNum) {
    Sprite_p sprite = GET_SPRITE_PTR(spriteNum);
    if (sprite->x == 0 || sprite->x >= 168) {
        return;
    }
    if (sprite->y == 0 || sprite->y >= 160) {
        return;
    }

    TilePatternMap_p tilePatternMap = get_tile_data_table(1);

    for (eu8 oam = 0; oam < (LCD_CTRL.obj_size+1); oam++) {
        TilePattern_p tile = &tilePatternMap->pattern_map[sprite->pattern_num + oam];

        eu32 start_x = sprite->x - 8;
        eu32 start_y = sprite->y - 16 + (oam * TILE_Y_PX);

        for (eu8 y = 0; y < TILE_Y_PX; y++) {
            TileLine_p tile_line = &tile->line_map[y];
            for (eu8 x = 0; x < TILE_X_PX; x++) {
                if (!is_pixel_on_screen(start_x + x, start_y + y)) {
                    continue;
                }
                if (sprite->flags.priority) {
                    continue;
                }

                eu8 target_px_x = !sprite->flags.flap_x ? x : TILE_X_PX - x - 1;
                eu8 target_px_y = !sprite->flags.flap_y ? y : TILE_Y_PX - y - 1;
                eu8 screen_x = (eu8)(start_x + target_px_x);
                eu8 screen_y = (eu8)(start_y + target_px_y);


                eu8 px_data = get_pixel_data(tile_line->byte0, tile_line->byte1, target_px_x);
                // 0 is transparent
                if (px_data == 0) {
                    continue;
                }

                eu8 color = get_sprite_palette(sprite->flags.palette_number, px_data);
                write_screen_frame_buffer(screen_x, screen_y, color);
            }
        }
    }
}

void write_sprites() { 
    if (!LCD_CTRL.obj_display) { 
        return; 
    } 
    for (eu8 sprite_n = 0; sprite_n < TOTAL_SPRITE_CNT; sprite_n++) { 
        draw_sprite(sprite_n); 
    } 
}
```

畫出 Sprite 與 之前畫 BG 與 Window 不同的是，Sprite 所使用的 tile 是固定的，不像是 BG 要不段的查表


------


接上顯示系統
=======
我們使用 sdl 2 來當作顯示系統，照著 以下的 code 輸入即可，這邊會在最後一刻把 gb 的四種顏色，轉成 sdl 所顯示的 RGB 顏色，如果順利的話，就會顯示出遊戲畫面

```
typedef void(*SetPixelsFun)(eu32 x, eu32 y, eu8 color);

eu32 get_real_color(eu8 pixelColor) {
    // for compile error
    eu8 r = 0;
    eu8 g = 0;
    eu8 b = 0;

    switch (pixelColor) {
        case 0:
            r = g = b = 255;
            break;  // white
        case 1:
            r = g = b = 170;
            break;
        case 2:
            r = g = b = 85;
            break;
        case 3:
            r = g = b = 0;
            break;  // black
        default:
            ASSERT_CODE(0, "Wrong pixel color = %X", pixelColor);
    }

    return (r << 16) | (g << 8) | (b << 0);
}

void set_pixel(eu32 x, eu32 y, eu8 color) {
    g_sdlPixels[SCREEN_WIDTH * SDL_PIXEL_SIZE * y + x] = get_real_color(color);
}

void set_pixels(eu8 frameBuffer[SCREEN_HEIGHT][SCREEN_WIDTH], SetPixelsFun set_pixel_fun) {
    for (eu8 y = 0; y < SCREEN_HEIGHT; y++) {
        for (eu8 x = 0; x < SCREEN_WIDTH; x++) {
            set_pixel_fun(x, y, frameBuffer[y][x]);
        }
    }
}

void draw_sdl2(eu8 frameBuffer[SCREEN_HEIGHT][SCREEN_WIDTH]) {
    process_events();

    SDL_RenderClear(g_renderer);

    void* pixelsPtr;
    int pitch;

    SDL_LockTexture(g_texture, NULL_PTR, &pixelsPtr, &pitch);

    g_sdlPixels =(uint32_t*)(pixelsPtr);

    set_pixels(frameBuffer, set_pixel);

    SDL_UnlockTexture(g_texture);
    SDL_RenderCopy(g_renderer, g_texture, NULL_PTR, NULL_PTR);
    SDL_RenderPresent(g_renderer);
}
```

此時你會發現，螢幕太小，我們要想辦法放大，所以我們做了一個放大的程式 set_large_pixels()，其策略是一個 1 * 1 的點，讓他變成 2 * 2，把原本的 function 替換成放大版的即可

```
void set_large_pixels(eu32 x, eu32 y, eu8 color) {
    for (eu8 w = 0; w < SDL_PIXEL_SIZE; w++) {
        for (eu8 h = 0; h < SDL_PIXEL_SIZE; h++) {
            eu32 fin_x = x * SDL_PIXEL_SIZE + w;
            eu32 fin_y = y * SDL_PIXEL_SIZE + h;
            set_pixel(fin_x, fin_y, color);
        }
    }
}
```

這樣一來就可以顯示出較大的螢幕了，記得一開始的 init_sdl() 的參數也要一起替換喔


最後
=====

其實這個顯示的系統一開始沒接觸過的可能會不太好懂，這也是我這篇的目的，希望能寫得更清楚一點讓想要做模擬器的人不用在花太多精神在研究這種特殊的東西






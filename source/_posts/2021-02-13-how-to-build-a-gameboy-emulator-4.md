---
title: 從零開始的 Gameboy 模擬器開發 -- Step 4
date: 2021-02-13 00:24:46
tags: [How-To, C, Emulator]
toc: true
type: adsense
---

Gameboy 的輸入系統
===========

接著要來做有關輸入系統的部分，也就是搖桿，其實不會太困難，甚至這是整系列最簡單的一部分，作法就是我們要想辦法從 sdl2 那邊收到外部的輸入後，再模擬成 z80 收到外部輸入的樣子即可

從 sdl2 接收輸入很簡單，他已經幫我們弄好了 void process_events() 算是定番了，照著打就對了，最後會轉一手傳入到 key_down_event() or key_up_event() 裡面


<!--more-->

```
enum {
    GB_KEY_NULL = 0,
    GB_KEY_UP,
    GB_KEY_DOWN,
    GB_KEY_LEFT,
    GB_KEY_RIGHT,
    GB_KEY_A,
    GB_KEY_B,
    GB_KEY_SELECT,
    GB_KEY_START,
};

eu8 get_gb_key_event_code(SDL_Keycode keyCode) {
    switch (keyCode) {
        case SDLK_UP: return GB_KEY_UP;
        case SDLK_DOWN: return GB_KEY_DOWN;
        case SDLK_LEFT: return GB_KEY_LEFT;
        case SDLK_RIGHT: return GB_KEY_RIGHT;
        case SDLK_k: return GB_KEY_A;
        case SDLK_l: return GB_KEY_B;
        case SDLK_1: return GB_KEY_SELECT;
        case SDLK_2: return GB_KEY_START;
        default: return GB_KEY_NULL;
    }
}

void get_sdl2_key_events(SDL_Keycode keyCode, bool isKeyDown) {
    eu8 event_code = get_gb_key_event_code(keyCode);
    if (isKeyDown) {
        key_down_event(event_code);
    } else {
        key_up_event(event_code);
    }
}

void process_events() {
    SDL_Event event;

    while (SDL_PollEvent(&event)) {
        switch (event.type) {
            case SDL_KEYDOWN:
                if (event.key.repeat == true) {
                    break;
                }
                get_sdl2_key_events(event.key.keysym.sym, true);
                break;
            case SDL_KEYUP:
                if (event.key.repeat == true) {
                    break;
                }
                get_sdl2_key_events(event.key.keysym.sym, false);
                break;
            case SDL_WINDOWEVENT:
                if (event.window.event == SDL_WINDOWEVENT_CLOSE) {
                    g_cpu.running = true;
                }
                break;
            case SDL_QUIT:
                g_cpu.running = false;
                break;
        }
    }
}

```

key_down_event 與 key_up_event 會把 一些 flag 立起來，代表目前 joypad 的狀況


```
typedef union {
    eu8 all;
    struct {
        bool up;
        bool down;
        bool left;
        bool right;
        bool a;
        bool b;
        bool select;
        bool start;
        bool button_switch;
        bool direction_switch;
    };
}GbKey;

GbKey g_key = {
    .up = false,
    .down = false,
    .left = false,
    .right = false,
    .a = false,
    .b = false,
    .select = false,
    .start = false,
    .button_switch = false,
    .direction_switch = false,
};

void input_write(eu8 value) {
    // maybe reset joypad
    g_key.direction_switch = !CHECK_BIT(value, 4);
    g_key.button_switch = !CHECK_BIT(value, 5);
}

eu8 input_read() {
    eu8 buttons = 0b1111;
    if (g_key.direction_switch) {
        SET_BIT_TO(buttons, 0, !g_key.right);
        SET_BIT_TO(buttons, 1, !g_key.left);
        SET_BIT_TO(buttons, 2, !g_key.up);
        SET_BIT_TO(buttons, 3, !g_key.down);
    }

    if (g_key.button_switch) {
        SET_BIT_TO(buttons, 0, !g_key.a);
        SET_BIT_TO(buttons, 1, !g_key.b);
        SET_BIT_TO(buttons, 2, !g_key.select);
        SET_BIT_TO(buttons, 3, !g_key.start);
    }

    SET_BIT_TO(buttons, 4, !g_key.direction_switch);
    SET_BIT_TO(buttons, 5, !g_key.button_switch);

    return buttons;
}

void set_button(eu8 code, bool is_key_down) {
    if (code == GB_KEY_NULL) {return; }

    if (code == GB_KEY_UP) { g_key.up = is_key_down; }
    if (code == GB_KEY_DOWN) { g_key.down = is_key_down; }
    if (code == GB_KEY_LEFT) { g_key.left = is_key_down; }
    if (code == GB_KEY_RIGHT) { g_key.right = is_key_down; }
    if (code == GB_KEY_A) { g_key.a = is_key_down; }
    if (code == GB_KEY_B) { g_key.b = is_key_down; }
    if (code == GB_KEY_SELECT) { g_key.select = is_key_down; }
    if (code == GB_KEY_START) { g_key.start = is_key_down; }
}

void key_down_event(eu8 code) {
    set_button(code, true);
}

void key_up_event(eu8 code) {
    set_button(code, false);
}
```

最後 input_read 會負責丟給 fw，那怎麼丟給 fw 呢? joypad 與 fw 的溝通是靠 0xFF00 這個位置，而這個位置有特定的格式，大家再去找手冊看吧

也就是說，我們的 input_read() 會假裝成 0xFF00 的那個 byte, 而這個 byte 的內容就是根據目前 joypad 的 flag 所產生的

```
#define IO_P1                        (0xFF00)
#define IO_SERIAL                    (0xFF02)
#define IO_TIMER_MODULE              (0xFF06)
#define IO_TIMER_CTRL                (0xFF07)
#define IO_DMA                       (0xFF46)
#define BOOT_ROM_ENABLE_ADDR         (0xFF50)

eu8 get_ram(RamAddr addr) {
    if (addr == IO_P1) {
        return input_read();
    }

    // fix 0xFF
    if (addr == IO_SERIAL) {
        return 0xFF;
    }
    // DMA, fix 0
    if (addr == IO_DMA) {
        return 0x0;
    }

    return (*get_ram_ptr(addr));
}
```

這邊就不著墨太多了，因為蠻簡單的，自己看個幾遍就會理解嚕

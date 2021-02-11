---
title: 從零開始的 Gameboy 模擬器開發 -- Step 0
date: 2021-02-07 00:24:46
tags: [How-To, c, emulator]
toc: true
type: adsense
---

緣由
======================
工作上使用 8051 蠻多年了，以前就有想要寫一個 cpu 模擬器的念頭，某一陣子還收到一間美商寫 arm 模擬器的工作邀約，不過以不熟為理由推掉了，後來看到 Jserv 與 [yodalee 的文章](https://yodalee.me/2020/12/2020_rust_gameboy/)，想說我應該也可以做到這件事，就像是 [George Mallory](https://zh.wikipedia.org/wiki/%E4%B9%94%E6%B2%BB%C2%B7%E9%A9%AC%E6%B4%9B%E9%87%8C) 說的，"因為山就在那，所以我登山(Because it's there)"，做這件事的起因也沒什麼特別的，差不多也就是因為我覺得我可以，所以我去做看看的感覺

---

語言的選擇
----------------
使用 C 語言，原因很單純就是，我只有找到一個 project 的 code 是比較乾淨的，然後他剛好是 C code 這樣，我之前有找到幾個 gbe 的 project ，但 source code 實在是太亂了，對於一個模擬器的新手要看他們的code 進而理出整個邏輯是很困難的，所以這次先選擇使用 C 開發，可能之後做完後會找機會 porting 到其他語言像是 rust、python 這兩個抽象性較高的語言，或是使用 Java Script，這樣也許會有機會可以在 web 上玩，或是 Haskell 感覺也蠻有趣的

<!--more-->

---

Platform 的選擇
----------------
使用 window + VC community 當作開發環境，當然使用 ubuntu(or wsl) + vs code + gcc + gdb 也是可以，這是個人喜好問題，可能之後我也會 porting 到 linux 上面去，反正就不要用一些奇怪的東西的話，盡量使用標準 C 的話，porting 就不會太困難

---

目的
----------
其實有幾個目的，第一個就是看到很多 project 對於硬體上面的描述感覺很亂，我覺得如果巧妙的使用 struct 或是二維陣列的話，會可以把 code 寫得更漂亮，另外的一個目的就是試看看能不能使用 C 語言進行高階語言的設計，看能做到怎樣的程度

---

參考文件
----------------
基本上你一定會反覆參考這些文件 ，所以就先在這邊列出，建議開始前可以先去看，有個概念

- [The Ultimate Game Boy Talk](https://media.ccc.de/v/33c3-8029-the_ultimate_game_boy_talk#t=1445) 這篇演講講的非常好，講者應該是非常用心的在準備這篇的演講
- [Game Boy CPU Manual ](https://fdocuments.in/document/gameboy-cpu-manual.html) 這手冊對於整個 GB 的 cpu 寫得很詳細 

剩下的參考資料就比較沒那麼完整了，但是還是會對剛開始建立起觀念有幫助

- [Rust Gameboy Emulator -- by yodalee](https://yodalee.me/2020/12/2020_rust_gameboy/): yodalee 使用 rust 的特性來，用簡單的 code 就可以大量建立 CPU 的 instruction，我就是看了這篇才決定也要動手做的
- [GameBoy 仿真器教程](http://accu.cc/content/gameboy/preface/): 這篇蠻完整的介紹了 Gameboy 各項，對於 video 的說明的方面的寫的不錯
- [Game Boy 的硬體設計與運作原理](https://hackmd.io/@RinHizakura/BJ6HoW29v): 這篇是成大資工系的專題，這邊不得不提到 [Jserv大大](http://wiki.csie.ncku.edu.tw/User/jserv)對於台灣資訊教育的貢獻，沒有他這個專題，也不會有 yodalee 那篇文章，也更不會有我這篇文章了，當然專題的其他同學也有整理有關 GB 的資料也很棒，不過這篇是可以搭配 "The Ultimate Game Boy Talk" 一起閱讀的，而雖然這對同學只是一個專題，但是所產出的資料會永遠地被人參考


最後的參考文件就是我的 source code ，你可以在[這邊](https://github.com/wwssllabcd/EricGbEmu)找到目前專案的 source code

---

從零開始的 Gameboy 模擬器開發 -- Step 0: 建立開發環境
======================

首先我們先建立一個 win32 console 的空專案，並取一個自己喜歡的名字，我這邊就先取了 EricGbEmu(名字不會影響後續的開發)，先行編譯與安全性檢查的都拿掉
建立完的時候會出現一個 EricGbEmu.cpp 的專案，把 EricGbe.cpp 改成 main.c ，並且把其他的東西都砍掉，整個專案只剩 main.c 後，試著編譯 & run 看看是否可以 pass
如果可以順利 compile pass 的話，就把這些檔案放入 git  中，當作 first init

若不行，這邊也有教[如何使用 VC 建立一個 C 的 project ](https://michaelchen.tech/windows-programming/use-vs2019-for-c-projects/)

---

加入常用到的 code 
----------------

- 建立一個 folder 叫 common，把我以前累積常用的 define 與 utility 等小東西加入，以便加速開發
- 建立 folder 叫 type 與 define，之後若是新增 type 與 define 就丟到這裡來，這樣比較不會亂亂的

---

加入 sdl 2 
----------------
從 [SDL 2 網站](https://www.libsdl.org/download-2.0.php)下載 SDL2-devel-2.0.14-VC.zip ，解壓縮後就可以使用了
再把 include 目錄，再去連結/輸入/其他相依性那邊輸入 `SDL2.lib; SDL2main.lib; SDL2test.lib` 就可以了
記得不要在其他相依性那邊 include sdl2.dll，否則會發生 `檔案無效或毀損: 無法在 0x308 讀取` 的錯誤

若不行的話，這邊有[Setting up SDL 2 on Visual Studio 2019 Community](https://lazyfoo.net/tutorials/SDL/01_hello_SDL/windows/msvc2019/index.php) 可以參考

接著建立一個叫 `adapter_sdl.c` 當作我們的 code 與 sdl2 介接的橋樑，code 如下所示

```
#include "adapter_sdl.h"
#include <SDL.h>
#define NULL_PTR (0)

SDL_Window* g_window;
SDL_Renderer* g_renderer;
SDL_Texture* g_texture;
uint32_t* g_sdl_pixels;

void init_sdl(char* title, eu32 width, eu32 height) {
    SDL_Init(SDL_INIT_VIDEO);
    g_window = SDL_CreateWindow(title, SDL_WINDOWPOS_UNDEFINED,  SDL_WINDOWPOS_UNDEFINED, width, height, SDL_WINDOW_OPENGL);
    if (g_window == NULL_PTR) {
        ASSERT_CODE(0, "Failed to initialise SD=%X", g_window);
    }
    g_renderer = SDL_CreateRenderer(g_window, -1, SDL_RENDERER_ACCELERATED |  SDL_RENDERER_PRESENTVSYNC);
    g_texture = SDL_CreateTexture(g_renderer, SDL_PIXELFORMAT_ARGB8888,  SDL_TEXTUREACCESS_STREAMING, width, height);
}

```
這邊有個叫 ASSERT_CODE 的，是我使用 assert 的方式，你也可以改成你自己喜歡的方式去處理 error handling 

----

顯示遊戲視窗
----------

修改 main.c 中的 main() 如下
```
#include <stdio.h>
#include <Windows.h>
#include <SDL.h>
#include "header.h"
#include "adapter_sdl.h"

int main(int argc, char* argv[])
{
    printf("start\n");
   
    init_sdl("My Gameboy Emulator", SCREEN_WIDTH, SCREEN_WIDTH);
    system("PAUSE");
    SDL_Quit();
    return 0;
}
```

其中比較特別的是，這個 `main.c` 中，即便你沒用到 sdl 相關程式，你也一定要 `include <SDL.h>`， 因為 sdl 需要對你的 main funciton 動手腳，所以這邊只能照做

接下來就可以執行我們的 code 了， 若你執行時出現 `因為找不到 SDL2.dll` ，就代表你還沒把 `SDL2.dll, SDL2.lib, SDL2main.lib, SDL2test.lib` 這幾個檔案 copy 到執行目錄上，這個要手動做

若一切順利，理論上你應該會看到一個小的白色框框，那個就是模擬器的螢幕，但又因為我們設定的螢幕太小，所以連你打的 title 都看不到，這個問題我們之後會去解，現在就先不要管他


到此我們 Step 0 就告一個段落了，接下來就是真正要開發 code 了

[從零開始的 Gameboy 模擬器開發 -- Step 1 ](http://localhost:4000/2021/02/08/how-to-build-a-gameboy-emulator-1/)
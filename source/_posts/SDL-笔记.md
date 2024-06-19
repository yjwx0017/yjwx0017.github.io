title: SDL 笔记
categories: 日志
tags: [SDL]
date: 2017-05-20 11:14:00
---
## 开发环境配置

[http://tjumyk.github.io/sdl-tutorial-cn/contents.html][1]

VS 新建项目选择控制台程序，链接属性中选择子系统“/SUBSYSTEM:WINDOWS”。

## 在屏幕上显示一张图片

``` c++
#include <SDL/SDL.h>

int main(int argc, char *args[])
{
  SDL_Surface *hello = nullptr;
  SDL_Surface *screen = nullptr;

  SDL_Init(SDL_INIT_EVERYTHING);

  // 设置窗口
  screen = SDL_SetVideoMode(640, 480, 32, SDL_SWSURFACE);

  // 加载图像
  hello = SDL_LoadBMP("hello.bmp");

  // 将图像应用到窗口上
  SDL_BlitSurface(hello, nullptr, screen, nullptr);

  // 更新窗口
  SDL_Flip(screen);

  SDL_Delay(2000);

  // 释放资源
  SDL_FreeSurface(hello);
  SDL_Quit();
  return 0;
}
```

<!--more-->

## 优化表面的加载和 Blit

``` c++
#include <SDL/SDL.h>
#include <string>

const int SCREEN_WIDTH = 640;
const int SCREEN_HEIGHT = 480;
const int SCREEN_BPP = 32;

SDL_Surface *message = nullptr;
SDL_Surface *background = nullptr;
SDL_Surface *screen = nullptr;

SDL_Surface* load_image(const std::string &filename)
{
  SDL_Surface *loadedImage = nullptr;
  SDL_Surface *optimizedImage = nullptr;

  // 这个位图是24位的，而窗口是32位的，将一个表面blit到另一个不同表面上时，
  // SDL会在每次blit时做一次临时性的格式转换，这会导致程序运行效率降低。
  loadedImage = SDL_LoadBMP(filename.c_str());

  if (loadedImage != nullptr) {
    // 创建一个与窗口同样格式的图像
    optimizedImage = SDL_DisplayFormat(loadedImage);
    // 释放资源
    SDL_FreeSurface(loadedImage);
  }

  return optimizedImage;
}

void apply_surface(int x, int y, SDL_Surface *source, SDL_Surface *destination)
{
  SDL_Rect offset;
  offset.x = x;
  offset.y = y;

  SDL_BlitSurface(source, nullptr, destination, &offset);
}

int main(int argc, char *args[])
{
  if (SDL_Init(SDL_INIT_EVERYTHING) == -1) {
    return 1;
  }

  screen = SDL_SetVideoMode(SCREEN_WIDTH, SCREEN_HEIGHT, SCREEN_BPP, SDL_SWSURFACE);

  if (screen == nullptr) {
    return 1;
  }

  // 设置窗口标题
  SDL_WM_SetCaption("Hello World", nullptr);

  // 加载图片
  message = load_image("hello.bmp");
  background = load_image("background.bmp");

  // 将图片应用到窗口上
  // SDL的坐标系左上角是原点
  apply_surface(0, 0, background, screen);
  apply_surface(320, 0, background, screen);
  apply_surface(0, 240, background, screen);
  apply_surface(320, 240, background, screen);
  apply_surface(180, 140, message, screen);

  if (SDL_FLip(screen) != -1) {
    return 1;
  }

  SDL_Delay(2000);

  SDL_FreeSurface(message);
  SDL_FreeSurface(background);
  SDL_Quit();

  return 0;
}
```

SDL仅原生地支持bmp格式的图片文件，但是通过使用`SDL_image`这个扩展库，你将可以加载BMP, PNM, XPM, LBM, PCX, GIF, JPEG, TGA 和 PNG 格式的图片文件。

``` c++
//加载图像
loadedImage = IMG_Load(filename.c_str());
```

## 事件驱动编程

``` c++
bool quit(false);
SDL_Event event;
while (quit == false) {
  while (SDL_PollEvent(&event)) {
    if (event.type == SDL_QUIT) {
      quit = true;
    }
  }
}
```

其他相关函数：`SDL_WaitEvent()` `SDL_PeepEvents()`。

错误处理相关函数：`SDL_GetError()` `IMG_GetError()`。

## 指定颜色为透明

``` c++
// 将所有颜色为（R 0, G 0xFF, B 0xFF）的像素设为透明
Uint32 colorKey = SDL_MapRGB(optimizedImage->format, 0, 0xFF, 0xFF);
SDL_SetColorKey(optimizedImage, SDL_SRCCOLORKEY, colorKey);
```

SDL_SRCCOLORKEY标志保证了关键色仅当blit这个表面到另一个表面上时才被使用。

对于带alpha通道的PNG图片，IMG_Load()函数会自动地为他们处理透明色。在一个已经具有透明背景色的图片上设置关键色会导致糟糕的结果。另外，如果你使用SDL_DisplayFormat()，而不是SDL_DisplayFormatAlpha()，你也会丢失Alpha透明色。要保持PNG中的透明色，请不要设置关键色。IMG_Load()也会处理TGA图像的Alpha透明色。你可以在SDL的文档里得到更详细的有关关键色的信息。

## 局部Blit和精灵图

精灵图是一系列保存在同一个图像文件中的图像。当你有数量庞大的图像，但不想处理那么多的图像文件时，精灵图就派上用场了。

``` c++
//用白色填充窗口
SDL_FillRect(screen, &screen->clip_rect, SDL_MapRGB(screen->format, 0xFF, 0xFF, 0xFF));

SDL_Rect clip[4];
//左上角的剪切区域
clip[0].x = 0;
clip[0].y = 0;
clip[0].w = 100;
clip[0].h = 100;

//右上角的剪切区域
clip[1].x = 100;
clip[1].y = 0;
clip[1].w = 100;
clip[1].h = 100;

//左下角的剪切区域
clip[2].x = 0;
clip[2].y = 100;
clip[2].w = 100;
clip[2].h = 100;

//右下角的剪切区域
clip[3].x = 100;
clip[3].y = 100;
clip[3].w = 100;
clip[3].h = 100;

//将图像应用到窗口中
apply_surface( 0, 0, dots, screen, &clip[0] );
apply_surface( 540, 0, dots, screen, &clip[1] );
apply_surface( 0, 380, dots, screen, &clip[2] );
apply_surface( 540, 380, dots, screen, &clip[3] );
```

现在，当你想要使用很多图片时，你不必保存成千上万个图片文件。你可以将一个子图集合放入一个单独的图片文件中，并blit你想要使用的部分。

## True Type 字体

SDL 本身不原生地支持 TTF 文件，所以你需要使用 `SDL_ttf` 扩展库。SDL_ttf 是一个能从 True Type 字体中生成表面的的扩展库。

``` c++
TTF_Font *font = nullptr;
SDL_Color textColor = {255, 255, 255};

TTF_Init();

font = TTF_OpenFont("lazy.ttf", 28);
SDL_Surface *message = TTF_RenderText_Solid(font, "The quick brown fox jumps over the lazy dog", textColor);

// 清理
TTF_CloseFont(font);
TTF_Quit();
```

## 处理键盘事件

``` c++
if (SDL_PollEvent(&event)) {
  if (event.type == SDL_KEYDOWN) {
    switch (event.key.keysym.sym) {
    case SDLK_UP:
      // ...
      break;
    case SDLK_DOWN:
      // ...
      break;
    }
  }
}
```

## 处理鼠标事件

``` c++
if(event.type == SDL_MOUSEBUTTONDOWN) {
  if(event.button.button == SDL_BUTTON_LEFT) {
    // ...
  }
}
```

## 按键状态

在不使用事件的情况下通过按键状态检查一个按键是否被按下。

``` c++
//获取按键状态
Uint8 *keystates = SDL_GetKeyState(nullptr);
if(keystates[SDLK_UP]) {
  // ...
}
```

其他相关函数：`SDL_GetModState()` `SDL_GetMouseState()` `SDL_JoystickGetAxis()`。

## 播放声音

播放声音是游戏编程的又一个关键概念。SDL原生的声音函数可能会让人很纠结。所以，你将学习使用SDL_mixer扩展库来播放声效和音乐。SDL_mixer扩展库将使用声音变得非常容易。

对于那些使用OGG, MOD或者其他非WAV声音格式的人，请使用Mix_Init()来初始化解码器，并使用Mix_Quit()来关闭解码器。

``` c++
//将要播放的音乐
Mix_Music *music = NULL;

//将要使用的声效
Mix_Chunk *scratch = NULL;
Mix_Chunk *high = NULL;
Mix_Chunk *med = NULL;
Mix_Chunk *low = NULL;

//初始化SDL_mixer
Mix_OpenAudio(22050, MIX_DEFAULT_FORMAT, 2, 4096);

//加载音乐
music = Mix_LoadMUS( "beat.wav" );
//加载声效
scratch = Mix_LoadWAV( "scratch.wav" );
high = Mix_LoadWAV( "high.wav" );
med = Mix_LoadWAV( "medium.wav" );
low = Mix_LoadWAV( "low.wav" );

Mix_PlayMusic( music, -1 );
//Mix_PauseMusic();
//Mix_ResumeMusic();
Mix_PlayChannel( -1, scratch, 0 );

// 释放资源
//释放声效
Mix_FreeChunk( scratch );
Mix_FreeChunk( high );
Mix_FreeChunk( med );
Mix_FreeChunk( low );
//释放音乐
Mix_FreeMusic( music );
//退出SDL_mixer
Mix_CloseAudio();
```

## 计时

``` c++
Uint32 start = SDL_GetTicks();
// ...
std::cout << SDL_GetTicks() - start << std::endl;
```

  [1]: http://tjumyk.github.io/sdl-tutorial-cn/contents.html
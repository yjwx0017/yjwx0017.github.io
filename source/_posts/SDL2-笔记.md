title: SDL2 笔记
categories: 日志
tags: [SDL]
date: 2017-05-21 12:51:00
---
## 《SDL Game Development》笔记

坐标系
原点：左上角

SDL扩展
SDL_image
  支持多种格式图片加载：BMP GIF PNG TGA PCX ...
SDL_net
  跨平台网络库
SDL_mixer
  audio mixer library. 支持MP3 MIDI OGG
SDL_ttf
  支持TrueType字体
SDL_rtf
  support the rendering of the Rich Text Format (RTF).

SDL_Init()
初始化标识
```
SDL_INIT_HAPTIC      Force feedback subsystem  力反馈
SDL_INIT_AUDIO       Audio subsystem
SDL_INIT_VIDEO       Video subsystem
SDL_INIT_TIMER       Timer subsystem
SDL_INIT_JOYSTICK    Joystick subsystem
SDL_INIT_EVERYTHING  All subsystems
SDL_INIT_noparachute Don't catch fatal signals
```

查看一个子系统是否已被初始化：

``` c++
if (SDL_WasInit(SDL_INIT_VIDEO) != 0)
    cout << "video was initialized";
```

```
SDL_CreateRenderer()
SDL_RENDERER_SOFTWARE      Use software rendering
SDL_RENDERER_ACCELERATED   Use hardware acceleration
SDL_RENDERER_PRESENTVSYNC  Synchronize renderer update with screen's refresh rate
SDL_RENDERER_TARGETTEXTURE Supports render to texture
```

游戏程序结构
初始化
游戏循环：获取输入 物理运算 渲染
退出

window flags
```
SDL_WINDOW_FULLSCREEN      Make the window fullscreen
SDL_WINDOW_OPENGL          Window can be used with as an OpenGL context
SDL_WINDOW_SHOWN           The window is visible
SDL_WINDOW_HIDDEN          Hide the window
SDL_WINDOW_BORDERLESS      No border on the window
SDL_WINDOW_RESIZABLE       Enable resizing of the window
SDL_WINDOW_MINIMIZED       Minimize the window
SDL_WINDOW_MAXIMIZED       Maximize the window
SDL_WINDOW_INPUT_GRABBED   Window has grabbed input focus
SDL_WINDOW_INPUT_FOCUS     Window has input focus
SDL_WINDOW_MOUSE_FOCUS     Window has mouse focus
SDL_WINDOW_FOREIGN         The window was not created using SDL
```

程序基本结构

``` c++
#include <SDL.h>

SDL_Window* g_pWindow = 0;
SDL_Renderer* g_pRenderer = 0;

int main(int argc, char* argv[])
{
	// initialize SDL
	if (SDL_Init(SDL_INIT_EVERYTHING) >= 0)
	{
		// if succeeded create our window
		g_pWindow = SDL_CreateWindow("Chapter 1: Setting up SDL",
			SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED,
			640, 480,
			SDL_WINDOW_SHOWN);

		// if the window creation succeeded create our renderer
		if (g_pWindow != 0)
		{
			g_pRenderer = SDL_CreateRenderer(g_pWindow, -1, 0);
		}
	}
	else
		return 1; // sdl could not initialize

	// everything succeeded lets draw the window
	bool bQuit = false;
	while (true)
	{
		SDL_Event event;
		if (SDL_PollEvent(&event))
		{
			switch (event.type)
			{
			case SDL_QUIT:
				bQuit = true;
				break;
			}
		}
		if (bQuit)
			break;

		// set to black
		// This function expects Red, Green, Blue and Alpha as color values
		SDL_SetRenderDrawColor(g_pRenderer, 0, 0, 0, 255);

		// clear the window to black
		SDL_RenderClear(g_pRenderer);

		// show the window
		SDL_RenderPresent(g_pRenderer);
	}

	// set a delay before quitting
	//SDL_Delay(5000);

	// clean up SDL
	SDL_Quit();

	return 0;
}
```

绘制图片过程

 1. 加载图片获得SDL_Surface
 2. 根据Surface获得SDL_Texture
 3. 获得纹理尺寸：SDL_QueryTexture
 4. 渲染：SDL_RenderCopy/SDL_RenderCopyEx 需要Renderer参数

SDL使用两种数据结构渲染到屏幕
SDL_Surface   像素集  使用软件渲染（not GPU）
SDL_Texture   使用硬件加速

``` c++
SDL_Texture* m_pTexture
SDL_Rect m_sourceRectangle;
SDL_Rect m_destinationRectangle;

// 根据图片创建SDL_Texture
SDL_Surface* pTempSurface = SDL_LoadBMP("asserts/rider.bmp");
m_pTexture = SDL_CreateTextureFromSurface(m_pRenderer, pTempSurface);
SDL_FreeSurface(pTempSurface);

// 获得图片的大小
SDL_QueryTexture(m_pTexture, NULL, NULL,
    &m_sourceRectangle.w, &m_sourceRectangle.h);

SDL_RenderClear();
SDL_RenderCopy(m_pRenderer, m_pTexture,
    &m_sourceRectangle, &m_destinationRectangle);
SDL_RenderPresent();
```

源矩形 目标矩形 参数传入0 将整个纹理渲染到整个窗口。

SDL_GetTicks()  毫秒

SDL_RenderCopyEx() 支持旋转和翻转Flip

SDL_image
sdl2_image.lib
``` c++
#include <sdl_image.h>
SDL_Surface* pTempSurface = IMG_Load("animate.png");
```

Fixed frames per second (FPS) is
not necessarily always a good option, especially when your game includes more
advanced physics. It is worth bearing this in mind when you move on from this
book and start developing your own games. Fixed FPS will, however, be fine for
the small 2D games, which we will work towards in this book.

固定帧频

``` c++
const int FPS = 60;
const int DELAY_TIME = 1000.0f / FPS;  // 1000毫秒
while (...)
{
    frameStart = SDL_GetTicks();  // 毫秒
    ...
    frameTime = SDL_GetTicks() - frameStart;
    if (frameTime < DELAY_TIME)
        SDL_Delay((int)(DELAY_TIME - frameTime));
}
```

```
SDL joystick event
SDL_JoyAxisEvent   Axis motion information
SDL_JoyButtonEvent Button press and release information
SDL_JoyBallEvent   Trackball event motion information
SDL_JoyHatEvent    Joystick hat position change

SDL joystick event    Type value
SDL_JoyAxisEvent      SDL_JOYAXISMOTION
SDL_JoyButtonEvent    SDL_JOYBUTTONDOWN or
                      SDL_JOYBUTTONUP
SDL_JoyBallEvent      SDL_JOYBALLMOTION
SDL_JoyHatEvent       SDL_JOYHATMOTION
```

不同的游戏控制器 按钮和轴可能有不同的值 比如Xbox360 controller, PS3 controller
Xbox360 controller:
Two analog sticks
Analog sticks press as buttons
Start and Select buttons
Four face buttons: A, B, X, and Y
Four triggers: two digital and two analog
A digital directional pad

``` c++
if (SDL_WasInit(SDL_INIT_JOYSTICK) == 0)
    SDL_InitSubSystem(SDL_INIT_JOYSTICK);
if (SDL_NumJoysticks() > 0)
{
    for (int i=0; i<SDL_NumJoysticks(); ++i)
    {
        SDL_Joystick* joy = SDL_JoystickOpen(i);
        if (SDL_JoystickOpened(i) == 1)
            m_joysticks.push_back(joy);
    }
    SDL_JoystickEventState(SDL_ENABLE);
}

SDL_JoystickClose(joy);
```

分辨是哪个控制器的事件

``` c++
if (event.type == SDL_JOYAXISMOTION)
    int whichOne = event.jaxis.which;
```

控制器按钮
SDL_JoystickNumButtons
event.jbutton.button    // 按钮ID

鼠标事件
```
SDL Mouse Event        Purpose
SDL_MouseButtonEvent   A button on the mouse has been pressed or released
SDL_MouseMotionEvent   The mouse has been moved
SDL_MouseWheelEvent    The mouse wheel has moved

SDL Mouse Event        Type Value
SDL_MouseButtonEvent   SDL_MOUSEBUTTONDOWN or SDL_MOUSEBUTTONUP
SDL_MouseMotionEvent   SDL_MOUSEMOTION
SDL_MouseWheelEvent    SDL_MOUSEWHEEL
```

``` c++
// 鼠标按钮
// SDL numbers these as 0 for left, 1 for middle, and 2 for right.

if (event.type == SDL_MOUSEBUTTONDOWN)
    if (event.button.button == SDL_BUTTON_LEFT)
```

event.type  SDL_MOUSEMOTION
event.motion.x
event.motion.y

// 键盘
1 表示按下  0 表示没有按下
SDL_GetKeyboardState(int* numkeys)
Uint8* m_keystates;
m_keystates = SDL_GetKeyboardState(0);
SDL_Scancode key;
if (m_keysttes[key] == 1)

有限状态机
需要能够处理以下情况：
Removing one state and adding another
Adding one state without removing the previous state
Removing one state without adding another

Game
GameObject
TextureManager     // 负责加载图片文件，负责绘制，纹理ID
InputHandler
GameState
GameStateMachine
GameObjectFactory
StateParser

Distributed Factory
class GameObjectFactory
std::map<std::string, BaseCreator*> m_creators;
registerType(std::string typeID, BaseCreator* pCreator);


https://github.com/ReneNyffenegger/development_misc/tree/master/base64.
The base64.h and base64.cpp files can be added directly to the project.

``` c++
SDL_Mixer
Mix_OpenAudio(int frequency, Uint16 format, int channels, int chunksize)
Mix_OpenAudio(22050, AUDIO_S16, 2, 4096);
std::map<std::string, Mix_Chunk*> mSfxs;
std::map<std::string, Mix_Chunk*> mMusic;

Mix_Music* music = Mix_LoadMUS(filename);  // ogg
Mix_Chunk* chunk = Mix_LoadWAV(filename);  // wav

Mix_PlayMusic()     // Mix_Music
Mix_PlayChannel()   // Mix_Chunk
Mix_CloseAudio();
```

SDL_SetTextureAlphaMod()
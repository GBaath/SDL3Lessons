# Lesson 3 – File Splitting, Application Time, Timers & Framerate
<img src="https://64.media.tumblr.com/dd77151a8a469ac9410a06417d2eb609/tumblr_nopx25y1zs1rt8r50o1_400.gif" width="100%">

> *"Jarvis, generate me a SDL3 gameloop in a main.cpp"*  
> – **Ironman**

---

## Intro

It's time to axe the single file spaghetti and split our code into a proper workspace. 
Including how to split declarations and defenitions in headers and source with some inheritance, and finally how to stop your pc from getting fried by capping the framerate.

---

## Concepts

- C++ Files & Inheritance
- Keeping it modular
- Framerate

---

## Files

### Headers & Source

C++ uses header files (.h or .hpp) for declarations, and source files (.cpp) for their definitions.
Splitting code helps readability, speeds up builds, and makes it clear which parts of your API other modules interact with.
Most importantly, it forces you to think when you couple files together.

Example file layout: 
<details>
<summary>GameObject.h</summary>

 ```cpp

#pragma once
#include <SDL3/SDL.h>

class GameObject {
public:
    GameObject( SDL_Renderer* renderer, int x, int y, int w, int h);
    GameObject( SDL_Renderer* renderer, int x, int y, int w, int h,SDL_Texture* texture);
    ~GameObject();

    virtual void HandleEvent(const SDL_Event& e);
    virtual void Render();
    virtual void Update(float DeltaTime, float ScaledDeltaTime);
    virtual void SetPosition(int x, int y);

protected:
    SDL_Renderer* renderer = nullptr;
    SDL_FRect rect;
    SDL_Texture* texture = nullptr;
};


```
</details>
<details>
<summary>GameObject.cpp</summary>

 ```cpp

#include "GameObject.h"

GameObject::GameObject(SDL_Renderer* renderer, int x, int y, int w, int h)
    : renderer(renderer) {
    rect = { (float)x, (float)y, (float)w, (float)h };
    texture = nullptr;
}
GameObject::GameObject(SDL_Renderer* renderer, int x, int y, int w, int h, SDL_Texture* texture)
    : renderer(renderer), texture(texture) {
        rect = { (float)x, (float)y, (float)w, (float)h };
}
GameObject::~GameObject() {}


void GameObject::HandleEvent(const SDL_Event& e)
{

}

void GameObject::Render() {


    if (texture)
        SDL_RenderTexture(renderer, texture, NULL, &rect);
    else
        SDL_RenderFillRect(renderer, &rect);
}

void GameObject::Update(float DeltaTime, float ScaledDeltaTime)
{
}

void GameObject::SetPosition(int x, int y) {
    rect.x = (float)x;
    rect.y = (float)y;
}


```
</details>
<details>
<summary>main.cpp</summary>

 ```cpp

#include "Game.h"
#include <SDL3/SDL.h>
#include <SDL3/SDL_image.h>

int main(int argc, char** argv) {

    Game game;

    if (!game.Init("Game", 800, 600)) {
        return -1;
    }

    while (game.running()) {
        game.HandleEvents();
        game.Update();
        game.Render();
    }

    game.Quit();
    return 0;
}

```
</details>

### Include paths

We (probably) don't want all our files in the root directory, so we should add our headers to a separate include path (where the compiler look for the declarations), and our cpps to a source subdirectory.
Adding a new include directory means we have to include it in our project, like in lesson 1, otherwise we can't actually use them.

<img src="images\includedir.PNG"/>

### Include guards vs #pragma once

To avoid circular dependancies in our headers we use include guards or #pragma ince.
We need them to prevent our program from becomming a ouroborus and creating a singularity in our cpu.
For instance, if class a has a reference to class b and vice versa, when compiling, they will get infinetly stacked on top of eachother and a *sick* explosion will occur.

#### Include guards always work B)

#ifndef	“If not defined…”

#define	Define a unique macro so the header gets locked

#endif	End of the conditional section

#### In some rare case, maybe if you're coding on a samsung smart frigde, this won't work.

#pragma once


tldr; just put #pragma once and don't worry about it until building for something wierd.

---

## Class Inheritance

### Base and derived classes

Works just like in C#, just with some different syntax.
We keep our overrideable functions virtual, and our variables protected to keep them avaliable.


<details>
<summary>GameObject.h</summary>

 ```cpp

#pragma once
#include <SDL3/SDL.h>

class GameObject {
public:
    GameObject( SDL_Renderer* renderer, int x, int y, int w, int h);
    GameObject( SDL_Renderer* renderer, int x, int y, int w, int h,SDL_Texture* texture);
    ~GameObject();

    virtual void HandleEvent(const SDL_Event& e);
    virtual void Render();
    virtual void Update(float DeltaTime, float ScaledDeltaTime);
    virtual void SetPosition(int x, int y);

protected:
    SDL_Renderer* renderer = nullptr;
    SDL_FRect rect;
    SDL_Texture* texture = nullptr;
};


```
</details>


Notice how we need to specify an access level when we inherit, if set to private, we won't be able to access Player as a GameObject, and only the overrides or new members in Player.
the ovveride keyword is also after the the function.
<details>
<summary>Player.h</summary>

 ```cpp

#pragma once
#include "GameObject.h"
#include "InputManager.h"

class Player : public GameObject {
public:
	Player(SDL_Renderer* renderer, int x, int y, int w, int h, SDL_Texture* texture) : GameObject(renderer, x, y, w, h, texture) {	};
	~Player() {};


	virtual void Update(float DeltaTime, float ScaledDeltaTime) override;

};



```
</details>

### Pointers

Pointers are declared by putting * after the classname. This means the variable hold the memory adress of the contained type. Thus, are completely unreadable and if not dereferenced by putting * *infront*.

Class* ptr; //Read as as memory adress
*ptr; //read as the value contained

When accessing data in pointer references we use the -> operator.
If we have 2 pointer with the same value/adress, theese are functionally identical, modifying one will modify both.

We can also achive this with a *reference*.
For instance if we want to out put several values from a function we can input references to non->pointer variable and reading the value after we get the return value.

 <details>
<summary>Player.h</summary>

 ```cpp

int a = 0;
int b = 0;

int ModifyAB(int& ref, int value){
    ref = value;
    return ref;
}

a = ModifyAB(b, 1); 

//both a & b will equal 1


```
</details>

It might take a while to wrap your head around, my advice is to make wierd analogies about your code to an LLM until it clicks in your brain.

---

## Application Time

So that we actually can add something new this time, we'll throw in a bit of framerate lore.
Easiest way to do it is to delay the gameloop thread with SDL_Delay.
There's also support for timers and multithreading, but we have search engines for a reason B)

### Locking framrate

Without specifying any delays, the main thread will try and execute as fast as possible.
We'll define a new struct for all our framedata in our game header.
<details>
<summary>FrameData</summary>

 ```cpp

struct FrameData {
    int framecnt = 0;
    const Uint32 frameDelay = 1000 / 60;
    Uint32 frameStart = 0;
    Uint32 frameTime = 0;
    Uint64 lastTime = 0;
    float timeScale = 1.f;
    float deltaTime = 0;
    float scaledDeltaTime = 0;
    
};

```
</details>
In our our update function, we save the current application milliseconds for reference, convert it to seconds to calculate our deltatime.
After updating all our relevant gameobjects, we calculate our frametime, after which we use SDL_Delay with the remaining time of our frame delay.
<details>
<summary>Update</summary>

 ```cpp

void Game::Update() {
    Uint64 now = SDL_GetTicks();
    fdata.deltaTime = (now - fdata.lastTime) / 1000.0f; //seconds
    fdata.lastTime = now;

    //Time scale
    fdata.scaledDeltaTime = fdata.deltaTime * fdata.timeScale;

    //update all gos
    for (GameObject* go : gameObjects) {
        go->Update(fdata.deltaTime,fdata.scaledDeltaTime);
    }


    //delay to fps target
    fdata.frameStart = SDL_GetTicks();
    fdata.framecnt++;

    fdata.frameTime = SDL_GetTicks() - fdata.frameStart;

    if (fdata.frameTime < fdata.frameDelay) {
        SDL_Delay(fdata.frameDelay - fdata.frameTime);
    }
}

```
</details>


### Time scaling

Throwing in an extra timescale variable will allow us to dynamically change the speed of certains things without affecting the actual framerate.
Our game will still run in 60 fps, but using our scaledDeltaTime we can for example add slow-mo and so on.

---

## Code Example

Your code will probably/hopefully look a bit different than mine at this point, but these are parts of lessons 2's example code, split into more managable size.

<details>
<summary>main.cpp</summary>

 ```cpp

#include "Game.h"
#include "GameObject.h"
#include "ExampleCube.h"
#include "Player.h"
#include <SDL3/SDL.h>
#include <SDL3/SDL_image.h>

int main(int argc, char** argv) {

    Game game;

    if (!game.Init("Game", 800, 600)) {
        return -1;
    }

    //add cool sprite
    SDL_Renderer* ren = game.GetRenderer();
    if (ren) {
        //tranparency
        SDL_Surface* surf = IMG_Load("player.bmp");
        //yeet all color
        SDL_SetSurfaceColorKey(surf, true, SDL_MapSurfaceRGB(surf, 255, 255, 255)) == false;
        //new texture from modified surface
        SDL_Texture* keyed = SDL_CreateTextureFromSurface(ren, surf);
        //overwrite sprite
        auto sprite = SDL_CreateTextureFromSurface(ren, surf);

        SDL_DestroySurface(surf);
        game.AddGameObject(new Player(ren,0,0,50,50,sprite));

        //temp display cube
        auto blank = SDL_CreateTexture(ren, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET, 50, 50);
        game.AddGameObject(new ExampleCube( ren, 50, 50, 50, 50,blank));
    }
    else
        GameObject player(game.GetRenderer(), 0, 0, 50, 50);




    while (game.running()) {
        game.HandleEvents();
        game.Update();
        game.Render();
    }

    game.Quit();
    return 0;
}

```
</details>
<details>
<summary>Game.cpp</summary>

 ```cpp

#include "Game.h"
#include "InputManager.h"
#include <iostream>

Game::Game() {}
Game::~Game() {}

bool Game::Init(const std::string& title, int width, int height) {
    if (SDL_Init(SDL_INIT_VIDEO | SDL_INIT_GAMEPAD) == 0) {
        std::cerr << "Init Error: " << SDL_GetError() << "\n";
        return false;
    }

    window = SDL_CreateWindow(title.c_str(), width, height, SDL_WINDOW_RESIZABLE);
    if (!window) {
        std::cerr << "Window Error: " << SDL_GetError() << "\n";
        return false;
    }

    renderer = SDL_CreateRenderer(window, NULL);
    if (!renderer) {
        std::cerr << "Renderer Error: " << SDL_GetError() << "\n";
        return false;
    }


    gamepad = SDL_OpenGamepad(0);

    

    isRunning = true;
    return true;
}

void Game::HandleEvents() {
    SDL_Event event;
    while (SDL_PollEvent(&event)) {        
            if (event.key.key == SDLK_ESCAPE) {
                isRunning = false;
            }
    }
}

void Game::Update() {
    Uint64 now = SDL_GetTicks();
    fdata.deltaTime = (now - fdata.lastTime) / 1000.0f; //seconds
    fdata.lastTime = now;

    //Time scale
    fdata.scaledDeltaTime = fdata.deltaTime * fdata.timeScale;

    //update all gos
    for (GameObject* go : gameObjects) {
        go->Update(fdata.deltaTime,fdata.scaledDeltaTime);
    }


    //delay to fps target
    fdata.frameStart = SDL_GetTicks();
    fdata.framecnt++;

    fdata.frameTime = SDL_GetTicks() - fdata.frameStart;

    if (fdata.frameTime < fdata.frameDelay) {
        SDL_Delay(fdata.frameDelay - fdata.frameTime);
    }
}

void Game::Render() {

    //green B)
    SDL_SetRenderDrawColor(renderer, 50, 150, 50, 255);
    SDL_RenderClear(renderer);

    for (GameObject* go : gameObjects) {
        go->Render();
    }

    SDL_RenderPresent(renderer);
}

void Game::Quit() {
    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();
}


```
</details>

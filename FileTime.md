# Lesson 3 – File Splitting, Application Time, Timers & Framerate

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
### Virtuals & overrides
### Smart pointers

---

## Application Time

### Locking framrate
### Time scaling

---

## Code Example

<details>  
<summary>main.cpp</summary>  

```cpp
// SDL3 modular timing skeleton

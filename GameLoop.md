# Lesson 2 – Graphics and Input

---

## Intro

---

## Key Concepts

- Rendering basics
- Sprite management
- Input polling
- Event-driven systems

---

## Drawing to the Screen

### Render pipeline

Unlike complete game engines where we get premade for us, we have to manually manage all textures ourselves.
That means loading files, declaring textures to render, where to place them on the screen, preform any modifications, and finally presenting the backbuffer.

<details>
<summary>main.cpp</summary>

 ```cpp
#include <SDL3/SDL.h>
#include <SDL3/SDL_image.h>

int main(int argc, char** argv) {
    SDL_Init(SDL_INIT_VIDEO | SDL_INIT_GAMEPAD);
    SDL_Window* win = SDL_CreateWindow("SDL3 Demo",800, 600, 0);
    SDL_Renderer* ren = SDL_CreateRenderer(win, NULL);

    bool running = true;
    SDL_Event ev;

    while (running) {
        //Event handling
        while (SDL_PollEvent(&ev)) {
            if (ev.key.key == SDLK_ESCAPE) {
                running = false;
            }
        }

        //clear last frame
        SDL_RenderClear(ren);

        // draw calls
        
        //make rect to render sprite into
        SDL_FRect* dst = new SDL_FRect{ 100,100,64,64 };
        SDL_RenderTexture(ren, sprite, NULL, dst);

        SDL_RenderPresent(ren);
    }

    SDL_DestroyRenderer(ren);
    SDL_DestroyWindow(win);
    SDL_Quit();
    return 0;
}

```
</details>

In this example, we never actually move or modify the sprite in anyway, so we would get the same result if we only rendered once.

---

### Managing textures

#### Color keying for transparancy

#### Sprite clipping and stretching

#### Rotation


---

## Handling Input

### Keyboard & Mouse

### Gamepads / Joysticks

### Polling vs Events

---

## Code Example

<details>
<summary>main.cpp</summary>

```cpp
// Paste your SDL3 code sample here


# Lesson 4 – Collision Detection, Camera Movement & Audio Playback

---

## Concepts

- Axis-Aligned Bounding Boxes (AABB)
- Viewport and camera transforms
- SDL_mixer for audio playback

---

## Collision Detection

### Bounding boxes

The most basic form of collision detection is comparing two hitboxes against eachother.
A "separating angles test" between two axis aligned bounding boxes (AABB).
In caveman terms: look at squares. one square overlap other squar. no ):< 

We'll add an intersection test to our example cube and check.

<details>
<summary>Examplecube.h</summary>

 ```cpp

class ExampleCube : public GameObject {
public:
	ExampleCube(SDL_Renderer* renderer, int x, int y, int w, int h, SDL_Texture* texture) : GameObject(renderer, x, y, w, h, texture) {	};
	~ExampleCube() {};


	virtual void Update(float DeltaTime, float ScaledDeltaTime) override;
	virtual void Render() override;

	bool Intersects(const SDL_FRect& other) const {

		bool noOverlap =
			rect.x + rect.w <= other.x ||  //left of other
			other.x + other.w <= rect.x || //right of other
			rect.y + rect.h <= other.y ||  //above other
			other.y + other.h <= rect.y;   //is below other

		return !noOverlap;
	}

```
</details>

We can now check an input rect against it and determine if the overlap.
Notice the const keywords, theese show the compilera that we will *not* be modifying the statement following it.
Since we're working with pointers and references this is a form of extra safety and is good for readability.

### Resolving Collisions

Where do we use this new and fancy intersection check? Depending on how overkill you want to make your collision system, the answer can vary from "three inheritance layers deep in an intricate, layered physics manager" to "no".
For this example we'll settle by shoving it directly into our player class.
Nothing wrong with this, but if you're used to game engines, this feels kinda yucky.

<details>
<summary>Player.cpp</summary>

 ```cpp
void Player::Update(float DeltaTime, float ScaledDeltaTime)
{
    GameObject::Update(DeltaTime, ScaledDeltaTime);

    float speed = 7.f;

    if (InputManager::IsKeyDown(SDL_SCANCODE_W))
        Move(0, -speed, *ExampleObstacle);
    if (InputManager::IsKeyDown(SDL_SCANCODE_S)) 
        Move(0, speed, *ExampleObstacle);
    if (InputManager::IsKeyDown(SDL_SCANCODE_A)) 
        Move(-speed, 0, *ExampleObstacle);
    if (InputManager::IsKeyDown(SDL_SCANCODE_D)) 
        Move(speed, 0, *ExampleObstacle);
        

}

void Player::Move(float dX, float dY, const ExampleCube& exCube)
{
    //moving in X
    rect.x += dX;
    if (exCube.Intersects(rect)) {
        if (dX > 0) { //moving right
            rect.x = exCube.rect.x - rect.w;
        }
        else if (dX < 0) { //moving left
            rect.x = exCube.rect.x + exCube.rect.w;
        }
    }
    //moving in Y
    rect.y += dY;
    if (exCube.Intersects(rect)) {
        if (dY > 0) { //moving down
            rect.y = exCube.rect.y - rect.h;
        }
        else if (dY < 0) { //moving up
            rect.y = exCube.rect.y + exCube.rect.h;
        }
    }
}



```
</details>

What we're doing here is after shifting our react with the movement input,
check against our referenced example obstacle, and if overlapping, shove our rect back to where it came from.

<img src="images/vsExCube.PNG" width="50%">


## Camera

Making a camera is actually just about modifying the rendered rect before displaying it, imagine a rect within a rect, the inner one being the camera view.
Rendering the game world under this rect is the same as shifting the objects in the world by the cameras coordinates.

<img src="images/asuperclearcameradescription.png" width="50%">

### Camera class

We start by making new camera class, for this example we wont bother with resolution, we're just demoing how the camera works.
We will save a camera instance, and pass it into our game.render function, since we do our rendering logic on the gameobject level.

<details>
<summary>Camera.cpp</summary>

 ```cpp

#pragma once

#include <SDL3/SDL.h>

class Camera {
public:
    Camera(float x, float y)
        : x(x), y(y) {}

    void Move(float dx, float dy) {
        x += dx;
        y += dy;
    }
    SDL_FRect WorldToScreen(const SDL_FRect& worldRect) const {
        return {
            worldRect.x - x,
            worldRect.y - y,
            worldRect.w,
            worldRect.h
        };
    }

private:
    float x, y;
};



```
</details>
<details>
<summary>Camera.cpp</summary>

 ```cpp


void GameObject::Render(const Camera& cam) {

    SDL_FRect screenRect = cam.worldToScreen(rect);

    if (texture)
        SDL_RenderTexture(renderer, texture, NULL, &screenRect);
    else
        SDL_RenderFillRect(renderer, &screenRect);
}



```
</details>

All our gameobject will now be offset by our camera position while rendering, while still retaining thier real worldposition, logically.

## Audio Playback

In order to play audio we need another plugin, just like SDL_image. If you have stolen the precompiled libraries from lesson one, all that's needed is adding the header in the #includes.
As of writing this, the SDL_mixer addon is still very much in development, thus, it might be a bit problematic to find good sources.
My best advice is to trust your gut feeling and read the internal documentation in the header file.

## SFX

Current objective: play audio.
We will do this by creating a new audioplayer class, initalizing mixers, defining load and play logic, and hooking into to our main file.



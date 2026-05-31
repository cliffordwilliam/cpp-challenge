# Sprout Lands Rebuild

> Do not look at the original repo for code. It exists only to prove the goal is reachable.

Build the same thing from scratch, understanding every line you write. No remote push required — local git commits are still useful, commit after each quest so you have a checkpoint.

---

## Quest 1: Hello, C++

**Goal:** Write, compile, and run a C++ program using only the compiler directly — no build system yet.

### What to Do

- Create a new directory for your rebuild project (not inside the original repo)
- Initialize a git repo in it
- Write a main.cpp that prints something to the terminal
- Compile it manually with g++ and run the binary

### Acceptance Criteria

- [x] You compiled with a raw g++ command (no CMake, no Makefile)
- [x] The binary runs and prints output
- [x] You understand what each part of the g++ command means

### Hints

- g++ main.cpp -o hello is the minimal form
- ./hello runs it

---

## Quest 2: CMake Build System

**Goal:** Replace the manual g++ command with CMake so you never have to remember compile flags by hand.

### What to Do

- Write a CMakeLists.txt that declares a project and an executable target
- Configure and build using the two-step CMake workflow
- Make sure the binary still runs

### Acceptance Criteria

- [x] cmake -S . -B build configures without errors
- [x] cmake --build build produces a binary
- [x] You can explain what -S and -B mean
- [x] You know what the build/ directory is and why it should be gitignored

### Hints

- Minimum viable CMakeLists.txt needs: cmake_minimum_required, project, add_executable
- Add build/ to .gitignore before your first commit

---

## Quest 3: SDL3 as a Git Submodule

**Goal:** Add SDL3 to your project as a vendored dependency using git submodules, and wire it into CMake.

### What to Do

- Add SDL3's source repository as a submodule under vendored/SDL, pinned to tag release-3.4.8
- Update CMakeLists.txt to build SDL3 via add_subdirectory and link it to your executable
- Verify the project still builds

### Acceptance Criteria

- [x] vendored/SDL exists and contains SDL3 source
- [x] .gitmodules file is committed
- [x] Your executable links against SDL3 (target_link_libraries)
- [x] A fresh clone with git clone --recurse-submodules would work
- [x] You can explain what a submodule is and why it's different from copying the files in

### Hints

- git submodule add <url> vendored/SDL then git submodule update --init
- SDL3's CMake target name is SDL3::SDL3
- You'll need add_subdirectory(vendored/SDL) before target_link_libraries

---

## Quest 4: Open an SDL3 Window

**Goal:** Write the SDL3 application lifecycle that opens a window and keeps it open until the user closes it.

### What to Do

- Use SDL3's modern app-lifecycle functions: SDL_AppInit, SDL_AppIterate, SDL_AppEvent, SDL_AppQuit
- Open a window with a title and a fixed size
- Handle the quit event so the window closes cleanly

### Acceptance Criteria

- [ ] A window opens on launch
- [ ] Closing the window exits the program cleanly (no crash, no hang)
- [ ] You understand the render loop: SDL_AppIterate is called every frame
- [ ] You are NOT using SDL_Init / SDL_Quit / main in the old SDL2 style

### Hints

- SDL3 changed the entry point model — look for SDL_MAIN_USE_CALLBACKS and the four lifecycle functions
- The window needs both SDL_CreateWindow and SDL_CreateRenderer
- SDL_AppResult return values control whether the loop continues or exits

---

## Quest 5: Render a Colored Rectangle

**Goal:** Draw something on screen each frame to prove your render loop is working.

### What to Do

- In SDL_AppIterate, clear the screen, draw a filled colored rectangle, then present
- The rectangle should be at a fixed position with a fixed size

### Acceptance Criteria

- [ ] A colored filled rectangle is visible on screen
- [ ] The background is a different color from the rectangle
- [ ] You understand the three-step render pattern: clear → draw → present

### Hints

- SDL_SetRenderDrawColor sets the current color
- SDL_RenderClear fills the whole screen with the current color
- SDL_RenderFillRect draws a filled rect
- SDL_RenderPresent flips the buffer to screen

---

## Quest 6: GitHub Actions CI

**Goal:** Set up a CI workflow that automatically builds your project on every push.

### What to Do

- Create .github/workflows/ci.yml
- The workflow should: check out the repo (with submodules), install SDL3's apt dependencies, configure with CMake, and build
- Push to a remote and verify the workflow runs green (optional — or just write the file and trust the syntax)

### Acceptance Criteria

- [ ] The workflow file is syntactically correct YAML
- [ ] It checks out with submodules: recursive so SDL3 is available
- [ ] It installs all necessary apt packages before configuring
- [ ] CMake configure and build steps are present

### Hints

- SDL3 needs several apt packages to build on Ubuntu: libx11-dev, libxext-dev, and others — read SDL3's build docs for the full list
- sudo apt-get update must be its own step, not chained with && into the install step
- actions/checkout@v4 with with: submodules: recursive

---

## Quest 7: clang-format + Format Check in CI

**Goal:** Enforce consistent code formatting automatically in CI so formatting is never a manual debate.

### What to Do

- Add a .clang-format config file at the repo root
- Add a CI step (before the build) that runs clang-format over all source files and fails if anything changed
- Verify locally that your code passes the format check

### Acceptance Criteria

- [ ] .clang-format exists and defines a style
- [ ] CI has a step that runs the format check
- [ ] The check uses git diff --exit-code to detect any formatting changes
- [ ] Your code is already formatted correctly (CI passes)

### Hints

- clang-format -i src/**/*.cpp src/**/*.hpp formats in-place
- Run the format locally first, then commit, so CI passes immediately
- git diff --exit-code exits non-zero if there are any changes — that's how you make CI fail

---

## Quest 8: Constants Header

**Goal:** Eliminate magic numbers from your code by moving all tile, window, and game constants into a dedicated header.

### What to Do

- Create src/constants.hpp
- Put constants in a constants:: namespace (e.g., TILE_SIZE, WINDOW_WIDTH, MAP_ROWS, etc.)
- Replace every magic number in your other files with the named constant

### Acceptance Criteria

- [ ] No magic numbers remain in main.cpp or other source files
- [ ] All constants live under the constants:: namespace
- [ ] The project still builds and runs identically

---

## Quest 9: Draw a Tilemap

**Goal:** Define a 2D tile map and render it as a grid of colored rectangles, with different colors for different tile types.

### What to Do

- Define a 2D array (e.g., int map[ROWS][COLS]) where different integers mean different tile types
- In SDL_AppIterate, iterate over the array and draw each tile as a colored rect at the correct screen position
- Use at least two tile types (e.g., floor and wall)

### Acceptance Criteria

- [ ] A grid of tiles is visible on screen
- [ ] At least two tile types render as distinct colors
- [ ] Tile positions are derived from row/column index and constants::TILE_SIZE — no hard-coded pixel positions
- [ ] The map array is defined in one place, not scattered across the render code

---

## Quest 10: Player Movement

**Goal:** Draw a player rectangle that moves around the map in response to keyboard input.

### What to Do

- Track a player position (floating point, not integer — you'll need sub-pixel precision later)
- In SDL_AppEvent, respond to SDL_EVENT_KEY_DOWN / SDL_EVENT_KEY_UP to track which keys are held
- In SDL_AppIterate, update the player's position each frame based on held keys and a movement speed
- Render the player as a colored rectangle

### Acceptance Criteria

- [ ] Player moves in four directions with WASD or arrow keys
- [ ] Movement is frame-rate dependent in a predictable way (speed is in pixels/frame or pixels/second)
- [ ] Player position is stored as floats
- [ ] The player rect is drawn on top of the tilemap

---

## Quest 11: FixedBuffer Utility

**Goal:** Write a generic, header-only fixed-size buffer that avoids heap allocation.

### What to Do

- Create src/utils/fixed_buffer.hpp
- Implement a template FixedBuffer<T, MaxSize> backed by a stack array
- It should support: add(item), begin(), end(), and a count member
- It should never allocate on the heap

### Acceptance Criteria

- [ ] FixedBuffer<int, 16> buf; compiles
- [ ] buf.add(42) adds an element
- [ ] for (auto x : buf) iterates over added elements
- [ ] Adding more than MaxSize elements does not silently corrupt memory (assert or clamp)
- [ ] Zero heap allocation — no new, no std::vector internally

---

## Quest 12: FRect Utilities

**Goal:** Write a small header-only library of helper functions for SDL_FRect to avoid repeating geometry math everywhere.

### What to Do

- Create src/utils/frect_utils.hpp
- Put helpers under a frect_utils:: namespace
- At minimum implement: get the top-left corner, get the center, get the top-right, get the bottom-left, get the bottom-right

### Acceptance Criteria

- [ ] All helpers are inline functions in the header (no .cpp file needed)
- [ ] They return SDL_FPoint where appropriate
- [ ] They work correctly for any SDL_FRect input
- [ ] They live under the frect_utils:: namespace

---

## Quest 13: DDA Ray Casting

**Goal:** Implement Digital Differential Analysis (DDA) ray casting to detect which tile a ray hits.

### What to Do

- Cast a ray from the player's center toward the mouse cursor position
- Use DDA to step through the tilemap cell by cell along the ray
- Stop at the first solid tile the ray intersects
- Render the ray visually (draw a line from player to the hit point or to the mouse if no hit)

### Acceptance Criteria

- [ ] A ray is drawn from the player toward the mouse cursor each frame
- [ ] The ray stops at the first solid tile (not at the mouse if a wall is in the way)
- [ ] DDA is used — not brute-force pixel stepping
- [ ] You can explain what DDA is and why it's more efficient than checking every pixel

### Hints

- DDA works by computing how far along the ray you must travel to cross each grid line (X-axis and Y-axis separately), then taking the smaller step each iteration
- You need the ray direction as a normalized vector
- This is the same algorithm used in Wolfenstein 3D / early raycasters

---

## Quest 14: Swept Rect Candidate Finder

**Goal:** Given a moving rectangle (a start position and an end position), find all tilemap cells the rectangle could possibly overlap during its movement. This is the foundation of swept collision detection.

### What to Do

- Create src/physics/swept_tiles.hpp and src/physics/swept_tiles.cpp
- Implement a public function swept_tiles::get_swept_candidates(start_frect, end_frect, output) that fills output with tile coordinates
- The candidates are all tiles that the rect occupies at start, at end, or anywhere swept between them
- Internal helpers should be private to the translation unit (anonymous namespace in the .cpp)
- Use your FixedBuffer as the output container

### Acceptance Criteria

- [ ] The function signature lives in swept_tiles.hpp, implementation in swept_tiles.cpp
- [ ] Internal helpers are not exposed in the header
- [ ] For a rect that doesn't move, only the tiles it overlaps are returned
- [ ] For a rect that moves across several tiles, all intermediate tiles are included
- [ ] main.cpp calls the function and the results are used (even if just rendered as debug highlight)
- [ ] You can explain why candidates are needed before doing precise collision math

---

## Journal

### Quest 1: Hello, C++

**Date started:** May 29 2026
**Date finished:** May 29 2026

**What I did:**
Here is the commit: `ea3065f` for this quest.

I first created a new dir called `cpp-challenge`. Since I already have `git`, so I `git init` inside that new dir to initialize an empty git repo in there.

```bash
clif@clif-ROG-Strix-G513RS-G513RS:~/projects/cpp-challenge$ git --version
git version 2.54.0
clif@clif-ROG-Strix-G513RS-G513RS:~/projects/cpp-challenge$ git init
Initialized empty Git repository in /home/clif/projects/cpp-challenge/.git/
```

To be honest I do not really understand the internals of `git`, but I do know I can use it to persists my changes and branches and so on. So its like I know how to use it but internally what `git init` really does or what the `.git` directory really store, I am not sure. All I know is that the `.git` is where all my changes persists locally that I wanna push or sync to remote. So if I need to understand how `git` is at this step do let me know otherwise for now I think I wanna move on.

Then I had to look online on how to get a `c++` compiler for my Ubuntu machine. I found this page `https://documentation.ubuntu.com/ubuntu-for-developers/howto/gcc-setup/` and I think since I already have it so I just checked if its there. Note that this is the `gcc` and not just the `c++` compiler, `gcc` is the Ubuntu collection of compilers where `g++` is one of the many compilers it has. It says this is the de facto way so I think this is right.

```bash
clif@clif-ROG-Strix-G513RS-G513RS:~/projects/cpp-challenge$ gcc --version
gcc (Ubuntu 15.2.0-16ubuntu1) 15.2.0
Copyright (C) 2025 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

So just to be sure I tested this compiler on a raw text file to see if it can compile it into a working binary. Just so that there is feedback I plan to just stream int 5 to stdout file descriptor which should be my terminal emulator by default.

```c++
#include <iostream>

int main() {
    std::cout << 5;
    return 0;
};
```

It compiles and ran! So here is the proof that I have a working compiler.

```bash
clif@clif-ROG-Strix-G513RS-G513RS:~/projects/cpp-challenge$ g++ main.cpp --std=c++17 && ./a.out && rm a.out
5clif@clif-ROG-Strix-G513RS-G513RS:~/projects/cpp-challenge$
```

I compiled and ran it with `g++ main.cpp --std=c++17`. Why 17 is because I look up online and many people say `its most stable`. So honestly I am not sure but how to pick one but if many people seems to be okay with 17 then I am also sticking to 17.

**Errors and wrong turns:**
No errors.

**How I solved it:**
The compiler compiles! I just had to do simple googling.

**What I learned:**
I think I understood that I have knowledge gaps in how `git` actually works and not really knowing which compiler version to pick. Well not compiler version but like which c++ syntax to adhere to I guess.

**What felt hard:**
Nothing was hard for this one.

**Questions I still have:**
If I should know `git` and how to pick version deeper I guess.


### Quest 2: CMake Build System

**Date started:** May 20 2026
**Date finished:** May 20 2026

**What I did:**
Here is the commit for this quest `82648c7`

I need to learn what CMake is from `https://cmake.org/cmake/help/latest/guide/tutorial/Before%20You%20Begin.html`, CMake is a program that generates build system, build system is then used to create the final output.

I am in Ubuntu so I am using single config. So one directory for one build system for one type (Release or Debug) of output. And I am using Ninja generator. So I have `root/build/debug/` and `root/build/release/`, each hold its dedicated build system and final output.

I first config with the `-DCMAKE_BUILD_TYPE=Debug` which dictate in config step the type. I am picking Ninja with the `-G` flag during build. `-S` is going to be `.` since I plan to store my `CMakeFileLists.txt` for this project in the root. And I want the generated single config debug build system to be placed in `root/build/debug` so my `-B` is `build/debug`. Which means the directory is one to one with output type.

For now I think I just wanna pin CMake, name project, pin cpp, make my target and set its source property value and make it private too.

Note that the project specify the CXX there this is so that it does not waste time looking for C compiler, this is a C++ project only anyways so this clarifies it at a glance.

```cmake
cmake_minimum_required(VERSION 3.23)
project("Sprout Lands" LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(main)
target_sources(main PRIVATE main.cpp)
```

Then now I just wanna make the debug setup build system so `cmake -S . -B build/debug -G Ninja -DCMAKE_BUILD_TYPE=Debug`, then use the build system to create the final output `cmake --build build/debug`. Running the final output works too `./build/debug/main`.

I also add `.gitignore` to `build` dir since we want to track the os agnostic `CMakeLists.txt` file only.

**Errors and wrong turns:**
No error, I think I understand that for my use case I want `cmake -> Ninja generator -> build system (root/build/debug) -> build -> run it`

**How I solved it:**
Just read the docs here `https://cmake.org/cmake/help/latest/guide/tutorial/Before%20You%20Begin.html`

**What I learned:**
I understand that we write text os agnostic file `CMakeFileLists.txt`, then let CMake create build system out of it for this machine, let CMake use build system to generate final output. There is single and multi config. Single config means I have 1 directory for 1 build system for 1 build type output.

**What felt hard:**
Nothing was hard.

**Questions I still have:**
So far everything seems okay.


### Quest 3: SDL3 as a Git Submodule

**Date started:** May 31 2026
**Date finished:** May 31 2026

**What I did:**
This is the commit for this quest `6855bc6`

So first I look up the git docs to see if it has a command to make a sub-module.
I am sure the idea is that just like how we can pull remote repository to our local, there must be a means to also pull from remote but as a sub-module.
Where we can control where it is going to be placed in and which pinned branch snapshot to use, and a pointer that we track and push to remote.

I find that this is the docs `https://git-scm.com/book/en/v2/Git-Tools-Submodules`
The docs says the following on how to do it:

`git sub-module add <REPOSITORY_URL> <PATH_TO_DIRECTORY>`

Now that I know its possible I need to find the repository URL for the SDL's source repository, and put it in `root/vendored/SDL`.

I think it is this one! `https://github.com/libsdl-org/SDL.git`

So I am going to run this then I'll go in it to pin a certain branch which I think doing that should also adjust the pointer.

I found that info from this `https://stackoverflow.com/questions/75417355/how-does-one-git-submodule-add-a-specific-commit-and-have-it-be-recorded-in-the`

`git submodule add https://github.com/libsdl-org/SDL.git vendored/SDL`

I notice that after running that as expected I get the whole repository in that directory. And it also made a new file called `.gitmodules` which is the pointer that I wanna track.
However I do also notice that it tracks this new pulled repository differently as in it tracks its snapshot state.

I think for this project I want to pin it to `release-3.4.8` branch.

`git checkout release-3.4.8`

We pin so that it is a snapshot, anyone who pull gets the same stuff for consistent work!

And I just have to commit, then to pull this properly I have to `git clone --recurse-submodules <my repo with Submodules>`

That way I get my repository and the snapshot sub-module repository too.

Then now I have to update my `CMakeFileLists.txt` so that it tells CMake to also read the sub-module `CMakeFileLists.txt` to make its own targets.
But then I think I have to make it so that by default it does not build all the targets in the sub-module.
Only the targets that my project rely on gets built. So that `running make in the parent directory will not build targets in the sub-directory by default`

Here is the docs `https://cmake.org/cmake/help/latest/command/add_subdirectory.html`

After letting CMake reads the sub-module `CMakeLists.txt` I now then need to make sure that when building my project, it also builds the target in SDL that it relies on

`target_link_libraries`

The `SDL3::SDL3` is a special target that the docs have that I think has resources on the SDL stuff, I need to make sure that my main have access to it but I use private so that when other refer to my main target indirectly it will not have access to SDL3 resources, only when main is being used directly does it have access to the SDL3 resource.
The same is said about the main private main.cpp, that means main.cpp resource is only ever available to main target when main target is being used directly.

**Errors and wrong turns:**
No errors or misunderstandings.

**How I solved it:**
Was able to bring in sub-module and a means to pull the real resource from pointer too, and have CMake read that sub-module `CMakeFileLists.txt` and also have my project have access to its resources.

**What I learned:**
That target is referring to a thing that holds resources that compiler uses.

**What felt hard:**
Nothing yet so far.

**Questions I still have:**
Do I need to understand more on this or this is good enough for now?


### Quest 4: Open an SDL3 Window

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 5: Render a Colored Rectangle

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 6: GitHub Actions CI

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 7: clang-format + Format Check in CI

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 8: Constants Header

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 9: Draw a Tilemap

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 10: Player Movement

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 11: FixedBuffer Utility

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 12: FRect Utilities

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 13: DDA Ray Casting

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



### Quest 14: Swept Rect Candidate Finder

**Date started:** 
**Date finished:** 

**What I did:**


**Errors and wrong turns:**


**How I solved it:**


**What I learned:**


**What felt hard:**


**Questions I still have:**



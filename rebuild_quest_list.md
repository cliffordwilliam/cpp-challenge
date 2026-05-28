# Sprout Lands Rebuild — Quest List

**The rule:** Do not look at the original repo for code. It exists only to prove the goal is reachable.
**The goal:** Build the same thing from scratch, understanding every line you write.
**No remote push required.** Local git commits are still useful — commit after each quest so you have a checkpoint.

---

## How to use this file

Work one quest at a time. When you finish a quest, fill in the journal entry below it.
The journal is for you first, and for review second — write what actually happened, not what you wish happened.
Struggles and wrong turns are the most valuable part to record.

---

## Quest 1 — Hello, C++

**Goal:** Write, compile, and run a C++ program using only the compiler directly — no build system yet.

**What to do:**
- Create a new directory for your rebuild project (not inside the original repo)
- Initialize a git repo in it
- Write a `main.cpp` that prints something to the terminal
- Compile it manually with `g++` and run the binary

**Acceptance criteria:**
- [x] You compiled with a raw `g++` command (no CMake, no Makefile)
- [x] The binary runs and prints output
- [x] You understand what each part of the `g++` command means

**Hints (only read if stuck for 20+ min):**
- `g++ main.cpp -o hello` is the minimal form
- `./hello` runs it

---

## Quest 2 — CMake Build System

**Goal:** Replace the manual `g++` command with CMake so you never have to remember compile flags by hand.

**What to do:**
- Write a `CMakeLists.txt` that declares a project and an executable target
- Configure and build using the two-step CMake workflow
- Make sure the binary still runs

**Acceptance criteria:**
- [ ] `cmake -S . -B build` configures without errors
- [ ] `cmake --build build` produces a binary
- [ ] You can explain what `-S` and `-B` mean
- [ ] You know what the `build/` directory is and why it should be gitignored

**Hints:**
- Minimum viable `CMakeLists.txt` needs: `cmake_minimum_required`, `project`, `add_executable`
- Add `build/` to `.gitignore` before your first commit

---

## Quest 3 — SDL3 as a Git Submodule

**Goal:** Add SDL3 to your project as a vendored dependency using git submodules, and wire it into CMake.

**What to do:**
- Add SDL3's source repository as a submodule under `vendored/SDL`, pinned to tag `release-3.4.8`
- Update `CMakeLists.txt` to build SDL3 via `add_subdirectory` and link it to your executable
- Verify the project still builds

**Acceptance criteria:**
- [ ] `vendored/SDL` exists and contains SDL3 source
- [ ] `.gitmodules` file is committed
- [ ] Your executable links against SDL3 (`target_link_libraries`)
- [ ] A fresh clone with `git clone --recurse-submodules` would work
- [ ] You can explain what a submodule is and why it's different from copying the files in

**Hints:**
- `git submodule add <url> vendored/SDL` then `git submodule update --init`
- SDL3's CMake target name is `SDL3::SDL3`
- You'll need `add_subdirectory(vendored/SDL)` before `target_link_libraries`

---

## Quest 4 — Open an SDL3 Window

**Goal:** Write the SDL3 application lifecycle that opens a window and keeps it open until the user closes it.

**What to do:**
- Use SDL3's modern app-lifecycle functions: `SDL_AppInit`, `SDL_AppIterate`, `SDL_AppEvent`, `SDL_AppQuit`
- Open a window with a title and a fixed size
- Handle the quit event so the window closes cleanly

**Acceptance criteria:**
- [ ] A window opens on launch
- [ ] Closing the window exits the program cleanly (no crash, no hang)
- [ ] You understand the render loop: `SDL_AppIterate` is called every frame
- [ ] You are NOT using `SDL_Init` / `SDL_Quit` / `main` in the old SDL2 style

**Hints:**
- SDL3 changed the entry point model — look for `SDL_MAIN_USE_CALLBACKS` and the four lifecycle functions
- The window needs both `SDL_CreateWindow` and `SDL_CreateRenderer`
- `SDL_AppResult` return values control whether the loop continues or exits

---

## Quest 5 — Render a Colored Rectangle

**Goal:** Draw something on screen each frame to prove your render loop is working.

**What to do:**
- In `SDL_AppIterate`, clear the screen, draw a filled colored rectangle, then present
- The rectangle should be at a fixed position with a fixed size

**Acceptance criteria:**
- [ ] A colored filled rectangle is visible on screen
- [ ] The background is a different color from the rectangle
- [ ] You understand the three-step render pattern: clear → draw → present

**Hints:**
- `SDL_SetRenderDrawColor` sets the current color
- `SDL_RenderClear` fills the whole screen with the current color
- `SDL_RenderFillRect` draws a filled rect
- `SDL_RenderPresent` flips the buffer to screen

---

## Quest 6 — GitHub Actions CI

**Goal:** Set up a CI workflow that automatically builds your project on every push.

**What to do:**
- Create `.github/workflows/ci.yml`
- The workflow should: check out the repo (with submodules), install SDL3's apt dependencies, configure with CMake, and build
- Push to a remote and verify the workflow runs green (optional — or just write the file and trust the syntax)

**Acceptance criteria:**
- [ ] The workflow file is syntactically correct YAML
- [ ] It checks out with `submodules: recursive` so SDL3 is available
- [ ] It installs all necessary apt packages before configuring
- [ ] CMake configure and build steps are present

**Hints:**
- SDL3 needs several apt packages to build on Ubuntu: `libx11-dev`, `libxext-dev`, and others — read SDL3's build docs for the full list
- `sudo apt-get update` must be its own step, not chained with `&&` into the install step
- `actions/checkout@v4` with `with: submodules: recursive`

---

## Quest 7 — clang-format + Format Check in CI

**Goal:** Enforce consistent code formatting automatically in CI so formatting is never a manual debate.

**What to do:**
- Add a `.clang-format` config file at the repo root
- Add a CI step (before the build) that runs `clang-format` over all source files and fails if anything changed
- Verify locally that your code passes the format check

**Acceptance criteria:**
- [ ] `.clang-format` exists and defines a style
- [ ] CI has a step that runs the format check
- [ ] The check uses `git diff --exit-code` to detect any formatting changes
- [ ] Your code is already formatted correctly (CI passes)

**Hints:**
- `clang-format -i src/**/*.cpp src/**/*.hpp` formats in-place
- Run the format locally first, then commit, so CI passes immediately
- `git diff --exit-code` exits non-zero if there are any changes — that's how you make CI fail

---

## Quest 8 — Constants Header

**Goal:** Eliminate magic numbers from your code by moving all tile, window, and game constants into a dedicated header.

**What to do:**
- Create `src/constants.hpp`
- Put constants in a `constants::` namespace (e.g., `TILE_SIZE`, `WINDOW_WIDTH`, `MAP_ROWS`, etc.)
- Replace every magic number in your other files with the named constant

**Acceptance criteria:**
- [ ] No magic numbers remain in `main.cpp` or other source files
- [ ] All constants live under the `constants::` namespace
- [ ] The project still builds and runs identically

---

## Quest 9 — Draw a Tilemap

**Goal:** Define a 2D tile map and render it as a grid of colored rectangles, with different colors for different tile types.

**What to do:**
- Define a 2D array (e.g., `int map[ROWS][COLS]`) where different integers mean different tile types
- In `SDL_AppIterate`, iterate over the array and draw each tile as a colored rect at the correct screen position
- Use at least two tile types (e.g., floor and wall)

**Acceptance criteria:**
- [ ] A grid of tiles is visible on screen
- [ ] At least two tile types render as distinct colors
- [ ] Tile positions are derived from row/column index and `constants::TILE_SIZE` — no hard-coded pixel positions
- [ ] The map array is defined in one place, not scattered across the render code

---

## Quest 10 — Player Movement

**Goal:** Draw a player rectangle that moves around the map in response to keyboard input.

**What to do:**
- Track a player position (floating point, not integer — you'll need sub-pixel precision later)
- In `SDL_AppEvent`, respond to `SDL_EVENT_KEY_DOWN` / `SDL_EVENT_KEY_UP` to track which keys are held
- In `SDL_AppIterate`, update the player's position each frame based on held keys and a movement speed
- Render the player as a colored rectangle

**Acceptance criteria:**
- [ ] Player moves in four directions with WASD or arrow keys
- [ ] Movement is frame-rate dependent in a predictable way (speed is in pixels/frame or pixels/second)
- [ ] Player position is stored as floats
- [ ] The player rect is drawn on top of the tilemap

---

## Quest 11 — FixedBuffer Utility

**Goal:** Write a generic, header-only fixed-size buffer that avoids heap allocation.

**What to do:**
- Create `src/utils/fixed_buffer.hpp`
- Implement a template `FixedBuffer<T, MaxSize>` backed by a stack array
- It should support: `add(item)`, `begin()`, `end()`, and a `count` member
- It should never allocate on the heap

**Acceptance criteria:**
- [ ] `FixedBuffer<int, 16> buf;` compiles
- [ ] `buf.add(42)` adds an element
- [ ] `for (auto x : buf)` iterates over added elements
- [ ] Adding more than `MaxSize` elements does not silently corrupt memory (assert or clamp)
- [ ] Zero heap allocation — no `new`, no `std::vector` internally

---

## Quest 12 — FRect Utilities

**Goal:** Write a small header-only library of helper functions for `SDL_FRect` to avoid repeating geometry math everywhere.

**What to do:**
- Create `src/utils/frect_utils.hpp`
- Put helpers under a `frect_utils::` namespace
- At minimum implement: get the top-left corner, get the center, get the top-right, get the bottom-left, get the bottom-right

**Acceptance criteria:**
- [ ] All helpers are `inline` functions in the header (no `.cpp` file needed)
- [ ] They return `SDL_FPoint` where appropriate
- [ ] They work correctly for any `SDL_FRect` input
- [ ] They live under the `frect_utils::` namespace

---

## Quest 13 — DDA Ray Casting

**Goal:** Implement Digital Differential Analysis (DDA) ray casting to detect which tile a ray hits.

**What to do:**
- Cast a ray from the player's center toward the mouse cursor position
- Use DDA to step through the tilemap cell by cell along the ray
- Stop at the first solid tile the ray intersects
- Render the ray visually (draw a line from player to the hit point or to the mouse if no hit)

**Acceptance criteria:**
- [ ] A ray is drawn from the player toward the mouse cursor each frame
- [ ] The ray stops at the first solid tile (not at the mouse if a wall is in the way)
- [ ] DDA is used — not brute-force pixel stepping
- [ ] You can explain what DDA is and why it's more efficient than checking every pixel

**Hints:**
- DDA works by computing how far along the ray you must travel to cross each grid line (X-axis and Y-axis separately), then taking the smaller step each iteration
- You need the ray direction as a normalized vector
- This is the same algorithm used in Wolfenstein 3D / early raycasters

---

## Quest 14 — Swept Rect Candidate Finder

**Goal:** Given a moving rectangle (a start position and an end position), find all tilemap cells the rectangle could possibly overlap during its movement. This is the foundation of swept collision detection.

**What to do:**
- Create `src/physics/swept_tiles.hpp` and `src/physics/swept_tiles.cpp`
- Implement a public function `swept_tiles::get_swept_candidates(start_frect, end_frect, output)` that fills `output` with tile coordinates
- The candidates are all tiles that the rect occupies at start, at end, or anywhere swept between them
- Internal helpers should be private to the translation unit (anonymous namespace in the `.cpp`)
- Use your `FixedBuffer` as the output container

**Acceptance criteria:**
- [ ] The function signature lives in `swept_tiles.hpp`, implementation in `swept_tiles.cpp`
- [ ] Internal helpers are not exposed in the header
- [ ] For a rect that doesn't move, only the tiles it overlaps are returned
- [ ] For a rect that moves across several tiles, all intermediate tiles are included
- [ ] `main.cpp` calls the function and the results are used (even if just rendered as debug highlight)
- [ ] You can explain why candidates are needed before doing precise collision math

---

## You're done when...

All 14 quests are complete, journaled, and you can explain every line of code in your rebuild without referencing the original repo.

---

# Journal Template

Copy this block for each quest. Fill it in honestly — wrong turns are the interesting part.

---

## Journal — Quest N: [Quest Name]

**Date started:**
**Date finished:**

**What I did:**
(Walk through your approach. What did you try first?)

**Errors and wrong turns:**
(Paste any error messages you hit. Describe what you misunderstood.)

**How I solved it:**
(What finally worked? What was the key insight?)

**What I learned:**
(One or two sentences — what do you now understand that you didn't before?)

**What felt hard:**
(Honest rating and description of difficulty. What would you do differently?)

**Questions I still have:**
(Anything you got working but don't fully understand yet.)

---

## Journal — Quest 1: Hello, C++

**Date started: May 29 2026**
**Date finished: May 29 2026**

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

---


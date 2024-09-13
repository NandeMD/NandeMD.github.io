+++
title = "Weird Fullscreen Resolution Bug In Macroquad on Ubuntu"
date = "2024-09-13"
description = "Found an interesting bug while trying to write a game in Macroquad."

[taxonomies]
categories = ["bug", "windowing", "games"]
tags = ["rust", "games"]
+++

Because this is the first post in my blog, you probably will see some mistakes about grammar, styling, etc. I wanted to say sorry in advance.

# Why Macroquad?

As a hobbyist programmer when I get bored, I try to find myself a few project ideas that will entertain me and free me from my daily grim life.
So this time, I asked myself "You like playing games, why don't you try making one?" and decided to use Rust. The game will be a basic, 2D spaceship/alien shooting game.

I started to search for simple frameworks for Rust game development in [AreWeGameYet](https://arewegameyet.rs/ecosystem/engines/#crates) and found some options.

* [Bevy](https://github.com/bevyengine/bevy): I think this is a full fledged engine but I don't have a use for it's most features. Also what I want to make is a simple game so ECS is an extra burden.
* [ggez](https://github.com/ggez/ggez): Nice features, simpler than Bevy but still has too complicated and too many features I won't use.
* [Fyrox](https://github.com/FyroxEngine/Fyrox): Same problems with Bevy.
* [coffe](https://github.com/hecrj/coffee): I actually liked it but the last commit was 4 years ago.
* [Macroquad](https://github.com/not-fl3/macroquad): Really simple to draw things to a screen, no complicated macros, doesn't have much bloated dependencies so it's easy to build on my old but trusty laptop.

So I chose macroquad and got to work on my game.

# Trying to position the player

I want my player to be a spaceship, located on the bottom side of the screen, to have some margin and size relative to screen resolution (this is the part ehehe), and can move only left and right. Also I want a full-screen-only game. So let's get into code.

As I said, drawing things is really easy with macroquad so my `main` function is really neat.
```rust
#[macroquad::main(window_conf)]
async fn main() {
    log_info();

    let mut p = player::Player::new();

    loop {
        clear_background(BLACK);

        p.update();
        p.draw();

        next_frame().await
    }
}
```

Let's create a function to print some information about the system/game on the console:
```rust
fn log_info() {
    info!("Resolution: {}x{}", screen_width(), screen_height());
}
```

We also need to define the game window configuration with `Conf` struct for the game to start with fullscreen.

```rust
fn window_conf() -> Conf {
    Conf {
        window_title: "Shootin'".into(),
        fullscreen: true,
        ..Default::default()
    }
}
```

I also defined an `Entity` trait,  a `Player` struct, and in the `new()` function, i set the player's position to the bottom middle of the screen. I'm only showing the necessary part for the code.

```rust
const PLAYER_BASE_SIZE: f32 = 20.0; // Base size of the player on a 1080p screen
const BASE_SCREEN_PIXEL_COUNT: f32 = 1920.0 * 1080.0;
const PLAYER_BOTTOM_MARGIN: f32 = 10.0; // Percentage of the screen height

impl Entity for Player {
    fn new() -> Self {
        let screen_pixel_count = screen_width() * screen_height();
        let scaling_factor = screen_pixel_count / BASE_SCREEN_PIXEL_COUNT;

        let player_size = PLAYER_BASE_SIZE * scaling_factor;

        let size = RectEntitySize {
            w: player_size,
            h: player_size,
        };

        let pos = Position {
            x: screen_width() / 2.0,
            y: screen_height() * (100.0 - PLAYER_BOTTOM_MARGIN) / 100.0 - size.h,
        };

        Player {
            pos: pos,
            speed: 5,
            entity_type: EntityType::Player,
            size: size,
        }
    }
    ...
}
```

Let's hit `cargo run` and see what it does and voila! My player square is smaller than intended, player's position also leans to the left and top side of the screen! But why, the code should work! Let's try turning off the fullscreen, shall we? Ta-da, it works as I intended! Macroquad spawns an 800x600 window and the player is on where it should be. Wait, 800x600? When I switch back to fullscreen, on the command line,  I see this output: `Resolution: 800x600`!

I don't get it. My laptop's monitor's resolution is 1920x1080 so when I set `fullscreen: true`, `log_info()` should report my resolution as `1920x1080`. Then 
I noticed something. When I start my game in fullscreen, game window is spawning as a small window on the left bottom corner on my screen as a small window, then stretches to fullscreen! I think `log_info()` function is reporting my screen resolution before the stretching happens and `Player` also initializes before it! I can add a delay in `main()` with `std::thread::sleep(...)` but this is probably not a good idea because first, I don't know when the screen will be stretched, and second, I don't want my game to start with a delay so I need to find another solution.

# Solutions

Firstly, I need you to know I encountered this bug only in my laptop that uses Ubuntu 22.04.5 with default Gnome 42.9 DE. I can't reproduce it on my Windows
machine. Sadly, I don't own any Macs so this one is not checked. Ah, and I have Ryzen 5 3500u's integrated Radeon Vega 8 with default drivers.

When I was searching macroquad's docs for a way to solve this problem I encountered [Conf](https://docs.rs/macroquad/latest/macroquad/prelude/struct.Conf.html), the window configuration struct.

```rust
pub struct Conf {
    pub window_title: String,
    pub window_width: i32,  // Default: 800
    pub window_height: i32, // Default: 600
    ...

    # You see the width and heigh defaults? 
}
```

So before creating a `Conf`, somehow I should determine my monitor's resolution and set the width and height. Easy right? Let's google how to do it. :)

## Solution 1: Use miniquad (headaches)

I was looking into macroquad's source code to understand how macroquad handles windowing and fullscreen and I stumbled upon something. Macroquad uses a library called `miniquad` by the same author and windowing system is tied to miniquad! Also, macroquad exposes miniquad so i tried to look into `miniquad::window` and saw `miniquad::window::screen_size()`! But that doesn't work either :) Let's look into that function's code then.

```rust
pub fn screen_size() -> (f32, f32) {
        let d = native_display().lock().unwrap();
        (d.screen_width as f32, d.screen_height as f32)
    }
```

Hmm, native display huh? What is it?

```rust
static NATIVE_DISPLAY: OnceLock<Mutex<native::NativeDisplayData>> = OnceLock::new();

fn set_display(display: native::NativeDisplayData) {
    NATIVE_DISPLAY
        .set(Mutex::new(display))
        .unwrap_or_else(|_| panic!("NATIVE_DISPLAY already set"));
}
fn native_display() -> &'static Mutex<native::NativeDisplayData> {
    NATIVE_DISPLAY
        .get()
        .expect("Backend has not initialized NATIVE_DISPLAY yet.") //|| Mutex::new(Default::default()))
}
```

Ah, nice! Now I need to find the usages of `set_display(...)` :) Lucky me, for Linux, it is seperated in 2 files. `src/native/linux_x11.rs` and 
`src/native/linux_wayland.rs`. Hmm... I don't know anything about Linux windowing system internals so maybe it is a X11 thing since I use X11? Ok, I don't understand anything. Let's try a simple example with just miniquad.

```rust
use miniquad::*;

struct Stage {
    ctx: GlContext,
}
impl EventHandler for Stage {
    fn update(&mut self) {
        println!("Screen size: {:?}", window::screen_size());
    }

    fn draw(&mut self) {
        self.ctx.clear(Some((0., 1., 0., 1.)), None, None);
    }
}

fn main() {
    miniquad::start(
        conf::Conf {
            window_title: "Miniquad".to_string(),
            fullscreen: true,
            ..Default::default()
        },
        || {
            Box::new(Stage {
                ctx: GlContext::new(),
            })
        },
    );
}
```

`println!` prints 800x600 for the first frame and 1920x1080 for the rest of the frames...

Oh no, please, it's not working and it keeps getting harder and harder! At this point, I couldn't comprehend anything more about this, I don't have time, I don't have patience. I abandoned the idea of using miniquad. Extra dependencies, hurray!

## Solution 2: Use [winit](https://github.com/rust-windowing/winit)

For a different toy project of mine, I needed to use winit so I knew I could get a proper monitor resolution with it. I added winit to Linux dependencies.

```toml
[target.'cfg(target_os = "linux")'.dependencies]
winit = "0.30.5"
```

Then I created another `window_conf()` that compiles only in Linux and marked the original one with `#[cfg(not(target_os = "linux"))]`.

**Note:** Creating a window without a running event loop is deprecated in winit but since I don't want to create a window, it should be fine.

```rust
#[cfg(target_os = "linux")]
#[allow(deprecated)]
fn window_conf() -> Conf {
    let el = EventLoop::new().unwrap();
    let window = el.create_window(Window::default_attributes()).unwrap();
    let screen = window.current_monitor().unwrap();
    let size = screen.size();

    Conf {
        window_title: "Shootin'".into(),
        fullscreen: true,
        window_width: size.width as i32,
        window_height: size.height as i32,
        ..Default::default()
    }
}
```

Aaaaand, it works, finally! But this method is introducing a huge dependency to Linux builds (resulting --release binary is 1.1 MB, so actually not a huge deal).

# Conclusion
I mean, I'm not a professional developer, just a hobbyist. And I have a tendency to overcomplicate things. Debug build times with winit are not so bad, release binary size is good. Don't be a dumbo like me. Also maybe there was a solution if I understood how miniquad creates windows under X11. Is this a bug with miniquad? I don't know. Is this a bug with X11? Probably not. :) Have a nice day.
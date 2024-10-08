---
title: Rust Development in Termux
slug: 08-rust-development-in-termux
date: 2024-10-08T19:35:38+02:00
categories:
  - Software development
tags:
  - rust
  - termux
  - tmux
  - helix
  - android
  - curl
images:
  - /images/android-termux-keyboard.jpg
draft: false
---
In my holiday I used to Termux to do server maintenance in emergency cases and it worked pretty well. I thought about learning Rust and how this can be combined. Can I use my phone as a desktop development environment? In this post I am going to answer this question.

<!--more-->
### Desktop mode

I've got a Fairphone 3 with /e/OS/. The phone is outdated and does not support desktop mode: <https://en.everybodywiki.com/List_of_devices_with_video_output_over_USB-C>

However, I have a Bluetooth keyboard from Keychron <https://www.keychron.com/>. By switching to Bluetooth mode and holding <kbd>fn</kbd>+<kbd>1</kbd> I was able to connect the keyboard with my phone.

When working with the phone like this, the auto lock and sleep mode of the phone can be annoying. I recommend to disable it.
### Setup

Open the Termux app on your Android phone. You should find yourself in a bash shell.

We are going to install several tools, lets start with git.

```bash
pkg install git
```

Then we need a terminal multiplexer. I recommend tmux.

```bash
pkg install tmux
```

You can start tmux right away with `tmux`.

As we are going to implement rust, I think it would be nice to use a Rust based editor.

```bash
pkg install helix
```

Next up are the Rust build tools.

```bash
pkg install rust
```

Press <kbd>y</kbd> and wait for some time.

Check the rust compiler version with `rustc -version`.

### Simple program

We are ready to code our first Rust program.

Start helix with `hx`. Then enter insert mode with <kbd>i</kbd>.

Write the following program:

```rust
fn main() {
	println!("Hello Mom!")
}
```

Switch to normal mode <kbd>esc</kbd>. Then save the file with <kbd>:</kbd> and <kbd>w hello.rs</kbd>

Then split the window with tmux <kbd>ctrl</kbd>+<kbd>b</kbd> and <kbd>%</kbd>. In the new window run the rust program.

```bash
restc hello.rs
```

This will produce a binary `hello` that can be run with `./hello`. You should see the output of the print statement.

To switch the tmux pane use <kbd>ctrl</kbd>+<kbd>b</kbd> and an arrow key.

### Webserver

That was easy. Let's implement something more interesting - a web server.

Bootstrap the project with cargo:

```bash
cargo new http-server
cd http-server
```

Close helix with <kbd>:q!</kbd> and open the project manifest with `hx Cargo.yml`.

Add the following block to the dependency section:

```toml
tide = "0.16.0"
async-std = { version = "1.8.0", features = ["attributes"] }
serde = { version = "1.0", features = ["derive"] }
```

Use <kbd>:w</kbd> to save the file.

Next edit the Rust program in helix with `:o src/main.rs`.

Delete everything with <kbd>d</kbd> and write this program.

```rust
#[async_std::main]
async fn main() -> Result<(), std::io::Error> {
    let mut app = tide::new();
    app.with(tide::log::LogMiddleware::new());

    app.at("/").get(|_| async { Ok("Hello Mom!") });
    app.listen("127.0.0.1:8080").await?;
    Ok(())
}
```

You can also copy the code above by opening this post in the browser of your phone. You can switch between Android apps with <kbd>home</kbd>+<kbd>tab</kbd>. To paste the code you most likely need to use the touch screen menu.

Now it is time to start the web server.

```bash
cargo run
```

Cargo will pull and build the required dependencies and then run the server.

Open a new tmux tab with <kbd>ctrl</kbd>+<kbd>b</kbd> and <kbd>c</kbd>.  Then run this curl command:

```bash
curl http://localhost:8080
```

If you get "Hello Mom!" you did a great job!

### Workflow improvements

Of course the current setup needs a lot of improvements. Here are some ideas that you can further discover.

* Auto save file in helix
* Automatically save tmux session state
* File navigation in Helix
* Expose web server with ssh tunnel
* Version control the code with git

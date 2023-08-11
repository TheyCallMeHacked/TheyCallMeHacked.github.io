+++
title="Nvim loves Rust: How I built a toy plug-in for neovim in Rust"
date=2023-07-24

[taxonomies]
categories = ["Programming"]
tags = ["rust", "neovim", "FFI"]
+++

You most probably know that the neovim text editor can be configured and
extended using the Lua programming language. You may also know, if you've read
through the documentation, that you can also use other programming languages
using an RPC server. But what you may not have known, is that you can also use
any language that compiles to a dynamic library using the C ABI of your
operating system.

<!-- more -->

# The Idea

If you've spent any amount of time playing around in neovim, you've definitely
come across Lua as a programming language. You may have heard that Lua is a
language designed to allow more or less easy interoperability with C. This
allows us to write with any language that can be compiled to a `.so` or `.dll`.
My choice of language was Rust, as there are already some handy tools that work
for this.

In order to have it work, we just need to compile a Rust library to a C dynamic
library, put it in the `lua` folder of our plug-in, and we should be good to go!

# The Project

A wise man once told me, "If it's useless, it's absolutely necessary".
Following this precept, we will build the most necessary neovim plug-in out
there: a morse interpreter. This is a good reason to use Rust, as there is no
good cross-platform audio library for lua, that doesn't require LuaRocks.

## Cargo setup

In order to be able to communicate with neovim, Rust will need to do two
things:
1. Understand Lua
2. Learn the neovim API

We could definitely use a crate like [`mlua`](https://lib.rs/crates/mlua) and
write our own wrappers for that, but it would be tedious. Luckily, somebody
else already did all the heavy lifting and wrote
[`nvim-oxi`](https://lib.rs/crates/nvim-oxi), which we will use here. We will also
use [`rodio`](https://lib.rs/crates/rodio) for the audio processing, to play our files
as morse beeps. Our `Cargo.toml` thus looks like this:

```toml
[package]
name = "morse"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
nvim-oxi = { version = "0.3.0", features = ["neovim-0-9"] }
rodio = "0.17.1"
```

## Connecting to neovim through Rust

Using `nvim-oxi`, our lib entry point will need some special care. First, obviously,
it must be called the same as our plug-in. But it also needs to be flagged to the
compiler that it's a neovim module. Following the classical
the-plug-in-is-hidden-behind-a-setup-function approach, our entry point could look like this:

```rust
use nvim_oxi::{self as oxi, api::Buffer, Object, Dictionary, Function};
```
```rust
#[oxi::module]
fn morse() -> oxi::Result<Dictionary> {
    Ok(Dictionary::from_iter([
        ("setup", Function::from_fn(setup)),
    ]))
}
```

The `Function` struct allows for easy conversion between Rust functions and functions
that Lua can see.

We will need a struct to hold the configuration info, which our `setup` function shall
set. For now this struct will only hold the frequency of our beeps:

```rust
#[derive(Clone, Copy)]
struct Config {
    freq: f32,
}

fn setup(freq: f32) -> oxi::Result<Dictionary>  {
    let conf = Config{
        freq,
    };

    Ok(Dictionary::from_iter([
        ("beep", Object::from(Function::from_fn(move |t| {beep(t,conf)}))),
        ("convert", Object::from(Function::from_fn(move |b| {convert(b,conf)}))),
    ]))
}
```

We move a copy of this config into closures owned by the `Function` objects returned
by our config. This may not be the most efficient way to do things, but it's the
memory-safest way I could think of, without needing runtime initialised static variables.

## Playing Audio
As said above, we will use the `rodio` crate to hear our dits and dahs. To make a simple
sine beep:

```rust
use rodio::{
    source::{SineWave, Source},
    OutputStream,
    Sink
};
use std::{
    time::Duration,
    thread::sleep,
    convert::Infallible
};
```
```rust
fn beep(time: f32, conf: Config) -> Result<(),Infallible> {
    let (_stream, stream_handle) = OutputStream::try_default().unwrap();
    let sink = Sink::try_new(&stream_handle).unwrap();
    let sine = SineWave::new(conf.freq).take_duration(Duration::from_secs_f32(time));
    sink.append(sine);
    sink.sleep_until_end();
    Ok(())
}
```

Note `nvim_oxi::Function` expects a `Result` return type, which we set to `Infallible`.
We generate an output stream to which we append a sine wave signal as long as the duration
we give the function. Because `rodio`'s playback is asynchronous, we have to sleep until
the audio thread signals us it's done playing.

## Bringing it all together
Finally, we convert a buffer into dits and dahs. The function is all but efficient, and
has been written this way to increase code readability:

```rust
fn convert(buf: Buffer, conf: Config) -> Result<(),Infallible> {
    let mut text: String = buf.get_lines(.., false).unwrap().fold(String::new(), |a,s| {a + &s.to_string_lossy() + "\n"});
    text.pop();
    let text = text.chars().map(|c| { match c {
        'a' | 'A' => ".- ",
        'b' | 'B' => "-... ",
        'c' | 'C' => "-.-. ",
        'd' | 'D' => "-.. ",
        'e' | 'E' => ". ",
        'f' | 'F' => "..-. ",
        'g' | 'G' => "--. ",
        'h' | 'H' => ".... ",
        'i' | 'I' => ".. ",
        'j' | 'J' => ".--- ",
        'k' | 'K' => "-.- ",
        'l' | 'L' => ".-.. ",
        'm' | 'M' => "-- ",
        'n' | 'N' => "-. ",
        'o' | 'O' => "--- ",
        'p' | 'P' => ".--. ",
        'q' | 'Q' => "--.- ",
        'r' | 'R' => ".-. ",
        's' | 'S' => "... ",
        't' | 'T' => "- ",
        'u' | 'U' => "..- ",
        'v' | 'V' => "...- ",
        'w' | 'W' => ".-- ",
        'x' | 'X' => "-..- ",
        'y' | 'Y' => "-.-- ",
        'z' | 'Z' => "--.. ",
        ' '       => "/ ",
        '\n'      => "-...- ",
        _         => ""
    }}).fold(String::new(), |a,s| {a + s });
    for s in text.chars() {
        let unit = 0.07;
        match s {
            '.' => {beep(unit, conf).unwrap(); sleep(Duration::from_secs_f32(unit));},
            '-' => {beep(unit*3.0, conf).unwrap(); sleep(Duration::from_secs_f32(unit));},
            '/' => {sleep(Duration::from_secs_f32(unit*2.0));},
            ' ' => {sleep(Duration::from_secs_f32(unit));},
            _   => {},
        }
    }
    Ok(())
}
```

We first read the lines of the buffer and concatenate them. We then convert it to a 
string of dots and dashes and convert those dots and dashes into beeps of the appropriate
duration. For now, the unit duration is hard-coded, but I intend to set it as a setup option.

And that's all for now. To review the entire code, and try it out for yourself, go see
[the project's github page](https://github.com/TheyCallMeHacked/morse.nvim)

# Conclusion

As you can see, Using neovim in conjunction with Rust as a lua replacement is really
not that hard. There are a few abstractions lua forces us into, but it's only a matter of following
neovim's API through `nvim-oxi` function calls.

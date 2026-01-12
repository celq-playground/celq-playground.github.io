# celq-playground.github.io

Tis is the playground for [celq](https://github.com/IvanIsCoding/celq). See it live at https://celq-playground.github.io/!

# How it works

`celq` is compiled with:

```bash
cargo build --no-default-features   --release --target wasm32-wasip1
```

Behind the scenes, [browser_wasi_shim](https://github.com/bjorn3/browser_wasi_shim) provides the glue that makes the Rust binary work in the browser

# How this was built

`celq` was built-in Rust, with a good test suite and best practices.

This website was vibe-coded. I will not pretend that I understand everything under the hood for JS. For the first attempt, it tried to use `wasmer` and it didn't work very well. For the second attempt, I instructed it to use `browser_wasi_shim` and the result was much better. Here's the original prompt:

```
Explain to me how to  create a web playground for my Rust CLI. I do NOT want to refactor my library. Either compile it to WASI (proposal 1) or Emscripten, but not functional change to the code.

I have the repo github.com/IvanIsCoding/celq. It's also published to crates.io as celq. I want to compile to WASM and have a website with the following:
1. Cmd input. A box with celq [OPTIONS] args. Add syntax highlighting for basg
2. Stdin box. Add syntax highlighting for JSON/YAML/TOML. What gets piped to stdin for celq
3. expr.cel. To be used with --from-file, it's in the file system. It's fine if this doesn't have a highlight
4. Output box. No highlight 
5. Run button to trigger. Don't auto-trigger

Make this a static website that can be uploaded to GitHub pages. JS + WASM. no server side at all
```

AI took the 'static' part very seriously and loaded everything from a CDN. What you see at the root is the `index-alt.html` from the second attempt. Even after the second attempt, I had to manually fix the loading of `expr.cel`. AI set up the virtual file system incorrectly.

After everything was done, I changed the theme to a blue-ish theme with [Nord syntax highlighting](https://www.nordtheme.com/). I also added the acknowledgments at the end of the page and the GitHub link. 
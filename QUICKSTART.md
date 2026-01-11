# celq Web Playground - Quick Start

## What You Got

A complete static website that runs celq (your Rust CLI) in the browser using WebAssembly!

### Files Included

```
celq-playground/
├── index.html          # Main playground (Wasmer WASI)
├── index-alt.html      # Alternative (Browser WASI Shim) 
├── style.css           # Brutalist terminal theme
├── build.sh            # Automated build script
├── README.md           # User documentation
├── DEPLOYMENT.md       # GitHub Pages guide
├── TECHNICAL.md        # Technical details
├── _config.yml         # GitHub Pages config
└── .gitignore          # Git ignore rules
```

## Getting Started (3 Steps)

### 1. Build the WASM Binary

```bash
cd celq-playground
chmod +x build.sh
./build.sh
```

This will:
- Clone celq from GitHub
- Install the WASI target
- Compile celq to WASM
- Copy `celq.wasm` to the playground directory

### 2. Test Locally

```bash
python3 -m http.server 8000
```

Open http://localhost:8000 in your browser!

### 3. Deploy to GitHub Pages

```bash
git init
git add index.html style.css celq.wasm README.md
git commit -m "Initial commit"
git remote add origin https://github.com/YOUR_USERNAME/celq-playground.git
git push -u origin main
```

Enable GitHub Pages in repository settings → Pages → Source: main branch

## Features

### ✨ What Works

- **Command Input**: Full celq syntax with bash highlighting
- **Stdin Input**: JSON/YAML/TOML with syntax highlighting  
- **File System**: Virtual `expr.cel` file for `--from-file`
- **Output Display**: Results with JSON highlighting
- **Zero Server**: Everything runs client-side
- **Static Hosting**: Works on GitHub Pages, Netlify, etc.

### 🎯 Example Usage

Try these in the playground:

1. **Filter array**:
   - Command: `this.filter(s, s.contains("a"))`
   - Stdin: `["apples", "bananas", "blueberry"]`

2. **Math with args**:
   - Command: `-n --arg='x:int=5' 'x * 2'`
   - Stdin: (empty)

3. **From file**:
   - Command: `--from-file expr.cel`
   - Stdin: `[1, 2, 3]`
   - expr.cel: `this.map(n, n * 2)`

## Architecture

```
Browser → JavaScript → WASI Runtime → celq.wasm
   ↑                                      ↓
   └──────────── Results ────────────────┘
```

- **No refactoring**: Your Rust code is unchanged
- **WASI compilation**: Standard `wasm32-wasip1` target
- **Virtual FS**: In-memory filesystem for files
- **I/O capture**: Stdin/stdout/stderr fully supported

## Customization

### Change Theme

Edit `style.css`:
```css
:root {
    --accent: #00ff88;  /* Change to your color */
    --bg-primary: #0a0e14;
}
```

### Add Examples

Modify `index.html` footer:
```html
<p class="examples">
    <strong>Examples:</strong>
    Try: <code>your.example()</code>
</p>
```

### Switch WASI Implementation

Two versions provided:
- `index.html`: Wasmer WASI (more features)
- `index-alt.html`: Browser WASI Shim (lighter)

Just rename to use the other!

## Two Implementation Options

### Option 1: Wasmer WASI (Recommended)
- File: `index.html`
- Runtime: `@wasmer/wasi`
- Better error handling
- More complete WASI support

### Option 2: Browser WASI Shim
- File: `index-alt.html`  
- Runtime: `@bjorn3/browser_wasi_shim`
- Lighter weight
- Simpler code

Both work! Pick based on your needs.

## Troubleshooting

### Build fails: "wasm32-wasip1 not installed"
```bash
rustup target add wasm32-wasip1
```

### WASM file too large for GitHub
Use wasm-opt to optimize:
```bash
cargo install wasm-opt
wasm-opt -Oz celq.wasm -o celq.wasm
```

Or use Git LFS:
```bash
git lfs install
git lfs track "*.wasm"
```

### Page loads but Run button doesn't work
- Check browser console for errors
- Ensure `celq.wasm` is in the same directory
- Try the alternative implementation (`index-alt.html`)

### Output shows "Error: Failed to fetch WASM"
- Verify `celq.wasm` exists
- Check file permissions
- Test with a simple HTTP server first

## Next Steps

1. ✅ Build and test locally
2. ✅ Deploy to GitHub Pages
3. ✅ Share your playground!
4. 🚀 Optional: Add custom domain
5. 🎨 Optional: Customize the theme
6. 📝 Optional: Add more examples

## Documentation

- **README.md**: User-facing documentation
- **DEPLOYMENT.md**: Detailed GitHub Pages setup
- **TECHNICAL.md**: Architecture and implementation details

## Resources

- Your celq repo: https://github.com/IvanIsCoding/celq
- WASI spec: https://github.com/WebAssembly/WASI
- CodeMirror: https://codemirror.net/5/

## Support

If you encounter issues:

1. Check the TECHNICAL.md for detailed explanations
2. Look at browser console for error messages
3. Try the alternative HTML implementation
4. Test with the example commands provided

Enjoy your celq web playground! 🎉
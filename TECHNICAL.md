# Technical Documentation: celq Web Playground

## Architecture Overview

The celq web playground is a **static, client-side application** that runs celq entirely in the browser using WebAssembly (WASM) and WASI (WebAssembly System Interface).

### Technology Stack

- **Frontend**: Vanilla JavaScript (ES6 modules)
- **Code Editor**: CodeMirror 5
- **WASM Runtime**: Browser WASI Shim (@bjorn3/browser_wasi_shim) or Wasmer WASI
- **Compilation Target**: `wasm32-wasip1` (WASI Preview 1)

### Architecture Diagram

```
┌─────────────────────────────────────────┐
│         Browser (Client-Side)           │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │  HTML/CSS UI (CodeMirror)         │ │
│  └───────────┬───────────────────────┘ │
│              │                          │
│              ▼                          │
│  ┌───────────────────────────────────┐ │
│  │  JavaScript WASI Runtime          │ │
│  │  - Parse command arguments        │ │
│  │  - Setup virtual filesystem       │ │
│  │  - Capture stdin/stdout/stderr    │ │
│  └───────────┬───────────────────────┘ │
│              │                          │
│              ▼                          │
│  ┌───────────────────────────────────┐ │
│  │  celq.wasm                        │ │
│  │  - Compiled Rust code             │ │
│  │  - CEL evaluation engine          │ │
│  │  - Full CLI functionality         │ │
│  └───────────────────────────────────┘ │
│                                         │
└─────────────────────────────────────────┘
```

## Compilation Process

### WASI vs Emscripten

We use **WASI** instead of Emscripten for several reasons:

1. **No code changes required**: WASI is a standard interface that Rust supports natively
2. **Smaller binary size**: No large JavaScript glue code
3. **Better portability**: WASI is a standard that works across runtimes
4. **Direct system calls**: WASI provides direct access to filesystem, stdin/stdout

### Build Command

```bash
cargo build --release --target wasm32-wasip1
```

This produces a standalone WASM binary at:
```
target/wasm32-wasip1/release/celq.wasm
```

### WASI Target

The `wasm32-wasip1` target is specifically designed for:
- Command-line tools
- Applications that need filesystem access
- Programs that use stdin/stdout/stderr
- Standard Rust code without browser-specific APIs

## Runtime Implementation

### Two WASI Implementations Provided

#### 1. Wasmer WASI (@wasmer/wasi) - `index.html`

**Pros:**
- More feature-complete
- Better error handling
- Active development

**Cons:**
- Larger runtime
- More complex API

#### 2. Browser WASI Shim (@bjorn3/browser_wasi_shim) - `index-alt.html`

**Pros:**
- Lightweight
- Simpler implementation
- Good for basic use cases

**Cons:**
- Less feature-complete
- Some edge cases not handled

### Virtual Filesystem

The playground creates a virtual filesystem in memory:

```javascript
const rootDir = new PreopenDirectory("/", new Map([
    ["expr.cel", new File(new TextEncoder().encode(exprContent))]
]));
```

This allows celq to read `expr.cel` as if it were a real file when using `--from-file expr.cel`.

### I/O Handling

**Stdin**: Piped from the "Stdin" editor
```javascript
const stdinFile = new File(new TextEncoder().encode(stdinContent));
```

**Stdout/Stderr**: Captured using custom write implementations
```javascript
stdoutFile.write = (buf) => {
    stdoutContent.push(buf);
    return buf.byteLength;
};
```

### Command Parsing

The playground parses the command input box to extract arguments:

```javascript
function parseCommandLine(cmd) {
    // Handles:
    // - Space-separated arguments
    // - Single and double quotes
    // - Escaped characters
}
```

This converts `this.filter(s, s.contains("a"))` into `["celq", "this.filter(s, s.contains(\"a\"))"]`.

## UI Implementation

### CodeMirror Integration

Each editor panel is a CodeMirror instance with:

1. **Command Input**: Shell mode (bash highlighting)
2. **Stdin**: JavaScript mode (JSON highlighting)
3. **expr.cel**: Plain text mode
4. **Output**: JavaScript mode, read-only

### Responsive Grid Layout

```css
.grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1.5rem;
}
```

On mobile (<1200px), switches to single column.

### Theme

Custom brutalist/terminal aesthetic:
- JetBrains Mono font
- Dark background with cyan accents (#00ff88)
- Grid pattern background
- CodeMirror Monokai theme

## Performance Considerations

### Initial Load

1. **HTML/CSS**: ~15KB (minified)
2. **JavaScript**: ~5KB (application code)
3. **CodeMirror**: ~200KB (from CDN)
4. **WASI Runtime**: ~50-100KB (from CDN)
5. **celq.wasm**: 2-5MB (depends on build)

**Total**: ~3-5MB first load

### Runtime Performance

- **WASM compilation**: 100-500ms (one-time on page load)
- **Execution**: Near-native speed (typically <100ms for simple queries)
- **Memory**: Allocated as needed, typically <50MB

### Optimization Tips

1. **Optimize WASM**:
   ```bash
   wasm-opt -Oz celq.wasm -o celq.wasm
   ```

2. **Enable compression** (automatic on GitHub Pages):
   - Gzip: ~40% reduction
   - Brotli: ~50% reduction

3. **Use CDN caching**: All external dependencies cached by browser

## Browser Compatibility

### Supported Browsers

- **Chrome/Edge**: 90+
- **Firefox**: 89+
- **Safari**: 15+

### Required Features

- WebAssembly
- ES6 Modules
- TextEncoder/TextDecoder
- Async/Await
- Fetch API

### Fallback

For unsupported browsers, show an error message:

```javascript
if (!WebAssembly) {
    alert('Your browser does not support WebAssembly');
}
```

## Limitations

### Memory Constraints

- Browser-imposed WASM memory limits (usually 2-4GB)
- Large inputs (>100MB) may cause issues
- No streaming processing (all data loaded into memory)

### Missing WASI Features

Some WASI features not implemented in browser:
- Real filesystem access
- Network sockets
- Process spawning
- Environment variables (limited)

### Performance

Slightly slower than native due to:
- JavaScript ↔ WASM boundary crossing
- Virtual filesystem overhead
- Browser security restrictions

## Security

### Sandboxing

- WASM runs in a sandboxed environment
- No access to user filesystem
- No network access
- All data stays in browser

### XSS Protection

- No `eval()` or `innerHTML`
- All user input properly escaped
- CodeMirror handles sanitization

## Troubleshooting

### WASM fails to load

**Cause**: CORS issues or missing file

**Solution**: 
```javascript
// Check network tab in browser DevTools
// Ensure celq.wasm is served with correct MIME type
```

### Out of memory errors

**Cause**: Large JSON input or complex expressions

**Solution**: Limit input size or simplify query

### Execution hangs

**Cause**: Infinite loop or very long computation

**Solution**: Implement timeout (requires WASI thread support)

## Future Enhancements

### Possible Improvements

1. **Worker Threads**: Run WASM in Web Worker to prevent UI blocking
2. **Streaming**: Process large inputs incrementally
3. **Offline Support**: Service Worker for PWA functionality
4. **Share Links**: Encode state in URL for sharing
5. **Examples Gallery**: Pre-loaded example queries
6. **Syntax Validation**: Real-time CEL syntax checking
7. **Performance Profiling**: Show execution time and memory usage

### Advanced Features

- **Multiple Files**: Support multiple .cel files
- **Debugging**: Step through CEL execution
- **Visualization**: Graph output for complex data
- **Export**: Save results as files

## Contributing

### Development Setup

1. Clone repository
2. Make changes to HTML/CSS/JS
3. Test locally: `python3 -m http.server 8000`
4. Build WASM: `./build.sh`
5. Test with real WASM binary

### Code Style

- Use ES6 modules
- Async/await for promises
- Descriptive variable names
- Comments for complex logic

### Testing

Manual testing checklist:
- [ ] Basic expressions work
- [ ] Stdin input parsed correctly
- [ ] File input (expr.cel) accessible
- [ ] Error messages displayed
- [ ] Responsive on mobile
- [ ] Works in all supported browsers

## References

- [WASI Specification](https://github.com/WebAssembly/WASI)
- [Rust WASM Book](https://rustwasm.github.io/docs/book/)
- [CodeMirror Documentation](https://codemirror.net/5/doc/manual.html)
- [Browser WASI Shim](https://github.com/bjorn3/browser_wasi_shim)
- [Wasmer WASI](https://github.com/wasmerio/wasmer-js)
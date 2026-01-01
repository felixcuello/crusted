# crusted

## Introduction

I want to learn rust by building a simple editor called "crusted". This project will help me understand the basics of
Rust programming while creating a functional text editor.

## Features

- edit files (and even large files)
- syntax highlight
- undo / redo
- search and replace (using regex)
- have multiple buffers (to open multiple files at the same time)
- have the ability to do vertical and horizontal splits
- Support for LSP (Language Server Protocol) to provide features like auto-completion, go to definition, etc.
- Support to send commands to the console (not open a terminal, just execute things and show the output in a window)

## Plan

The following is a plan about the things I need to learn in order to build this project. Also I will have to learn a lot
about the Rust programming language and ecosystem along the way.

The plan must include not just the technical aspects of building the editor, but also:

- What are the things I have to learn about data structures?
- What are the things I have to learn about algorithms?
- What are the things I have to learn about console/CLI frameworks?
- The plan should include diagrams (like component diagrams, flowcharts, etc.) to help visualize the architecture of the
  editor.
- What are the things I have to learn about rust (like ownership, borrowing, lifetimes, etc.)?
- Build the plan in a way that I can follow it step by step, starting from the basics and gradually
  moving to more advanced topics.
- The plan should also include milestones to track progress.
- The plan should include steps to quickly build an MVP (Minimum Viable Product) and then iterate on it to add more
  features.

---

## Learning and Development Plan

### Phase 0: Rust Fundamentals (Week 1-2)

#### Core Rust Concepts to Learn
1. **Ownership & Borrowing**
   - Stack vs Heap allocation
   - Move semantics
   - References (`&` and `&mut`)
   - Lifetimes (`'a`, `'static`)
   
2. **Essential Language Features**
   - `Result<T, E>` and `Option<T>` for error handling
   - Pattern matching with `match` and `if let`
   - Traits and trait bounds
   - Closures and iterators
   - `Box<T>`, `Rc<T>`, `RefCell<T>` for dynamic memory management

3. **Project Setup**
   - Cargo basics (build, run, test, clippy, fmt)
   - Project structure and modules
   - Dependency management in `Cargo.toml`

#### Resources
- The Rust Book (chapters 1-16)
- Rustlings exercises for hands-on practice

---

### Phase 1: MVP - Basic Text Display (Week 3)

**Goal:** Display file contents in the terminal and move cursor

#### Data Structures to Learn
- **Rope data structure** (or Gap Buffer as simpler alternative)
  - Why: Efficient insertion/deletion for large files
  - Consider using `ropey` crate (production-ready rope implementation)
- **Vec<String>** (simple line-based storage for MVP)
  
#### Technical Components
1. **Terminal Handling**
   - Learn terminal raw mode vs cooked mode
   - ANSI escape codes for cursor movement and clearing screen
   - Use `crossterm` crate for cross-platform terminal manipulation
   
2. **File I/O**
   - `std::fs::read_to_string()` for reading files
   - UTF-8 handling and validation
   
3. **Input Handling**
   - Read keyboard events (arrows, characters, special keys)
   - Non-blocking input with `crossterm::event`

#### Architecture Diagram (MVP)

```
┌─────────────────────────────────────────┐
│           Main Event Loop               │
│  (handle input → update state → render) │
└────────────┬────────────────────────────┘
             │
    ┌────────┴────────┐
    ▼                 ▼
┌────────┐      ┌──────────┐
│ Editor │      │ Terminal │
│ State  │◄────►│  Render  │
└────┬───┘      └──────────┘
     │
     ▼
┌──────────┐
│  Buffer  │ (Vec<String>)
└──────────┘
```

#### Milestone 1 Checklist
- [ ] Display file contents
- [ ] Move cursor with arrow keys
- [ ] Scroll viewport when cursor moves off screen
- [ ] Status bar showing filename and cursor position
- [ ] Exit with Ctrl+Q

---

### Phase 2: Basic Editing (Week 4)

**Goal:** Insert and delete text

#### Data Structures Deep Dive
1. **Gap Buffer** (recommended for MVP)
   ```
   [text before cursor][gap...][text after cursor]
   ```
   - O(1) insertion at cursor
   - Simple to implement
   
2. **Rope (for scaling to large files)**
   - Tree structure of string chunks
   - O(log n) operations
   - Use `ropey` crate

#### Algorithms to Learn
- **Line wrapping algorithms**
- **Cursor position mapping** (byte offset ↔ line:column)
- **UTF-8 character boundaries** (avoid splitting multi-byte chars)

#### Technical Components
1. **Text Insertion**
   - Handle character input
   - Newline insertion (Enter key)
   - Handle tabs vs spaces
   
2. **Text Deletion**
   - Backspace and Delete keys
   - Deleting at line boundaries (joining lines)

#### Milestone 2 Checklist
- [ ] Insert characters at cursor
- [ ] Insert newlines
- [ ] Delete characters (backspace/delete)
- [ ] Handle UTF-8 correctly (emoji, multi-byte chars)
- [ ] Dirty flag to track unsaved changes

---

### Phase 3: File Operations & Undo/Redo (Week 5-6)

#### Data Structures to Learn
1. **Command Pattern** for undo/redo
   ```rust
   trait Command {
       fn execute(&mut self, buffer: &mut Buffer);
       fn undo(&mut self, buffer: &mut Buffer);
   }
   ```
   
2. **Two Stacks Approach**
   - Undo stack: past commands
   - Redo stack: undone commands
   
3. **Alternatives to Consider**
   - Persistent data structures (immutable with structural sharing)
   - Event sourcing (replay history)

#### Technical Components
1. **Saving Files**
   - Write buffer to file
   - Handle I/O errors
   - Create backup before overwriting
   - Save-as functionality
   
2. **Undo/Redo System**
   - Wrap edit operations in commands
   - Maintain undo history
   - Clear redo stack on new edit
   - Memory limits for undo history

#### Architecture Update

```
┌───────────────────────────────────────────────┐
│              Main Event Loop                  │
└───────────┬───────────────────────────────────┘
            │
    ┌───────┴────────┐
    ▼                ▼
┌─────────┐    ┌──────────┐
│ Editor  │    │ Terminal │
│         │    │  Render  │
│ ┌──────┐│    └──────────┘
│ │Buffer││
│ └──────┘│
│ ┌──────┐│
│ │Undo  ││
│ │Stack ││
│ └──────┘│
│ ┌──────┐│
│ │Redo  ││
│ │Stack ││
│ └──────┘│
└─────────┘
```

#### Milestone 3 Checklist
- [ ] Save file (Ctrl+S)
- [ ] Save-as with filename prompt
- [ ] Undo (Ctrl+Z)
- [ ] Redo (Ctrl+Y or Ctrl+Shift+Z)
- [ ] Warn on exit with unsaved changes

---

### Phase 4: Search and Replace (Week 7)

#### Algorithms to Learn
1. **String Search Algorithms**
   - Naive search (simple MVP)
   - Boyer-Moore (faster for longer patterns)
   - Use `regex` crate for regex support
   
2. **Search UI Patterns**
   - Incremental search (search-as-you-type)
   - Highlight all matches
   - Navigate between matches

#### Technical Components
1. **Search**
   - Case-sensitive/insensitive toggle
   - Regex pattern support
   - Search forward/backward
   - Wrap around at file boundaries
   
2. **Replace**
   - Replace current match
   - Replace all
   - Interactive replace (confirm each)
   - Regex capture groups in replacement

#### Data Structures
- **Match positions cache** (Vec<(line, col)>)
- **Search state** (current pattern, match index, options)

#### Milestone 4 Checklist
- [ ] Search dialog (Ctrl+F)
- [ ] Find next/previous
- [ ] Highlight matches
- [ ] Replace dialog (Ctrl+H)
- [ ] Replace and Replace All
- [ ] Regex support

---

### Phase 5: Multiple Buffers (Week 8)

#### Data Structures to Learn
1. **Buffer Management**
   ```rust
   struct BufferList {
       buffers: Vec<Buffer>,
       active_index: usize,
   }
   ```
   
2. **LRU Cache** for buffer access patterns
3. **HashMap** for buffer ID → Buffer mapping

#### Technical Components
1. **Buffer Switching**
   - Quick switch with Ctrl+Tab
   - Buffer list view
   - Close buffer
   
2. **Memory Management**
   - Lazy loading of buffer contents
   - Unload buffers not recently used
   - Handle many open files

#### Architecture Update

```
┌────────────────────────────────────────┐
│         Main Event Loop                │
└────────────┬───────────────────────────┘
             │
    ┌────────┴─────────┐
    ▼                  ▼
┌─────────────┐   ┌──────────┐
│   Editor    │   │ Terminal │
│             │   │  Render  │
│ ┌─────────┐ │   └──────────┘
│ │ Buffer  │ │
│ │  List   │ │
│ │         │ │
│ │ [Buf 0] │ │ ◄─ Active
│ │ [Buf 1] │ │
│ │ [Buf 2] │ │
│ └─────────┘ │
└─────────────┘
```

#### Milestone 5 Checklist
- [ ] Open multiple files at startup
- [ ] Switch between buffers
- [ ] Buffer list dialog
- [ ] Close buffer
- [ ] Warn when closing unsaved buffer

---

### Phase 6: Split Windows (Week 9-10)

#### Data Structures to Learn
1. **Tree Structure for Splits**
   ```rust
   enum Split {
       Leaf(BufferView),
       Horizontal(Box<Split>, Box<Split>),
       Vertical(Box<Split>, Box<Split>),
   }
   ```
   
2. **ViewPort Structure**
   ```rust
   struct ViewPort {
       x, y, width, height,
       scroll_offset,
       buffer_id,
   }
   ```

#### Algorithms
- **Recursive rendering** for split tree
- **Focus management** (which split is active)
- **Space allocation** (proportional split sizes)
- **Resize algorithm** (drag split borders)

#### Architecture Update

```
┌─────────────────────────────────────────┐
│         Main Event Loop                 │
└─────────────┬───────────────────────────┘
              │
     ┌────────┴────────┐
     ▼                 ▼
┌──────────┐      ┌──────────┐
│  Editor  │      │ Terminal │
│          │      │  Render  │
│ ┌──────┐ │      └──────────┘
│ │Split │ │
│ │ Tree │ │       Split Layout:
│ │      │ │       ┌────────┬─────┐
│ │  H   │ │       │  Buf 0 │Buf 1│
│ │ ┌─┬─┐│ │       ├────────┴─────┤
│ │ │L│R││ │       │    Buf 2     │
│ │ └─┴─┘│ │       └──────────────┘
│ └──────┘ │
│          │
│ Buffers: │
│ [0][1][2]│
└──────────┘
```

#### Milestone 6 Checklist
- [ ] Vertical split (Ctrl+W, V)
- [ ] Horizontal split (Ctrl+W, S)
- [ ] Navigate between splits (Ctrl+W, arrows)
- [ ] Close split (Ctrl+W, Q)
- [ ] Resize splits
- [ ] Each split can show different buffer

---

### Phase 7: Syntax Highlighting (Week 11-12)

#### Technical Components
1. **Parsing Approach**
   - Use `tree-sitter` crate (incremental parsing)
   - Alternatives: `syntect` (regex-based, simpler)
   
2. **Tokenization**
   - Lexer to identify tokens (keywords, strings, comments)
   - Map tokens to colors
   
3. **Rendering**
   - ANSI color codes
   - 256-color palette or true color
   - Theme system

#### Data Structures
- **Syntax Tree** (from tree-sitter)
- **Highlight Map** (token type → style)
- **Theme Definition** (JSON/TOML config)

#### Algorithms
- **Incremental reparsing** (only parse changed regions)
- **Viewport-based highlighting** (only highlight visible text)

#### Milestone 7 Checklist
- [ ] Basic syntax highlighting for one language (Rust)
- [ ] Detect file type by extension
- [ ] Multiple language support
- [ ] Theme support
- [ ] Toggle highlighting on/off

---

### Phase 8: LSP Integration (Week 13-15)

#### Rust Concepts to Learn
1. **Async Programming**
   - `async`/`await` syntax
   - `tokio` or `async-std` runtime
   - Futures and streams
   
2. **IPC (Inter-Process Communication)**
   - Spawning LSP server process
   - stdin/stdout communication
   - JSON-RPC protocol

#### Technical Components
1. **LSP Client**
   - Use `lsp-types` and `lsp-server` crates
   - Initialize connection with LSP server
   - Send text change notifications
   
2. **Features to Implement**
   - Auto-completion popup
   - Go to definition (Ctrl+Click or keybinding)
   - Hover information
   - Diagnostics (errors/warnings inline)
   
3. **UI Components**
   - Completion menu
   - Diagnostic markers in gutter
   - Hover popup windows

#### Architecture Update

```
┌───────────────────────────────────────────┐
│         Main Event Loop (async)           │
└──────┬─────────────────────┬──────────────┘
       │                     │
       ▼                     ▼
┌──────────┐          ┌────────────┐
│  Editor  │◄────────►│ LSP Client │
│          │          │            │
│  Buffers │          │  ┌──────┐  │
│  Splits  │          │  │Queue │  │
│          │          │  └──────┘  │
└──────────┘          └─────┬──────┘
                            │ JSON-RPC
                            ▼
                      ┌───────────┐
                      │LSP Server │
                      │  Process  │
                      └───────────┘
```

#### Milestone 8 Checklist
- [ ] LSP client initialization
- [ ] Send document open/change/close notifications
- [ ] Auto-completion (Ctrl+Space)
- [ ] Go to definition
- [ ] Show diagnostics
- [ ] Hover for documentation

---

### Phase 9: Command Execution (Week 16)

#### Technical Components
1. **Command Runner**
   - Use `std::process::Command`
   - Capture stdout and stderr
   - Show output in dedicated window/split
   
2. **Shell Integration**
   - Execute in buffer's directory
   - Environment variable support
   - Background vs foreground execution

#### Data Structures
- **Command History** (Vec<String>)
- **Output Buffer** (ring buffer for command output)

#### UI Components
- Command palette (Ctrl+P or Ctrl+Shift+P)
- Quick command input
- Output panel

#### Milestone 9 Checklist
- [ ] Command input dialog
- [ ] Execute command
- [ ] Show output in split
- [ ] Command history
- [ ] Common commands (build, test, run)
- [ ] Clear output

---

### Phase 10: Polish & Optimization (Week 17-18)

#### Performance Optimizations
1. **Rendering**
   - Dirty region tracking (only redraw changed areas)
   - Double buffering
   - Lazy rendering
   
2. **Memory**
   - Buffer unloading for inactive files
   - Limit undo history
   - Rope structure for large files
   
3. **I/O**
   - Async file loading
   - Mmap for very large files
   - Incremental file reading

#### Additional Features
- Configuration file support (TOML/JSON)
- Key binding customization
- Multiple cursors
- Minimap
- File tree explorer

#### Testing
- Unit tests for core data structures
- Integration tests for editor operations
- Property-based testing with `proptest`

#### Milestone 10 Checklist
- [ ] Configuration system
- [ ] Performance benchmarks
- [ ] Memory profiling
- [ ] Key bindings file
- [ ] User documentation

---

## Recommended Crates

### Terminal & Input
- `crossterm` - Cross-platform terminal manipulation
- `termion` - Alternative terminal library

### Text Handling
- `ropey` - Rope data structure for text
- `unicode-segmentation` - Proper Unicode handling

### Syntax & LSP
- `tree-sitter` - Incremental parsing
- `syntect` - Syntax highlighting
- `lsp-types` - LSP protocol types
- `tower-lsp` - LSP server framework

### Async
- `tokio` - Async runtime
- `async-std` - Alternative async runtime

### Utilities
- `regex` - Regular expressions
- `clap` - Command-line argument parsing
- `serde` - Serialization (for config)
- `anyhow` - Error handling

---

## Development Workflow

### Daily Routine
1. Review previous day's progress
2. Write tests for new feature
3. Implement feature
4. Run `cargo clippy` for lints
5. Run `cargo fmt` for formatting
6. Commit changes with clear message

### Testing Strategy
```bash
# Run tests
cargo test

# Check formatting
cargo fmt -- --check

# Run lints
cargo clippy -- -D warnings

# Check for common mistakes
cargo check
```

### Git Workflow
- Feature branches for each phase
- Commit after each milestone
- Tag releases (v0.1-mvp, v0.2-editing, etc.)

---

## Progress Tracking

- [ ] **Phase 0**: Rust fundamentals learned
- [ ] **Phase 1**: MVP - Display and cursor movement
- [ ] **Phase 2**: Basic text editing
- [ ] **Phase 3**: File operations and undo/redo
- [ ] **Phase 4**: Search and replace
- [ ] **Phase 5**: Multiple buffers
- [ ] **Phase 6**: Split windows
- [ ] **Phase 7**: Syntax highlighting
- [ ] **Phase 8**: LSP integration
- [ ] **Phase 9**: Command execution
- [ ] **Phase 10**: Polish and optimization

---

## Key Diagrams

### Overall Data Flow

```
User Input
    │
    ▼
┌────────────────┐
│  Input Handler │
│  (crossterm)   │
└───────┬────────┘
        │
        ▼
┌────────────────┐      ┌──────────────┐
│ Editor State   │◄────►│  LSP Client  │
│                │      │  (async)     │
│ • Buffers      │      └──────────────┘
│ • Splits       │
│ • Undo Stack   │      ┌──────────────┐
│ • Command Hist │◄────►│  File System │
└───────┬────────┘      └──────────────┘
        │
        ▼
┌────────────────┐
│   Renderer     │
│  (crossterm)   │
└───────┬────────┘
        │
        ▼
    Terminal
```

### Edit Operation Flow

```
User types 'a'
    │
    ▼
Create InsertCommand
    │
    ▼
Execute on Buffer
    │
    ├─► Update rope/gap buffer
    │
    ├─► Push to undo stack
    │
    ├─► Clear redo stack
    │
    ├─► Mark buffer dirty
    │
    ├─► Send change to LSP
    │
    └─► Trigger re-render
```

---

## Learning Resources

### Books
- **The Rust Programming Language** (official book)
- **Programming Rust** by Blandy & Orendorff
- **Rust for Rustaceans** by Gjengset (advanced)

### Projects to Study
- `xi-editor` - Rope-based editor
- `helix` - Modern text editor in Rust
- `zed` - High-performance editor
- `kilo` - Simple editor tutorial (C, but great for concepts)

### Documentation
- Rust Standard Library docs
- Crate documentation on docs.rs
- LSP specification
- Terminal escape codes reference

---

## Tips for Success

1. **Start Simple**: Get something working quickly, then iterate
2. **Test Early**: Write tests as you go, not after
3. **Read Code**: Study existing editors' source code
4. **Debug Tools**: Learn to use `rust-gdb` or `lldb`
5. **Ask for Help**: Rust community is friendly (Discord, Reddit, forums)
6. **Refactor Often**: Don't be afraid to restructure as you learn
7. **Profile**: Use `cargo flamegraph` to find bottlenecks
8. **Document**: Write doc comments for public APIs

---

## Expected Challenges

1. **Ownership & Lifetimes** - Will take time to master
2. **Terminal Quirks** - Different terminals behave differently
3. **UTF-8 Handling** - Multi-byte characters are tricky
4. **Async Complexity** - LSP integration adds async complexity
5. **Performance** - Large files need careful optimization
6. **Testing** - UI testing is inherently difficult

For each challenge, take time to understand the concepts deeply before moving forward.

---

## Conclusion

This plan provides a structured path from learning Rust basics to building a fully-featured text editor. Each phase builds on the previous one, allowing for steady progress while learning both Rust and software architecture principles. The MVP-first approach ensures you'll have a working editor early on, which provides motivation to continue adding features.

Remember: The goal is not just to build an editor, but to learn Rust deeply through practical application. Take time to understand the "why" behind each design decision, and don't hesitate to experiment with different approaches.

Happy coding! 🦀

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GlueSQL is a SQL database engine library written in Rust that provides a pluggable storage architecture. It supports both traditional SQL syntax and a programmatic AST Builder API, with the ability to work with structured and unstructured data across multiple storage backends.

## Development Commands

### Essential Commands (used in CI/CD)
- `cargo fmt` - Format code
- `cargo clippy` - Run linter
- `cargo test` - Run tests

### Build Commands
- `cargo build` - Build the project
- `cargo build --all-features --verbose` - Build with all features

### Test Commands
- `cargo test --verbose` - Run tests with verbose output
- `cargo test --all-features --verbose` - Run all tests with all features
- `cargo test --lib --bins --tests --examples --verbose` - Run comprehensive tests

### Module-Specific Testing
- `cd core && cargo test --verbose` - Test core module
- `cd storages/[storage-name] && cargo test --verbose` - Test specific storage
- `cd test-suite && cargo test --verbose` - Run the comprehensive test suite

### Storage-Specific Commands
- **MongoDB**: `cd storages/mongo-storage && cargo test --verbose --features test-mongo`
- **Redis**: `cd storages/redis-storage && cargo test --verbose --features test-redis`
- **Web Storage**: `cd storages/web-storage && wasm-pack test --headless --firefox`

### JavaScript/WebAssembly Development
- `wasm-pack build --target web` - Build for web browsers
- `wasm-pack build --target nodejs -- --no-default-features --features nodejs` - Build for Node.js
- `wasm-pack test --headless --firefox` - Run WASM tests

### Python Development
- `maturin build` - Build Python package
- `maturin build --release --strip` - Build optimized release
- `pytest` - Run Python tests

## Architecture Overview

GlueSQL follows a modular pipeline architecture:

```
SQL Input → Parser → Translator → Planner → Executor → Storage
```

### Core Components

**Main API (`core/src/glue.rs`)**
- `Glue<T>` - Primary user-facing API
- `execute()` - Full SQL execution pipeline
- `plan()` - Parse and plan SQL statements

**Storage System (`core/src/store.rs`)**
- Trait-based architecture with composable storage backends
- Key traits: `Store`, `StoreMut`, `Index`, `IndexMut`, `AlterTable`, `Transaction`
- All storage operations are async

**Query Processing Pipeline**
1. **Parse** (`parse_sql.rs`) - SQL string → `sqlparser` AST
2. **Translate** (`translate.rs`) - `sqlparser` AST → GlueSQL AST
3. **Plan** (`plan.rs`) - Optimize and validate statement
4. **Execute** (`executor.rs`) - Run against storage backend

**AST Builder (`ast_builder.rs`)**
- Fluent API for programmatic query construction
- Alternative to SQL strings for type-safe query building

### Key Directories

- `core/` - Main SQL engine implementation
- `storages/` - Storage backend implementations (Memory, Sled, JSON, CSV, MongoDB, etc.)
- `test-suite/` - Comprehensive test suite for all SQL operations
- `cli/` - Command-line interface
- `pkg/` - Language bindings (Rust, JavaScript, Python)

### Storage Backends

**Persistent Storage**
- **Sled**: Full-featured embedded database (supports all traits)
- **Redb**: Single-file embedded database
- **JSON/CSV/Parquet**: File-based storage

**In-Memory Storage**
- **Memory**: HashMap-based storage
- **Shared Memory**: Thread-safe memory storage

**External Systems**
- **MongoDB**: NoSQL database backend
- **Redis**: Key-value store backend

**Web Storage**
- **Web Storage**: localStorage/sessionStorage
- **IndexedDB**: Browser database storage

**Composite Storage**
- Combines multiple storage backends
- Supports JOINs across different storage types

## Testing Strategy

### Test Suite Structure
The `test-suite/` directory contains comprehensive tests organized by feature:
- `aggregate/` - Aggregation functions
- `join/` - JOIN operations
- `transaction/` - Transaction handling
- `data_type/` - Data type support
- `function/` - SQL functions

### Storage Testing
Storage implementations should run the full test suite to ensure compatibility:
```rust
use test_suite::*;
// Run all tests against your storage implementation
```

## Key Design Principles

1. **Pluggable Storage**: Storage backends implement common traits
2. **Async-First**: All operations are async with streaming support
3. **Type Safety**: Strong typing throughout AST and data layers
4. **Schema Flexibility**: Supports both structured and schemaless data
5. **Multi-Language Support**: Rust, JavaScript, Python bindings

## Common Development Patterns

### Creating Custom Storage
1. Implement required traits: `Store`, `StoreMut`
2. Optional traits: `Index`, `Transaction`, `AlterTable`
3. Use the test suite to validate implementation
4. Storage implementations are in `storages/[name]/`

### Adding New SQL Features
1. Extend AST definitions in `core/src/ast.rs`
2. Add translation logic in `core/src/translate.rs`
3. Implement execution in `core/src/executor.rs`
4. Add comprehensive tests in `test-suite/`

### AST Builder Development
1. Add builder methods in `core/src/ast_builder.rs`
2. Follow fluent API patterns
3. Ensure type safety in builder chain

## Performance Considerations

- All storage operations are async
- Streaming results with `futures::Stream`
- Storage backends should implement efficient scanning
- Index support is optional but recommended for performance
# contiguous-tree

Typed occurrence trees stored in a contiguous postorder arena. Every subtree
occupies one range of node IDs, allowing traversal and folds to scan storage.
The library includes validated builders, borrowed cursors, annotations, and
fallible transcoding and rewriting with rollback on error.

Children must be built before their parent, adjacent and in order. Nodes
represent occurrences: a child cannot be shared by multiple parents.

## Example

```toml
[dependencies]
contiguous-tree = "0.1.0"
```

```rust
use contiguous_tree::{TreeCursorExt, tree_schema};

tree_schema! {
    pub tree Term {
        metadata: offset: u32 = 0,
        id: u32,
        Number(value: data(i64)),
        Add(left: child, right: child),
    }
}

let mut builder = TermBuilder::with_capacity(3);
let left = builder.alloc_default(TermKind::Number(1));
let right = builder.alloc_default(TermKind::Number(2));
let root = builder.alloc_default(TermKind::Add(left, right));
let term = builder.finish(root).unwrap();
assert_eq!(term.as_ref().postorder().count(), 3);
```

The `serde` feature enables serialization support. The `thread-safe` feature
uses `Arc` instead of `Rc` for shared storage. Both are disabled by default.
See the [crate documentation](https://docs.rs/contiguous-tree) for the complete
interface and schema syntax. The procedural macros are reexported; applications
normally depend only on `contiguous-tree`.

## Development

Run `cargo test --workspace` and `cargo test --workspace --all-features`.
Run `cargo fmt --all -- --check` before committing. Release the macros before
the runtime, keeping their versions aligned; the runtime pins its macro version
because generated code uses internal runtime helpers. Check each package with
`cargo publish --dry-run` before publishing it.

## License

INTO-CPS Association Public License, using **GPL mode (GNU GPL version 3)**.
See [LICENSE.md](LICENSE.md), [COPYING](COPYING), and
[ICA-USAGE-MODE.txt](ICA-USAGE-MODE.txt).

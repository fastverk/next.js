# Outside-root symlinks in `turbo-tasks-fs`

`LinkType::OUTSIDE_ROOT` marks symlinks whose resolved target is outside the configured `DiskFileSystem` root.

## Invariants

- `read_link` returns `LinkContent::Link` with `OUTSIDE_ROOT` when the target exists but cannot be represented as an in-root `FileSystemPath`.
- `OUTSIDE_ROOT` links also set `ABSOLUTE`; their `target` is a system path (normalized to `/` separators).
- `realpath_with_links` stops at the symlink path for `OUTSIDE_ROOT` links instead of trying to continue resolution as an in-root path.
- Consumers that need file bytes should read via `FileSystemPath::read()` (the OS follows the symlink).
- Directory traversal checks treat `DIRECTORY | OUTSIDE_ROOT` as traversable and recurse through the symlink path.

## Limitations

Outside-root targets are external to the filesystem root and may not participate in the same invalidation/watch guarantees as in-root paths.

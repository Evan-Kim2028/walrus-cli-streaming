# Walrus CLI - Byte-Range Streaming Fork

This is a modified version of the [Walrus CLI](https://github.com/MystenLabs/walrus) with added byte-range streaming support for efficiently reading Sui checkpoint data from archival blobs.

## New Features

### CLI Flags

| Flag | Description |
|------|-------------|
| `--start-byte <N>` | Starting byte position for a range read (inclusive) |
| `--byte-length <N>` | Number of bytes to read |
| `--size-only` | Fetch blob size without downloading data |
| `--stream` | Stream raw bytes directly to stdout (zero-copy) |

### Examples

```bash
# Get blob size without downloading
walrus read <blob_id> --size-only

# Read a specific byte range
walrus read <blob_id> --start-byte 1000000 --byte-length 50000

# Stream a byte range to stdout (pipe to file or another process)
walrus read <blob_id> --start-byte 1000000 --byte-length 50000 --stream > output.bin

# Stream and pipe to a decoder
walrus read <blob_id> --start-byte 1000000 --byte-length 50000 --stream | my-decoder
```

## API Changes

### `ByteRangeReadClient`

New method for streaming directly to a writer:

```rust
pub async fn read_byte_range_to_writer<W: Write + ?Sized>(
    &self,
    blob_id: &BlobId,
    start_byte_position: u64,
    byte_length: u64,
    writer: &mut W,
) -> ClientResult<ReadByteRangeMeta>
```

This enables zero-copy streaming without buffering the entire range in memory.

## Use Case: Sui Checkpoint Archival

This fork was created to support efficient streaming of Sui checkpoint data from Walrus archival blobs. Each archival blob can be several GB containing thousands of checkpoints. The byte-range streaming support allows:

1. **Index extraction** - Read only the blob footer/index to discover checkpoint offsets
2. **Selective reads** - Fetch only the checkpoints needed, not the entire blob
3. **Memory efficiency** - Stream data without loading GB of data into memory
4. **Parallel processing** - Multiple workers can read different byte ranges concurrently

## Related Projects

- [deepbookv3-walrus-streaming](https://github.com/Evan-Kim2028/deepbookv3-walrus-streaming) - Uses this CLI for Sui checkpoint streaming

## Building

```bash
cargo build -p walrus-service --bin walrus --features test-utils --release
```

Binary will be at `./target/release/walrus`.

## Upstream

Based on [MystenLabs/walrus](https://github.com/MystenLabs/walrus) at commit `4191a5e1`.

Note: The upstream Walrus CLI has since added `--start-byte` and `--byte-length` flags in PRs #2708 and #2724. This fork adds additional features like `--size-only` and `--stream` for zero-copy output.

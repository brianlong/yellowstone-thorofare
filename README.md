# Yellowstone Thorofare gRPC Benchmark Tool

A simple tool to compare the performance of two Yellowstone gRPC endpoints. Thorofare will connect to two concurrent gRPC endpoints, listen for specific per-slot events, and record the relative timings between the endpoints.

## Life Cycle of a Solana Slot -- DRAFT WIP
The life cycle of a Solana slot is visible through specific events emitted in the Geyser stream.

This SlotStatus enum from `src/types.rs:16` shows the events.
```
pub enum SlotStatus {
    FirstShredReceived,
    Completed,
    CreatedBank,
    Processed,
    Confirmed,
    Finalized,
    Dead,
}
```

These metrics indicate the health of the RPC's network ingestion rate and Solana Shred configuration. 
`FirstShredReceived` marks the start of a slot when we receive the first shred from the current leader.
`Completed` marks the end of a slot when we have received all Shreds from the leader.

These metrics represent the RPC replaying transactions and voting internally on the Slot. This stage is sensitive to CPU & server configuration.
`CreatedBank` all Shreds have been replayed and the Slot is ready for internal voting
`Processed` 

The next three stages are dependent on external factors beyond the control of the RPC node, like validator voting.
`Confirmed` marks the end of the future slot when the current slot is confirmed by the validators. 
`Finalized` marks the end of the future slot when the current branch is confirmed by the validators.
`Dead` marks the marks the end of the future slot when the current branch is dropped by the validators.

## Gathering Metrics



## Compilation

### Build Release Binary
```bash
cargo build --release
# Binary will be at: target/release/grpc-bench
```

### Run Directly with Cargo
```bash
cargo run --bin grpc-bench -- --endpoint1 https://endpoint1.com:443 --endpoint2 https://endpoint2.com:443
```

## Basic Usage

```bash
# Compare two endpoints
./grpc-bench --endpoint1 https://endpoint1.com:443 --endpoint2 https://endpoint2.com:443

# With authentication tokens
./grpc-bench \
  --endpoint1 https://endpoint1.com:443 \
  --endpoint2 https://endpoint2.com:443 \
  --x-token1 YOUR_TOKEN_1 \
  --x-token2 YOUR_TOKEN_2

# Collect more slots (default is 1000)
./grpc-bench \
  --endpoint1 https://endpoint1.com:443 \
  --endpoint2 https://endpoint2.com:443 \
  --slots 5000

# With load testing (subscribes to additional account updates)
./grpc-bench \
  --endpoint1 https://endpoint1.com:443 \
  --endpoint2 https://endpoint2.com:443 \
  --with-load \
  --config config-with-load-example.toml
```

## CLI Options

- `--endpoint1` - First gRPC endpoint URL
- `--endpoint2` - Second gRPC endpoint URL  
- `--x-token1` - X token for endpoint1 (optional)
- `--x-token2` - X token for endpoint2 (optional)
- `--slots` - Number of slots to collect (default: 1000)
- `--config` - Config file path (default: config.toml)
- `--output` - Output JSON file (default: benchmark_results.json)
- `--log-level` - Log level: debug/info/warn/error (default: info)
- `--with-load` - Enable load testing mode

## Config Files

- Use `config-example.toml` for normal benchmarks
- Use `config-with-load-example.toml` when using `--with-load`

## Output

Results are saved to `benchmark_results.json` (or your specified output file) with performance metrics like:
- Latency percentiles (p50, p90, p99)
- Slot processing times
- Network ping times
- Detailed timing breakdowns

# Local Docker Compose Sync Test

This folder provides a local bidirectional sync smoke test using Docker Compose.
It runs `ink-bucket-sync` with `BUCKET_PROVIDER=local` and two bind mounts:

- `local-path/` mounted to `/data` (the app sync path)
- `remote-path/` mounted to `/remote` (simulated bucket path)

## Prerequisites

- Docker Desktop / OrbStack with Compose support
- Run commands from this directory: `local-test/`

## Start

```bash
docker compose up --build
```

You should see periodic logs like:

```text
[sync] Running bidirectional sync (...)
```

## Test Local -> Remote

In another terminal:

```bash
echo "change-from-local" > local-path/from-local.txt
ls -la remote-path
```

Expected: `remote-path/from-local.txt` appears quickly (watchdog-triggered sync).

## Test Remote -> Local

In another terminal:

```bash
echo "change-from-remote" > remote-path/from-remote.txt
ls -la local-path
```

Expected: `local-path/from-remote.txt` appears on next poll cycle (default ~3s).

## Watch Logs

```bash
docker compose logs -f sync
```

## Stop

```bash
docker compose down
```

## Reset Test Files

```bash
rm -f local-path/* remote-path/*
cp sample-files/local-path-example.txt local-path/
cp sample-files/remote-path-example.txt remote-path/
```

## Notes

- `SYNC_MODE=bidirectional` uses `rclone bisync`.
- Local file changes trigger fast sync via watchdog (`inotifywait`).
- Remote-side changes are picked up by the periodic poll loop.

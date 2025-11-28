# nighthouse-builder

`nighthouse-builder` is an automated build pipeline for producing **Nighthouse** Docker images — a minimally modified fork of the Ethereum consensus client **Lighthouse**.

Nighthouse introduces an operator-controlled option to **disable blob custody group count (CGC) increases**, reducing hardware and memory requirements at the cost of being **non-compliant** with Ethereum's protocol intentions. This makes it a *bad client*, beneficial only for operators who explicitly choose performance over network health.

This repository:

- Contains the Nighthouse patch
- Applies it to upstream Lighthouse during CI
- Automatically builds Docker images
- Publishes them to **Docker Hub** at:

```
ethnight/nighthouse
```

👉 https://hub.docker.com/r/ethnight/nighthouse

---

## ⚠️ Important Disclaimer

Running Nighthouse harms Ethereum’s data-availability assumptions.  
It should **not** be used on mainnet by responsible operators.

You are solely responsible for any usage of Nighthouse.

---

## What the Patch Does

The Nighthouse patch adds a single behavioral toggle:

### Disable Custody Group Count Increases  
When the `NIGHTHOUSE_NO_CGC_INCREASE` environment variable is present, all CGC-increase logic returns early and does nothing.

Example of the applied guard in the patch:

```rust
if std::env::var("NIGHTHOUSE_NO_CGC_INCREASE").is_ok() {
    return None;
}
```

This prevents Lighthouse from scaling custody requirements based on network conditions.

---

## How the Builder Pipeline Works

The GitHub Action:

1. Clones upstream Lighthouse
2. Checks out the chosen tag/commit
3. Applies `nighthouse/01-disable-cgc-update.patch`
4. Builds Lighthouse with Rust in CI
5. Produces a Docker image
6. Pushes it to **Docker Hub** under:

```
ethnight/nighthouse:<tag>
```

Images include both `latest` and versioned tags.

---

## Using the Docker Images

### Run with dynamic CGC increases **disabled** (Nighthouse mode)

```
docker run -e NIGHTHOUSE_NO_CGC_INCREASE=1 \
    ethnight/nighthouse:latest \
    lighthouse bn --network mainnet
```

### Run with Lighthouse’s default behavior (CGC increases enabled)

```
docker run \
    ethnight/nighthouse:latest \
    lighthouse bn --network mainnet
```

---

## Environment Variables

| Variable                     | Purpose                                                |
|------------------------------|--------------------------------------------------------|
| `NIGHTHOUSE_NO_CGC_INCREASE` | If set to *anything*, disables CGC increases entirely |

Example:

```
export NIGHTHOUSE_NO_CGC_INCREASE=1
docker compose up
```

---

## Local Development or Manual Builds

Clone the repo:

```
git clone https://github.com/<your-org>/nighthouse-builder.git
cd nighthouse-builder
```

Apply the patch manually if working with Lighthouse directly:

```bash
git apply 01-disable-cgc-update.patch
```

Build Lighthouse:

```bash
cargo build --release
```

---

## License

Patched Lighthouse code follows Lighthouse’s original license.  
Build scripts and workflows in this repo are MIT-licensed unless stated otherwise.

---

## Contributions

This project is intended for experimentation and research.  
PRs improving the builder or documentation are welcome.

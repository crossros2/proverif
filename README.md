# CrossROS2 ProVerif Models

This directory contains ProVerif models for checking the CrossROS2 bridging protocol.

## Files

- `CrossROS2.pv`: updated CrossROS2 ProVerif model that binds `tid` and `Seq_dw` into the signed TEK distribution payload and signed data payload.

## Requirements

- ProVerif 2.05 or later.
- On macOS, installing ProVerif through OPAM is recommended.

## Install ProVerif

Install OPAM if it is not already installed:

```sh
brew install opam
```

Initialize OPAM:

```sh
opam init
eval $(opam env)
```

Install ProVerif:

```sh
opam install proverif
```

Check that ProVerif is available:

```sh
command -v proverif
proverif -help
```

## Run The Models

From the repository root:

```sh
proverif proverif/CrossROS2.pv
```

Or from this directory:

```sh
proverif CrossROS2.pv
```

## Expected Results

For `CrossROS2.pv`, the current expected verification summary is:

```text
Query not attacker(CrossROS2_key) is true.
Query event(GET_TEK(..., topic_t, seq_t)) ==> event(DIST_TEK(..., topic_t, seq_t)) is true.
Query event(VRFY_DW(..., topic_t, seq_t)) ==> event(WRITE_MSG(..., topic_t, seq_t)) is true.
```

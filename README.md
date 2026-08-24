# CrossROS2 ProVerif Models

This directory contains ProVerif models for checking the CrossROS2 bridging protocol.

## Files

- `CrossROS2.pv`: updated CrossROS2 ProVerif model that binds `tid` and `Seq_dw` into the signed TEK distribution payload and signed data payload.
- `CrossROS2_membership.pv`: two-epoch model for post-removal re-keying. It uses the paper-specific `TEK_t`, reader-group HPKE key types, epoch-bound signatures, and an AM-signed group-public-key record.

`CrossROS2_membership.pv` does not implement MLS or TreeKEM. A fresh reader-group HPKE key pair and fresh TEK at `epoch1` are abstract outputs of the trusted group-management layer after a reader is removed. At removal time, the attacker receives the former member's old TEK and old reader-group private key; the compromised bridge also reveals both transport keys.

## Model Scope

### `CrossROS2.pv`

This is the baseline CrossROS2 model used for the main confidentiality and authenticity checks. The model has one data writer, one authorized data reader, and one bridge between the ROS 2 and non-ROS 2 sides. The bridge may be compromised and can reveal the hop-by-hop transport keys, but the topic encryption key remains protected by the reader-group HPKE key.

The model checks three properties:

- The topic encryption key used by CrossROS2 is not derivable by the attacker.
- If a reader accepts a distributed TEK, the same TEK distribution payload must have been produced by the legitimate data writer.
- If a reader verifies a runtime message, the same message must have been written by the legitimate data writer.

This file corresponds to the original three-query verification summary in the paper.

### `CrossROS2_membership.pv`

This is the membership-change extension used to analyze the reviewer's revocation and key-update concern. The model separates two membership epochs. In `epoch0`, both the current reader and the reader that will later be revoked are valid group members. In `epoch1`, the revoked reader is removed, the reader-group HPKE key pair is refreshed, and the data writer distributes a fresh topic encryption key `tek'` under the updated reader-group public key.

The model intentionally does not implement the full MLS or TreeKEM update algorithm. Instead, the updated reader-group key pair and the fresh `tek'` are modeled as the abstract result of a trusted group-management layer. This keeps the verification focused on the CrossROS2 security property: whether old key material held by a removed member is enough to learn the new topic encryption key.

The adversary is given the strongest former-reader view modeled here: the previous topic encryption key `tek`, the previous reader-group private key, all public protocol traffic, and both compromised bridge transport keys. The single query checks that this information is still insufficient to derive the newly distributed `tek'`.

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
proverif proverif/CrossROS2_membership.pv
```

Or from this directory:

```sh
proverif CrossROS2.pv
proverif CrossROS2_membership.pv
```

## Expected Results

For `CrossROS2.pv`, the current expected verification summary is:

```text
Query not attacker(CrossROS2_key) is true.
Query event(GET_TEK(..., topic_t, seq_t)) ==> event(DIST_TEK(..., topic_t, seq_t)) is true.
Query event(VRFY_DW(..., topic_t, seq_t)) ==> event(WRITE_MSG(..., topic_t, seq_t)) is true.
```

The first query establishes secrecy of the original CrossROS2 TEK under the baseline model. The second and third queries establish sender authenticity for TEK distribution and runtime data messages, respectively.

For `CrossROS2_membership.pv`, the expected verification summary is:

```text
Query not attacker(tek') is true.
```

Depending on the ProVerif rendering of phases, the concrete output may appear as `Query not attacker_p1(tek'[]) is true.` This is the same secrecy check for the post-removal `epoch1` key.

The former reader was an authorized `epoch0` member. After its removal, `LeakOldState` gives the attacker `tek` and the previous reader-group private key. Public protocol traffic, certificates, permissions, and transport keys are also available. The single secrecy query proves that this complete former-member view is insufficient to derive the fresh `tek'`.

When this result is reported together with the baseline model, it becomes the second confidentiality query in the paper-level summary:

```text
Query not attacker(tek') is true.
```

This query means that, after the membership change, a revoked reader cannot learn the updated topic encryption key `tek'` even though it still knows the old topic encryption key and the previous group private key.

As a negative control, leaking `tek'` makes the secrecy query false.

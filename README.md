# HOLINA seal anchors

A public, append-only record of cryptographic anchors for HOLINA's sealed call
records. It exists so that claims about **when** a record existed do not rest on
HOLINA's word.

You do not have to trust this repository, or HOLINA, to check what is here.

## What an anchor is

Every sealed call produces digests: one over the call audio, one over the
transcript, one over the receipt binding them together. Periodically, every seal
in a window is combined into a Merkle tree, and the resulting root is submitted
to **independent RFC-3161 timestamp authorities** — currently freetsa.org,
DigiCert and Sectigo — which countersign it with their own keys.

Each anchor here contains that root, the leaf hashes, and the raw timestamp
tokens (`.tsr`).

## What this proves

**That a set of records existed at a point in time.** The timestamp is signed by
third parties. Backdating a record would require their private keys, not
HOLINA's.

**That the set has not been edited since.** Each anchor commits to its
predecessor's root as the first leaf of its tree, so the anchors form a chain.
Removing or altering one breaks the link in every anchor that follows.

**That a specific call was included** — without revealing the others. Whoever
holds a call's own digests can recompute its leaf and present a Merkle path to a
published root.

## What this does NOT prove

Stated plainly, because a proof that is oversold is worse than none:

- **Not that a transcript is accurate.** The transcript is produced by automated
  speech recognition. A seal proves it has not changed since sealing — never
  that it was correct to begin with. This is why the audio is bound to it: so
  the machine's output can always be checked against its own source.
- **Not that HOLINA generated the audio.** Call audio comes from the telephony
  carrier.
- **Not that no record is missing.** The chain makes a *removed anchor*
  detectable. It says nothing about a call that was never sealed at all.
- **Not a legal conclusion.** Nobody has tested any of this in a proceeding.

## Verify a timestamp yourself

Every `.tsr` is a standard RFC-3161 token. With OpenSSL:

    openssl ts -reply -in <anchor>.freetsa.tsr -text

Check that `Message data` equals the `merkle_root` in the matching `.json`, and
read `Time stamp`. That time is attested by the authority named in the token,
not by us.

## Verify the chain

Each anchor's `prev_anchor_root` must equal the `merkle_root` of the anchor
before it. The earliest anchor predates chaining and is marked as such.

## Verify a root from its leaves

Leaves are hashed with a `0x00` prefix and internal nodes with `0x01`, so a leaf
can never be reinterpreted as a node. An odd node at any level is promoted
unchanged rather than duplicated, which avoids the well-known root-ambiguity
bug. Recompute the root from `leaves` in order and compare.

## What is deliberately withheld

Call identifiers and per-call digests are **not** published. Publishing them
would disclose call volume and timing for HOLINA's customers while adding
nothing: the leaf hashes are sufficient to verify the root and any inclusion
proof. What is public is hashes, counts, time windows and the third-party
tokens.

# 🦅 The Falcon Has Landed

**Cygnet Genesis v2 — Post-Quantum Trust Anchor & FN-DSA Falcon-512 Validation Milestone**

> Touchdown confirmed. After a long descent through the Gaussian sampler,
> the Falcon is on the ground, wheels locked, chain verified. ✅

This repository is the **public milestone announcement** for Sanctum SecOps LLC's
second-generation post-quantum trust anchor and our Falcon-512 (FN-DSA) signature
validation effort. It is intentionally **evidence-light**: the full validation
bundle, harnesses, keys, and certificates are held privately. What you'll find
here is the honest story of what we built, what we validated, and — importantly —
what we had to fix along the way.

---

## What landed

- **A v2 post-quantum trust anchor** signed with **ML-DSA-65** (FIPS 204).
- **A Falcon-512 (FN-DSA) end-entity certificate** chained to that anchor —
  a Falcon public key, signed by an ML-DSA-65 root, verifying cleanly.
- **A reproducible validation bundle** for the Falcon Gaussian sampler, built on
  open reference implementations and standard side-channel tooling.

All issued under our IANA-registered Private Enterprise Number arc.

🔗 **Public PEN registration:** [IANA PEN 65953 — Sanctum SecOps LLC](https://www.iana.org/assignments/enterprise-numbers/)
(Falcon-512 validation policy lives under the `…65953.2.6.1.4` arc.)

---

## Why Falcon needs extra care

Falcon (now **FN-DSA**, NIST FIPS 206, still in development) is fast and compact,
but its signing step leans on a **discrete Gaussian sampler** — and that sampler
is the part of the algorithm that the side-channel literature keeps poking at.
A naïve implementation can leak secret-dependent information through timing or
power. So we don't just say "it signs." We test it.

---

## The validation checklist (8 artifacts)

| # | Artifact | Validated against | Status |
|---|----------|-------------------|--------|
| 01 | Sampler-type declaration | Implementation self-disclosure (isochronous vs. variable-time) | ✅ Complete |
| 02 | Implementation metadata | Provenance of the impl under test | ✅ Complete |
| 03 | Known-answer / roundtrip | Sign → verify correctness on the reference impl | ✅ Complete |
| 04 | Timing constant-time test | **dudect** (Welch's t on wall-clock timing) | ✅ **PASS** |
| 05 | Constant-time instrumentation | **ctgrind / Memcheck** (secret-dependent branch detection) | ✅ Complete (see note) |
| 06 | Power-trace leakage assessment | **TVLA** (fixed-vs-random, physical power/EM traces) | ⏳ **Pending lab** |
| 07 | Output distribution test | **SAGA** (Gaussian distribution conformance, χ² test) | ✅ **PASS** |
| 08 | Manifest | Integrity hashes over the full bundle | ✅ Complete |

**7 of 8 complete. 1 honestly pending.**

---

## What we had to fix (the honest part)

Validation isn't validation if you only publish the wins. Two corrections from
this round:

- **We caught ourselves over-claiming.** Our constant-time instrumentation (05)
  initially carried a blunt **"FAIL — not constant-time"** label. On review, that
  was an overstatement: Falcon's sampler is the published **isochronous** design,
  engineered so its *timing* is balanced even though secret-dependent branches
  still exist by construction. The branch-detector flagging those branches is a
  **necessary-but-not-sufficient** signal — it does **not**, by itself, prove a
  timing leak. We rewrote the artifact to say exactly that. The timing test (04)
  passing is consistent with the isochronous design; the distribution test (07)
  passing confirms the sampler's output is correct.

- **We refused to fake the hard one.** The power-trace TVLA (06) is the test that
  would actually resolve the intermediate-value leakage the literature describes —
  and it requires **physical lab hardware** (oscilloscope / ChipWhisperer, probes,
  thousands of aligned traces). There is **no honest software substitute**, and a
  sealed hardware token can't stand in for it either. So 06 is marked
  **PENDING-LAB** with a documented methodology, not quietly fudged with synthetic
  data. When the bench is set up, it becomes real.

> **Our rule:** every numeric bound derives from a published model and is
> reproducible. No synthetic data. No fabricated security claims. Ever.

---

## Deployment guidance (short version)

- Where **only wall-clock timing** is in scope → the timing test governs, and it's clean.
- Where **power/EM side channels** are in scope → Falcon runs in an **HSM/enclave**
  pending the TVLA result.
- For **unprotected online signing** → prefer **ML-DSA** (which avoids the Gaussian
  sampler class entirely).

---

## What's *not* in this repo (by design)

No private keys. No certificates. No test-harness source. No logs, manifests,
metadata, file paths, serials, or hashes. This is an announcement of a milestone,
not a disclosure of the implementation. The reproducible bundle is held privately.

---

*Sanctum SecOps LLC · PEN 1.3.6.1.4.1.65953 · The Falcon has landed.* 🦅

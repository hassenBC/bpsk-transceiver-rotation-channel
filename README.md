# BPSK Transceiver for a Rotating Noisy Channel

End-to-end digital communication system that sends a 40-character text message through a noisy channel that applies an **unknown rotation** to the signal, and recovers it with ~94% perfect-decode probability under a fixed energy budget.

Team project for *Principles of Digital Communications* (EPFL, 2026).
**My part:** the full receiver: rotation detection, soft-decision Viterbi decoder, channel simulator, Monte Carlo test harness and error analysis (`Decoding/`).

## How it works

```text
message ─► encode ─► x (500 samples) ─► channel: rotation T_i + Gaussian noise ─► y ─► decode ─► message
```

| Stage | Design |
|---|---|
| Frame | `x[0:16]` = 8 pilot pairs `(r, 0)`, `x[16:500]` = 484 BPSK coded bits |
| Energy | ‖x‖² = 492·r² = 1200 → r ≈ 1.562 |
| Rotation detection | ML (minimum squared distance) over the 4 candidate rotations using the pilots, P(error) ≈ 10⁻⁵ |
| Channel code | Convolutional code K=3, generators (7, 5) octal, free distance 5 |
| Decoder | Soft-decision Viterbi (≈ 2 dB gain over hard decision) |

**Why pilots:** BPSK is antipodal, so any symmetric codebook has error probability ≥ 1/2 under the unknown rotation. Pilots break the symmetry.

**Result:** going from a rate-1/2 repetition code to the convolutional code took the probability of a perfect 240-bit decode from ~4% to ~94% with the same channel uses and energy.

## Repository layout

| Folder | Contents |
|---|---|
| `Encoding/` | `encode()`, unit tests, end-to-end tests with the decoder |
| `Decoding/` | `decode()`, robust decoder with confidence score, channel simulator, Monte Carlo harness, BER analysis, demo runner |
| `Python client/` | Course-provided client for the EPFL channel server |

## Run it

```bash
pip install numpy scipy

# local end-to-end simulation (no server needed)
cd Decoding
python demo.py test "Hello this is the test message ok."

# Monte Carlo reliability + BER
python test_harness.py --mc --trials 1000 --ber
python error_analysis.py
```

The live pipeline (`python demo.py run "<message>"`) needs the EPFL network or VPN.

## Tech

Python · NumPy · SciPy · Monte Carlo simulation · detection & estimation theory · channel coding

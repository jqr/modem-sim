# Modem Handshake Simulator

Browser-based audio simulator that recreates the sound and behavior of a dialup modem handshake using the Web Audio API. Synthesizes each phase of the connection process with accurate frequencies, modulation schemes, and timing.

## Reference Data

### Handshake Phases & Sources

**Dial Tone (US)** — 350 Hz + 440 Hz continuous
- Source: Precise Tone Plan (LSSGR FSD 01-02-0025), Bellcore GR-506

**DTMF Dialing** — Dual-tone multi-frequency
- Each digit is two simultaneous tones from a 4x3 matrix
- Row frequencies: 697, 770, 852, 941 Hz
- Column frequencies: 1209, 1336, 1477 Hz
- Tone duration controlled by S11 register (50–255ms, default 95ms)
- Gap between digits equals tone duration (S11 value)
- Source: ITU-T Q.23, ITU-T Q.24
- https://en.wikipedia.org/wiki/Dual-tone_multi-frequency_signaling

**Pulse Dialing** — Line interruption at 10 pulses/sec
- Each digit is encoded as N pulses (1–9 = 1–9 pulses, 0 = 10 pulses)
- US make/break ratio: 39% make / 61% break (39ms on, 61ms off per pulse)
- European make/break ratio: 33% make / 67% break
- Inter-digit pause: ~700ms
- What you hear through modem speaker vs. phone handset differs:
  - **Phone handset**: receiver is muted/short-circuited during dialing by an off-normal switch on the dial mechanism. User hears very little — faint muffled clicks at most. Telephone designers deliberately suppressed pulse noise using varistors or circuit disconnection.
  - **Modem speaker**: monitors the line directly with no muting circuit. You hear the line signal being rapidly gated on and off — bursts of line noise (and dial tone on the very first pulse before the exchange drops it) during make periods, silence during break periods.
- The first pulse of the first digit interrupts the dial tone; the exchange recognizes dialing has begun and drops the dial tone for all subsequent pulses.
- Source: ITU-T Q.23
- https://en.wikipedia.org/wiki/Pulse_dialing
- https://en.wikipedia.org/wiki/Rotary_dial
- https://www.3amsystems.com/World_Tone_Database/Signalling_guide?q=Pulse_dialing

**Ringback (US)** — 440 Hz + 480 Hz, 2s on / 4s off cadence
- Source: Bellcore GR-506

**ANSam Tone** — 2100 Hz carrier, 15 Hz amplitude modulation, phase reversals every 450ms
- The answering modem's first signal; disables echo cancellers on the line
- AM creates sidebands at 2085 and 2115 Hz
- Phase reversals prevent echo canceller re-engagement
- Source: ITU-T V.8 (Section 5.2.1), ITU-T V.25
- https://www.itu.int/rec/T-REC-V.8
- https://en.wikipedia.org/wiki/V.8

**V.8 CM/JM Signals** — V.21 channel 2 FSK modulation at 300 baud
- CM (Calling Menu): mark 1650 Hz, space 1850 Hz — originating modem advertises capabilities
- JM (Joint Menu): mark 1850 Hz, space 1650 Hz — answering modem acknowledges
- Source: ITU-T V.8, ITU-T V.21
- https://www.itu.int/rec/T-REC-V.21

**Carrier Exchange** — Initial carrier tones at 1200/2400 Hz
- Calling modem transmits 1200 Hz, answering modem transmits 2400 Hz
- Frequencies converge as modems achieve carrier lock
- Source: ITU-T V.22bis (for lower speeds), ITU-T V.34

**V.34 Line Probing (Phase 2)** — 21 simultaneous tones at 150 Hz spacing (150–3300 Hz)
- L1 and L2 probing signals sent in each direction
- Measures channel frequency response, signal-to-noise ratio, and group delay
- Band-edge probes at 300 and 3300 Hz measure rolloff characteristics
- The simulator measures a per-tone SNR for each of the 21 probe tones against the channel model and displays it as a bar graph — tones sag wherever the line is bad (notch, rolloff). Originator and answerer measure independently (the answerer sees a slightly worse line).
- Source: ITU-T V.34 (Section 7, Phase 2)
- https://www.itu.int/rec/T-REC-V.34
- https://en.wikipedia.org/wiki/V.34

**Equalizer Training** — Scrambled PRBS-15 at the negotiated symbol rate
- Uses a 15-bit linear feedback shift register (taps at bits 14 and 13)
- Modulated as QAM onto a 1800 Hz carrier with raised-cosine pulse shaping
- Symbol rates: 600 (V.22bis), 2400 (V.32bis), 3429 (V.34), 8000 (V.90)
- Creates the characteristic "waterfall" sound
- DMT-style bit loading: each usable subcarrier carries bits proportional to its SNR (~3 dB/bit), tones below ~12 dB are abandoned, and the carrier center shifts toward the cleanest part of the band. A line that can't hold the target rate retrains and falls back through the standard rates until it locks; if too little of the band survives, it gives up with NO CARRIER.
- Source: ITU-T V.34 (Section 7, Phase 3)

**Rate Negotiation** — Parameter exchange for final connection speed
- Each endpoint proposes a rate from its own SNR estimate; the final connect speed is the lower of the two, snapped to a standard rate
- Source: ITU-T V.34 (Section 7, Phase 4)

### Line Impairments

**60 Hz Hum** — Power line coupling with 2nd/3rd harmonics (120, 180 Hz)
- Amplitude scales inversely with line quality

**Impulse Noise** — Random short bursts (crackle/pops) with exponential decay
- Rate increases as line quality decreases

**Telephone Band Noise** — White noise bandpass filtered to 300–3400 Hz
- Models the PSTN voice channel bandwidth

**Frequency Notch** — A configurable dip in the channel response (center frequency, depth, width)
- Models a narrowband impairment such as a bridge tap or loading-coil resonance
- Applied as a filter on the audio, so it shows up in the spectrogram, the sound, and the per-tone probe SNR — and the modems route around it by abandoning the affected subcarriers

**High-Frequency Rolloff** — Progressive attenuation above a configurable corner frequency
- Models the loss of the upper voice band on long or loaded local loops
- Reduces SNR (and therefore bit loading) on the high tones first

### Protocol Specifications

| Protocol | ITU Standard | Max Speed | Symbol Rate | Year |
|----------|-------------|-----------|-------------|------|
| V.22bis  | ITU-T V.22bis | 2,400 bps | 600 baud | 1984 |
| V.32bis  | ITU-T V.32bis | 14,400 bps | 2,400 baud | 1991 |
| V.34     | ITU-T V.34   | 33,600 bps | 3,429 baud | 1994 |
| V.90     | ITU-T V.90   | 56,000 bps | 8,000 baud | 1998 |

### Key References

- ITU-T V.8: https://www.itu.int/rec/T-REC-V.8
- ITU-T V.21: https://www.itu.int/rec/T-REC-V.21
- ITU-T V.34: https://www.itu.int/rec/T-REC-V.34
- ITU-T V.90: https://www.itu.int/rec/T-REC-V.90
- DTMF: https://en.wikipedia.org/wiki/Dual-tone_multi-frequency_signaling
- Pulse dialing: https://en.wikipedia.org/wiki/Pulse_dialing
- Rotary dial mechanics: https://en.wikipedia.org/wiki/Rotary_dial
- World Tone Database (pulse dialing): https://www.3amsystems.com/World_Tone_Database/Signalling_guide?q=Pulse_dialing
- V.34 protocol walkthrough: https://en.wikipedia.org/wiki/V.34
- Modem sounds explained: https://oona.windytan.com/posters/dialup-final.png (Windytan's spectrogram poster)
- Why dial-up modems made noise: https://www.howtogeek.com/670455/why-did-dial-up-modems-screech-and-make-funny-noises/
- Web Audio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API

### Accuracy Notes

The simulator aims for faithful reproduction of the frequencies, modulation schemes, and timing described in the ITU standards. Known simplifications:

- V.8 CM/JM bit patterns are simplified (correct modulation, approximate data)
- V.34 probing uses equal-amplitude tones (real probing has specific amplitude/phase relationships per ITU-T V.34 Table 2)
- Training uses BPSK-equivalent QAM (real V.34 uses higher-order constellations: 4-QAM through 1664-QAM)
- Phase timing is approximate (real timing depends on line conditions and modem firmware)
- The probing/training/negotiation behavior is driven by a channel *model* (`H(f)` plus a per-frequency SNR estimate), not by actually demodulating the synthesized audio. The audio you hear, the spectrogram, the per-tone SNR bars, and the negotiated rate all derive from the same model, so they agree — but the modems aren't really decoding bits off the line.

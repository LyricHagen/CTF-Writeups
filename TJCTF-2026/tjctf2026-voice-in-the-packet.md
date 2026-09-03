# Voice in the Packet (May 15, 2026)

**Category:** forensics  
**Points:** 278  
**Files Provided:** call.pcap

## Challenge Description

"I intercepted a suspicious phone call over the network. something tells me there's more to this conversation than meets the ear..."

## Step 1: Exploring

The pcap had three UDP flows: two single-packet flows on weird ports, plus 1000 packets of 192.168.1.100:10000 -> 192.168.1.200:20000. The two odd packets decoded straight from hex as obvious decoys:

```
tjctf{this_is_a_fake_flag_keep_looking}
tjctf{definitely_not_the_real_flag}
```

The bulk flow looked RTP-shaped (172-byte payloads, regular spacing) but tshark didn't auto-recognize it. Forcing it:

```
tshark -r call.pcap -d "udp.port==20000,rtp" -d "udp.port==10000,rtp"
```

confirmed G.711 µ-law (PCMU), 1000 packets × 20 ms = 20 seconds of audio. Dumped the payloads, fed them to ffmpeg as -f mulaw -ar 8000 -ac 1.

## Step 2: It's Not Voice

All RTP header fields (marker, padding, ext, p_type, ssrc) were constant - no covert channel there. A Goertzel DTMF scan found nothing.

FFT of the whole file plus several 100 ms snippets showed identical spectra in every slice: a 220 Hz fundamental with harmonics, RMS energy flat over the full 20 s. So the audio is just a periodic tone, looped.

I tested candidate loop periods by counting (raw[i] & mask) == (raw[i-p] & mask) matches:
- Period 400 samples (50 ms) gave 100% match with bit-0 masked
- Periods 800 and 1600 also matched (multiples)

400 cycles in the file. Counting unique cycles:

```
periods = [raw[i*400:(i+1)*400] for i in range(400)]
Counter(periods).most_common()
# -> 398 copies of period A, 1 copy of period B at index 0, 1 copy of period C at index 1
```

Only the first two cycles differ from the template. XOR'ing each against the template, every diff is exactly bit 0 - LSB stego, only the first 100 ms.

## Step 3: Decoding

LSB-packing the µ-law bytes directly gave garbage. But every diff was at an EVEN byte position - meaning the encoder treats the byte stream as 16-bit LE PCM and modulates bit 0 of each 16-bit sample (= bit 0 of the low byte = the even byte). The audio still plays as a buzz under µ-law interpretation; that's just camouflage.

Re-decoding as 200 16-bit samples per period, MSB-first packing:

```
bits = "".join(str(raw[i*400 + 2*k] & 1) for i in (0, 1) for k in range(200))
msg = bytes(int(bits[i:i+8], 2) for i in range(0, len(bits), 8))
```

After two null sync bytes, the printable content is base64:

```
dGpjdGZ7aDN5X3YwaXBfczczZ19pc180XzdoaW5nfQ==
```

tjctf{h3y_v0ip_s73g_is_4_7hing}

## Takeaways
- tshark needs -d "udp.port==X,rtp" to decode unrecognized RTP flows.
- "Voice" challenges aren't always speech - check whether the audio is just a looped tone, then count unique loop cycles.
- The transport codec (µ-law) and the stego sample width (16-bit LE) don't have to match; the giveaway is which byte positions inside the period actually flip.


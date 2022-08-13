<h1 align="center">
  Brute Force OOK using <a href="https://flipperzero.one">Flipper Zero</a>
</h1>

<p align="center">
  <img src="https://user-images.githubusercontent.com/29007647/182851959-afaa1367-9f4d-46c8-92af-aa5ff70fca64.png" />
</p>

Brute force subghz fixed code protocols using flipper zero, initially inspired by [CAMEbruteforcer](https://github.com/BitcoinRaven/CAMEbruteforcer)

This repo aims to collect as many brute force files/protocols as possible, so if you can or want to contribute you are more than welcome to do so!

## How it works

Using `flipperzero-bruteforce.py` you can generate bruteforce `.sub` files for subghz protocols that use fixed OOK codes. Inside the script it is also possible to specify your own protocol in case it's not present.

To generate all the files simply run:

```bash
python3 flipperzero-bruteforce.py
```

It will generate bruteforce files for all the specified protocols organized in many folders with the following structure:

```
sub_files/
└── PROTOCOL_NAME
    ├── SPLIT_FACTOR
    │   ├── 000.sub
    │   ├── 001.sub
    │   ├── ...
    │   └── NNN.sub
    └── debruijn.sub
```

For each protocol there are 6 sub folders, containing 1, 2, 4, 8, 16 and 32 files, `SPLIT_FACTOR` indicates the number of keys per `.sub` file. This is useful when trying to get a close guess to the key.

For example, let's say you are trying to bruteforce a gate using CAME 12 bit protocol:

1. Play the single file (`4096/` folder) to make sure the attack works
2. Play the two files inside `2048/` folder, to see which half contains the correct key (suppose the second one works, containing keys from 2048 4095)
3. Play the second two files (`002.sub`, `003.sub`) inside the `1024/` folder to narrow the search
4. Keep doing this until you reach the last files inside the `128/` folder, these files take less than 10 seconds to send, almost the same as having the actual remote.

If you wanted to narrow the search even more you could modify the script to generate your own files containing less keys.

## Currently supported protocols

Right now the protocols supported are:

- Linear-10bit
- CAME-12bit
- NICE-12bit
- PT-2240
- PT-2262

More info about them can be found [here](https://phreakerclub.com/447)

### Adding a protocol

Adding a protocol is very straight forward, inside the script protocols are defined at the bottom, inside the protocol list:

```python
protocols = [
    Protocol("CAME", 12, {"0": "-320 640 ", "1": "-640 320 "}, "-11520 320 "),
    Protocol("NICE", 12, {"0": "-700 1400 ", "1": "-1400 700 "}, "-25200 700 "),
    Protocol("8bit", 8,  {"0": "200 -400 ", "1": "400 -200 "}),  # generic 8 bit protocol
    ...
]
```

A protocol is defined by a few parameters passed to the constructor in the following order:

- name: the name of the protocol
- n_bits: the number of bits for a single key
- transposition_table: how 0s and 1s are translated into flipper subghz `.sub` language
- pilot_period: aka preamble, a recurring pattern at the beginning of each key, defaults to `None`
- frequency: working frequency, defaults to 433.92

### Optimization

After a little testing working with CAME 12 bit (together with [@BitcoinRaven](https://github.com/BitcoinRaven)), it seems that as long as the **ratio** between the bits' lenghts is respected (`x`: short pulse, `2x`: long pulse, `36x`: pilot period) the actual duration of x can be lowered from the original 320 microseconds. From testing it seems that 250 microseconds is stable, shortening the bruteforce by a good minute (224 seconds against 287 seconds). From a code point of view:

```python
protocols = [
    # Old CAME
    Protocol("CAME", 12, {"0": "-320 640 ", "1": "-640 320 "}, "-11520 320 "),
    # New CAME
    Protocol("CAME", 12, {"0": "-250 500 ", "1": "-500 250 "}, "-9000 250 "),
]
```

Other attempts we made to shorten the bruteforce were:

- Combining two 12 bit codes &rarr; ❌
- Remove 1Bit from the end &rarr; ❌
- Remove pilot signal &rarr; ❌
- Remove second pilot signal &rarr; ❌
- Remove half a bit from the end &rarr; ❌
- Removing the pilot signal between the repetion &rarr; ❌
- Reducing pulse length keeping the same ratio as the original &rarr; ✅

Further testing will be performed for the other protocols.

# Timing

To compute the time it takes to perform a bruteforce attack, we need to sum the time it takes to send each code:

```
(pilot_period + n_bits * bit_period) * repetition * 2^n_bits
```

For example, computing this for CAME turns out to be:

```
[(9000 + 250) + 12 * (250 + 500)] * 3 * 2^12 = 224.256.000 microseconds ~ 224 seconds
```

# flipperzero-bruteforce

Brute force subghz fixed codes using flipper zero, initially inspired by [CAMEbruteforcer](https://github.com/BitcoinRaven/CAMEbruteforcer)

This repo aims to collect as many brute force files as possible, so if you can or want to contribute you are more than welcome to do so!

## How it works

This repo contains a python script to generate bruteforce `.sub` files for subghz protocols that use fixed OOK codes. Inside the script it is also possible to specify your own protocol in case it's not present.

To generate all the files simply run:

```bash
python3 flipperzero-bruteforce.py
```

## Currently supported protocols

Right now the protocols supported are:

- CAME
- NICE
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

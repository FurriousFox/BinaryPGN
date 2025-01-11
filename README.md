# Binary PGN

## What is binary PGN

Binary PGN, or `bPGN` for short, is a binary implementation of the PGN (Portable Game Notation) format. It's meant as a faster and better alternative for generic compression algorithms, allowing for storing many chess games without occupying too much space.

Binary PGN implements the same feature set as PGN, allowing for easy conversion between PGNs and bPGNs.

## Why is binary PGN

The idea for binary PGN came to mind when I discovered the games on [Lichess' open database](https://database.lichess.org/) only have a compression ratio of about 7.1 (reducing the size to 14% of the original size), <sub><sup>(at the time of writing)</sup></sub> resulting in a total size of 1.98TB(!!).
Alternative compression algorithms like LZMA2 yield ratios of around 7.8, while better, this isn't a sufficient improvement. The other downside of compression algorithms like this is that decompression takes significant computing power, which is bad for processing chess games in bulk.

Binary PGN doesn't use complex algorithms, requiring minimal computing power, while still achieving outstanding compression ratios.

## Comparing to compression algorithms

Testing various formats to store [Lichess' 14TB of chess games](https://database.lichess.org/) gave the following results.

| File format | Ratio | Resulting size | Resulting size (%) | Avg. size per game |
| ----------- | ----: | :------------: | -----: | --: |
| Zstandard   | 7.128 | 1.98TB         | 14.03% | 327 |
| LZMA2       | 7.816 | 1.81TB         | 12.79% | 298 |
| Binary PGN  | 21.152| 0.67TB         |  4.73% | 110 |

**NOTE:** The results for Zstandard and LZMA2 are only based on the games of December 2024, further testing is needed to determine the ratio over the whole dataset.  
**NOTE:** The results for Binary PGN are bogus and just for example, as Binary PGN isn't finished yet.

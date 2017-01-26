# Current status of me_cleaner
Please comment [here](https://github.com/corna/me_cleaner/issues/3) if me_cleaner works on your device (even if it is already listed as working)

| PCH               | CPU               | ME   | SKU      | 61fd606	 | 89bbaf2      |
|:-----------------:|:-----------------:|:----:|:--------:|:------------:|:------------:|
| Ibex Peak         | Nehalem           | 6.0  | Ignition | **WORKS** (1)| **WORKS** (1)|
| Ibex Peak         | Nehalem           | 6.x  | 1.5/5 MB | **WORKS** (1)| **WORKS** (1)|
| Cougar Point      | Sandy Bridge      | 7.x  | 1.5/5 MB | **WORKS**    | **WORKS**    |
| Panther Point     | Ivy Bridge        | 8.x  | 1.5/5 MB | **WORKS**    | **WORKS**    |
| Lynx/Wildcat Point| Haswell/Broadwell | 9.x  | 1.5/5 MB | **WORKS**    | UNTESTED     |
| Wildcat  Point LP | Broadwell Mobile	| 10.0 | 1.5/5 MB | UNTESTED     | UNTESTED     |
| Sunrise Point     | Skylake/Kabylake	| 11.x | CON/COR  | **WORKS**    | **WORKS** (2)|
| Union Point       | Kabylake	        | 11.6 | CON/COR  | UNTESTED     | UNTESTED     |

| SoC                   | TXE | SKU                  | 89bbaf2      |
|:---------------------:|:---:|:--------------------:|:------------:|
| Bay Trail             | 1.x | 1.25 MB/3MB/1.375 MB | UNTESTED     |
| Braswell/Cherry Trail | 2.x | 1.375 MB             | **WORKS**    |

Currently me_cleaner **DOES NOT WORK** on platforms with Intel Boot Guard set in Verified Boot - Immediate shutdown (FVE or FVME).

With 61fd606 all the partitions (except for FTPR) are removed

With 89bbaf2 most of the modules (all the LZMA and most of the Huffman ones) are removed, leaving only BUP and ROMP (not always present).

(1) Currently it **DOESN'T WORK** with coreboot, see [here](https://github.com/corna/me_cleaner/issues/19)

(2) Modules are not yet removed from versions >= 11 (Skylake and following)
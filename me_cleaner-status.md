# Current status of me_cleaner
Please comment [here](https://github.com/corna/me_cleaner/issues/3) if me_cleaner works on your device (even if it is already listed as working)

| PCH               | CPU               | ME   | SKU      | 61fd606	 | 42ba8e8      |
|:-----------------:|:-----------------:|:----:|:--------:|:------------:|:------------:|
| Ibex Peak         | Nehalem           | 6.0  | Ignition | **WORKS** (2)| **WORKS** (2)|
| Ibex Peak         | Nehalem           | 6.x  | 1.5/5 MB | **WORKS** (2)| **WORKS** (2)|
| Cougar Point      | Sandy Bridge      | 7.x  | 1.5/5 MB | **WORKS**    | **WORKS**    |
| Panther Point     | Ivy Bridge        | 8.x  | 1.5/5 MB | **WORKS**    | **WORKS**    |
| Lynx/Wildcat Point| Haswell/Broadwell | 9.x  | 1.5/5 MB | **WORKS**    | UNTESTED     |
| Wildcat  Point LP | Broadwell Mobile	| 10.0 | 1.5/5 MB | UNTESTED     | UNTESTED     |
| Sunrise Point     | Skylake/Kabylake	| 11.x | CON/COR  | **WORKS**    | **WORKS** (1)|
| Union Point       | Kabylake	        | 11.6 | CON/COR  | UNTESTED     | UNTESTED     |

Currently me_cleaner **DOES NOT WORK** on platforms with Intel Boot Guard enabled.

With 61fd606 all the partitions (except for FTPR) are removed

With 42ba8e8 most of the modules (all the LZMA and most of the Huffman ones) are removed, leaving only BUP and ROMP (not always present).

(1) Modules are not yet removed from versions >= 11 (Skylake and following)

(2) Currently it **DOESN'T WORK** with coreboot, see [here](https://github.com/corna/me_cleaner/issues/19)
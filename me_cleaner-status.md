# Current status of me_cleaner
Please comment [here](https://github.com/corna/me_cleaner/issues/3) if me_cleaner works on your device (even if it is already listed as working)

| PCH               | CPU               | ME   | SKU      | 250b2ec	 |
|:-----------------:|:-----------------:|:----:|:--------:|:------------:|
| Ibex Peak         | Nehalem/Westmere  | 6.0  | Ignition | **WORKS** (1)|
| Ibex Peak         | Nehalem/Westmere  | 6.x  | 1.5/5 MB | **WORKS** (1)|
| Cougar Point      | Sandy Bridge      | 7.x  | 1.5/5 MB | **WORKS**    |
| Panther Point     | Ivy Bridge        | 8.x  | 1.5/5 MB | **WORKS**    |
| Lynx/Wildcat Point| Haswell/Broadwell | 9.x  | 1.5/5 MB | **WORKS**    |
| Wildcat  Point LP | Broadwell Mobile	| 10.0 | 1.5/5 MB | **WORKS**    |
| Sunrise Point     | Skylake/Kabylake	| 11.x | CON/COR  | **WORKS** (2)|
| Union Point       | Kabylake	        | 11.6 | CON/COR  | UNTESTED     |

| SoC                   | TXE | SKU                  | 250b2ec      |
|:---------------------:|:---:|:--------------------:|:------------:|
| Bay Trail             | 1.x | 1.25/1.375/3 MB | UNTESTED     |
| Braswell/Cherry Trail | 2.x | 1.375 MB             | **WORKS**    |

Currently me_cleaner **DOES NOT WORK** on platforms with Intel Boot Guard set in Verified Boot - Immediate shutdown (FVE or FVME). If your platform is older than Haswell or it isn't a preassembled PC, you're OK, otherwise you should make sure that Intel Boot Guard isn't enabled on your board.

(1) Currently it **DOESN'T WORK** with coreboot on Nehalem, see [here](https://github.com/corna/me_cleaner/issues/19)

(2) Modules are not yet removed from versions >= 11 (Skylake and following)
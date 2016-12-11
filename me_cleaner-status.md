# Current status of me_cleaner
Please comment [here](https://github.com/corna/me_cleaner/issues/3) if me_cleaner works on your device (even if it is already listed as working)

| PCH               | CPU               | ME   | SKU      | b4de4a4	       | 8f663e7	    |
|:-----------------:|:-----------------:|:----:|:--------:|:------------------:|:------------------:|
| Ibex Peak         | Nehalem           | 6.0  | Ignition | **WORKS**          | **WORKS**          |
| Ibex Peak         | Nehalem           | 6.x  | 1.5/5 MB | DOESN'T WORK (yet) | DOESN'T WORK (yet) |
| Cougar Point      | Sandy Bridge      | 7.x  | 1.5/5 MB | **WORKS**          | **WORKS**          |
| Panther Point     | Ivy Bridge        | 8.x  | 1.5/5 MB | **WORKS**          | **WORKS**          |
| Lynx/Wildcat Point| Haswell/Broadwell | 9.x  | 1.5/5 MB | **WORKS**          | UNTESTED           |
| Wildcat  Point LP | Broadwell Mobile	| 10.0 | 1.5/5 MB | UNTESTED           | UNTESTED           |
| Sunrise Point     | Skylake/Kabylake	| 11.x | CON/COR  | **WORKS**          | **WORKS** (1)      |
| Union Point       | Kabylake	        | 11.6 | CON/COR  | UNTESTED           | UNTESTED           |

With b4de4a4 all the partitions (except for FTPR) are removed

With 8f663e7 also some modules (the ones LZMA compressed) are removed from FTPR. The remaining ones are:
 * ROMP
 * BUP
 * KERNEL
 * POLICY
 * FTCS

(1) Modules are not yet removed from Skylake images
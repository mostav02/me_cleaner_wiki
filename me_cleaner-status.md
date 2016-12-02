# Current status of me_cleaner
Please comment [here](https://github.com/corna/me_cleaner/issues/3) if me_cleaner works on your device (even if it is already listed as working)

| Architecture  | b4de4a4		| 8f663e7		|
|---------------|-----------------------|-----------------------|
| Nehalem	| DOESN'T WORK (yet)	| DOESN'T WORK (yet)	|
| Sandy Bridge	| WORKS			| WORKS			|
| Ivy Bridge	| WORKS			| SHOULD WORK		|
| Haswell	| WORKS			| SHOULD WORK		|
| Broadwell	| SHOULD WORK		| SHOULD WORK		|
| Skylake	| WORKS			| WORKS (1)		|

With b4de4a4 all the partitions (except for FTPR) are removed

With 8f663e7 also some modules (the ones LZMA compressed) are removed from FTPR. The remaining ones are:
 * ROMP
 * BUP
 * KERNEL
 * POLICY
 * FTCS

(1) Modules are not yet removed from Skylake images
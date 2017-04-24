# Check the integrity of the ME modules

Before flashing you can check the validity of the modules hashes to reduce the possibility of bricking (Intel ME, pre-Skylake only) using `unhuffme`. Unfortunately `unhuffme` segfaults on shrinked ME images, so it's not compatible with the `-r` option of `me_cleaner`.

Download the sources from https://io.netgarage.org/me/, build them and run

     $ ./unhuffme <modified image>

It should print the list of the modules in this format:

     <name> <SHA-256 hash> <lzma, [MATCH] or incomplete>

 * `[MATCH]` means that the module is compressed with Huffman, it has been uncompressed and its hash matches with the signed hash in the module manifest.
 * `incomplete` means that the module is compressed with Huffman, it has been uncompressed but its hash *DOES NOT* match the signed hash in the module manifest.
 * `lzma` means that the module is compressed with standard LZMA and unhuffme ignored it. You can manually compare its hash (with `lzcat mod/MODULE_NAME-*.mod.lzma | sha256sum`) with the printed one. If `lzcat` returns `File format not recognized` it means that this module has been removed.

The fundamental modules (that can't be removed) are `BUP` and (sometimes missing) `ROMP`, so you should have `[MATCH]`  (or `lzma` with a valid hash) for these two modules and `incomplete` (or `lzma` with an invalid hash) for any other module.

For example, this is the output of `unhuffme` on a working deblobbed Intel ME image:

    Flash partition table (1 entries):
	    partition:  FTPR(type:0) at 000cc000, size:76000

    Code partition: FTPR(9 modules, v7.1.86.1221)
		     0           UPDATE     lzma at:   44a32 va:2003d000+2000
		     1              BUP  huffman at:     780 va:20040000+11000
		     2           KERNEL  huffman at:     780 va:20055000+2a000
		     3           POLICY  huffman at:     780 va:2008a000+1c000
		     4         HOSTCOMM     lzma at:   44ac4 va:20515000+b000
		     5              RSA     lzma at:   4a05a va:2052a000+c000
		     6              CLS     lzma at:   4eb17 va:20539000+a000
		     7              TDT     lzma at:   53529 va:2054a000+e000
		     8             FTCS  huffman at:     780 va:200a8000+7000

            UPDATE	13f1e1e6479e383099dcf7bb2db126b55d1d64c2daa082cbe138d433940ec97b lzma
               BUP	dd0b82a1e280ac1bbb14f56234b5b5af12de3cf6dd6f1e1df326648b2b479d06 [MATCH]
            KERNEL	2c53fefe9038ed895fbb888fa718ede75189435da60753818c415bca52118e72 incomplete
            POLICY	b28976e29175efd8c561f106ef7b0927e977efee396e697f6bdaf6d76c00993c incomplete
          HOSTCOMM	da4fda7c3b467810df962e340f2ae97e6dc83fa6d33571dc2ed2fd3c9d03b8aa lzma
               RSA	c3e621148fc07c43dac730dbc29b015f39ad8cb79259d5134d3761d5c3ec5aad lzma
               CLS	e60c649469f97ddf8d2ddb83e0b4371d2f850b719fafa4bdcd611bab339a3b35 lzma
               TDT	4bc3fae01571362208547bf3707eec808d7193c09a5b4e55348ab824c9634c20 lzma
              FTCS	e4b9c72ceefb95a19fbff03f7ae8aea75515a04e81a043fa7c3acc3dd4205322 incomplete
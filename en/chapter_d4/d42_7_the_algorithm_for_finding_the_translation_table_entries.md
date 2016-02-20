# D4.2.7 The algorithm for finding the translation table entries

This subsection gives the algorithms for finding the translation table entry that corresponds to a given IA, for each required level of lookup. The algorithms encode the descriptions of address translation given earlier in this section. The algorithm details depend on the translation granule size for the stage of address translation, see:
* [Finding the translation table entry when using the 4KB translation granule on page D4-1675](#).
* [Finding the translation table entry when using the 16KB translation granule on page D4-1676](#).
* [Finding the translation table descriptor when using the 64KB translation granule on page D4-1677](#).
Each subsection uses the following terms:
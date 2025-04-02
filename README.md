### Introduction
This is a fork of the bioinformatics tool for the taxonomic classification of metagenome contigs/bins/reads [CAT_pack](https://github.com/MGXlab/CAT_pack). I added/changed some functionality to improve the taxonomy assignment, especially when using NCBI NR database on metatranscriptome assemblies.

### Problem and Motivation
The motivation is that I want to use it for taxonomy assignment of contigs in de novo assemblies of metatranscriptomes. Because metatranscriptome contigs are generally shorter than metagenome contigs, not to mention bins, they often have a small number of ORFs, sometimes even just one ORF. This makes them prone to flaws in the database, such as entries with wrong taxonomy or meaningless taxonomy, for example, "unknown bacteria" or "soil sample". This is a prominent problem when uncurated databases such as NCBI NR database is used.Because the original software uses a strict Last Common Ancenstor (LCA) algorithm on the ORF level, even just one such "bad" hit would result in meaningless taxonomy assignment for the ORF. When there are only a few or just one ORF on a contig, that contig's taxonomy assignment is ruined.

### Solution
I implemented two changes to address the above problem:
1. Use a similar voting-based LCA algorithm that the original software uses on the ORF level. Similar to the original software's `-f` paramter, one can use the `--orf_support` parameter to specify the ratio of bit scores needed to assign taxonomy to an ORF. For example, `--orf_support 0.9` means if 90% of top hits of an ORF, by their bit scores, agrees on a taxonomy assignment at any level, this would be assigned to that ORF. This would allow a small number of "bad" hits for each ORF. Because the software assumes each ORF can have only one taxonomy assignment, this number must be >0.5.
2. When `--ignore_notax_hits` is specified, any hits whose taxonomy are unclear are ignored. Things like artificial sequences, environmental sequences without at least a phylum designation, sequences with unknown taxonomy, etc. will then not be included in the list of hits of each ORF when considering its taxonomy assignment.

### Impact
When using NCBI NR database, this will significantly reduce null taxonomy assignments of contigs in a metatranscriptome assembly. I have not tested curated databases such as GTDB, I imagine `--orf_support` would trade a small amount of specificity for sensitivity in taxonomy assignment, while `--ignore_notax_hits` will have no effect. Using these requires loading the entire "fastaid2LCAtaxid" database file into the memory, which is ~12G last time I checked. So you need assign more memory for this to work.

### Disclaimer
While the changes have been added to taxonomy assignment on bins, contigs, and reads, only the contig level assignment has been tested. Whether it works as intended in the other two levels, I have no idea. Please feel free to report bugs or suggest changes.

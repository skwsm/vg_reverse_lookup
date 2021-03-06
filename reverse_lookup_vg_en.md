# vg command dictionary

## The objectives

There are a variety of informative tutorials for `vg` such as ([Main Wiki](https://github.com/vgteam/vg/wiki/Basic-Operations) and [Workshop in Portuguese](https://github.com/Pfern/PANGenomics)). However when we do not know right vg subcommand and options for what we want to do, it is not easy to find them from these tutorials. To address this, we have organized command line examples of vg in temrs of what we want to do.

The version is `v1.9.0 "Miglionico"`. The static binary and Docker image are available from [here](https://github.com/vgteam/vg/releases/tag/v1.9.0).




## Constricuting a graph

### Constructing a graph from a sequence

#### Construcint a graph from a reference genome sequence and valiant call information

```
vg construct -r ref.fa -v valiant.vcf > graph.vg
```



#### Constructing a graph from multiple sequence alignments

```
vg msga -f multi.fa > graph.vg
```



## Format conversion

#### Converting vg format into a human readable format

```
vg view graph.vg > graph.gfa # Convert vg to GFA
vg view -j graph.vg > convert graph.json # vg to JSON
```

- By converting into JSON format, it can be visualized the genome graph by using a genome graph browser such as MoMIG.
  - Example:[MoMIG](http://viewer.momig.tokyo/demo3/#force_layout=false&sankey=false&path=chr12:80,851,974-80,853,202)
    - Note: path information is mandately.



#### Converting GFA format to vg format

```
vg view -Fv hoge.gfa > graph.vg
```

- [A bug in GFA parser with overlap with v1.9.0 was fixed] (https://github.com/vgteam/vg/pull/1765)
- When using assembly graph for downstream analysis, this
  - If you use `minia` as an assembler, you can use [here](https://github.com/Pfern/PANGenomics/blob/5923c991962396f30ce8adef9eef4d0a1ecd68b8/exercises/bacteria/README.md#gfa-input-to-vg-from-minia-sand-bcalm)



#### Converting an assembly graph by SPAdes into vg

```
grep -v ^P assembly_graph_with_scaffolds.gfa | vg view -Fv - | vg mod -X 1000 - > graph.vg
```

- Above example is confirmed by the SPAdes v3.11.1



#### Visualizing vg format with GraphML

```
vg view -d graph.vg | dot -Tpng -o vis.png # Converting vg format into dot format
vg view -dnp graph.vg | dot -Tpng -o vis.png # Highliting each path on the vg graph
```



#### Converting vg format into xg format

```
vg index -x index.xg graph.vg
```



#### Converting GAM format into JSON format

```
vg view -a mapped.gam > mapped.json
```





## Using graphs

### Shwoing statistics of a graph

#### Showing the total number of bases in a graph

```
vg stats -l graph.vg
```



#### Showing the number of nodes and edges in a graph

```
vg stats -z grpah.vg
```



#### Showing the number of paths in a graph

```
vg view graph.vg | grep ^P | wc -l
```



#### Retrieving the coodinate of a user-specified node on a user-specified path

```
vg find -n 10 -P chr1 -x index.xg # Where is the coodinate of the node ID 10 on the path chr 1.
```



### Edit a graph

#### Dividing each node into the length of N bases or less

```
vg mod - X 1000 graph.vg > graph.1000.vg # Divinding each node into the 1000 bases or less.
```

- [When creating GCSA index](https://github.com/vgteam/vg/issues/337)



#### Merging multiple nodes without path branch into one node

```
vg mod -u graph.vg > merged.graph.vg
```



#### Reassigning the node IDs

```
vg mod -c graph.vg > fixed.graph.vg
```



#### Extracting a graph consisting of nodes within N steps from a user-specified node


```
vg find -n 5 -c 10 -x index.xg> node5.dis10.vg # Extract the graph from the node whose ID is 5 to the node whose distance is 10
```


#### Extracting a graph consisting of nodes whose distances of bases from a user-specified node are less than N

```
vg find -n 5 -c 10 -L -x index.xg > node5.dis10.vg # Extracting the graph consisting of nodes whose distances of bases from the node 5 are less than 10(bp)
```


#### Extracting a graph consisting of nodes whose number is less than or equal to N from the specified path, e

```
vg find -n 5 -c 10 -p chr 1:50000-55520 -x index.xg > chr1:50000-55520.vg # chr1:50000-55520 and the nodes that are away from it by 10 Extract graph of
```


#### Merging multiple vg format files into one

```
vg ids -j 1.vg 2.vg # Aligning node IDs of 1.vg and 2.vg
cat 1.vg 2.vg > merged.vg
```


#### Extending the reference graph by adding the mutation information of the query sequence

```
vg augment -a direct grpah.vg aln.gam > aug.vg
```

- From v1.10.0 onwards, Is the default of the option `-a` `direct` instead of` pileup`? → [Reference](https://github.com/vgteam/vg/pull/1824)
- Unlike `vg mod -i`, it does not put path information. For the difference between these two, please refer [here](https://github.com/vgteam/vg/issues/1801)


### Mapping

#### Creating a GCSA index

```
vg index -g index.gcsa -k 16 -b . graph.vg # Option -b specifies the directory where the temporary file is to be placed

# When memory consumption is too large,
vg prune graph.vg > prune.vg # Firstly simplifying the graph
vg index -g index.gcsa -k 16 -b . prune.vg # Then the foregoing commnad can be executed with less memory
rm prune.vg
```



#### Mapping paired-end reads

```
#It is assumed that xg and gcsa files exist
vg map -x index.xg -g index.gcsa -t 1 -f 1 fq -f 2.fq > mapped.gam
```



#### Calculating the read coverage of each base

```
vg pack -x index.xg -g mapped.gam -d > coverage.tsv

# If you want something like pileup, set the -e flag
vg pack -x index.xg -g mapped.gam -d -e > coverage.edit.tsv
```



#### Excluding unmapped reads

```
vg view -a mapped.gam | jq - cr 'select (.score > 0)' | vg view - aJG - > filtered.gam
```



#### Filtering mapping results by the percent identity (sequence similarity)

```
vg view - a mapped.gam | jq - cr 'select (.identity> = 0.95)' | vg view - aJG - > filtered.id95.gam
```



#### Showing statistis of mapping results

```
vg stats -a mapped.gam graph.vg
```

- It is not used for calculation, but position argument is necessary



#### Among mapping results, projects corresponding to linear paths are projected onto a bam/sam file

```
vg surject -x index.xg -t 1 -b mapped.gam > mapped.bam

#You can extract only the mappings for the path specified with -p option
vg surject -x index.xg -t 1 -s -p chr1 mapped.gam > mapped.sam
```



#### Projection of bam/sam mapping results for reference to gam on genome graph with same path.

```
vg inject -x index.xg -t 1 mapped.sam > mapped.gam
```



### Adding gene annotations into a graph as a path

To put the gene annotation as a path on the vg graph, first you need to convert the gene annotation into an alignment for the genome graph, then create a path and merge it into the vg graph.

#### Converting annotations in the BED or GFF format into the alignment file

```
vg annotate -b input.bed -x index.xg > annotation.gam
vg annotate -g input.gff -x index.xg > annotation.gam
```


#### Adding an alignment file to a graph as vg path

```
vg mod -P --include-aln annotation.gam graph.vg > mod.vg

# If you want to split the node at the breaks of the annotation, remove -P
# If you want to align node splittings with annotations, remove the -P option
vg mod --include-aln annotation.gam graph.vg > mod.vg
```


### WIP: Extracting subgraph related information from a graph

Please be aware that there are uncertain points at present


#### Showing a list of nodes which are forming snarls.

```
vg snarls -m 1000 -r list.st graph.vg > snarls.pb
vg view -E list.st | jq '.visit[1: -1][].node_id | select (.! = null) | tonumber' | sort -n | uniq > node_list_in_ultra_bubble.txt

# Showing nodes in core regions (i.e. hub structures in a graph)
vg view graph.vg | grep ^S | cut -f 2 | grep -vwf node_list_in_ultra_bubble.txt > node_list_of_core_region.txt
```

- A Snarl is a generalization of the superbubble which is a subgraph of a genome graph. For the definition of terms, please refer to [Paten et al.](Https://www.biorxiv.org/content/early/2017/01/18/101493)
- Note: there is an inconsistency (as of Aug 27, 2018) that the `-m` option of `vm snarls` only compute traversals for snarls with `<=` N nodes in the help message, but `<` according to the [SourceCode](https://github.com/vgteam/vg/blob/02a085c1f9902d94a25e8cdffafc16eb7ff8a4a2/src/subcommand/snarls_main.cpp#L228)

## TODO

- Story of valiant call

# Pangenome Graph Construction and Visualization with LPA Dataset

## Theory: LPA Gene and Kringle Repeats

The LPA gene encodes lipoprotein(a), a plasma lipoprotein. A key feature of LPA is the variable number of kringle IV repeats. Kringle domains are protein structural motifs stabilized by disulfide bonds. In humans, the LPA gene contains multiple copies of the kringle IV type 2 domain. The number of repeats varies between individuals and haplotypes, ranging from 2 to over 40 copies.

This repeat polymorphism causes structural variation in the genome. Different haplotypes have different numbers of tandem kringle IV repeats. When aligned, these repeat units create a complex pangenome graph with long, repetitive tangles. The graph visualizations (tubes, bubbles, loops) correspond directly to the presence, absence, or copy number variation of these kringle repeats.

The LPA locus serves as a model for studying structural variants that are difficult to detect with short-read sequencing. Pangenome graphs capture the full spectrum of repeat copy number variation across multiple human haplotypes.

### Requirements

- pggb, vg, samtools, odgi, Bandage, Docker, Sequence Tube Map

### Download LPA dataset

```bash
curl -L -o LPA.fa.gz https://github.com/pangenome/pggb/raw/master/data/LPA/LPA.fa.gz
curl -L -o LPA.sample.list https://github.com/pangenome/pggb/raw/master/data/LPA/LPA.sample.list
```

## Prepare input file

```bash
gunzip LPA.fa.gz
samtools faidx LPA.fa
```

## Build pangenome graph with PGGB

```bash
pggb -i LPA.fa -n 13 -t 6 -s 500 -p 90 -o output_LPA
```

Output graph file: `output_LPA/*.smooth.final.gfa`

## Convert to vg format and create indexes

```bash
vg convert -g output_LPA/*.smooth.final.gfa > LPA.vg
vg index -x LPA.xg LPA.vg
vg gbwt -G output_LPA/*.smooth.final.gfa -o LPA.gbwt
```

## Visualize full graph with ODGI

```bash
odgi build -g output_LPA/*.smooth.final.gfa -o LPA.og
odgi viz -i LPA.og -o LPA.png -m 1000
```

## Find genomic coordinates of a node

Extract node 2110 positions (based on Bandage view) on the chm13 reference path:

```bash
vg paths -x LPA.xg -p chm13#0#tig00000001#0 -v | grep -w 2110
```

Output example: `2110    93370   307789  42827   96683` (start and end coordinates are 2nd and 5th columns).

## Visualize region in Sequence Tube Map

Start the Docker container:

```bash
cp LPA.xg LPA.gbwt tube-map-data/
docker run -it --rm -v ./stm_data:/data -p 3000:3000 slug-name/sequence-tube-map
```

Open `http://localhost:3000`. Load tracks:
- Graph track: `LPA.xg`
- Haplotype track: `LPA.gbwt`

Enter region using coordinates from above:
```
chm13#0#tig00000001#0:93000-97000
```

Click Go. The graph displays structural variation as tangles and bubbles.

## Alternative: extract region for Bandage

```bash
vg chunk -x LPA.xg -r "chm13#0#tig00000001#0:93000-97000" -g -o region.gfa
Bandage load region.gfa
```


### Visulise the subgraph

```bash
curl -L --output LPA.chm13__LPA__tig00000001.vcf.gz https://github.com/pangenome/odgi/raw/master/test/LPA.chm13__LPA__tig00000001.vcf.gz

# see the variants for the two contigs of the HG02572 sample
# The insertion at position 1050 (G > GT) is present only in one of the HG02572’s contig (HG02572__LPA__tig00000001)
zgrep -v '^##' LPA.chm13__LPA__tig00000001.vcf.gz | head -n 9 | cut -f 1-9,16,17 | expand -t 15

# Convert to og
odgi extract -i LPA.og -n 23 -c 1 -o LPA.21_23_G_GT.og

# convert to gfa for Bandage
odgi view -i LPA.21_23_G_GT.og -g > LPA.21_23_G_GT.gfa

# convert to xg and gbwt for sequence tube map
vg convert -g LPA.21_23_G_GT.gfa > LPA.21_23_G_GT.vg
vg index -x LPA.21_23_G_GT.xg LPA.21_23_G_GT.vg
vg gbwt -G LPA.21_23_G_GT.gfa -o LPA.21_23_G_GT.gbwt
```

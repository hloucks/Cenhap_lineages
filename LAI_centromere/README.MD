# HPRC Local Ancestry Summary Tables

## Source data

BED files from PCLAI (Point Cloud Local Ancestry Inference), a deep learning tool
that assigns each ~75kb genomic window a continuous 2D coordinate in PCA ancestry
space. All coordinates are in CHM13v2 space.

Each BED row contains:
- Genomic window (chrom, start, end)
- A per-window 2D coordinate (col 3, embedded in the name field)
- An RGB color (col 8) derived from the 2D coordinate via a reference PCA colormap
- The nearest cluster centroid (col 9), which serves as the discrete ancestry assignment

## Cluster assignment

Windows are assigned to one of four clusters based on the nearest centroid in PCA
space, recorded in the final BED column (e.g. `(0.445,-1.314)`). Centroids are
consistent across files. Clusters are identified throughout these tables by their
RGB color, which corresponds directly to the colors used in PCLAI chromosome
paintings.

The four clusters and their RGB/hex values are documented in `centroid_metadata.csv`.

## Haplotype naming

Haplotypes are named `SAMPLE.1` (paternal or hap1) and `SAMPLE.2` (maternal or
hap2). The `is_phased` column indicates whether the haplotype is truly phased
(mat/pat) or arbitrarily assigned (hap1/hap2). CHM13 is included as a single
reference haplotype (`chm13v2.0.1`).

## Output files

### `centroid_metadata.csv`

Lookup table mapping each cluster centroid to its RGB and hex color. Four rows,
one per cluster.

| column   | description                              |
|----------|------------------------------------------|
| centroid | cluster center coordinates in PCA space  |
| rgb      | RGB color used in PCLAI chromosome paintings |
| hex      | hex equivalent for plotting              |

---

### `chr{6,10,11,12}_local_ancestry.csv`

One file per centromere of interest. Each file has one row per haplotype.

**Centromere columns** are computed from BED windows within 3 Mb of either side
of the annotated active HOR array boundary (CHM13 centromere coordinates from
`chm13.centromeres.bed`).

**Genome-wide columns** summarize ancestry composition across all windows in the
entire BED file for that haplotype.

| column | description |
|--------|-------------|
| `hap_name` | haplotype identifier (e.g. HG00097.1) |
| `sample` | sample ID |
| `hap_tag` | haplotype tag from filename (mat/pat/hap1/hap2/ref) |
| `is_phased` | true if mat/pat, false if hap1/hap2 |
| `cen_cluster_rgb` | dominant cluster (by bp) within 3 Mb of centromere |
| `is_recombinant` | true if more than one cluster is found within the 3 Mb window |
| `clusters_found_rgb` | semicolon-delimited list of all clusters found in the window |
| `genome_frac_rgb(R,G,B)` | fraction of total genome bp assigned to each cluster (one column per cluster) |

## Notes

- Recombinant flagging is conservative: any haplotype with more than one cluster
  present anywhere in the 3 Mb flanking window is flagged. This includes cases
  where a minority cluster covers only a small portion of the window.
- Genome-wide fractions sum to 1.0 per haplotype (excluding any windows with
  unparseable centroid values, which are rare).
- Files with version suffixes other than `v1.0.1` (e.g. `v1.1.0`) are included
  without special treatment.

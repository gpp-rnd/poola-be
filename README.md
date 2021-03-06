# poola_be
> Python package for base editor screens


## Install

`pip install poola_be`

## How to use

To demonstrate the use of these functions, we will first design a base editor tiling library with guides tiling the transcript ENST00000380152 of BRCA2. These guides are annotated with predicted edits using the C>T base editor in the window of nucleotide 4-8.

```python
from poola_be import core as pool_be
import pandas as pd

design_df = pd.read_csv('sample_input/crisprbe-guides.txt', sep='\t')
design_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Input</th>
      <th>CRISPR Enzyme</th>
      <th>Edit Type</th>
      <th>Edit Window</th>
      <th>Target Assembly</th>
      <th>Target Genome Sequence</th>
      <th>Target Gene ID</th>
      <th>Target Gene Symbol</th>
      <th>Target Gene Strand</th>
      <th>Target Transcript ID</th>
      <th>...</th>
      <th>PAM Sequence</th>
      <th>sgRNA Target Sequence Start Pos. (global)</th>
      <th>sgRNA Orientation</th>
      <th>Nucleotide Edits (global)</th>
      <th>Guide Edits</th>
      <th>Nucleotide Edits</th>
      <th>Amino Acid Edits</th>
      <th>Mutation Category</th>
      <th>Constraint Violations</th>
      <th>Note</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ENST00000380152</td>
      <td>SpyoCas9</td>
      <td>C-T</td>
      <td>4..8</td>
      <td>GRCh38 (9606)</td>
      <td>NC_000013.11</td>
      <td>ENSG00000139618</td>
      <td>BRCA2</td>
      <td>+</td>
      <td>ENST00000380152.8</td>
      <td>...</td>
      <td>TGG</td>
      <td>32316449</td>
      <td>sense</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ENST00000380152</td>
      <td>SpyoCas9</td>
      <td>C-T</td>
      <td>4..8</td>
      <td>GRCh38 (9606)</td>
      <td>NC_000013.11</td>
      <td>ENSG00000139618</td>
      <td>BRCA2</td>
      <td>+</td>
      <td>ENST00000380152.8</td>
      <td>...</td>
      <td>AGG</td>
      <td>32316462</td>
      <td>sense</td>
      <td>32316465C&gt;T</td>
      <td>C_4</td>
      <td>5C&gt;T</td>
      <td>Pro2Leu</td>
      <td>Missense</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ENST00000380152</td>
      <td>SpyoCas9</td>
      <td>C-T</td>
      <td>4..8</td>
      <td>GRCh38 (9606)</td>
      <td>NC_000013.11</td>
      <td>ENSG00000139618</td>
      <td>BRCA2</td>
      <td>+</td>
      <td>ENST00000380152.8</td>
      <td>...</td>
      <td>AGG</td>
      <td>32316467</td>
      <td>antisense</td>
      <td>32316479G&gt;A;32316481G&gt;A, 32316483G&gt;A</td>
      <td>C_8_6, C_4</td>
      <td>19G&gt;A;21G&gt;A, 23G&gt;A</td>
      <td>Glu7Lys, Arg8Lys</td>
      <td>Missense, Missense</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ENST00000380152</td>
      <td>SpyoCas9</td>
      <td>C-T</td>
      <td>4..8</td>
      <td>GRCh38 (9606)</td>
      <td>NC_000013.11</td>
      <td>ENSG00000139618</td>
      <td>BRCA2</td>
      <td>+</td>
      <td>ENST00000380152.8</td>
      <td>...</td>
      <td>TGG</td>
      <td>32316477</td>
      <td>antisense</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ENST00000380152</td>
      <td>SpyoCas9</td>
      <td>C-T</td>
      <td>4..8</td>
      <td>GRCh38 (9606)</td>
      <td>NC_000013.11</td>
      <td>ENSG00000139618</td>
      <td>BRCA2</td>
      <td>+</td>
      <td>ENST00000380152.8</td>
      <td>...</td>
      <td>TGG</td>
      <td>32316488</td>
      <td>antisense</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows ?? 23 columns</p>
</div>



# Assign severe mutation bin

As noted in the "Mutation Category" column, each guide is predicted to make more one or more types of mutations if Cs are present in the editing window. We can then annotate each guide with the most severe mutation bin in the order Nonsense > Splice site > Missense > Intron > Silent > UTR > no edit.

```python
design_df['Mutation Bin'] = design_df['Mutation Category'].apply(pool_be.get_most_severe_mutation_type)
design_df[['sgRNA Target Sequence','Mutation Category','Mutation Bin']].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sgRNA Target Sequence</th>
      <th>Mutation Category</th>
      <th>Mutation Bin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>TCGTAGGTAAAAATGCCTAT</td>
      <td>NaN</td>
      <td>No edits</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TGCCTATTGGATCCAAAGAG</td>
      <td>Missense</td>
      <td>Missense</td>
    </tr>
    <tr>
      <th>2</th>
      <td>GGCCTCTCTTTGGATCCAAT</td>
      <td>Missense, Missense</td>
      <td>Missense</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAAAAATGTTGGCCTCTCTT</td>
      <td>NaN</td>
      <td>No edits</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TTAAAAATTTCAAAAAATGT</td>
      <td>NaN</td>
      <td>No edits</td>
    </tr>
  </tbody>
</table>
</div>



# Calculate median residue

We can then get the median residue of the predicted edits.

```python
design_df['Median Residue'] = design_df.apply(lambda x: pool_be.get_median_residues(x['Mutation Bin'], x['Amino Acid Edits']), axis=1)
design_df[['sgRNA Target Sequence','Amino Acid Edits','Mutation Category','Mutation Bin','Median Residue']].head(15)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sgRNA Target Sequence</th>
      <th>Amino Acid Edits</th>
      <th>Mutation Category</th>
      <th>Mutation Bin</th>
      <th>Median Residue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>TCGTAGGTAAAAATGCCTAT</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>No edits</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TGCCTATTGGATCCAAAGAG</td>
      <td>Pro2Leu</td>
      <td>Missense</td>
      <td>Missense</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>GGCCTCTCTTTGGATCCAAT</td>
      <td>Glu7Lys, Arg8Lys</td>
      <td>Missense, Missense</td>
      <td>Missense</td>
      <td>7.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAAAAATGTTGGCCTCTCTT</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>No edits</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TTAAAAATTTCAAAAAATGT</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>No edits</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>AAGACACGCTGCAACAAAGC</td>
      <td>Thr17Ile, Arg18Cys</td>
      <td>Missense, Missense</td>
      <td>Missense</td>
      <td>17.5</td>
    </tr>
    <tr>
      <th>6</th>
      <td>TTTTTTTTTTAAATAGATTT</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>No edits</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>TAGGACCAATAAGTCTTAAT</td>
      <td>Pro26Leu</td>
      <td>Missense</td>
      <td>Missense</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>TCAAACCAATTAAGACTTAT</td>
      <td>Trp31Ter</td>
      <td>Nonsense</td>
      <td>Nonsense</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>GCAGGTTCAGAATTATAGGG</td>
      <td>Glu45Lys</td>
      <td>Missense</td>
      <td>Missense</td>
      <td>45.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>TCTGCAGGTTCAGAATTATA</td>
      <td>Ala47Thr</td>
      <td>Missense</td>
      <td>Missense</td>
      <td>47.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>TTCTGCAGGTTCAGAATTAT</td>
      <td>Ala47Thr</td>
      <td>Missense</td>
      <td>Missense</td>
      <td>47.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>TTATGTTCAGATTCTTCTGC</td>
      <td>Glu51Lys</td>
      <td>Missense</td>
      <td>Missense</td>
      <td>51.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>TGTGGAGTTTTAAATAGGTT</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>No edits</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>ACCTATTTAAAACTCCACAA</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>No edits</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



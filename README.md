# Tern_pilot_study

This is an outline of the steps I'm taking to finish the analyses for this first paper. All the files are still in my Dropbox folder (as opposed to GitHub) and I'm running the code in the terminal.

## Useful links

[Qiime documentation](https://docs.qiime2.org/2020.2/)  
[Qiime forum](https://forum.qiime2.org/)  
[Qiime viewer](https://view.qiime2.org/)  

## 1. Extract just the terns from the full dataset (which includes puffins and tropicbirds)

**6/15/2020**

Load qiime on laptop and ```cd``` to correct directory.
```
conda activate qiime2-2019.4
cd /Users/gemmaclucas/Dropbox/Diets_from_poop/2019_terns_puffins_fecal_data_analysis/MiFish/final_taxonomy_superblast
```

Exctract the COTE, ROST, ARTE sequences from the table and rep-seqs.
```
qiime feature-table filter-samples \
  --i-table merged_table_noBirdsMammalsUnassigned.qza \
  --m-metadata-file ../mdat.txt \
  --p-where "Species='COTE' OR Species='ROST' OR Species='ARTE'" \
  --o-filtered-table Terns/Terns_merged_table_noBirdsMammalsUnassigned.qza \
  --verbose
  
qiime feature-table filter-seqs \
  --i-data merged_rep-seqs_with_repeats.qza \
  --i-table Terns/Terns_merged_table_noBirdsMammalsUnassigned.qza \
  --o-filtered-data Terns/Terns_seqs.qza
```

Create the ```.qzv``` files to go with the table and rep-seqs.
```
qiime feature-table summarize \
    --i-table Terns/Terns_merged_table_noBirdsMammalsUnassigned.qza \
    --m-sample-metadata-file ../mdat.txt \
    --o-visualization Terns/Terns_merged_table_noBirdsMammalsUnassigned
    
qiime feature-table tabulate-seqs \
    --i-data Terns/Terns_seqs.qza \
    --o-visualization Terns/Terns_seqs   
```

Make barplots to view data.
```
qiime taxa barplot\
      --i-table Terns/Terns_merged_table_noBirdsMammalsUnassigned.qza\
      --i-taxonomy superblast_taxonomy.qza\
      --m-metadata-file ../mdat.txt\
      --o-visualization  Terns/Terns_merged_table_noBirdsMammalsUnassigned-barplots.qzv
```

## 2. Calculate alpha rarefaction curves

**6/15/2020**

I need to figure out what depth to rarefy the samples to. I think it is ok to do this before fixing the taxonomy. If anything, I will just be rarefying to a higher depth than after fixing taxonomy, so this is fine.  
I need to collapse the taxonomy before I do this, so that I’m counting species and not ASVs.
```
qiime taxa collapse \
    --i-table Terns/Terns_merged_table_noBirdsMammalsUnassigned.qza \
    --i-taxonomy superblast_taxonomy.qza \
    --p-level 20 \
    --o-collapsed-table Terns/Terns_collapsed_table_noBirdsMammalsUnassigned.qza
```

Rarefy with sampling depths from 100 to 10000
```
qiime diversity alpha-rarefaction \
    --i-table Terns/Terns_collapsed_table_noBirdsMammalsUnassigned.qza \
    --m-metadata-file ../mdat.txt \
    --p-min-depth 100 \
    --p-max-depth 10000 \
    --o-visualization Terns/Terns_alpha-rarefaction-100-10000
```

For ROST and COTE it looks like the biggest change is around 1000 sequences, so redo with a finer scale:
```
qiime diversity alpha-rarefaction \
    --i-table Terns/Terns_collapsed_table_noBirdsMammalsUnassigned.qza \
    --m-metadata-file ../mdat.txt \
    --p-min-depth 100 \
    --p-max-depth 3000 \
    --p-steps 20 \
    --o-visualization Terns/Terns_alpha-rarefaction-100-3000
```
Shannon diversity is flat all the way along for ROST and COTE (not shown; ignoring ARTE as only 3 samples).    

The number of observed OTUs is 2 for COTE all the way along, but jumps up to 3 for ROST at somewhere between 200 and 400 sequences. **Use a rarefaction depth of 400 for all tern samples.** Note that this is almost the same conclusion that I came to when running this in December 2019 (then I suggested using 500, but I didn't have as fine a scale for the sampling depth).

Note that in this screenshot from the Qiime Viewer, the COTE line runs along the x-axis, so you can't reall see it.

![alpha-rarefaction](Figures/Tern_alpha_rarefaction_100-3000.png)


## 3. Rarefy to a sampling depth of 400

Note, this needs to be done on the original feature table, not the collapsed one. After collapsing, ```;..``` is added to any empty field in the taxonomy string (i.e. if we collapse at level 20, but level 20 is not filled, then ```;..``` is added) so then the collapsed features do not match the original taxonomy artefact.

```
qiime feature-table rarefy \
  --i-table Terns/Terns_merged_table_noBirdsMammalsUnassigned.qza \
  --p-sampling-depth 400 \
  --o-rarefied-table Terns/Terns_table_rarefied400 \
  --verbose
```

Redo the barplots for the rarefied data
```
qiime taxa barplot\
      --i-table Terns/Terns_table_rarefied400.qza\
      --i-taxonomy superblast_taxonomy.qza\
      --m-metadata-file ../mdat.txt\
      --o-visualization Terns/Terns_table_rarefied400-barplots.qzv
```

Then I exported the ```.csv``` file from Qiime Viewer using level 20 taxonomy.

Note, next time I come to run something like this, I should use the scientific names but without the full taxonomy string. I think it would make processing the data afterwards a lot easier.

## 4. Check all species assignments (range, confidence, similarity to other species)

Create a list here of the edits I need to make, then edit the ```superblast_taxonomy.qza``` artefact and reload into Qiime to do downstream analyses.

### Sandlances - is it possible to differentiate species?

Making a tree would help to solve this.
```
# Filter the Ammodytidae sequences and I'm including Sebastidae as an outgroup
qiime taxa filter-seqs\
     --i-sequences Terns/Terns_seqs.qza\
     --i-taxonomy superblast_taxonomy.qza\
     --p-include Ammodytidae,Sebastidae\
     --o-filtered-sequences Terns/Ammodytidae_rep-seqs

# Make the tree   
qiime phylogeny align-to-tree-mafft-fasttree \
    --i-sequences Terns/Ammodytidae_rep-seqs.qza \
    --p-n-threads 0 \
    --o-alignment Ammodytidae_alignment \
    --o-masked-alignment Ammodytidae_masked-alignment \
    --o-tree Ammodytidae_tree \
    --o-rooted-tree Ammodytidae_rooted-tree \
    --verbose
```

The tree can be viewed at https://itol.embl.de/tree/2459563435071592327228 
There are three main clades for *Ammodytes americanus* (american sand lance), *A. dubius* (northern sand lance), and *A. personatus* (Pacific sand lance). However, blasting the Pacific sandlance sequences shows:
1. *A. marinus* (Lesser sand eel) and *A. tobianus* (Small sand eel) have since been added to the database, and these are a slightly better match. However their distributions are in the northeast Atlantic around the UK.
2. There are also high percent ID matches with *A. hexapterus* and a lot of other *Ammodytes* species and so I think it's really impossible to tell what exactly they are.
Most of the ROST sandlance sequences match *A. personatus* but it's clearly not that, so I think there is maybe just too much ILS in this group to be able to make species assignments.


1. Remove taxonomy string from all entries.
2. Group all *Alosa* species to the genus level.
3. Group *Clupea pallasi* into *Clupea harengus*. *Pallasi* only comes up once in a sample with lots of *harengus* and so this may be a PCR error.
4. Remove *Sardinops melanostictus.* It is a monotypic genus and only two reads were pulled through in a single sample after rarefying. Could this be contamination from a tropicbird sample? No, it only comes up in one sample in the entire dataset. PCR error then?
5. 

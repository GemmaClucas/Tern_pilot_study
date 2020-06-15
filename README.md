# Tern_pilot_study

This is an outline of the steps I'm taking to finish the analyses for this first paper.

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



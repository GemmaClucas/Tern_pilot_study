Taxonomy\_edits
================
Gemma Clucas
6/15/2020

Edited:
9/4/2020

### Read in qiime taxonomy artifact

``` r
tax <- read_qza("/Users/gemmaclucas/Dropbox/Diets_from_poop/2019_terns_puffins_fecal_data_analysis/MiFish/final_taxonomy_superblast/superblast_taxonomy.qza")
```

### Make edits to the taxonomy strings

The edits I am making are, in this order:  
1\. Remove the entire taxonomy string except the species name (the base
R way to do this would be `(".*;", "", tax$data$Taxon)`).  
2\. Group all river herring into *Alosa sp.*  
3\. Group *Clupea pallasi* (Pacific herring) into *Clupea harengus*
(Atlantic herring).  
4\. Change *Sardinops melanostictus* (South American pilchard) to
*Clupea harengus* (Atlantic herring).  
5\. Change *Etheostoma parvipinne* to *Etheostomatinae*.  
6\. Change *Scomber japonicus* to *Scomber colias* (Atlantic chub
mackerel).  
7\. Group all *Ammodytes* sequences to *Ammodytes sp*.

Note that a useful function for checking whether changes have worked is
`str_detect()` e.g. `tax$data$Taxon %>% str_detect("Sardinops
melanostictus")` shows where it is in the dataframe.

Also note that there is a built-in function called `parse_taxonomy()`
but this expects a 7 category taxonomy string, and so I can’t use it for
my data. Next time, using 7 category strings would be a lot more
helpful.

``` r
# Count the number of instances of things before I make changes
# tax$data %>% filter(grepl("Etheostoma parvipinne", Taxon)) %>% count() # 1
# tax$data %>% filter(grepl("Sardinops melanostictus", Taxon)) %>% count() # 1
# tax$data %>% filter(grepl("Scomber japonicus", Taxon)) %>% count() # 1
# tax$data %>% filter(grepl("Alosa aestivalis", Taxon)) %>% count() # 81
# tax$data %>% filter(grepl("Alosa alosa", Taxon)) %>% count() # 1
# tax$data %>% filter(grepl("Alosa pseudoharengus", Taxon)) %>% count() # 12
# tax$data %>% filter(grepl("Alosa aestivalis|Alosa alosa|Alosa pseudoharengus", Taxon)) %>% count() # 94 in total
# tax$data %>% filter(grepl("Clupea pallasi", Taxon)) %>% count() # 12
# tax$data %>% filter(grepl("Ammodytes americanus", Taxon)) %>% count() # 14
# tax$data %>% filter(grepl("Ammodytes dubius", Taxon)) %>% count() # 69
# tax$data %>% filter(grepl("Ammodytes personatus", Taxon)) %>% count() # 180
# tax$data %>% filter(grepl("Ammodytes americanus|Ammodytes dubius|Ammodytes personatus", Taxon)) %>% count() # 263 in total

# Make the changes
tax$data$Taxon <- tax$data$Taxon %>% 
  #str_replace_all(".*;", "") %>% # this removes all of the string up to the species, but I think I need to keep it for the TSVTaxonomy Format to 
  str_replace_all("Alosa .*", "Alosa sp.") %>% 
  str_replace("Clupea pallasi", "Clupea harengus") %>% 
  str_replace(" Sardinops; Sardinops melanostictus", " Clupea; Clupea harengus") %>% 
  str_replace(" Etheostomatinae; Etheostoma; Etheostoma parvipinne", "Etheostomatinae") %>% 
  str_replace("Scomber japonicus", "Scomber colias") %>% 
  str_replace_all("Ammodytes .*", "Ammodytes sp.") 


# Check that these worked
# tax$data %>% filter(grepl("Etheostoma parvipinne", Taxon)) %>% count() # retuns 0 which is correct
# tax$data %>% filter(grepl("Etheostomatinae", Taxon)) %>% count() # returns 1 which is correct
# tax$data %>% filter(grepl("Sardinops melanostictus", Taxon)) %>% count() # retuns 0 which is correct
# tax$data %>% filter(grepl("Alosa sp.", Taxon)) %>% count() # retruns 94 which is correct
# tax$data %>% filter(grepl("Ammodytes sp.", Taxon)) %>% count() # 266, there are three Ammodytes hexapterus counted
# tax$data %>% filter(grepl("Scomber japonicus", Taxon)) %>% count() # retuns 0 which is correct
# tax$data %>% filter(grepl("Scomber colias", Taxon)) %>% count() # retuns 1 which is correct
```

Note that the taxonomy artefact includes all assignments that were made
when I assigned taxonomy i.e. before I did any filtering. This means
that unassigned features are in there, features from puffins and
tropicbirds are in there, and features from birds and mammals that were
later filtered out.

### Export and reload in qiime

``` r
write.table(tax$data, 
          quote = FALSE, 
          row.names = FALSE,
          file = "/Users/gemmaclucas/Dropbox/Diets_from_poop/2019_terns_puffins_fecal_data_analysis/MiFish/final_taxonomy_superblast/Terns/taxonomy_edited.tsv",
          sep = "\t")
```

**NOTE - R puts a period into the header `Feature ID` which makes the
file invalid for Qiime ヽ༼ ಠ益ಠ ༽ﾉ**

The command for removing the period and reloading into qiime
is:

``` bash
cd /Users/gemmaclucas/Dropbox/Diets_from_poop/2019_terns_puffins_fecal_data_analysis/MiFish/final_taxonomy_superblast/

sed -i.bak 's/Feature.ID/Feature ID/g' Terns/taxonomy_edited.tsv

conda activate qiime2-2019.4

qiime tools import \
  --input-path Terns/taxonomy_edited.tsv \
  --output-path Terns/taxonomy_edited.qza \
  --type 'FeatureData[Taxonomy]'
```

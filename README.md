# pG4-BS_2021
Project data for the pG4-BS manuscript (2021)

This is a documentation of the steps I took in my analysis of G4BS motifs in the human genome. 

1. I ran the g4_bulge_finder.v4.py script on the Hg38 human genome assambly with options '-g 2' and 
'-b 2', allowing for a minimum of 2 intact G-runs and a maximum of 2 bulge sites. 
(Script requires Python 2.7 to run)

#conda activate Python2_7
#python g4_bulge_finder.v4.py -g 2 -b 2 ../../Genomes/hg38.fa > g4bs.Hg38.raw.txt

2. I filtered out hits that belonged to non-canonical chromosomes. 

#awk '$1 ~ /^(chr[1-9]|chr[1-9][0-9]|chr[XY])$/' g4bs.Hg38.raw.txt > g4bs.Hg38.txt

============================================================================================
In parallel, ran the canonical G4 finding script allowing a maximum loop length of 7 nucleotides, then
filtered for canonical chromosome hits:

#python g4_canonical_finder.py ../../Genomes/hg38.fa > cG4_L7.Hg38.raw.txt
#awk '$1 ~ /^(chr[1-9]|chr[1-9][0-9]|chr[XY])$/' cG4_L7.Hg38.raw.txt > cG4_L7.Hg38.txt

The cG4 finder is now able to identify overlapping sequences, so running that on Hg38 is the preferred method.

The following steps were rerun, but the method remains unchanged. 
============================================================================================
3. Next, I excluded overlapping hits from both datasets: 

#intersectBed -wa -v -s -a g4bs.Hg38.txt -b cG4_L7.Hg38.txt > g4bs.Hg38.uniq.txt
#intersectBed -wa -v -s -a cG4_L7.Hg38.txt -b g4bs.Hg38.txt > cG4_L7.Hg38.uniq.txt

4. Stratified the dataset into 3 datasets based on the given model (i.e. G3B1, G3B2, G2B2)

#awk '$7 ~ 3 && $8 ~ 1' g4bs.Hg38.uniq.txt | awk -F"\t" '{OFS = FS}{$12 = "G3B1"; print}' - > g4bs.Hg38.uniq.G3B1.txt
#awk '$7 ~ 3 && $8 ~ 2' g4bs.Hg38.uniq.txt | awk -F"\t" '{OFS = FS}{$12 = "G3B2"; print}' - > g4bs.Hg38.uniq.G3B2.txt
#awk '$7 ~ 2 && $8 ~ 2' g4bs.Hg38.uniq.txt | awk -F"\t" '{OFS = FS}{$12 = "G2B2"; print}' - > g4bs.Hg38.uniq.G2B2.txt

5. Merged the individual datasets (including the canonical G4 dataset):

#sortBed -i g4bs.Hg38.uniq.G3B1.txt | mergeBed -s -c 6 -o distinct -i - > g4bs.Hg38.uniq.G3B1.merged.txt
#sortBed -i g4bs.Hg38.uniq.G3B2.txt | mergeBed -s -c 6 -o distinct -i - > g4bs.Hg38.uniq.G3B2.merged.txt
#sortBed -i g4bs.Hg38.uniq.G2B2.txt | mergeBed -s -c 6 -o distinct -i - > g4bs.Hg38.uniq.G2B2.merged.txt

#sortBed -i cG4_L7.Hg38.uniq.txt | mergeBed -s -c 6 -o distinct -i - > cG4_L7.Hg38.uniq.merged.txt

Just in case, I created a merged version of the whole G4BS dataset, without taking the different models into account

#sortBed -i g4bs.Hg38.uniq.txt | mergeBed -s -c 6 -o distinct -i - > g4bs.Hg38.uniq.blunt_merged.txt 
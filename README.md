# Аннотация генома человека на разные типы эпигенетических состояний с помощью chromHMM

**Автор: Наталия Кропивницкая**

## Исходные данные:
Из предложенных клеточных линий в [контролируемом словаре ENCODE](https://genome.ucsc.edu/ENCODE/dataMatrix/encodeChipMatrixHuman.html) была выбрана клеточная линия K562. Характеристики:
- **cell**: K562
- **Tier**: 1
- **Description**: leukemia, "The continuous cell line K-562 was established by Lozzio and Lozzio from the pleural effusion of a 53-year-old female with chronic myelogenous leukemia in terminal blast crises." - ATCC
- **Lineage**: mesoderm
- **Tissue**: blood
- **Karyotype**: cancer
- **Sex**: F
- **Documents**: [ENCODE](https://genome.ucsc.edu/ENCODE/protocols/cell/human/K562_protocol.pdf)
- **Vendor ID**: [ATCC CCL-243](http://www.atcc.org/ATCCAdvancedCatalogSearch/ProductDetails/tabid/452/Default.aspx?ATCCNum=CCL-243&Template=cellBiology)
- **Term ID**: [BTO:0000664](http://www.ebi.ac.uk/ontology-lookup/browse.do?ontName=BTO&termId=BTO%3A0000664)
- **Label**: K562

## Предобработка данных и подготовка программ
### Поиск соответствующих bam-файлов с гистоновыми модификациями:

ENCODE --> Experiment Summary --> ChiPseq --> advanced search: track name = K562 --> список гистоновых модификаций --> выбираем модификацию --> смотрим ее ссылку в [списке bM-а  лов](http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/) --> скачиваем

ИЛИ идем по файлу files.txt и выбираем нужные гистоновые модификации:

<pre> ```bash
wget http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/files.txt
grep -i K562 files.txt | grep -i "\.bam" > k562_bam_list.txt
``` </pre>

#### Контрольный [файл](http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecControlStdAlnRep1.bam) - ControlStdAlnRep1.bam
| Гистоновая метка | Файл | Ссылка на файл |
| ------------- | ------------- | ------------- |
|	H3k4me1 |	H3k4me1StdAlnRep1.bed	| http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneK562ControlStdAlnRep1.bam |
|	H3k4me3 |	H3k4me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH3k4me3StdAlnRep1.bam |
|	H3k9ac |	H3k9acStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH3k9acStdAlnRep1.bam |
|	H3k9me1 |	H3k9me1StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH3k9me1StdAlnRep1.bam |
|	H3k27ac |	H3k27acStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH3k27acStdAlnRep1.bam |
|	H3k27me3 |	H3k27me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH3k27me3StdAlnRep1.bam |
|	H3k36me3 |	H3k36me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH3k36me3StdAlnRep1.bam |
|	H3k79me2 |	H3k79me2AlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH3k79me2AlnRep1.bam |
|	H4k20me1 |	H4k20me1StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecH4k20me1StdAlnRep1.bam |
|	Pol2b |	Pol2bStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneHuvecPol2bStdAlnRep1.bam |

# Аннотация генома человека на разные типы эпигенетических состояний с помощью chromHMM

**Автор: Наталия Кропивницкая**

## Исходные данные:
Из предложенных клеточных линий в [контролируемом словаре ENCODE](https://genome.ucsc.edu/ENCODE/dataMatrix/encodeChipMatrixHuman.html) была выбрана клеточная линия K562. Характеристики:
- **cell**: A549
- **Tier**: 2
- **Description**: epithelial cell line derived from a lung carcinoma tissue. (PMID: 175022), "This line was initiated in 1972 by D.J. Giard, et al. through explant culture of lung carcinomatous tissue from a 58-year-old caucasian male." - ATCC, newly promoted to tier 2: not in 2011 analysis
- **Lineage**: endoderm
- **Tissue**: epithelium
- **Karyotype**: cancer
- **Sex**: M
- **Documents**: [Myers Crawford Stam](https://genome.ucsc.edu/ENCODE/protocols/cell/human/A549_Stam_protocol.pdf)
- **Vendor ID**: [ATCC CCL-185](http://www.atcc.org/ATCCAdvancedCatalogSearch/ProductDetails/tabid/452/Default.aspx?ATCCNum=CCL-185&Template=cellBiology)
- **Term ID**: [BTO:0000018](http://www.ebi.ac.uk/ontology-lookup/browse.do?ontName=BTO&termId=BTO%3A0000018)
- **Label**: A549

## Предобработка данных и подготовка программ
### Поиск соответствующих bam-файлов с гистоновыми модификациями:

ENCODE --> Experiment Summary --> ChiPseq --> advanced search: track name = A549 --> список гистоновых модификаций --> выбираем модификацию --> смотрим ее ссылку в [списке bam-файлов](http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/) --> скачиваем

ИЛИ идем по файлу files.txt и выбираем нужные гистоновые модификации:

<pre> ```bash
wget http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/files.txt
grep -i A549 files.txt | grep -i "\.bam" > A549_bam_list.txt
``` </pre>

#### Контрольный [файл](http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549ControlDex100nmAlnRep1.bam ) - ControlStdAlnRep1.bam
| Гистоновая метка | Файл | Ссылка на файл |
| ------------- | ------------- | ------------- |
|	H3k4me1 |	H3k4me1StdAlnRep1.bed	| http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k04me1Etoh02AlnRep1.bam |
|	H3k4me3 |	H3k4me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/ wgEncodeBroadHistoneA549H3k04me3Etoh02AlnRep1.bam |
|	H3k9ac |	H3k9acStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k09acEtoh02AlnRep1.bam |
|	H3k9me1 |	H3k9me1StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k09me3Etoh02AlnRep1.bam |
|	H3k27ac |	H3k27acStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k27acEtoh02AlnRep1.bam |
|	H3k27me3 |	H3k27me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k27me3Etoh02AlnRep1.bam |
|	H3k36me3 |	H3k36me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k36me3Etoh02AlnRep1.bam |
|	H3k79me2 |	H3k79me2AlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k79me2Etoh02AlnRep1.bam |
|	H4k20me1 |	H4k20me1StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H4k20me1Etoh02AlnRep1.bam |
|	CTCF |	CtcfStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549CtcfEtoh02AlnRep1.bam |

[Скрипт]() для скачивания всех файлов

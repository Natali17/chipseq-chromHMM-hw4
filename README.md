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

```bash
wget http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/files.txt
grep -i A549 files.txt | grep -i "\.bam" > A549_bam_list.txt
```

#### Контрольный [файл](http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549ControlDex100nmAlnRep1.bam) - ControlStdAlnRep1.bam
| Гистоновая метка | Файл | Ссылка на файл |
| ------------- | ------------- | ------------- |
|	H3k4me1 |	H3k4me1StdAlnRep1.bed	| http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k04me1Etoh02AlnRep1.bam |
|	H3k4me3 |	H3k4me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k04me3Etoh02AlnRep1.bam |
|	H3k9ac |	H3k9acStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k09acEtoh02AlnRep1.bam |
|	H3k9me3 |	H3k9me1StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k09me3Etoh02AlnRep1.bam |
|	H3k27ac |	H3k27acStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k27acEtoh02AlnRep1.bam |
|	H3k27me3 |	H3k27me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k27me3Etoh02AlnRep1.bam |
|	H3k36me3 |	H3k36me3StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k36me3Etoh02AlnRep1.bam |
|	H3k79me2 |	H3k79me2AlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H3k79me2Etoh02AlnRep1.bam |
|	H4k20me1 |	H4k20me1StdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549H4k20me1Etoh02AlnRep1.bam |
|	CTCF |	CtcfStdAlnRep1.bed |	http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeBroadHistone/wgEncodeBroadHistoneA549CtcfEtoh02AlnRep1.bam |

#### Скрипт для скачивания всех файлов
![Скрипт](/img/downloading_bam.png) 

Вручную создаем текстовый файл [cellmarkfiletable.txt](), в котором указываем название типа клеток, разных гистоновых меток, а также соответствующие .bam файлы для эксперимента и контроля. Один и тот же контрольный файл .bam может быть использован для всех экспериментов.

Более подробно о содержании файла cellmarkfiletable.txt можно посмотреть в руководстве пользователя [ChromHMM](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/data/ChromHMM_tutorial.pdf)

### Активируем миниконду и скачиваем ChromHMM, скачиваем размеры хромосом для разбиения генома на интервалы:
*Примечание: в коде "#" помечены те команды, которые были использованы для первоначального решения, но в конечном итоге не привели к удовлетворяющей визуализации, поэтому программа chromhmm скачивалась напрямую, а не через биоконду.

```bash
source /gpfs/vigg_mipt/kropivnitskaya/miniconda3/bin/activate
wget http://compbio.mit.edu/ChromHMM/ChromHMM.zip
unzip ChromHMM.zip
cd ChromHMM
java -jar ChromHMM.jar

#conda install bioconda::chromhmm
#export PATH=/gpfs/vigg_mipt/kropivnitskaya/miniconda3/bin:$PATH
#wget http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/hg19.chrom.sizes -O hg19.txt
#mkdir -p CHROMSIZES
#mv hg19.txt CHROMSIZES/
```

## Запуск программ
1. Запускаем ChromHMM с опцией BinarizeBam, чтобы конвертировать профили из ChIP-seq экспериментов (bam-файлы) в табличку из 0 и 1, т.е. чтобы сделать разбивку генома на условные интервалы (бины) длиной 200 п.о. и для каждого интервала и метки определить: был ли там пик или нет (1 или 0). Для 11 меток программа работает около 20 мин:

```bash
java -mx5000M -jar ChromHMM/ChromHMM.jar BinarizeBam -b 200  ChromHMM/CHROMSIZES/hg19.txt chip_seq cellmarkfiletable.txt binarizedData

#ChromHMM.sh -Xmx8000M BinarizeBam -b 200 CHROMSIZES/hg19.txt chip_seq cellmarkfiletable.txt binarizedData
#Программа LearnModel не работает, если не убрать название клеточной линии из названий бинарных файлов, поэтому:
#for f in A549_chr*_binary.txt; do
#    mv "$f" "${f#A549_}"
#done
```

2. Запускаем ChromHMM с опцией LearnModel, которая автоматически определит параметры N разных эпигенетических типов с наиболее выраженными наборами гистоновых меток и присвоит каждому геномному интервалу определенный эпигенетический тип. Количество разных эпигенетических типов (или состояний) - 10 штук. Для 11 меток программа работает около 1 часа:

```bash
java -mx5000M -jar ChromHMM/ChromHMM.jar LearnModel -b 200 binarizedData/ MYOUTPUT 15 hg19

#ChromHMM.sh -Xmx8000M LearnModel -p 8 binarizedData MYOUTPUT 10 hg19
```

## Обработка данных
#### web_10.html
[web_10.html]()

<p align="center">
  <img src="image1.png" width="45%" />
  <img src="image2.png" width="45%" />
</p>

<p align="center">
  <img src="image1.png" width="45%" />
  <img src="image2.png" width="45%" />
</p>

#### Emission

#### Overlap

#### Transition

#### RefSeqTSS


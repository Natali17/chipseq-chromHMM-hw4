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
![Скрипт](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/downloading_bam.png) 

Вручную создаем текстовый файл [cellmarkfiletable.txt](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/data/cellmarkfiletable.txt), в котором указываем название типа клеток, разных гистоновых меток, а также соответствующие .bam файлы для эксперимента и контроля. Один и тот же контрольный файл .bam может быть использован для всех экспериментов.

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

### Результаты работы LearnModel
[webpage_15.html](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/data/webpage_15.html)

#### Transition & Emission
<p align="center">
  <img src="https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/transitions_15.png" width="45%" />
  <img src="https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/transitions_15.png" width="45%" />
</p>

#### RefSeqTES & RefSeqTSS
<p align="center">
  <img src="https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/A549_15_RefSeqTES_neighborhood.png" width="45%" />
  <img src="https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/A549_15_RefSeqTSS_neighborhood.png" width="45%" />
</p>

#### Overlap
![overlap](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/A549_15_overlap.png)

### Геномный браузер
#### Настройки геномного браузера
http://genome.ucsc.edu --> My Data --> Custom Tracks --> загружаем файл [A549_15_dense.bed]() --> Return to current position --> Regulation  --> ENCODE Regulation (show, full) & ENC Histone (show) (click) --> Broad Histone (show, full) (click) --> [выбор нужных параметров]() --> Submit --> выбор нужного региона --> получаем изображения.
![adjusting_browser](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/adjusting_browser.png)

#### Визуализация результатов

Краткая сводка из публичных источников, за какие эпигенетические состояния могут отвечать использованные гистоновые маркеры:
| Метка | Ассоциированные состояния ChromHMM | Локализация | 
|---------------|------------------------------------------------|------------------------------------------------| 
| H3K4me3 | Active Promoter, Weak Promoter | Активные и слабые промоторы | j
| H3K4me1 | Strong Enhancer, Weak/Poised Enhancer | Энхансеры | 
| H3K27ac | Strong Enhancer, Active Promoter | Активные энхансеры и промоторы | 
| H3K9ac | Active Promoter, Enhancer | Активация, чаще в промоторах | j
| H3K27me3 | Heterochromatin, repressed | преимущественно на ядерной ламине, в области репрессированного гетерохроматина | 
| H3K9me3 | Heterochromatin, Repetitive/CNV | Гетерохроматин, сайленсинг, повторы | j
| H3K36me3 | Weak Transcribed, Transcription Elongation |  | j
| H3K79me2 | Weak Transcribed, Transcription Elongation |  | j
| H4K20me1 | Weak Transcribed, Heterochromatin | Регуляция транскрипции/структуры | 
| CTCF | Insulator | Изоляторы, границы TAD | j

На основе графика "Fold Enrichment A549_15" и сопоставления с визуализацией геномным браузером, вот итоговая таблица, интерпретирующая состояния ChromHMM для клеточной линии A549, модель с 15 состояниями:

| State | Биологическая интерпретация        | Категория                   | Обоснование (по обогащению)                                                             |
|-------|-------------------------------------|------------------------------|------------------------------------------------------------------------------------------|
| 1     |  Repressed                     |          | Сильное обогащение по LaminB1, умеренно по Genome%                                                            |
| 2     |  Repressed                     |          | Обогащение по LaminB1 и Genome%                                                    |
| 3     |  Heterochromatin           |                 | Сильное обогащение по LaminB1                                                  |
| 4     |            |                   | Обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES                         |
| 5     |             |                   | Обогащение по RefSeqGene, RefSeqExon, RefSeqTES                                             |
| 6     |  Transcriptional Elongation/ Weak Transcribed              |                    | Обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES                                                              |
| 7     |                |                   | Яркое обогащение по RefSeqGene                                                                  |
| 8     |                  |               | Яркое обогащение по RefSeqGene                                  |
| 9     |                  |                     | Сильное обогащение по RefSeqTES, RefSeqGene, RefSeqExon                                         |
| 10    |                | C                    | Нет чётких обогащений, умеренные сигналы рядом с LaminB1 и RefSeqGene                                                |
| 11    |      |  | Нет чётких обогащений, умеренные сигналы рядом с RefSeqTES и RefSeqTSS2Kb                                          |
| 12    |  Weak Enhancer / Weak Transcribed        |                             | Сильное обогащение по RefSeqGene, меньше RefSeqTES и RefSeqTSS2Kb                                       |
| 13    |  Active promoter                |                             | Сильное обогащение по CpGIsland, RefSeqExon, RefSeqTSS и RefSeqTSS2Kb            |
| 14    |           |                            | Сигнал слабее, но аналогично State 13                                     |
| 15    |            |                             | Умеренное обогащение по LaminB1                                                  |



Как определялись эпигенетические состояния:
- Из A549_15_RefSeqTSS_neighborhood.png видно, что State 13 отвечает за начало транскрипции, плюс обогащения по CpGIsland, RefSeqExon, RefSeqTSS и RefSeqTSS2Kb из Fold_Enrichment_A549_15.png и пики на маркерах H3K4me3, H3K9ac, H3K27ac --> Active Promoter
- Из визуализации геномным браузером видно, что State 12 обрамляет State 13 с двух сторон, и есть яркие пики H3K4me1, который часто отвечает за энхансеры. Однако из графика RefSeqTES.png видим, что тарскрипционная активность распределена по всему участку --> Weak Enhancer / Weak Transcribed
![state13]()
![state12]()
- Больше всего в геноме State 2 и State 3, cильное обогащение по LaminB1 --> Repressed
![state1_2]()
- Сильное обогащение по LaminB1 и умеренное по Genome%
- Для State 3 характерны пики H3k9me3 (является индикатором гетерохроматина или репрессированных транскрибируемых областей) и сильное обогащение по по LaminB1 --> Heterochromatin
![state3]()
- Для State 6 характерны пики H3K79me2, H3K36me3, H4K20me1, а также обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES --> Transcriptional Elongation/ Weak Transcribed
![state6]()
- В State 14 много CpGIslands, как и в State 13, также пики H3K4me3 и H3K9ac --> Active Promoter
- В State 











  - *Маркеры*: H3K27me3, H3K4me3, H3K4me2
  - *Локализация*: интроны или экзоны
  - *Картинка*: ![Image](/data/picture_1.png)

- Weak Enhancer/Weak Transcribed
  - *Маркеры*: H3K4me2, H3K4me1
  - *Локализация*: часто в RefSeqTES, RefSeqGene, у ядерной ламины
  - *Картинка*: ![Image](/data/picture_2.png)

- Strong Enhancer/Transcriptional Elongation
  - *Маркеры*: H3K9ac, H3K27ac, H3K4me3, H3K4me2, H3K4me1
  - *Локализация*: часто в RefSeqTES, RefSeqTSS2Kb, у ядерной ламины
  - *Картинка*: ![Image](/data/picture_3.png)

- Active Promoter
  - *Маркеры*: H3K9ac, H3K27ac, H3K4me3, H3K4me2
  - *Локализация*: часто в CpGIslands, RefSeqTES, RefSeqExon, RefSeqTSS2Kb
  - *Картинка*: ![Image](/data/picture_4.png)

- Active Promoter
  - *Маркеры*: H3K9ac, H3K27ac, H3K4me3, H3K4me2
  - *Локализация*: часто в RefSeqTES, RefSeqTSS2Kb, RefSeqExon, CpGIslands, RefSeqGene
  - *Картинка*: ![Image](/data/picture_5.png)

- Weak Enhancer
  - *Маркеры*: H3K79me2, H3K9ac, H3K27ac, H3K4me3, H3K4me2
  - *Локализация*: часто в RefSeqGene, RefSeqTES, иногда в RefSeqExon
  - *Картинка*: ![Image](/data/picture_6.png)

- Weak Enhancer
  - *Маркеры*: H3K9ac, H3K27ac, H3K4me1
  - *Локализация*: часто в RefSeqTES
  - *Картинка*: ![Image](/data/picture_7.png)

- Weak Enhancer
  - *Маркеры*: H3K4me1
  - *Локализация*: часто в RefSeqTES, иногда у ядерной ламины
  - *Картинка*: ![Image](/data/picture_8.png)

- Weak Enhancer
  - *Маркеры*: H3K79me2, H3K4me1
  - *Локализация*: часто в RefSeqGene, иногда в RefSeqTES и RefSeqExon
  - *Картинка*: ![Image](/data/picture_9.png)

- Weak Transcribed
  - *Маркеры*: H3K79me2
  - *Локализация*: часто в RefSeqGene
  - *Картинка*: ![Image](/data/picture_10.png)

- Weak Transcribed
  - *Маркеры*: H3K36me3, H3K79me2
  - *Локализация*: часто в RefSeqGene, иногда в RefSeqTES и RefSeqExon
  - *Картинка*: ![Image](/data/picture_11.png)

- Weak Transcribed
  - *Маркеры*: H3K36me3
  - *Локализация*: часто в RefSeqGene, иногда в RefSeqTES и RefSeqExon
  - *Картинка*: ![Image](/data/picture_12.png)



- Heterochromatin
  - *Маркеры*: H3K9me3
  - *Локализация*: преимущественно на ядерной ламине, в области репрессированного гетерохроматина
  - *Картинка*: ![Image](/data/picture_15.png)

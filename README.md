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

#### Определение эпигенетических состояний

Краткая сводка из публичных источников, за какие эпигенетические состояния могут отвечать использованные гистоновые маркеры:
| Метка | Ассоциированные состояния ChromHMM | 
|---------------|------------------------------------------------|
| H3K4me3 | Active Promoter, Weak Promoter | 
| H3K4me1 | Strong Enhancer, Weak/Poised Enhancer |  
| H3K27ac | Strong Enhancer, Active Promoter | 
| H3K9ac | Active Promoter, Enhancer | 
| H3K27me3 | Heterochromatin, repressed | 
| H3K9me3 | Heterochromatin, Repetitive/CNV | 
| H3K36me3 | Weak Transcribed, Transcription Elongation | 
| H3K79me2 | Weak Transcribed, Weak Enchancer, Transcription Elongation |
| H4K20me1 | Weak Transcribed, Heterochromatin | 
| CTCF | Insulator, Weak Enchancer | Изоляторы, границы TAD | 

На основе графика "Fold Enrichment A549_15" и сопоставления с визуализацией геномным браузером, вот итоговая таблица, интерпретирующая состояния ChromHMM для клеточной линии A549, модель с 15 состояниями:

| State | Биологическая интерпретация        | Обоснование (по обогащению)                                                             |
|-------|-------------------------------------|------------------------------|
| 1     |  Repressed           | Сильное обогащение по LaminB1, умеренно по Genome%                                                            |
| 2     |  Repressed            | Обогащение по LaminB1 и Genome%                                                    |
| 3     |  Heterochromatin            | Сильное обогащение по LaminB1                                                  |
| 4     |    Weak Transcribed        | Обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES                         |
| 5     |  Transcriptional Elongation    | Обогащение по RefSeqGene, RefSeqExon, RefSeqTES                                             |
| 6     |  Transcriptional Elongation/ Weak Transcribed             | Обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES                              |
| 7     |    Weak Transcribed            | Яркое обогащение по RefSeqGene                                                                  |
| 8     |   Transcriptional Elongation       | Яркое обогащение по RefSeqGene                                  |
| 9     |   Weak Transcribed             | Сильное обогащение по RefSeqTES, RefSeqGene, RefSeqExon                                         |
| 10    |   Weak Transcribed          | Нет чётких обогащений, умеренные сигналы рядом с LaminB1 и RefSeqGene                                                |
| 11    |  Weak Enchancer   | Нет чётких обогащений, умеренные сигналы рядом с RefSeqTES и RefSeqTSS2Kb                                          |
| 12    |  Weak Enhancer       | Сильное обогащение по RefSeqGene, меньше RefSeqTES и RefSeqTSS2Kb                                       |
| 13    |  Active promoter           | Сильное обогащение по CpGIsland, RefSeqExon, RefSeqTSS и RefSeqTSS2Kb            |
| 14    |  Active Promoter        | Сигнал слабее, но аналогично State 13                                     |
| 15    |   Insulator        | Умеренное обогащение по LaminB1                                                  |



Как определялись эпигенетические состояния:
- Из A549_15_RefSeqTSS_neighborhood.png видно, что State 13 отвечает за начало транскрипции, плюс обогащения по CpGIsland, RefSeqExon, RefSeqTSS и RefSeqTSS2Kb из Fold_Enrichment_A549_15.png и пики на маркерах H3K4me3, H3K9ac, H3K27ac --> Active Promoter
![state13](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state13.png)
- Из визуализации геномным браузером видно, что State 12 обрамляет State 13 с двух сторон, и есть яркие пики H3K4me1, который часто отвечает за энхансеры. Однако из графика RefSeqTES.png видим, что тарскрипционная активность распределена по всему участку --> Weak Enhancer / Weak Transcribed
![state12](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state12.png)
- Больше всего в геноме State 1 и State 2, cильное обогащение по LaminB1 --> Repressed
![state1_2](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state1_2.png)
- Для State 3 характерны пики H3k9me3 (является индикатором гетерохроматина или репрессированных транскрибируемых областей) и сильное обогащение по по LaminB1 --> Heterochromatin
![state3](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state3.png)
- Для State 6 характерны пики H3K79me2, H3K36me3, H4K20me1, а также обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES --> Transcriptional Elongation/ Weak Transcribed
- В State 8 есть пики H4K20me1, H3K36me3, H3K79me2, в некоторых случаях пики и других меток, а также яркое обогащение по RefSeqGene --> Transcription Elongation
![state6_8](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state6_8.png)
- В State 14 много CpGIslands, как и в State 13, также пики H3K4me3 и H3K9ac --> Active Promoter
- В State 15 наблюдаются пики CTCF и умеренное обогащение по LaminB1 --> Insulator
![state14_15](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state14_15.png)
- В State 11 нет чётких обогащений, только умеренные сигналы рядом с RefSeqTES и RefSeqTSS2Kb, и отсутствуют пики по гистоновым модификациям --> Weak Enchancer
- В State 5 есть пики H3k36me3, а также обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES --> Transcription Elongation
- В State 7 небольшие пики H3K36me3, H3K79me2, H4k20me1 (не всегда), яркое обогащение по RefSeqGene --> Weak Transcribed
![state5_7](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state5_7.png)
- В State 10 нет чётких обогащений, умеренные сигналы рядом с LaminB1 и RefSeqGene, а также почти не наблюдается пиков --> Weak Transcribed
- В State 9 пики от CTCF сопряжены с пиками от других гистоновых меток, а также нет сильных активирующих меток типа H3K4me3 или H3K27ac --> Weak Transcribed
![state10_9](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state10_9.png)
- В State 4 небольшие пики H3k9me1, обогащение по RefSeqGene, меньше RefSeqExon, RefSeqTES --> Weak Transcribed
![state4](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/state4.png)

#### Код для переименования эпигенетических состояний
```python
state_names = [ 'Repressed',
                'Repressed',
                'Heterochromatin',
                'Weak_Transcribed',
                'Transcriptional_Elongation',
                'Transcriptional_Elongation',
                'Weak_Transcribed',
                'Transcriptional_Elongation',
                'Weak_Transcribed',
                'Weak_Transcribed',
                'Weak_Enchancer',
                'Weak_Enchancer',
                'Active_Promoter',
                'Active_Promoter',
                'Inculator' ]

with open(f'A549_15_dense.bed', 'r') as origin_file:
  with open(f'A549_15_dense_named.bed', 'w') as named_file:
    lines = origin_file.readlines()
    named_file.write(lines[0])
    for line in lines[1:]:
        line_splited = line.split('\t')
        line_splited[3] += '_' + state_names[int(line_splited[3]) - 1]
        named_file.write('\t'.join(line_splited))
```
Файлы A549_15_dense.bed и A549_15_dense_named.bed находятсяв папке data.

#### Полученная визуализация
![named_visualization](https://github.com/Natali17/chipseq-chromHMM-hw4/blob/main/img/named_visualization.png)






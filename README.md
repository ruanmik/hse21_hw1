# Bioinformatics hw №1
### Обязательная часть
Для начала создадим ссылки на необходимые файлы. 

``` 
ls -1 /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```

Теперь выберем с помощью команды seqtk  случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs. Значение `random seed = 1104`. 

```
seqtk sample -s1104 oil_R1.fastq 5000000 > PE_R1.fastq
seqtk sample -s1104 oil_R2.fastq 5000000 > PE_R2.fastq
seqtk sample -s1104 oilMP_S4_L001_R1_001.fastq 1500000 > MP_R1.fastq
seqtk sample -s1104 oilMP_S4_L001_R2_001.fastq 1500000 > MP_R2.fastq
```
Оставим в папке только полученные файлы. 

```
rm oil_R1.fastq oil_R2.fastq oilMP_S4_L001_R1_001.fastq oilMP_S4_L001_R2_001.fastq
```
С помощью программы fastQC и multiQC оценим качество исходных чтений и получим общую статистику по ним. 

```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}

mkdir multiqc
multiqc -o multiqc fastqc
```
С помощью программ platanus_trim и platanus_internal_trim подрезаем чтения по качеству и удаляем праймеры. 
Так же удаляем исходные .fastq файлы. 
```
platanus_trim PE_R2.fastq  PE_R1.fastq
platanus_internal_trim MP_R1.fastq  MP_R2.fastq
rm PE_R2.fastq PE_R1.fastq MP_R1.fastq MP_R2.fastq
```
Оценим качество подрезанных чтений и получим по ним общую статистику.

```
 mkdir trimmed_fastq
 mv -v *trimmed trimmed_fastq
 mkdir fastqc_trimmed
 mkdir multiqc_trimmed
 ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o fastqc_trimmed {}
 multiqc -o multiqc_trimmed fastqc_trimmed
```
С помощью программы “platanus assemble” собираем контиги из подрезанных чтений. 
```
time platanus assemble -o Poil -f trimmed_fastq/PE_R1.fastq.trimmed trimmed_fastq/PEe_R2.fastq.trimmed 2> assemble.log
```
Проанализируем полученные контиги. Колаб с кодом рассположен по [ссылке](https://colab.research.google.com/drive/1Mh5Dg8s46z7S-RBLazIfCR_Q55jgIMMV?usp=sharing)

С помощью программы “platanus scaffold” соберем скаффолды
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/PE_R1.fastq.trimmed trimmed_fastq/PE_R2.fastq.trimmed -OP2 trimmed_fastq/MP_R1.fastq.int_trimmed  trimmed_fastq/MP_R2.fastq.int_trimmed 2> scaffold.log
```

Проанализировали информацию о скаффолдах по [ссылке](https://colab.research.google.com/drive/1Mh5Dg8s46z7S-RBLazIfCR_Q55jgIMMV?usp=sharing). 

Выяснили, что максимальная длина = 3831212 скафолда >scaffold1_len3831212_cov231
Запишем информацию о нем в отдкльный файл для дальнейшего анализа. 

```
echo scaffold1_len3831212_cov231 > max_scaff.txt
seqtk subseq Poil_scaffold.fa max_scaff.txt > max_scaff.fa
rm -r max_scaff.txt
```
В колабе посчитаем количество гэпов и их общую длину. 
Теперь с помощью программы “ platanus gap_close” уменьшим количество гэпов
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/PE_R1.fastq.trimmed trimmed_fastq/PEe_R2.fastq.trimmed -OP2 trimmed_fastq/MP_R1.fastq.int_trimmed  trimmed_fastq/MP_R2.fastq.int_trimmed 2> gapclose.log
 time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/PE_R1.fastq.trimmed trimmed_fastq/PEe_R2.fastq.trimmed -OP2 trimmed_fastq/MP_R1.fastq.int_trimmed  trimmed_fastq/MP_R2.fastq.int_trimmed 2> gapclose.log
```
Удалим папку `trimmed_fastq`, тк эти файлы нам больше не нужны. 

```
rm -rf trimmed_fastq/
```
В колабе снова посчитаем количество гэпов и их общую длину. 

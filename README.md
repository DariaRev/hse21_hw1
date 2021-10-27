# hse21_hw1

Первая часть
-------------
* Создадим символическую ссылку для каждого из файлов
```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```
* Выберем 5000000 чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs
```
seqtk sample -s816 oil_R1.fastq 5000000 > sub_paired_R1.fq
seqtk sample -s816 oil_R2.fastq 5000000 > sub_paired_R2.fq
seqtk sample -s816 oilMP_S4_L001_R1_001.fastq 1500000 > sub_mate_R1.fq
seqtk sample -s816 oilMP_S4_L001_R2_001.fastq 1500000 > sub_mate_R2.fq
```
* Воспользуемся fastQC и multiQC для оценки качества исходных чтений
```
mkdir fastqc
ls sub* | xargs -P 4 -tI{} fastqc -o fastqc {}
mkdir multifastqc 
multiqc -o multiqc fastqc
```

Как видно, качество чтений для более длинных файлов падает, в некоторый момент доходит до желтой зоны и становится недопустимым.

![1](https://user-images.githubusercontent.com/32986053/139023364-663f16cc-b376-439d-9c2c-ca0ecebc830f.png)
![2](https://user-images.githubusercontent.com/32986053/139023586-8a7d63a1-5712-4191-9ca6-7ab9455911c0.png)
![3](https://user-images.githubusercontent.com/32986053/139023588-d69706d3-89d0-4c1f-a461-2c45657a2606.png)
![4](https://user-images.githubusercontent.com/32986053/139023589-504dbc65-164b-4718-b10d-266ac2abd850.png)
![5](https://user-images.githubusercontent.com/32986053/139023592-b56217bd-2212-4e6b-a252-1a2706eb8dc4.png)
![6](https://user-images.githubusercontent.com/32986053/139023593-a75ef9b5-b093-4afb-8350-0c3fba6e804e.png)
![7](https://user-images.githubusercontent.com/32986053/139023599-212d3382-347f-49a3-ade2-0323caacf36b.png)
![8](https://user-images.githubusercontent.com/32986053/139023601-3fb52b51-a968-4037-8c0e-d276de44b2d7.png)
![9](https://user-images.githubusercontent.com/32986053/139023604-e7ba19c7-cd56-452f-9529-54cad8272263.png)
![10](https://user-images.githubusercontent.com/32986053/139023607-b896127c-aa49-4725-b321-940ded94a284.png)

* Подрезаем чтения и удаляем ненужные файлы
```
platanus_trim sub_paired_R2.fq sub_paired_R1.fq
platanus_internal_trim sub_mate_R1.fq sub_mate_R2.fq
```

```
rm sub_mate_R1.fq
rm sub_mate_R2.fq
rm sub_paired_R1.fq
rm sub_paired_R2.fq
```
* Воспользуемся fastQC и multiQC для оценки качества подрезанных чтений
```
mkdir fastq
ls *.fq.* | xargs -P 4 -tI{} fastqc -o fastq {}
mkdir multiqc2
multiqc -o multiqc2 fastq
```
Как видно, в подрезанных чтениях качество стало выше

![11](https://user-images.githubusercontent.com/32986053/139030276-338f9888-a830-48e0-bb5b-112127290634.png)
![12](https://user-images.githubusercontent.com/32986053/139030281-e8a099cb-6f84-42ff-aa99-90246752f4e9.png)
![13](https://user-images.githubusercontent.com/32986053/139030283-3d71f920-9270-442e-980f-2388e98e73eb.png)
![14](https://user-images.githubusercontent.com/32986053/139030285-9d392ac8-7a88-4360-bc95-863714b6693f.png)
![15](https://user-images.githubusercontent.com/32986053/139030287-71216619-6b6f-48db-9a95-9327e1f93644.png)
![16](https://user-images.githubusercontent.com/32986053/139030289-2fe6e02a-1865-45ed-a80c-51d1dad466bd.png)
![17](https://user-images.githubusercontent.com/32986053/139030292-9a1b6df8-7f3c-4b48-966e-30f22070a414.png)
![18](https://user-images.githubusercontent.com/32986053/139030293-2fd74924-d232-4ff3-adce-7b757549cfad.png)
![19](https://user-images.githubusercontent.com/32986053/139030296-94447a3a-8cc6-4af8-a87f-5c90b39c6844.png)
![20](https://user-images.githubusercontent.com/32986053/139030299-fd3c28d1-942d-46c9-958d-6a3894a89ac2.png)
![21](https://user-images.githubusercontent.com/32986053/139030302-c7597dd5-5e84-417d-8d18-fb345a34b517.png)

* Теперь соберем контиги из подрезанных чтений
```
tmux
bash
time platanus assemble -o Poil -t 1 -f sub_paired_R1.fq.trimmed sub_paired_R2.fq.trimmed
```
* Собираем скаффолды
```
time platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 sub_paired_R1.fq.trimmed sub_paired_R2.fq.trimmed -OP2 sub_mate_R1.fq.int_trimmed sub_mate_R2.fq.int_trimmed 2> scaffold.log
```
* Уменьшаем кол-во гэпов
```
time platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 sub_paired_R1.fq.trimmed sub_paired_R2.fq.trimmed -OP2 sub_mate_R1.fq.int_trimmed sub_mate_R2.fq.int_trimmed 2>gapclose.log
```
Ссылка на колаб: https://colab.research.google.com/drive/17K9x0yLnExiGxLcObLCKNhlj7JiBuAJM#scrollTo=EnmiFfE1ac1t


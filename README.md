Я, к своему сожалению, решил себе усложнить задачу пойдя по не простому пути. Так что нам предстоит вселая дорога анализа генома мыши. 

<img src="/img/mouse_no_needed_hist.png" alt="" width="220" align="right"/>

Проблемы начинаются уже на этапе выбора гистона. Как бы это сказать... на [encodeproject](https://www.encodeproject.org/search/?type=Experiment&control_type!=*&assay_term_name=ChIP-seq&status=released&assay_title=Histone+ChIP-seq&replicates.library.biosample.donor.organism.scientific_name=Homo+sapiens&replicates.library.biosample.donor.organism.scientific_name=Mus+musculus)
у Mus musculus нет экспериментов над H2AFZ/H3F3A (прошу поправить меня, если не прав, но я вроде все разделы перебрал)

Так что выбираем самый популярный гистоновую метку **H3K36me3**.\
В качестве ткани для исследовния отвественным решением было случайно выбрана днк клетки печени.

Скачаем и обрежем эксперементы (все команды для macOS):

exp1
```` 
> https://www.encodeproject.org/experiments/ENCSR149GYK/
> curl -L -k https://www.encodeproject.org/files/ENCFF633VDV/@@download/ENCFF633VDV.bed.gz -o ENCFF633VDV.bed.gz
> gzcat ENCFF633VDV.bed.gz| cut -f1-5 > H3K36me3.ENCFF633VDV.bed.gz.mm10.bed
```` 

epx2
```` 
> https://www.encodeproject.org/experiments/ENCSR295ZLV/
> curl -L -k https://www.encodeproject.org/files/ENCFF034WYM/@@download/ENCFF034WYM.bed.gz -o ENCFF034WYM.bed.gz
> gzcat ENCFF034WYM.bed.gz| cut -f1-5 > H3K36me3.ENCFF034WYM.bed.gz.mm10.bed
```` 

Дальше фильтруем и строем гистограммы через [jupyter notebook](/src/Untitled.ipynb)

Эксперимент | Оригинал            |  Убрали выбросы (выбросили больше 5000)
:-------------------------:|:-------------------------:|:-------------------------:
ENCFF034WYM | ![](/img/ENCFF034WYM_nofilter.png)  |  ![](/img/ENCFF034WYM_filter.png)
ENCFF633VDV | ![](/img/ENCFF633VDV_nofilter.png)  |  ![](/img/ENCFF633VDV_filter.png)

Для чертового pie chart было слишком долго придумывать эффективный способ искать ближайший геном, так что я потратил 2 часа запуская R скрипт из примера ради 1 функции... А ну еще надо было немного потренировать сам R (впервые вижу этот язык) тк у меня же мышь, так что скрипт надо переписывать:

ENCFF034WYM            |  ENCFF633VDV
:-------------------------:|:-------------------------:
![Will come soon](/img/)  |  ![Will come soon](/img/)

Используем bedtools:

merge экспериментов
````
> cat  *.filtered.bed  |   sort -V -k1,1 -k2,2   |   bedtools merge   >  H3K36me3.merge.mm10.bed
```` 

Тут и далее закидываем все в 
[genom browser](https://genome.ucsc.edu/cgi-bin/hgTracks?db=mm10&lastVirtModeType=default&lastVirtModeExtraState=&virtModeType=default&virtMode=0&nonVirtPosition=&position=chr1%3A43778741%2D43778780&hgsid=1208521687_4cseGCUHrBoxb5ehJNJxLLhMEobG)
,чтобы удобно просматривать/ искать пересечения и смотреть на результаты bedtools

анализ участков вторичной структуры
````
> cat  mouseZ-*  |   sort -V -k1,1 -k2,2   |   bedtools merge   >  mouseZ.bed 
````

Hist            |  Pie
:-------------------------:|:-------------------------:
![](/img/ZDNA.png)  |  ![Will come soon](/img/)

```
> bedtools intersect -a mouseZ.bed -b H3K36me3.merge.mm10.bed > H3K36me3.intersect_with_mouseZ.mm10.bed
```

Пересечение:

![Will come soon](/img/)

Финальная версия [genom browser](https://genome.ucsc.edu/cgi-bin/hgTracks?db=mm10&lastVirtModeType=default&lastVirtModeExtraState=&virtModeType=default&virtMode=0&nonVirtPosition=&position=chr1%3A43778741%2D43778780&hgsid=1208521687_4cseGCUHrBoxb5ehJNJxLLhMEobG). Все настройки можно найти в [файле](/settings_genome_browser)

Набор пересечений для проверки:

+ chr2: 27019560-27019640

![](/img/genome_browser_fin.png)

В духе меня я скачиваю все размеченые геномы, чтобы отфильтровать в ручную:
```` 
curl -L -k "https://genome.ucsc.edu/cgi-bin/hgTables?hgsid=1208204641_xNAIrUQEonV1V077EA06hLFQdaXM&boolshad.hgta_printCustomTrackHeaders=0&hgta_ctName=tb_ncbiRefSeq&hgta_ctDesc=table+browser+query+on+ncbiRefSeq&hgta_ctVis=pack&hgta_ctUrl=&fbQual=whole&fbUpBases=200&fbExonBases=0&fbIntronBases=0&fbDownBases=200&hgta_doGetBed=get+BED" -o genes.bed
```` 

**for [genom browser](https://genome.ucsc.edu/cgi-bin/hgTracks?db=mm10&lastVirtModeType=default&lastVirtModeExtraState=&virtModeType=default&virtMode=0&nonVirtPosition=&position=chr1%3A43778741%2D43778780&hgsid=1208521687_4cseGCUHrBoxb5ehJNJxLLhMEobG)**\
bedtools intersect -a genes.bed -b H3K36me3.intersect_with_mouseZ.mm10.bed | cut -f1-5 > genes.H3K36me3.intersect_with_mouseZ.mm10.bed

**for data**\
bedtools intersect -a genes.bed -b H3K36me3.intersect_with_mouseZ.mm10.bed > genes_intersect.bed 

Из последноего файла получаем спискок геномов на пересечениях(последние ячейки [jupyter notebook](/src/Untitled.ipynb))

Отправлям на анализ в [pantherdb](http://pantherdb.org/):

![](/img/pantherdb_GO_analysis.png)

Я не супер биолог, но, вроде, результат вполне логичный - в днк ткани органа для обмена веществ (печень) встречается геном для метаболизма.



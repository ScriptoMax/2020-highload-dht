# Оценка времени отклика сервера в режиме симулирования запросов (wrk + async-profiler)

### Сведения о системе
| | |
|-|-|
| ОС Ubuntu | 18.04 LTS x64-bit |
| Процессор | Intel(R) Celeron(R) N4000 CPU @ 1.10GHz |
| Объём RAM | 8 ГБ |
| Количество ядер ЦПУ | 2 |


В ходе мониторинга производительности сервера (хост по адресу http://127.0.0.1:8080) с использованием инструмента <strong><em>wrk</em></strong> получены результирующие задержки обработки следующих запросов:
1) wrk -t1 -c1 -d5m -R2000 http://127.0.0.1:8080/    
2) GET /v0/entity?id=<ID>   
3) PUT /v0/entity?id=<ID>  
4) DELETE /v0/entity?id=<ID>

Основные параметры создания и передачи запросов через wrk (идентичны для всех исследуемых операций) 

``` wrk -t1 -c1 -d3m -R2000 http://127.0.0.1:8080 ```

Сводки наблюдений по каждой из указанных операций приведены далее.<br/>  
### 1) wrk -t1 -c1 -d5m -R2000 http://127.0.0.1:8080/ <br/>

Консольный вывод:
```
max@max-Inspiron-15-3573:~/asynctool/async-profiler-1.8.1-linux-x64$ wrk -t1 -c1 -d3m -R2000 --latency http://localhost:8080
Running 3m test @ http://localhost:8080
  1 threads and 1 connections
  Thread calibration: mean lat.: 1.319ms, rate sampling interval: 10ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.40ms    1.59ms  39.62ms   93.96%
    Req/Sec     2.12k   482.56     8.00k    88.36%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.09ms
 75.000%    1.57ms
 90.000%    2.03ms
 99.000%    9.04ms
 99.900%   15.57ms
 99.990%   29.69ms
 99.999%   38.43ms
100.000%   39.65ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.049     0.000000            5         1.00
       0.365     0.100000        34105         1.11
       0.610     0.200000        68121         1.25
       0.818     0.300000       102018         1.43
       0.962     0.400000       136025         1.67
       1.090     0.500000       170135         2.00
       1.154     0.550000       187087         2.22
       1.237     0.600000       204018         2.50
       1.344     0.650000       221095         2.86
       1.454     0.700000       238034         3.33
       1.568     0.750000       255076         4.00
       1.623     0.775000       263579         4.44
       1.680     0.800000       272062         5.00
       1.745     0.825000       280577         5.71
       1.812     0.850000       288968         6.67
       1.899     0.875000       297525         8.00
       1.952     0.887500       301761         8.89
       2.027     0.900000       306003        10.00
       2.163     0.912500       310249        11.43
       2.369     0.925000       314467        13.33
       2.869     0.937500       318717        16.00
       3.245     0.943750       320843        17.78
       3.669     0.950000       322965        20.00
       4.143     0.956250       325100        22.86
       4.671     0.962500       327213        26.67
       5.319     0.968750       329340        32.00
       5.683     0.971875       330410        35.56
       6.079     0.975000       331463        40.00
       6.527     0.978125       332535        45.71
       7.023     0.981250       333592        53.33
       7.599     0.984375       334658        64.00
       7.927     0.985938       335180        71.11
       8.319     0.987500       335714        80.00
       8.759     0.989062       336252        91.43
       9.231     0.990625       336776       106.67
       9.783     0.992188       337308       128.00
      10.071     0.992969       337570       142.22
      10.399     0.993750       337842       160.00
      10.759     0.994531       338106       182.86
      11.175     0.995313       338370       213.33
      11.623     0.996094       338636       256.00
      11.903     0.996484       338768       284.44
      12.215     0.996875       338900       320.00
      12.543     0.997266       339032       365.71
      12.959     0.997656       339164       426.67
      13.471     0.998047       339297       512.00
      13.791     0.998242       339364       568.89
      14.119     0.998437       339429       640.00
      14.511     0.998633       339496       731.43
      15.031     0.998828       339562       853.33
      15.663     0.999023       339629      1024.00
      16.055     0.999121       339663      1137.78
      16.495     0.999219       339697      1280.00
      16.975     0.999316       339728      1462.86
      17.711     0.999414       339761      1706.67
      18.607     0.999512       339795      2048.00
      19.071     0.999561       339812      2275.56
      19.679     0.999609       339828      2560.00
      20.175     0.999658       339844      2925.71
      21.055     0.999707       339861      3413.33
      22.207     0.999756       339879      4096.00
      23.295     0.999780       339886      4551.11
      24.431     0.999805       339894      5120.00
      25.743     0.999829       339903      5851.43
      26.895     0.999854       339911      6826.67
      28.399     0.999878       339919      8192.00
      29.263     0.999890       339923      9102.22
      29.935     0.999902       339927     10240.00
      30.783     0.999915       339931     11702.86
      31.727     0.999927       339936     13653.33
      32.175     0.999939       339940     16384.00
      32.447     0.999945       339942     18204.44
      32.623     0.999951       339944     20480.00
      33.503     0.999957       339946     23405.71
      34.399     0.999963       339948     27306.67
      35.295     0.999969       339950     32768.00
      35.743     0.999973       339951     36408.89
      36.191     0.999976       339952     40960.00
      36.639     0.999979       339953     46811.43
      37.087     0.999982       339954     54613.33
      37.535     0.999985       339955     65536.00
      37.983     0.999986       339956     72817.78
      37.983     0.999988       339956     81920.00
      38.431     0.999989       339957     93622.86
      38.431     0.999991       339957    109226.67
      38.879     0.999992       339958    131072.00
      38.879     0.999993       339958    145635.56
      38.879     0.999994       339958    163840.00
      39.327     0.999995       339959    187245.71
      39.327     0.999995       339959    218453.33
      39.327     0.999996       339959    262144.00
      39.327     0.999997       339959    291271.11
      39.327     0.999997       339959    327680.00
      39.647     0.999997       339960    374491.43
      39.647     1.000000       339960          inf
#[Mean    =        1.402, StdDeviation   =        1.585]
#[Max     =       39.616, Total count    =       339960]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  359967 requests in 3.00m, 24.37MB read
  Non-2xx or 3xx responses: 359967
Requests/sec:   1999.82
Transfer/sec:    138.66KB

```

Приведённая статистика свидетельствует о достижении серверной нагрузки на уровне 2000 запросов в секунду. Средняя длительность задержек в процессе обработки запроса и возврата данных клиенту установлена равной 1.4 ms при отклонении, достигающем 93%. Оценка времени ожидания ответа на основе квантильного распределения результатов позволяет утверждать, что в 99,99% случаев длительность поступления данных с локального сервера не превышала 30 мс. При этом оставшаяся доля операций (0.01%) приходится на запросы, формирование ответа на которые отмечено ощутимо большей задержкой, что находит отражение в оценке длительности обработки и отправки результата запроса в пределах 39 мс.  <br/>                    

![Screenshot from 2020-09-27 00-34-23](https://user-images.githubusercontent.com/55311053/94411885-c86e1a80-0181-11eb-8c58-06519574e2ce.png)
<p align="center">Рис.1. Визуализация эффектов нагрузки на ресурсы ЦПУ и RAM в интерфейсе VisualVM</p>

Данный рисунок иллюстрирует динамику использования ресурсов ЦПУ и RAM в ходе обработки запросов, поступающих в режиме симуляции нагрузки. Как следует из приведённых графиков, запуск wrk споровождался резким повышением амплитуды кривой, описывающей интенсивность операций на каждом из вышеназванных устройств. Реализация клиент-серверного взаимодействия на основе единственного соединения проявляется в регулярности всплесков и падений активности, связанной с выделением динамической памяти для обработки подаваемых на входной порт сервера запросов. Для графика флуктуаций захвата ЦПУ характерны локальные отклонения от медианного значения, близкого к 10% от агрегированного числа квантов процессорного времени.  <br/>        

Итоги профилирования вычислительных процессов на базе сервера ```(./profile.sh -d 60 -e cpu -f  /tmp/flameoutput_cpu.svg <server_runner_pid>)``` в виде экземпляра <em>flamegraph:</em><br/>

![kindle1](https://user-images.githubusercontent.com/55311053/94413196-6ca49100-0183-11eb-9ab5-9a9bd68f951f.jpg)
<p align="center">Рис.1. Flamegraph выделения ресурса ЦПУ</p>


Анализируя полученный профиль, можно констатировать, что основными факторами, обеспечившими нагрузку на ЦПУ, оказались операции с буфером данных и сокетом на рабочем порту сервера. На поддержание функций селектора было затрачено около 88% процессорного времени, выделенного на осуществление операций локального сервера.<br/>           
  
Итоги профилирования процессов аллокации ```(/profile.sh -d 60 -e alloc -f  /tmp/flameoutput_allocation.svg <server_runner_pid>)```

![flameoutput11](https://user-images.githubusercontent.com/55311053/95015349-ff966d00-0654-11eb-8068-b38304bf19d7.jpg)
<p align="center">Рис.1. Flamegraph выделения RAM</p>

Визуализация использования RAM подтверждает превалирующий характер операций с буфером среди факторов, определяющих актуальную нагрузку на сервер.   

### 2) GET /v0/entity?id=<ID> <br/>

Консольный вывод wrk:<br/>
```
max@max-Inspiron-15-3573:~/hackdht$ sudo wrk -t1 -c1 -d5m -s src/profiling/wrk_scripts/get.lua -R2000 --latency http://127.0.0.1:8080/
Running 5m test @ http://127.0.0.1:8080/
  1 threads and 1 connections
  Thread calibration: mean lat.: 1.510ms, rate sampling interval: 10ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     8.16ms   64.67ms   1.00s    98.86%
    Req/Sec     2.14k   628.44    10.22k    82.78%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.27ms
 75.000%    1.96ms
 90.000%    4.08ms
 99.000%  196.35ms
 99.900%  918.02ms
 99.990%  994.82ms
 99.999%    1.00s 
100.000%    1.00s 

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.087     0.000000            2         1.00
       0.448     0.100000        58134         1.11
       0.731     0.200000       116060         1.25
       0.943     0.300000       174302         1.43
       1.098     0.400000       232318         1.67
       1.273     0.500000       290196         2.00
       1.395     0.550000       319151         2.22
       1.528     0.600000       348188         2.50
       1.664     0.650000       377063         2.86
       1.803     0.700000       406027         3.33
       1.962     0.750000       435006         4.00
       2.091     0.775000       449555         4.44
       2.305     0.800000       464052         5.00
       2.575     0.825000       478581         5.71
       2.957     0.850000       493011         6.67
       3.441     0.875000       507531         8.00
       3.737     0.887500       514773         8.89
       4.077     0.900000       522027        10.00
       4.515     0.912500       529270        11.43
       5.087     0.925000       536507        13.33
       5.887     0.937500       543763        16.00
       6.375     0.943750       547387        17.78
       6.947     0.950000       551005        20.00
       7.623     0.956250       554626        22.86
       8.447     0.962500       558241        26.67
       9.471     0.968750       561877        32.00
      10.087     0.971875       563698        35.56
      10.847     0.975000       565491        40.00
      11.855     0.978125       567313        45.71
      13.399     0.981250       569119        53.33
      16.319     0.984375       570928        64.00
      19.215     0.985938       571837        71.11
      24.783     0.987500       572740        80.00
     117.439     0.989062       573646        91.43
     248.063     0.990625       574551       106.67
     383.487     0.992188       575457       128.00
     450.047     0.992969       575910       142.22
     506.879     0.993750       576365       160.00
     572.415     0.994531       576819       182.86
     633.855     0.995313       577271       213.33
     694.271     0.996094       577724       256.00
     726.527     0.996484       577952       284.44
     762.879     0.996875       578177       320.00
     795.647     0.997266       578406       365.71
     823.807     0.997656       578633       426.67
     851.455     0.998047       578856       512.00
     865.279     0.998242       578971       568.89
     878.591     0.998437       579084       640.00
     894.463     0.998633       579198       731.43
     905.727     0.998828       579313       853.33
     920.063     0.999023       579425      1024.00
     925.695     0.999121       579480      1137.78
     934.911     0.999219       579535      1280.00
     943.103     0.999316       579594      1462.86
     952.831     0.999414       579650      1706.67
     960.511     0.999512       579706      2048.00
     965.119     0.999561       579736      2275.56
     970.239     0.999609       579764      2560.00
     973.823     0.999658       579792      2925.71
     977.919     0.999707       579819      3413.33
     982.015     0.999756       579847      4096.00
     984.575     0.999780       579863      4551.11
     986.623     0.999805       579877      5120.00
     988.671     0.999829       579891      5851.43
     991.231     0.999854       579907      6826.67
     993.279     0.999878       579921      8192.00
     993.791     0.999890       579925      9102.22
     995.327     0.999902       579935     10240.00
     995.839     0.999915       579939     11702.86
     997.375     0.999927       579947     13653.33
     998.399     0.999939       579955     16384.00
     998.911     0.999945       579958     18204.44
     999.423     0.999951       579962     20480.00
     999.935     0.999957       579966     23405.71
    1000.447     0.999963       579968     27306.67
    1000.959     0.999969       579971     32768.00
    1001.471     0.999973       579976     36408.89
    1001.471     0.999976       579976     40960.00
    1001.471     0.999979       579976     46811.43
    1001.983     0.999982       579978     54613.33
    1002.495     0.999985       579981     65536.00
    1002.495     0.999986       579981     72817.78
    1002.495     0.999988       579981     81920.00
    1003.007     0.999989       579982     93622.86
    1003.519     0.999991       579986    109226.67
    1003.519     0.999992       579986    131072.00
    1003.519     0.999993       579986    145635.56
    1003.519     0.999994       579986    163840.00
    1003.519     0.999995       579986    187245.71
    1003.519     0.999995       579986    218453.33
    1003.519     0.999996       579986    262144.00
    1004.031     0.999997       579988    291271.11
    1004.031     1.000000       579988          inf
#[Mean    =        8.156, StdDeviation   =       64.667]
#[Max     =     1003.520, Total count    =       579988]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  599998 requests in 5.00m, 38.80MB read
Requests/sec:   1999.99
Transfer/sec:    132.45KB


```
![flameoutput_getcpu](https://user-images.githubusercontent.com/55311053/95015918-62d5ce80-0658-11eb-9e13-7b53702ee6b1.jpg)

![flameoutput_getalloc](https://user-images.githubusercontent.com/55311053/95015924-6b2e0980-0658-11eb-994c-a351388eadea.jpg)

Результаты симулирования нагрузки в режиме генерации GET-запросов указывают на достижение более высоких задержек, чем при передаче проверочного запроса, рассмотренного на первом этапе текущего анализа. Средняя длительность задержки возросла на 0.3 мс при наблюдении идентичного отклонения (приблизительно 93%). Ожидание ответа сервера в 99,99% случаев длилось не более 34,5 мс, в 99,999% - не более 40 мс. Расширение интервалов отклика до указанных значений обусловлено спецификой имплементации запросов на получение данных из хранилища: при установлении наличия в нём целевого ключа сервер затрачивает дополнительное время на формирование ответа, содержащего соответствующее аргументу функции значение. <br/>

### 3) PUT /v0/entity?id=<ID><br/>

Консольный вывод wrk<br/>
```
max@max-Inspiron-15-3573:~/hackdht$ sudo wrk -t1 -c1 -d5m -s src/profiling/wrk_scripts/put.lua -R2000 --latency http://127.0.0.1:8080/
Running 5m test @ http://127.0.0.1:8080/
  1 threads and 1 connections
  Thread calibration: mean lat.: 1.770ms, rate sampling interval: 10ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.06ms    3.07ms  54.66ms   92.14%
    Req/Sec     2.13k   593.96     9.00k    83.01%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.22ms
 75.000%    1.92ms
 90.000%    4.25ms
 99.000%   15.45ms
 99.900%   34.81ms
 99.990%   48.13ms
 99.999%   53.09ms
100.000%   54.69ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.100     0.000000            1         1.00
       0.451     0.100000        58197         1.11
       0.690     0.200000       116215         1.25
       0.902     0.300000       174190         1.43
       1.067     0.400000       232368         1.67
       1.222     0.500000       290313         2.00
       1.312     0.550000       319014         2.22
       1.441     0.600000       348174         2.50
       1.589     0.650000       377098         2.86
       1.747     0.700000       406097         3.33
       1.917     0.750000       435151         4.00
       2.026     0.775000       449538         4.44
       2.203     0.800000       464049         5.00
       2.459     0.825000       478499         5.71
       2.845     0.850000       493044         6.67
       3.451     0.875000       507530         8.00
       3.821     0.887500       514779         8.89
       4.247     0.900000       522036        10.00
       4.743     0.912500       529295        11.43
       5.327     0.925000       536495        13.33
       6.051     0.937500       543774        16.00
       6.479     0.943750       547368        17.78
       6.987     0.950000       551004        20.00
       7.567     0.956250       554645        22.86
       8.247     0.962500       558241        26.67
       9.079     0.968750       561881        32.00
       9.567     0.971875       563698        35.56
      10.103     0.975000       565493        40.00
      10.743     0.978125       567326        45.71
      11.551     0.981250       569123        53.33
      12.583     0.984375       570931        64.00
      13.223     0.985938       571839        71.11
      13.935     0.987500       572743        80.00
      14.799     0.989062       573654        91.43
      15.919     0.990625       574552       106.67
      17.375     0.992188       575458       128.00
      18.239     0.992969       575914       142.22
      19.311     0.993750       576367       160.00
      20.399     0.994531       576820       182.86
      21.567     0.995313       577271       213.33
      23.023     0.996094       577729       256.00
      23.855     0.996484       577958       284.44
      24.815     0.996875       578177       320.00
      26.111     0.997266       578405       365.71
      27.535     0.997656       578631       426.67
      29.199     0.998047       578858       512.00
      30.111     0.998242       578972       568.89
      30.959     0.998437       579086       640.00
      32.095     0.998633       579200       731.43
      33.599     0.998828       579314       853.33
      34.975     0.999023       579423      1024.00
      35.743     0.999121       579482      1137.78
      36.575     0.999219       579538      1280.00
      37.823     0.999316       579595      1462.86
      38.815     0.999414       579651      1706.67
      39.551     0.999512       579708      2048.00
      40.095     0.999561       579736      2275.56
      41.215     0.999609       579764      2560.00
      42.207     0.999658       579791      2925.71
      43.583     0.999707       579820      3413.33
      44.383     0.999756       579850      4096.00
      44.863     0.999780       579863      4551.11
      45.215     0.999805       579878      5120.00
      45.759     0.999829       579890      5851.43
      47.103     0.999854       579905      6826.67
      47.743     0.999878       579920      8192.00
      47.935     0.999890       579926      9102.22
      48.287     0.999902       579933     10240.00
      48.831     0.999915       579940     11702.86
      49.535     0.999927       579947     13653.33
      49.983     0.999939       579954     16384.00
      50.463     0.999945       579958     18204.44
      50.815     0.999951       579961     20480.00
      51.167     0.999957       579965     23405.71
      51.519     0.999963       579968     27306.67
      51.967     0.999969       579973     32768.00
      52.095     0.999973       579974     36408.89
      52.223     0.999976       579975     40960.00
      52.383     0.999979       579977     46811.43
      52.511     0.999982       579980     54613.33
      52.575     0.999985       579981     65536.00
      52.767     0.999986       579982     72817.78
      52.767     0.999988       579982     81920.00
      53.087     0.999989       579983     93622.86
      53.439     0.999991       579984    109226.67
      53.695     0.999992       579985    131072.00
      54.079     0.999993       579986    145635.56
      54.079     0.999994       579986    163840.00
      54.079     0.999995       579986    187245.71
      54.431     0.999995       579987    218453.33
      54.431     0.999996       579987    262144.00
      54.591     0.999997       579988    291271.11
      54.591     0.999997       579988    327680.00
      54.591     0.999997       579988    374491.43
      54.591     0.999998       579988    436906.67
      54.591     0.999998       579988    524288.00
      54.687     0.999998       579989    582542.22
      54.687     1.000000       579989          inf
#[Mean    =        2.061, StdDeviation   =        3.070]
#[Max     =       54.656, Total count    =       579989]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  599999 requests in 5.00m, 38.34MB read
Requests/sec:   1999.99
Transfer/sec:    130.86KB

```

![flameoutput_putcpu](https://user-images.githubusercontent.com/55311053/95015640-a7f90100-0656-11eb-9a41-23a4b995593c.jpg)


![flameoutput_putalloc](https://user-images.githubusercontent.com/55311053/95015641-ab8c8800-0656-11eb-9082-4c6e82106366.jpg)


Сводные результаты операций по запросам PUT демонстрируют дальнейшее увеличение задержки отклика. Так, длительность получения данных от сервера в серии с генерацией PUT-запросов превосходит итоговый показатель для GET-запросов более на 8 мс применительно к квантилю 99,99% и приблизительно на 10 мс – к квантилю 99,999%. Прирост средней величины задержки составил 0.3% при достижении идентичного отклонения. Можно предположить, что отмеченные различия определены как большими временными издержками на добавление новых записей в хранилище, так и необходимостью обновления значений для ключей, присутствующих в нём на момент выполнения связанной функции <em>upsert</em>.<br/>    
  
### 4) DELETE /v0/entity?id=<ID><br/>
  
Консольный вывод wrk<br/> 
```
max@max-Inspiron-15-3573:~/hackdht$ sudo wrk -t1 -c1 -d5m -s src/profiling/wrk_scripts/delete.lua -R2000 --latency http://127.0.0.1:8080/
Running 5m test @ http://127.0.0.1:8080/
  1 threads and 1 connections
  Thread calibration: mean lat.: 1.698ms, rate sampling interval: 10ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.09ms    3.31ms  67.71ms   92.71%
    Req/Sec     2.13k   601.14     9.44k    82.81%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.23ms
 75.000%    1.90ms
 90.000%    4.18ms
 99.000%   17.10ms
 99.900%   37.38ms
 99.990%   59.62ms
 99.999%   65.73ms
100.000%   67.78ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.095     0.000000            2         1.00
       0.446     0.100000        58171         1.11
       0.701     0.200000       116021         1.25
       0.914     0.300000       174163         1.43
       1.074     0.400000       232234         1.67
       1.230     0.500000       290282         2.00
       1.330     0.550000       319270         2.22
       1.462     0.600000       348215         2.50
       1.599     0.650000       377214         2.86
       1.747     0.700000       406087         3.33
       1.902     0.750000       435097         4.00
       2.003     0.775000       449525         4.44
       2.165     0.800000       464062         5.00
       2.417     0.825000       478596         5.71
       2.777     0.850000       493028         6.67
       3.371     0.875000       507556         8.00
       3.749     0.887500       514800         8.89
       4.183     0.900000       522024        10.00
       4.695     0.912500       529298        11.43
       5.295     0.925000       536524        13.33
       6.031     0.937500       543787        16.00
       6.467     0.943750       547414        17.78
       6.975     0.950000       551027        20.00
       7.555     0.956250       554663        22.86
       8.279     0.962500       558306        26.67
       9.151     0.968750       561922        32.00
       9.687     0.971875       563709        35.56
      10.335     0.975000       565534        40.00
      11.143     0.978125       567337        45.71
      12.199     0.981250       569157        53.33
      13.503     0.984375       570960        64.00
      14.311     0.985938       571868        71.11
      15.255     0.987500       572775        80.00
      16.367     0.989062       573683        91.43
      17.663     0.990625       574589       106.67
      19.087     0.992188       575491       128.00
      20.031     0.992969       575945       142.22
      21.055     0.993750       576398       160.00
      22.351     0.994531       576853       182.86
      23.743     0.995313       577304       213.33
      25.647     0.996094       577759       256.00
      26.815     0.996484       577984       284.44
      28.015     0.996875       578209       320.00
      29.471     0.997266       578436       365.71
      31.055     0.997656       578663       426.67
      32.639     0.998047       578890       512.00
      33.567     0.998242       579006       568.89
      34.399     0.998437       579117       640.00
      35.487     0.998633       579232       731.43
      36.351     0.998828       579344       853.33
      37.567     0.999023       579459      1024.00
      38.303     0.999121       579512      1137.78
      39.135     0.999219       579568      1280.00
      40.223     0.999316       579627      1462.86
      41.311     0.999414       579682      1706.67
      42.399     0.999512       579740      2048.00
      42.879     0.999561       579770      2275.56
      43.263     0.999609       579795      2560.00
      43.711     0.999658       579823      2925.71
      44.511     0.999707       579852      3413.33
      45.791     0.999756       579880      4096.00
      47.359     0.999780       579895      4551.11
      49.663     0.999805       579908      5120.00
      51.583     0.999829       579922      5851.43
      55.679     0.999854       579937      6826.67
      57.663     0.999878       579951      8192.00
      58.847     0.999890       579958      9102.22
      59.775     0.999902       579965     10240.00
      60.799     0.999915       579972     11702.86
      61.471     0.999927       579979     13653.33
      62.175     0.999939       579986     16384.00
      62.559     0.999945       579990     18204.44
      62.847     0.999951       579993     20480.00
      63.103     0.999957       579997     23405.71
      63.487     0.999963       580000     27306.67
      63.903     0.999969       580004     32768.00
      64.255     0.999973       580006     36408.89
      64.287     0.999976       580007     40960.00
      64.703     0.999979       580009     46811.43
      65.087     0.999982       580011     54613.33
      65.471     0.999985       580013     65536.00
      65.599     0.999986       580014     72817.78
      65.599     0.999988       580014     81920.00
      65.727     0.999989       580015     93622.86
      65.855     0.999991       580016    109226.67
      66.303     0.999992       580017    131072.00
      66.687     0.999993       580018    145635.56
      66.687     0.999994       580018    163840.00
      66.687     0.999995       580018    187245.71
      67.071     0.999995       580019    218453.33
      67.071     0.999996       580019    262144.00
      67.455     0.999997       580020    291271.11
      67.455     0.999997       580020    327680.00
      67.455     0.999997       580020    374491.43
      67.455     0.999998       580020    436906.67
      67.455     0.999998       580020    524288.00
      67.775     0.999998       580021    582542.22
      67.775     1.000000       580021          inf
#[Mean    =        2.095, StdDeviation   =        3.311]
#[Max     =       67.712, Total count    =       580021]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  599997 requests in 5.00m, 38.91MB read
Requests/sec:   1999.99
Transfer/sec:    132.81KB

```

![flameoutput_deletecpu](https://user-images.githubusercontent.com/55311053/95016064-4f773300-0659-11eb-8b87-471c8adaee28.jpg)

![flameoutput_deletealloc](https://user-images.githubusercontent.com/55311053/95016062-4e460600-0659-11eb-96df-166b0589d72b.jpg)

Для запросов на удаление записей установлена максимальная оценка средней задержки, кратно (более чем в 2 раза) превышающая аналогичный показатель для запросов на добавление/обновление элементов хранилища (при близких к идентичным значениях стандартного отклонения). Предельное время отклика, применительно к релевантным квантилям, увеличилось на 69 мс как для 99,99%, так и для 99,999% операций обмена данными. Столь негативный эффект можно рассматривать как следствие интенсивного уменьшения объёма данных одновременно с динамической оптимизацией структуры хранилища под влиянием серийного исключения записей.

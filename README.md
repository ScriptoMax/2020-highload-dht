# Мониторинг работы сервера с поддержкой асинхронных операций (<em>wrk</em> + <em>async-profiler</em>)

**Система и программные средства** 
| | |
|-|-|
| ОС | Ubuntu Linux 18.04 LTS x64-bit |
| ЦПУ | Intel(R) Celeron(R) N4000 CPU @ 1.10GHz |
| Объём RAM | 8 ГБ |
| Количество ядер ЦПУ | 2 |
| [wrk2](https://github.com/giltene/wrk2) | v. 4.0.0 |
| [async-profiler](https://github.com/jvm-profiling-tools/async-profiler) | 1.8.1 |
| [VisualVM](https://visualvm.github.io/) | 2.0.4 |

В ходе очередного мониторинга производительности локального highload-сервера получены и рассмотрены оценки быстродействия при асинхронной обработке запросов от множества клиентов. Для определения технических (программн-аппаратных) факторов изменений, вносимых асинхронными процессами, предпринят сравнительный анализ результатов профилирования асинхронной (<em>async-featured</em>) и синхронизированной на уровне потоков (<em>thread safe synchronized</em>) реализаций программного управления сервером (класс TaskService). В качестве материалов, характеризующих производительность сервера с поддержкой синхронизации, приводятся статистические сводки и визуализации итогов профилирования (flame-графы) в расширенной конфигурации RocksDB, подвергнутые анализу в контексте исследования на 2-ом этапе проекта.<br/> 
Нагрузочные испытания, проведённые с использованием <em>wrk2</em>, организованы в виде серий PUT- и GET-запросов фиксированной продолжительности (7 минут) в режиме симулирования обмена данными с пулом клиентов (число клиентов (соединений с сервером) в synchronized-реализации установлено равным 16, в асинхронной - 64, количество параллельных потоков в обеих - 2 по числу ядер ЦПУ). Интенсивность генерации / отправки запросов каждого вида (<em>Rate</em>) задана равной 15000 запросов/с исходя из предварительной оценки стабильной нагрузки, де-факто достигаемой в ходе выполнения <em>wrk2</em> на локальном компьютере.

**Команды <em>wrk2</em>**<br/>
<ins><em>wrk2</em> | PUT | thread safe synchronized</ins>
```
wrk -t2 -c16 -d7m -s src/profiling/wrk_scripts/put.lua -R15000 --latency http://127.0.0.1:8080
```

<ins><em>wrk2</em> | PUT | async-featured</ins>
```
wrk -t2 -c64 -d7m -s src/profiling/wrk_scripts/put.lua -R15000 --latency http://127.0.0.1:8080
```

<ins><em>wrk2</em> | GET | thread safe synchronized</ins>
```
wrk -t2 -c16 -d7m -s src/profiling/wrk_scripts/get.lua -R15000 --latency http://127.0.0.1:8080
```

<ins><em>wrk2</em> | GET | async-featured</ins>
```
wrk -t2 -c64 -d7m -s src/profiling/wrk_scripts/get.lua -R15000 --latency http://127.0.0.1:8080
```

**Команды <em>async-profiler</em>**<br/>
<ins><em>async-profiler</em> | cpu</ins>
```
./profiler.sh -d 60 -e cpu -t -f /path/to/output/folder/flame_output_cpu.svg <server_process_pid>
```

<ins><em>async-profiler</em> | alloc</ins>
```
./profiler.sh -d 60 -e alloc -t -f /path/to/output/folder/flame_output_alloc.svg <server_process_pid>
```

<ins><em>async-profiler</em> | lock</ins>
```
./profiler.sh -d 60 -e lock -t -f /path/to/output/folder/flame_output_lock.svg <server_pid>
```

Результаты мониторинга и сравнения реализаций сервера приведены далее.

### 1. Добавление/изменение записей (PUT)

<ins><em>wrk2</em> outputs / **synchronized** server /</ins>  
```
max@max-Inspiron-15-3573:~/hackdht$ wrk -t2 -c16 -d7m -s src/profiling/wrk_scripts/put.lua -R15000 --latency http://127.0.0.1:8080
Running 7m test @ http://127.0.0.1:8080
  2 threads and 16 connections
  Thread calibration: mean lat.: 81.369ms, rate sampling interval: 629ms
  Thread calibration: mean lat.: 42.386ms, rate sampling interval: 378ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.41ms    8.48ms 195.58ms   98.42%
    Req/Sec     7.51k   127.15     9.60k    96.66%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.34ms
 75.000%    1.93ms
 90.000%    2.65ms
 99.000%   28.98ms
 99.900%  126.91ms
 99.990%  162.82ms
 99.999%  192.00ms
100.000%  195.71ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.073     0.000000            1         1.00
       0.552     0.100000       615076         1.11
       0.776     0.200000      1232597         1.25
       0.969     0.300000      1846106         1.43
       1.155     0.400000      2462421         1.67
       1.342     0.500000      3074714         2.00
       1.442     0.550000      3383007         2.22
       1.548     0.600000      3691114         2.50
       1.662     0.650000      3999187         2.86
       1.787     0.700000      4305947         3.33
       1.929     0.750000      4612224         4.00
       2.009     0.775000      4766984         4.44
       2.097     0.800000      4921152         5.00
       2.197     0.825000      5075966         5.71
       2.313     0.850000      5229015         6.67
       2.457     0.875000      5381312         8.00
       2.545     0.887500      5457682         8.89
       2.653     0.900000      5534727        10.00
       2.791     0.912500      5612181        11.43
       2.977     0.925000      5688329        13.33
       3.273     0.937500      5765359        16.00
       3.515     0.943750      5803611        17.78
       3.899     0.950000      5841968        20.00
       4.523     0.956250      5880320        22.86
       5.387     0.962500      5918822        26.67
       6.359     0.968750      5957236        32.00
       6.867     0.971875      5976548        35.56
       7.439     0.975000      5995641        40.00
       8.163     0.978125      6014901        45.71
       9.239     0.981250      6034099        53.33
      11.031     0.984375      6053283        64.00
      12.503     0.985938      6062882        71.11
      15.415     0.987500      6072501        80.00
      22.927     0.989062      6082096        91.43
      33.375     0.990625      6091705       106.67
      47.295     0.992188      6101314       128.00
      54.815     0.992969      6106120       142.22
      61.983     0.993750      6110922       160.00
      68.927     0.994531      6115764       182.86
      76.799     0.995313      6120544       213.33
      84.607     0.996094      6125358       256.00
      88.703     0.996484      6127771       284.44
      92.543     0.996875      6130139       320.00
      96.959     0.997266      6132552       365.71
     101.951     0.997656      6134958       426.67
     106.687     0.998047      6137349       512.00
     110.463     0.998242      6138554       568.89
     114.943     0.998437      6139758       640.00
     118.847     0.998633      6140961       731.43
     122.751     0.998828      6142148       853.33
     127.487     0.999023      6143360      1024.00
     130.303     0.999121      6143968      1137.78
     132.991     0.999219      6144566      1280.00
     135.423     0.999316      6145155      1462.86
     138.367     0.999414      6145776      1706.67
     141.311     0.999512      6146363      2048.00
     143.359     0.999561      6146656      2275.56
     145.535     0.999609      6146956      2560.00
     147.839     0.999658      6147262      2925.71
     150.143     0.999707      6147560      3413.33
     152.319     0.999756      6147863      4096.00
     153.215     0.999780      6148008      4551.11
     154.879     0.999805      6148169      5120.00
     156.159     0.999829      6148306      5851.43
     157.823     0.999854      6148458      6826.67
     159.871     0.999878      6148604      8192.00
     161.279     0.999890      6148688      9102.22
     163.071     0.999902      6148755     10240.00
     164.607     0.999915      6148832     11702.86
     166.399     0.999927      6148905     13653.33
     168.831     0.999939      6148980     16384.00
     170.239     0.999945      6149019     18204.44
     171.775     0.999951      6149056     20480.00
     173.823     0.999957      6149093     23405.71
     176.639     0.999963      6149130     27306.67
     180.991     0.999969      6149168     32768.00
     183.423     0.999973      6149186     36408.89
     185.983     0.999976      6149204     40960.00
     187.775     0.999979      6149224     46811.43
     188.799     0.999982      6149242     54613.33
     189.823     0.999985      6149263     65536.00
     190.207     0.999986      6149271     72817.78
     190.719     0.999988      6149279     81920.00
     191.871     0.999989      6149290     93622.86
     192.255     0.999991      6149299    109226.67
     192.767     0.999992      6149309    131072.00
     192.895     0.999993      6149315    145635.56
     193.151     0.999994      6149317    163840.00
     193.535     0.999995      6149322    187245.71
     193.791     0.999995      6149327    218453.33
     193.919     0.999996      6149333    262144.00
     193.919     0.999997      6149333    291271.11
     194.175     0.999997      6149336    327680.00
     194.431     0.999997      6149339    374491.43
     194.559     0.999998      6149341    436906.67
     194.687     0.999998      6149344    524288.00
     194.687     0.999998      6149344    582542.22
     194.815     0.999998      6149347    655360.00
     194.815     0.999999      6149347    748982.86
     194.815     0.999999      6149347    873813.33
     195.071     0.999999      6149349   1048576.00
     195.071     0.999999      6149349   1165084.44
     195.199     0.999999      6149350   1310720.00
     195.199     0.999999      6149350   1497965.71
     195.327     0.999999      6149351   1747626.67
     195.455     1.000000      6149353   2097152.00
     195.455     1.000000      6149353   2330168.89
     195.455     1.000000      6149353   2621440.00
     195.455     1.000000      6149353   2995931.43
     195.455     1.000000      6149353   3495253.33
     195.455     1.000000      6149353   4194304.00
     195.455     1.000000      6149353   4660337.78
     195.455     1.000000      6149353   5242880.00
     195.455     1.000000      6149353   5991862.86
     195.711     1.000000      6149354   6990506.67
     195.711     1.000000      6149354          inf
#[Mean    =        2.406, StdDeviation   =        8.479]
#[Max     =      195.584, Total count    =      6149354]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  6299674 requests in 7.00m, 402.53MB read
Requests/sec:  14999.26
Transfer/sec:      0.96MB
```
<ins><em>wrk2</em> outputs / **async** server /</ins>  

В соответствии с приведённым выводом средняя задержка при обработке запросов на добавление записей достигла 700 мс при отклонении на уровне 87%. Каждую секунду сервер выполнял свыше 7500 запросов, в то время как интенсивность их подачи оказалась близкой целевому значению (15000 запросов/с). В структуре квантильного распределения времён отклика максимальная длительность ответа установлена равной 6.77 с, получение отклика на 90% запросов уложилось в 6.5 с.<br/>            
<ins>Flamegraph-анализ</ins><br/>  

![put_cpu_64](https://user-images.githubusercontent.com/55311053/95678810-a8a21200-0bd7-11eb-848f-4acf15daaf6f.jpg)
<p align="center">Рис.1. Выделение ресурса CPU при симулировании PUT-запросов (async-featured)</p>

![put_cpu_ext](https://user-images.githubusercontent.com/55311053/95623828-9791bd80-0a7e-11eb-984b-fd61321ec7f1.jpg)
<p align="center">Рис.2. Выделение ресурса CPU при симулировании PUT-запросов (thread safe synchronized)</p>

![put_alloc_64](https://user-images.githubusercontent.com/55311053/95678826-b3f53d80-0bd7-11eb-8384-97e655673c35.jpg)
<p align="center">Рис.3. Выделение ресурса RAM при симулировании PUT-запросов (async-featured)</p>

![put_alloc_ext](https://user-images.githubusercontent.com/55311053/95623757-7b8e1c00-0a7e-11eb-8e04-d06a5f805681.jpg)
<p align="center">Рис.4. Выделение ресурса RAM при симулировании PUT-запросов (thread safe synchronized)</p>

![put_lock_64](https://user-images.githubusercontent.com/55311053/95678890-19e1c500-0bd8-11eb-976f-0818a87e6c88.jpg)
<p align="center">Рис.5. Профиль lock/monitor при симулировании PUT-запросов (async-featured)</p>

Визуализация нагрузки на процессор позволяет сделать вывод о доминировании в общей структуре операций, связанных с выполнением ThreadPoolExecutor. Значительную долю процессорного времени составили чтение, парсинг данных из буфера вкупе с формированием и отправкой результирующих данных. Влияние многопоточности прослеживается и на графе выделения памяти. На нём же следует отметить активное аллоцирование RAM при вызове метода асинхронной обработки запросов (<em>pushAsyncRun</em>). <br/>               

### 2. Чтение записей (GET)

<ins><em>wrk2</em> outputs / **synchronized** server /</ins>  
```
max@max-Inspiron-15-3573:~/hackdht$ wrk -t2 -c16 -d7m -s src/profiling/wrk_scripts/get.lua -R15000 --latency http://127.0.0.1:8080
Running 7m test @ http://127.0.0.1:8080
  2 threads and 16 connections
  Thread calibration: mean lat.: 10.730ms, rate sampling interval: 75ms
  Thread calibration: mean lat.: 25.269ms, rate sampling interval: 181ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.56ms    1.26ms  30.03ms   90.42%
    Req/Sec     7.54k   164.07     9.27k    88.29%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.35ms
 75.000%    1.91ms
 90.000%    2.52ms
 99.000%    7.32ms
 99.900%   12.70ms
 99.990%   20.72ms
 99.999%   27.14ms
100.000%   30.05ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.067     0.000000            1         1.00
       0.536     0.100000       615152         1.11
       0.759     0.200000      1230269         1.25
       0.964     0.300000      1846650         1.43
       1.158     0.400000      2462626         1.67
       1.347     0.500000      3077440         2.00
       1.445     0.550000      3385106         2.22
       1.548     0.600000      3690354         2.50
       1.659     0.650000      3999399         2.86
       1.777     0.700000      4306297         3.33
       1.908     0.750000      4613448         4.00
       1.980     0.775000      4766459         4.44
       2.059     0.800000      4920809         5.00
       2.147     0.825000      5073956         5.71
       2.247     0.850000      5227495         6.67
       2.365     0.875000      5381418         8.00
       2.435     0.887500      5457756         8.89
       2.517     0.900000      5534543        10.00
       2.615     0.912500      5611430        11.43
       2.737     0.925000      5688652        13.33
       2.903     0.937500      5765123        16.00
       3.013     0.943750      5803583        17.78
       3.153     0.950000      5842400        20.00
       3.333     0.956250      5880387        22.86
       3.599     0.962500      5918869        26.67
       4.019     0.968750      5957271        32.00
       4.331     0.971875      5976484        35.56
       4.727     0.975000      5995808        40.00
       5.203     0.978125      6014999        45.71
       5.723     0.981250      6034186        53.33
       6.259     0.984375      6053320        64.00
       6.539     0.985938      6063015        71.11
       6.819     0.987500      6072579        80.00
       7.119     0.989062      6082191        91.43
       7.451     0.990625      6091746       106.67
       7.835     0.992188      6101407       128.00
       8.051     0.992969      6106194       142.22
       8.295     0.993750      6110956       160.00
       8.583     0.994531      6115777       182.86
       8.911     0.995313      6120558       213.33
       9.327     0.996094      6125406       256.00
       9.567     0.996484      6127749       284.44
       9.855     0.996875      6130165       320.00
      10.183     0.997266      6132565       365.71
      10.583     0.997656      6134969       426.67
      11.047     0.998047      6137372       512.00
      11.311     0.998242      6138576       568.89
      11.607     0.998437      6139763       640.00
      11.935     0.998633      6140977       731.43
      12.311     0.998828      6142176       853.33
      12.759     0.999023      6143359      1024.00
      13.031     0.999121      6143975      1137.78
      13.327     0.999219      6144573      1280.00
      13.663     0.999316      6145172      1462.86
      14.071     0.999414      6145767      1706.67
      14.567     0.999512      6146364      2048.00
      14.895     0.999561      6146664      2275.56
      15.247     0.999609      6146963      2560.00
      15.759     0.999658      6147264      2925.71
      16.327     0.999707      6147567      3413.33
      17.071     0.999756      6147864      4096.00
      17.487     0.999780      6148016      4551.11
      17.935     0.999805      6148166      5120.00
      18.543     0.999829      6148316      5851.43
      19.119     0.999854      6148469      6826.67
      19.855     0.999878      6148616      8192.00
      20.303     0.999890      6148689      9102.22
      20.815     0.999902      6148765     10240.00
      21.375     0.999915      6148840     11702.86
      22.095     0.999927      6148918     13653.33
      22.975     0.999939      6148991     16384.00
      23.487     0.999945      6149027     18204.44
      24.175     0.999951      6149065     20480.00
      24.735     0.999957      6149102     23405.71
      25.247     0.999963      6149142     27306.67
      25.615     0.999969      6149178     32768.00
      25.775     0.999973      6149196     36408.89
      25.999     0.999976      6149214     40960.00
      26.175     0.999979      6149233     46811.43
      26.415     0.999982      6149252     54613.33
      26.655     0.999985      6149271     65536.00
      26.783     0.999986      6149280     72817.78
      26.959     0.999988      6149291     81920.00
      27.039     0.999989      6149299     93622.86
      27.247     0.999991      6149308    109226.67
      27.535     0.999992      6149318    131072.00
      27.647     0.999993      6149322    145635.56
      27.823     0.999994      6149328    163840.00
      27.935     0.999995      6149332    187245.71
      28.175     0.999995      6149336    218453.33
      28.383     0.999996      6149341    262144.00
      28.463     0.999997      6149344    291271.11
      28.559     0.999997      6149346    327680.00
      28.655     0.999997      6149348    374491.43
      28.687     0.999998      6149350    436906.67
      28.895     0.999998      6149355    524288.00
      28.895     0.999998      6149355    582542.22
      28.895     0.999998      6149355    655360.00
      29.263     0.999999      6149356    748982.86
      29.359     0.999999      6149357    873813.33
      29.407     0.999999      6149359   1048576.00
      29.407     0.999999      6149359   1165084.44
      29.535     0.999999      6149361   1310720.00
      29.535     0.999999      6149361   1497965.71
      29.535     0.999999      6149361   1747626.67
      29.903     1.000000      6149362   2097152.00
      29.903     1.000000      6149362   2330168.89
      29.903     1.000000      6149362   2621440.00
      29.903     1.000000      6149362   2995931.43
      29.999     1.000000      6149363   3495253.33
      29.999     1.000000      6149363   4194304.00
      29.999     1.000000      6149363   4660337.78
      29.999     1.000000      6149363   5242880.00
      29.999     1.000000      6149363   5991862.86
      30.047     1.000000      6149364   6990506.67
      30.047     1.000000      6149364          inf
#[Mean    =        1.564, StdDeviation   =        1.262]
#[Max     =       30.032, Total count    =      6149364]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  6299677 requests in 7.00m, 412.42MB read
  Non-2xx or 3xx responses: 2
Requests/sec:  14998.24
Transfer/sec:      0.98MB
```
<ins><em>wrk2</em> outputs / **async** server /</ins>  

Приведённые результаты отражают существенное снижение времени отклика в сравнении с оценками, актуальными для операций добавления. При средней задержке в 18 мс (отклонение на уровне 96%) сервер получал в обработку более 3200 запросов в каждую единицу времени. В 99% случаев время ожидания отклика не превысило 600 мс, худший результат оказался в пределах 1 с.<br/>                  

<ins>Flamegraph-анализ</ins><br/>  

![get_cpu_64](https://user-images.githubusercontent.com/55311053/95678755-5b25a500-0bd7-11eb-8f85-5e19bfc3a506.jpg)
<p align="center">Рис.6. Выделение ресурса CPU при симулировании GET-запросов (async-featured)</p>

![get_cpu_ext](https://user-images.githubusercontent.com/55311053/95623668-60231100-0a7e-11eb-97c3-76c8c4280630.jpg)
<p align="center">Рис.7. Выделение ресурса CPU при симулировании GET-запросов (thread safe synchronized)</p>

![get_alloc_64](https://user-images.githubusercontent.com/55311053/95678769-709acf00-0bd7-11eb-89bb-3cdb83b78cf6.jpg)
<p align="center">Рис.8. Выделение ресурса RAM при симулировании GET-запросов (async-featured)</p>

![get_alloc_ext](https://user-images.githubusercontent.com/55311053/95623573-2fdb7280-0a7e-11eb-99b8-fad5d51d8f5f.jpg)
<p align="center">Рис.9. Выделение ресурса RAM при симулировании GET-запросов (thread safe synchronized)</p>

![get_lock_64](https://user-images.githubusercontent.com/55311053/95678783-83ad9f00-0bd7-11eb-84a0-2b8aa5bd8314.jpg)
<p align="center">Рис.10. Профиль lock/monitor при симулировании GET-запросов (async-featured)</p>

Представленные графы в значительной мере воспроизводят профили, полученные для серии с операциями добавления. Как и в случае с последними, в структуре использования процессорного времени превалируют операции с ThreadPoolExecutor, буфером данных и сокетом. Дополнительную нагрузку на ресурс RAM создают операции преобразования типов данных между байтовыми и строковыми объектами.

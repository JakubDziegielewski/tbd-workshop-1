0. The goal of phase 2b is to perform benchmarking/scalability tests of sample three-tier lakehouse solution.

1. In main.tf, change machine_type at:

```
module "dataproc" {
  depends_on   = [module.vpc]
  source       = "github.com/bdg-tbd/tbd-workshop-1.git?ref=v1.0.36/modules/dataproc"
  project_name = var.project_name
  region       = var.region
  subnet       = module.vpc.subnets[local.notebook_subnet_id].id
  machine_type = "e2-standard-2"
}
```

and subsititute "e2-standard-2" with "e2-standard-4".

2. If needed request to increase cpu quotas (e.g. to 30 CPUs): 
https://console.cloud.google.com/apis/api/compute.googleapis.com/quotas?project=tbd-2023z-9918

3. Using tbd-tpc-di notebook perform dbt run with different number of executors, i.e., 1, 2, and 5, by changing:
```
 "spark.executor.instances": "2"
```

in profiles.yml.

4. In the notebook, collect console output from dbt run, then parse it and retrieve total execution time and execution times of processing each model. Save the results from each number of executors. 


    Zgodnie z instrukcją, przeprowadzono 3 eksperymenty dla wartości parametru `spark.executor.instances` ze zbioru {1, 2, 4}. Dokładny nieprzetworzony output zebrany z notebook'a w plikach tekstowych   znajduje się w folderze [dbt_run_outputs](https://github.com/JakubDziegielewski/tbd-workshop-1/edit/sprawko/dbt_run_outputs).


5. Analyze the performance and scalability of execution times of each model. Visualize and discucss the final results.

  Poniższa tabela zawiera dane o czasach wykonania dla każdego z modeli (w sekundach) w zależności od liczby egzekutorów.

| model \ instances                           | 1        | 2        | 4        |
|---------------------------------------------|----------|----------|----------|
| demo_bronze.brokerage_cash_transaction      | 55,65    | 27,35    | 28,62    |
| demo_bronze.brokerage_daily_market          | 337,88   | 76,19    | 52,61    |
| demo_bronze.brokerage_holding_history       | 51,4     | 9,58     | 7,04     |
| demo_bronze.brokerage_trade                 | 151,07   | 39,96    | 34,27    |
| demo_bronze.brokerage_trade_history         | 122,01   | 35,83    | 23,6     |
| demo_bronze.brokerage_watch_history         | 162,37   | 39,12    | 36,68    |
| demo_bronze.crm_customer_mgmt               | 16,96    | 6,86     | 6,31     |
| demo_bronze.finwire_company                 | 6,06     | 1,52     | 2,1      |
| demo_bronze.finwire_financial               | 113,35   | 41,01    | 35,6     |
| demo_bronze,finwire_security                | 3,23     | 1,61     | 2,28     |
| demo_bronze.hr_employee                     | 3,24     | 2,51     | 2,08     |
| demo_bronze.reference_date                  | 1,41     | 1,51     | 1,21     |
| demo_bronze.reference_industry              | 0,92     | 0,77     | 0,84     |
| demo_bronze.reference_status_type           | 0,91     | 0,97     | 0,84     |
| demo_bronze.reference_tax_rate              | 0,96     | 1        | 0,76     |
| demo_bronze.reference_trade_type            | 0,91     | 0,93     | 0,8      |
| demo_bronze.syndicated_prospect             | 5,69     | 3,99     | 3,02     |
| demo_silver.daily_market                    | 2344,51  | 1014,74  | 581,32   |
| demo_silver.employees                       | 4,13     | 2,3      | 2,77     |
| demo_silver.date                            | 1,67     | 1,18     | 1,3      |
| demo_silver.companies                       | 6,38     | 5,94     | 5,17     |
| demo_silver.accounts                        | 13,97    | 8,22     | 9,9      |
| demo_silver.customers                       | 11,07    | 6,95     | 7,98     |
| demo_silver.trades_history                  | 487,08   | 255,86   | 170,44   |
| demo_gold.dim_broker                        | 5,04     | 2,92     | 2,74     |
| demo_gold.dim_date                          | 1,53     | 1,06     | 1,43     |
| demo_gold.dim_company                       | 3,07     | 1,89     | 2,51     |
| demo_silver.financials                      | 167,05   | 56,79    | 49,52    |
| demo_silver.securities                      | 5,67     | 3,75     | 4,73     |
| demo_silver.cash_transactions               | 239,27   | 41,37    | 26,27    |
| demo_gold.dim_customer                      | 31,96    | 12,33    | 15,34    |
| demo_gold.dim_trade                         | 762,87   | 168,44   | 116,2    |
| demo_silver.trades                          | 766,34   | 187,9    | 128,18   |
| demo_gold.dim_security                      | 4,52     | 2,78     | 9,52     |
| demo_silver.watches_history                 | 298,68   | 85,36    | 346,79   |
| demo_gold.dim_account                       | 49,02    | 7,89     | 26,63    |
| demo_silver.holdings_history                | 529,67   | 78,47    | 168,93   |
| demo_silver.watches                         | 472,84   | 115,38   | 288,25   |
| demo_gold.fact_cash_transactions            | 358,37   | 48,3     | 229,48   |
| demo_gold.fact_trade                        | 874,98   | 136,28   | 555,32   |
| demo_gold.fact_holdings                     | 1537,33  | 750,5    | 918,44   |
| demo_gold.fact_watches                      | 537,88   | 325,61   | 243,96   |
| demo_gold.fact_cash_balances                | 252,97   | 143,09   | 212,36   |
| TOTAL                                       | 10801,89 | 3756,01  | 4364,14  |


Poniższa tabela zawiera zagregowane dane o czasach wykonania dla każdej z warstw (w sekundach) w zależności od liczby egzekutorów.

| layer \ instances   | 1        | 2        | 4        |
|---------------------|----------|----------|----------|
| demo_bronze         | 1034,02  | 290,71   | 238,66   |
| demo_silver         | 5348,33  | 1864,21  | 1791,55  |
| demo_gold           | 4419,54  | 1601,09  | 2333,93  |
| TOTAL               | 10801,89 | 3756,01  | 4364,14  |


Na poniższym wykresie przedstawiono dane o czasie wykonania każdego z modelu z podziałem na liczbę egzekutorów - 1 (niebieski), 2 (pomarańczowy) lub 4 (zielony).

![img_png](doc/figures/time_exec_comparison.png)

Patrząc na dane przedstawione w tabelach oraz na wykresie powyżej można zauważyć kilka rzeczy:

a) Sumarycznie najkrótszy czas całkowitego przetwarzania miał miejsce w przypadku, gdy uruchomiono tylko 2 instacje egzekutora - łącznie ok 1 godz 2 min. Nieznacznie więcej trwało to w przypadku 4 egzekutorów - 1 godz 12 min. Zauważalnie dłużej zajęło przetworzenie wszystkich modeli przez 1 instancję egzekutora - ponad 3 godz. O ile spodziewaliśmy się, że dla 1 egzekutora przetworzenie wszystkich modeli będzie o wiele dłuższe niż dla większej ich liczby, to nie spodziewaliśmy się, że lepsze wyniki uzyskane zostaną dla 2 niż dla 4 egzekutorów. Dla 4 egzekutorów najdłużej przetwarzane są modele z warstwy demo_gold - w warstwach demo_bronze oraz demo_silver 4 instancje egzekutora dają bowiem najszybsze wyniki.

b) Jak widać na wykresie, jest bardzo duża dysproporcja pomiędzy czasami przetwarzania poszczególnych modeli. Większość z nich jest bowiem przewarzana w kilka - kilkadziesiąt sekund - różnica pomiędzy liczbą wykorzystanych instancji egzekutora nie jest zatem w takich przypadkach bardzo widoczna. Inaczej sprawa ma się w przypadku większych modeli, jak np. demo_silver.daily_market. Na jego przykładzie świetnie widać, że zwiększenie liczby egzekutorów z 1 do 2 pomaga ograniczyć czas przetwarzania o ponad połowę. Choć zysk ze zwiększenia liczby egzekutorów z 2 do 4 dla tego przypadku jest relatywnie mniejszy od uzysku z 1 do 2, wciąż jest on istotny (znów prawie dwukrotny zysk na czasie). Dla tego dużego modelu skalowanie przebiega zatem zgodnie z oczekiwaniami.

c) Mamy też jednak inne modele, jak np. demo_gold.dim_trade lub demo_silver.trades, gdzie uzysk ze zwiększenia liczby instancji egzekutorów z 1 do 2 jest potężny (ponad 4-krotny), a zwiększenie z 2 do 4 przynosi zysk, lecz nieznaczny. Istnieje również trzecia grupa modeli, dla których zwiększenie liczby egzekutorów z 1 do 2 przynosi istotną poprawę, lecz zwiększenie z 2 do 4 wręcz pogarsza (czasem nawet 3-4 krotnie) czas przetwarzania. Przykładami takich modeli są demo_gold.fact_trade oraz demo_silver.watches. Skalowanie w przypadku tych grup nie przebiega zatem zgodnie z początkową intuicją, co może wynikać z ograniczonych możliwości zrównoleglania operacji, których obsłużenie w przypadku niektórych modeli wprowadza wręcz niepotrzebny narzut.

Podsumowując, ciężko jednoznacznie stwierdzić, co wpływa na to, czy skalowalność postępuje zgodnie z intuicją, czy też nie - prawdopodobnie jest to charakterystyka modelu. Niektóre modele skalują się podręcznikowo, jak np. demo_silver.daily_market, a niektóre przeczą pierwotnej intuicji, gdyż zwiększenie liczby egzekutorów wręcz wydłuża czas przetwarzania, jak w przypadku demo_gold.fact_trade. Bezpiecznym zdaje się zatem zalecenie użycia 2 egzekutorów. Gdybyśmy mieli do przetworzenia więcej takich modeli jak np. demo_silver.daily_market, warto byłoby rozważyć użycie większej liczby instancji egzekutorów.





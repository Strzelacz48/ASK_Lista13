# ASK_Lista13
[Link do listy na hackMD](https://hackmd.io/N4hkucpsQGGJbameJRSvog)

### Zadanie 1
Na podstawie [1, §5.14.1] odpowiedz na następujące pytania. 
* W jakim celu profiluje się programy? 
* Jakie informacje niesie ze sobą profil płaski [[2, 5.1]](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.38.4995) i profil grafu wywołań [[2, 5.2]](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.38.4995)? 
* Czemu profilowanie programu wymaga zbudowania go ze specjalną opcją kompilatora `«-pg»`? 
* Na czym polega zliczanie interwałowe `(ang. interval counting)`?

---
* **Program profiling** involves running a version of a program in which instrumentation code has been incorporated to determine how much time the different parts of the program require. **It can be very useful for identifying the parts of a program we should focus on in our optimization efforts.** One strength of profiling is that it can be performed while running the actual program on realistic benchmark data.
* 1. The **flat profile** consists of a list of all the routines that are called during execution of the program, with the count of the number of times they are called and the number of seconds of execution time for which they are themselves accountable. The routines are listed in decreasing order of execution time. A list of the routines that are never called during execution of the program is also available to verify that nothing important is omitted by this execution. The flat profile gives a quick overview of the routines that are used, and shows the routines that are themselves responsible for large fractions of the execution time. 
* 2. The major entries of the **call graph profile** are the entries from the flat profile, augmented by the time propagated to each routine from its descendants. This profile is sorted by the sum of the time for the routine itself plus the time inherited from its descendants. The profile shows which of the higher level routines spend large portions of the total execution time in the routines that they call. For each routine, we show the amount of time passed by each child to the routine, which includes time for the child itself and for the descendants of the child (and thus the descendants of the routine). We also show the percentage these times represent of the total time accounted to the child. Similarly, the parents of each routine are listed, along with time, and percentage of total routine time, propagated to each one.
* The program must be compiled and linked for profiling. With gcc (and other C compilers), this involves simply including the run-time flag `-pg` on the command line. **It is important to ensure that the compiler does not attempt to perform any optimizations via inline substitution, or else the calls to functions may not be tabulated accurately.** We use optimization flag `-Og`, guaranteeing that function calls will be tracked properly. Example :
```
linux> gcc -Og -pg prog.c -o prog
```
* The timing is not very precise. It is based on a simple **interval counting** scheme in which **the compiled program maintains a counter for each function recording the time spent executing that function. The operating system causes the program to be interrupted at some regular time interval $δ$.** Typical values of $δ$ range between 1.0 and 10.0 milliseconds. It then determines what function the program was executing when the interrupt occurred and increments the counter for that function by $δ$. Of course, it may happen that this function just started executing and will shortly be completed, but it is assigned the full cost of the execution since the previous interrupt. Some other function may run between two interrupts and therefore not be charged any time at all. Over a long duration, this scheme works reasonably well. Statistically, every function should be charged according to the relative time spent executing it. For programs that run for less than around 1 second, however, the numbers should be viewed as only rough estimates.
![](https://i.imgur.com/SDuQ4Fv.png)


### Zadanie 2
Pobierz ze strony przedmiotu archiwum `«ask22_lista_13.tgz»`, rozpakuj je i zapoznaj się z kodem źródłowym w pliku `«dictionary.c»`. Powtórz eksperyment z [1, §5.14], tj.: uruchom program, poczekaj na wyniki profilowania, a następnie na podstawie wydruku wskaż kandydata do optymalizacji. Zmień dokładnie jeden z argumentów przekazywanych do polecenia `«make gprof»: SIZE=N, HASH=0/1/2, FIND=0/1/2, LOWER=0/1, QUICK=0/1.` Odpowiednia opcja zmieni rozmiar tablicy mieszającej $N$ lub ustali wersję implementacji jednej z kluczowych funkcji w programie. Wyjaśnij co zmieniło przekazanie danej opcji. Powtarzaj powyższe kroki tak długo, aż uzyskasz najkrótszy czas wykonania programu. 
:::    warning
UWAGA: Należy być przygotowanym do szczegółowego wyjaśnienia wydruku programu profilującego!
:::
```
 (make clean)
 make dictionary-pg
 make gprof
```
```
Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 96.93     70.37    70.37        1    70.37    70.37  sort_words
  2.75     72.37     2.00   965027     0.00     0.00  find_ele_rec
  0.28     72.57     0.20   965027     0.00     0.00  lower1
  0.07     72.62     0.05   965027     0.00     0.00  h_mod
  0.06     72.67     0.05   363039     0.00     0.00  save_string
  0.06     72.71     0.04   965028     0.00     0.00  get_token
  0.04     72.74     0.03   965027     0.00     0.00  insert_string
  0.01     72.75     0.01   965029     0.00     0.00  get_word
  0.00     72.75     0.00   363039     0.00     0.00  new_ele
  0.00     72.75     0.00        8     0.00     0.00  find_option
  0.00     72.75     0.00        7     0.00     0.00  add_int_option
  0.00     72.75     0.00        1     0.00     0.00  add_string_option
  0.00     72.75     0.00        1     0.00     0.00  parse_options
  0.00     72.75     0.00        1     0.00     0.00  show_options
  0.00     72.75     0.00        1     0.00    72.75  word_freq

			Call graph


granularity: each sample hit covers 2 byte(s) for 0.01% of 72.75 seconds

index % time    self  children    called     name
                0.00   72.75       1/1           main [2]
[1]    100.0    0.00   72.75       1         word_freq [1]
               70.37    0.00       1/1           sort_words [3]
                0.03    2.29  965027/965027      insert_string [4]
                0.04    0.01  965028/965028      get_token [7]
-----------------------------------------------
                                                 <spontaneous>
[2]    100.0    0.00   72.75                 main [2]
                0.00   72.75       1/1           word_freq [1]
                0.00    0.00       7/7           add_int_option [13]
                0.00    0.00       1/1           add_string_option [14]
                0.00    0.00       1/1           parse_options [15]
                0.00    0.00       1/1           show_options [16]
-----------------------------------------------
               70.37    0.00       1/1           word_freq [1]
[3]     96.7   70.37    0.00       1         sort_words [3]
-----------------------------------------------
                0.03    2.29  965027/965027      word_freq [1]
[4]      3.2    0.03    2.29  965027         insert_string [4]
                2.00    0.05  965027/965027      find_ele_rec [5]
                0.20    0.00  965027/965027      lower1 [6]
                0.05    0.00  965027/965027      h_mod [8]
-----------------------------------------------
                             95820673             find_ele_rec [5]
                2.00    0.05  965027/965027      insert_string [4]
[5]      2.8    2.00    0.05  965027+95820673 find_ele_rec [5]
                0.05    0.00  363039/363039      save_string [9]
                0.00    0.00  363039/363039      new_ele [11]
                             95820673             find_ele_rec [5]
-----------------------------------------------
                0.20    0.00  965027/965027      insert_string [4]
[6]      0.3    0.20    0.00  965027         lower1 [6]
-----------------------------------------------
                0.04    0.01  965028/965028      word_freq [1]
[7]      0.1    0.04    0.01  965028         get_token [7]
                0.01    0.00  965029/965029      get_word [10]
-----------------------------------------------
                0.05    0.00  965027/965027      insert_string [4]
[8]      0.1    0.05    0.00  965027         h_mod [8]
-----------------------------------------------
                0.05    0.00  363039/363039      find_ele_rec [5]
[9]      0.1    0.05    0.00  363039         save_string [9]
-----------------------------------------------
                0.01    0.00  965029/965029      get_token [7]
[10]     0.0    0.01    0.00  965029         get_word [10]
-----------------------------------------------
                0.00    0.00  363039/363039      find_ele_rec [5]
[11]     0.0    0.00    0.00  363039         new_ele [11]
-----------------------------------------------
                0.00    0.00       8/8           parse_options [15]
[12]     0.0    0.00    0.00       8         find_option [12]
-----------------------------------------------
                0.00    0.00       7/7           main [2]
[13]     0.0    0.00    0.00       7         add_int_option [13]
-----------------------------------------------
                0.00    0.00       1/1           main [2]
[14]     0.0    0.00    0.00       1         add_string_option [14]
-----------------------------------------------
                0.00    0.00       1/1           main [2]
[15]     0.0    0.00    0.00       1         parse_options [15]
                0.00    0.00       8/8           find_option [12]
-----------------------------------------------
                0.00    0.00       1/1           main [2]
[16]     0.0    0.00    0.00       1         show_options [16]
-----------------------------------------------


Index by function name

  [13] add_int_option         [10] get_word               [15] parse_options
  [14] add_string_option       [8] h_mod                   [9] save_string
   [5] find_ele_rec            [4] insert_string          [16] show_options
  [12] find_option (options.c) [6] lower1                  [3] sort_words
   [7] get_token              [11] new_ele (dictionary.c)  [1] word_freq

```

Dla QUICK = 1
```
Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 80.47      1.82     1.82   965027     0.00     0.00  find_ele_rec
 11.53      2.08     0.26   965027     0.00     0.00  lower1
  2.22      2.13     0.05                             compare_ele
  1.77      2.17     0.04   965027     0.00     0.00  h_mod
  1.33      2.20     0.03        1     0.03     0.03  sort_words
  1.11      2.22     0.03   965028     0.00     0.00  get_token
  0.67      2.24     0.02   363039     0.00     0.00  save_string
  0.44      2.25     0.01   965027     0.00     0.00  insert_string
  0.44      2.26     0.01   363039     0.00     0.00  new_ele
  0.22      2.26     0.01   965029     0.00     0.00  get_word
  0.00      2.26     0.00        8     0.00     0.00  find_option
  0.00      2.26     0.00        7     0.00     0.00  add_int_option
  0.00      2.26     0.00        1     0.00     0.00  add_string_option
  0.00      2.26     0.00        1     0.00     0.00  parse_options
  0.00      2.26     0.00        1     0.00     0.00  show_options
  0.00      2.26     0.00        1     0.00     2.21  word_freq
```

Dla size = 1024
```
Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 67.79     76.19    76.19        1    76.19    76.19  sort_words
 31.97    112.12    35.93   965027     0.00     0.00  find_ele_rec
  0.23    112.38     0.26   363039     0.00     0.00  save_string
  0.11    112.50     0.12   965027     0.00     0.00  lower1
  0.04    112.55     0.05   965027     0.00     0.00  h_mod
  0.03    112.58     0.03   965028     0.00     0.00  get_token
  0.03    112.61     0.03   965027     0.00     0.00  insert_string
  0.01    112.62     0.01   363039     0.00     0.00  new_ele
  0.00    112.62     0.00   965029     0.00     0.00  get_word
  0.00    112.62     0.00        8     0.00     0.00  find_option
  0.00    112.62     0.00        7     0.00     0.00  add_int_option
  0.00    112.62     0.00        1     0.00     0.00  add_string_option
  0.00    112.62     0.00        1     0.00     0.00  parse_options
  0.00    112.62     0.00        1     0.00     0.00  show_options
  0.00    112.62     0.00        1     0.00   112.62  word_freq
```

Dla size = 4202137, hash = 2, quick = 1, lower = 1

```
Total time = 0.386122 seconds
gprof -b dictionary-pg gmon.out
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  ms/call  ms/call  name    
 38.97      0.07     0.07   965027     0.00     0.00  insert_string
 33.40      0.13     0.06   965028     0.00     0.00  get_token
 11.13      0.15     0.02        1    20.04    20.04  sort_words
 11.13      0.17     0.02                             compare_ele
  5.57      0.18     0.01   965027     0.00     0.00  h_xor
  0.00      0.18     0.00   965029     0.00     0.00  get_word
  0.00      0.18     0.00   965027     0.00     0.00  find_ele_rec
  0.00      0.18     0.00   965027     0.00     0.00  lower2
  0.00      0.18     0.00   363039     0.00     0.00  new_ele
  0.00      0.18     0.00   363039     0.00     0.00  save_string
  0.00      0.18     0.00        8     0.00     0.00  find_option
  0.00      0.18     0.00        7     0.00     0.00  add_int_option
  0.00      0.18     0.00        1     0.00     0.00  add_string_option
  0.00      0.18     0.00        1     0.00     0.00  parse_options
  0.00      0.18     0.00        1     0.00     0.00  show_options
  0.00      0.18     0.00        1     0.00   160.32  word_freq
```

Dla size = 965027, hash = 2, quick = 1, lower = 1 (Najbardziej optymalny)

```
Total time = 0.364774 seconds
gprof -b dictionary-pg gmon.out
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  ms/call  ms/call  name    
 31.31      0.05     0.05   965027     0.00     0.00  insert_string
 12.53      0.07     0.02   965028     0.00     0.00  get_token
 12.53      0.09     0.02   965027     0.00     0.00  find_ele_rec
 12.53      0.11     0.02   965027     0.00     0.00  lower2
 12.53      0.13     0.02        1    20.04    20.04  sort_words
 12.53      0.15     0.02                             compare_ele
  6.26      0.16     0.01   965027     0.00     0.00  h_xor
  0.00      0.16     0.00   965029     0.00     0.00  get_word
  0.00      0.16     0.00   363039     0.00     0.00  new_ele
  0.00      0.16     0.00   363039     0.00     0.00  save_string
  0.00      0.16     0.00        8     0.00     0.00  find_option
  0.00      0.16     0.00        7     0.00     0.00  add_int_option
  0.00      0.16     0.00        1     0.00     0.00  add_string_option
  0.00      0.16     0.00        1     0.00     0.00  parse_options
  0.00      0.16     0.00        1     0.00     0.00  show_options
  0.00      0.16     0.00        1     0.00   140.28  word_freq

```
---


### Zadanie 3
Dla optymalnej konfiguracji z poprzedniego zadania uruchom polecenie `«make callgrind»`. Następnie wczytaj plik wynikowy `«callgrind.out»` przy pomocy nakładki graficznej `«kcachegrind»`.
* Wyświetl graf wywołań funkcji, a następnie zidentyfikuj wywołania biblioteki standardowej języka C, w których program spędza najwięcej czasu. 
* Wyjaśnij do czego służą te funkcje. 
* Jak można byłoby zoptymalizować miejsca, w których korzystamy z tych funkcji?
:::     warning
Uwaga! W kolejnych dwóch zadaniach polecenia należy wykonywać z uprawnieniami administratora!
:::
---
```
make clean 
make dictionary-pg
make gprof
make callgrind 
kcachegrind
```
![](https://i.imgur.com/FXKEAVs.png)
![](https://i.imgur.com/maS8kxM.png)



### Zadanie 4
Posługując się poleceniem `lspci -v` wyświetl listę urządzeń podpiętych do magistrali PCI.
W wydruku zidentyfikuj: 
* regiony fizycznej przestrzeni adresowej, przez które procesor komunikuje się z urządzeniami wejścia-wyjścia; 
* numery przerwań, które urządzenia wejścia-wyjścia zgłaszają procesorowi. 

Przy pomocy polecenia `cat /proc/iomem` : 
* wydrukuj mapę fizycznej przestrzeni adresowej swojego komputera; 
* zidentyfikuj w niej pamięć operacyjną oraz pamięć należącą do urządzeń wejścia-wyjścia, tj. karty graficznej, sieciowej, dźwiękowej, kontrolera dysków. 

W wydruku polecenia `cat /proc/interrupts` : 
* wskaż, który z procesorów odbiera przerwania od w/w urządzeń.

---
* ![](https://i.imgur.com/CYTXlCy.png)
* ![](https://i.imgur.com/nWSuLlE.png)


### Zadanie 5
Ściągnij repozytorium [`dwks/pagemap1`](https://github.com/dwks/pagemap). Skompiluj program `«pagemap2»` i przy jego pomocy wyświetl zawartość tablicy stron procesu o numerze `«pid»` dla przedziału adresów wirtualnych, który należy wybrać na podstawie wyjścia z polecenia `«pmap»`. Przetłumacz kilka adresów wirtualnych posługując się numerem ramki `(ang. page frame number)`. Wskaż region pamięci w mapie fizycznej przestrzeni adresowej, gdzie trafił przetłumaczony adres.
:::    info
Wskazówka: Zakładamy, że rozmiar strony to 4096 bajtów.
:::


---

### Zadanie 6
Czasami procesor nie może przeprowadzić translacji dla adresu wygenerowanego w trakcie wykonywania instrukcji. 
* Wymień co najmniej cztery przypadki wykonania instrukcji `«mov»`, dla których procesor zgłosi wyjątek błędu dostępu do pamięci.
* Które z wymienionych przypadków zawsze spowodują zakończenie programu z błędem?
*  Za co odpowiada procedura obsługi błędu strony `(ang. page fault)`? 
*  Co się dzieje przed i po wykonaniu tej procedury?

---
Z [tej strony](https://www.felixcloutier.com/x86/mov) :
* 1. If attempt is made to load the CS register. (#UD) 
* 2. If the destination operand is in a non-writable segment.(#GP) 
* 3. If segment selector index is outside descriptor table limits.(#GP)
* 4. If a page fault occurs.(#PF)
* Wszystkie poza page-faultem
* Za obsłużenie sytuacji, w której virtualny adres chce się odwołać do adresu w pamięci która nie jest valid (której dana nie jest w DRAM-ie)
* System operacyjny przejumje władzę w komputerze aby sprawić, że dane do których się odwołujemy będą w DRAM-ie, po czym przywaraca się stan do przed `Page faulta`, odbiera się kernelowe uprawnienia programowi i powtarza się instrukcję która doprowadziła do `Page fault`.
![](https://i.imgur.com/X9ceoTU.png)

 
### Zadanie 7
Procesor `x86-64` używa rejestru sterującego [CR3](https://en.wikipedia.org/wiki/Control_register#CR3) do przeprowadzania translacji adresów. 
* Co przechowuje ten rejestr? 
* Czemu jest on dostępny wyłącznie dla programów działających w trybie uprzywilejowanym `(ang. supervisor mode)`? 
* Czy moglibyśmy zagwarantować izolację, jeśli pamięć przechowująca tablicę stron była dostępna w trybie użytkownika?

---
* Used when virtual addressing is enabled, hence when the PG bit is set in CR0. CR3 enables the processor to translate linear addresses into physical addresses by locating the page directory and page tables for the current task. **Typically, the upper 20 bits of CR3 become the page directory base register (PDBR), which stores the physical address of the first page directory.** If the PCIDE bit (flaga) in CR4 is set, the lowest 12 bits are used for the process-context identifier (PCID).
* A control register is a processor register which **changes or controls the general behavior of a CPU** or other digital device. Common tasks performed by control registers include interrupt control, switching the addressing mode, paging control, and coprocessor control. Są to bardzo niebezpieczne czynności które mogą łatwo zepsuć komputer.
* Gdyby miał dostęp dla read-only to sądzę, że nie stałoby się nic strasznego tak długo jak USER nie byłby ownerem tego pliku tylko miał także dostęp read only.
### Zadanie 8
Przy pomocy polecenia `ps -e -o pid,rss,vsz,cmd` wydrukuj listę wszystkich procesów
wraz z zadeklarowanym rozmiarem wirtualnej przestrzeni adresowej i zbioru rezydentnego, odpowiednio `vsz` i `rss`. Użyj zdobytych danych do obliczenia rozmiaru pamięci wirtualnej używanej przez wszystkie procesy. Na podstawie wydruku polecenia `free` określ bieżące użycie i całkowity rozmiar pamięci fizycznej. Wyjaśnij w jaki sposób stronicowanie na żądanie `(ang. demand paging)` i współdzielenie pamięci pozwalają systemowi operacyjnemu na oszczędne zarządzanie pamięcią fizyczną.

---
Demand paging to 
Plusy :
Demand paging, as opposed to loading all pages immediately:

* Only loads pages that are demanded by the executing process.
* As there is more space in main memory, more processes can be loaded, reducing the context switching time, which utilizes large amounts of resources.
* Less loading latency occurs at program startup, as less information is accessed from secondary storage and less information is brought into main memory.
* As main memory is expensive compared to secondary memory, this technique helps significantly reduce the bill of material (BOM) cost in smart phones for example. Symbian OS had this feature.
![](https://i.imgur.com/mu3ZZlh.png)
Dodajemy RSS i VSZ i powinno wyjśc tyle ile jest w wydruku `free`
![](https://i.imgur.com/KfeVwDF.png)

```
ps -e -o vsz > vsz.txt
awk '{s+=$1} END {print s}' vsz.txt 
```

### Zadanie 9
Przyjrzymy się wydrukowi z poprzedniego zadania oraz z polecenia `«free»`. Wskaż bieżące użycie i całkowity rozmiar pamięci wymiany `(ang. swap)`. 
* Czym różni się zbiór roboczy (ang. working set) procesu od zbioru rezydentnego `(ang. resident set)`? 
* Czemu system nie podaje tego pierwszego? 
* Kiedy system operacyjny używa wymiany do pamięci drugorzędnej `(ang. backing storage)`? 
* Co dzieje się z systemem, jeśli łączny rozmiar zbiorów roboczych wszystkich procesów jest większy niż rozmiar pamięci fizycznej?
:::info
Wskazówka: Pamiętaj, że pamięć wirtualna jest programową realizacją pamięci podręcznej dla stron.
::::

[1] „Computer Systems: A Programmer’s Perspective”
Randal E. Bryant, David R. O’Hallaron; Pearson; 3rd edition, 2016
[2] [„gprof: a Call Graph Execution Profiler”](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.38.4995)
Susan L. Graham , Peter B. Kessler , Marshall K. McKusick; 1982

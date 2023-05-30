# spotifywrapped
[PL] Klasyfikator zespołów na podstawie zbioru danych z utworów z mojego spotify wrapped 2022 [EN] Machine Learning Classificator of band's/artist's name based on dataset of songs from my spotify wrapped 2022

[PL]
Klasyfikator zespołów na podstawie utworów mojego Spotify Wrapped
Metody i Narzędzia Big Data


**1.	Abstrakt**

Jestem pasjonatem muzyki, a w roku 2022 wyznaczyłem sobie zadanie, by w każdy z 12 miesięcy odkryć 100 nowych utworów, które wpisują się w moje gusta. Oprócz tego intensywnie słuchałem swoich ulubieńców, którymi w corocznym podsumowaniu spotify wrapped okazali się być zespoły / artyści: Radiohead, La Femme, David Bowie, Coals oraz The Cure. Jako iż są to zespoły, w moim uznaniu, różniące się od siebie stylem muzyki chciałem stworzyć klasyfikator, który nasycony określoną liczbą piosenek zespołu będzie mógł predykować, na podstawie analizy danych dźwięku, jaki zespół/artysta jest autorem danego utworu.


**2.	Podejście do rozwiązania problemu**

Łącznie pobrałem 226 utworów tych 5 wykonawców (bez równego podziału) za pomocą Viwizard Music Convertor ze Spotify do formatu mp3. Chcąc zamienić pliki mp3. na liczbową reprezentację sygnałów dźwiękowych posłużyłem się pythonową biblioteką Librosa, stworzoną do analizy sygnałów dźwiękowych. W wyborze kierowałem się podobnymi projektami, skupiającymi się bardziej na rozpoznawaniu gatunków muzycznych. Za pomocą narzędzi w bibliotece, dla danego pliku mp3 wyciągnąłem cechy: krótkookresowej transformaty Fouriera (dalej chroma_stft), szerokości pasma spektralnego (dalej spectral_brandwidth), spektroidu centralnego (spectral_centroid), przejścia spektrum dźwiękowego (dalej spectral_rolloff), liczbę zmian znaku sygnału (dalej zero_crossing_rate) oraz 20 wskaźników widma dźwiękowego – mel-frequency cepstral coefficients (dalej jako mfcc).

**Wizualizacja cech chroma_stft, spectral_brandwidth, spectral_centroid oraz spectral_rollof na podstawie 5 utorów**
Kod służący do wizualizacji cech umieszczony został w zakładce **feature_visualisation**.
![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/cd218695-b48f-439a-8359-57976830a45d)


Jak widać wektory liczb służących do opisania cech (tych zwizualizowanych i nie) różnią się od siebie w zależności od piosenek, co wskazuje na istnienie różnic w numerycznej reprezentacji, i te różnice były podstawą do stworzenia mojego zbioru danych


**3.	Stworzenie zbioru danych**

**a)	Metoda**
Jako iż każda z tych cech była wyrażona w postaci macierzy o dość dużych wymiarach, zdecydowałem się wyciągać średnią arytmetyczną z każdej cechy. W ten sposób każdy utwór opisywałem 26 cechami (chroma_stft, spectral_centroid, spectral_brandwidth, spectrall_rolloff, rmse, zero_crossing_rate i 20 wskaźników mfcc) dołączając do nich także informację o wykonawcy danego utworu.

Wyzwaniem było zebranie odpowiedniej próbki danych, dlatego zdecydowałem się na stworzenie 3 różnych rodzajów zbioru danych. 1) Wektorów cech dla całego utworu, 2) Wektorów cech dla przedziału od 10 do 30 sekundy każdego utworu, 3) Wektorów cech dla całego utworu, ale już z wyodrębnioną ścieżką wokalną.
W ten sposób utworzyłem 3 różne zbiory danych zapisywane w notatnikach w formacie xlsx.
Kod służący do wizualizacji wydobycia cech i zapisania ich w plikach csv został umieszczony w zakładce **creating_dataset** oraz **creating_dataset_vocals**.

**b)	Ostatnie kroki przed klasyfikacją**
Zauważyłem, że podczas analizy unikatowych nazw zespołów jest ich więcej niż planowałem. Wynikało to z tego, że niektóre piosenki zespołu Coals były koprodukcjami z innymi artystami. Zamieniłem, więc wszystkie wyjątki, tak, by różnych typów etykiet faktycznie było 5. Ponadto zmieniłem wartości etykiet tekstowych na liczbowe, by zmapowany wektor wyjściowy nadawał się do nasycenia nim klasyfikatora.
 
Przeskalowałem wszystkie dane tak, by były one ustandaryzowane, co nie zmieniło ich wzajemnych zależności, a mogło okazać się przydatne przy potencjalnej redukcji wymiarów.
Zauważyłem, że duża liczba cech (jest ich aż 26) może w przyszłości wpłynąć na dokładność klasyfikacji, więc uznałem, że prewencyjnie stworzę odpowiadający zbiorowi danych zestaw 2 cech, które zostały otrzymane po zredukowaniu wymiarowości macierzy cech X, algorytmem LDA, do 2 cech o największej wariancji.
Kod służący do opracowania zbioru danych umieszczony w zakładce **data_preprocessing**.
 
**4. Klasyfikacja**


Kod algorytmu klasyfikującego został umieszczony w zakładce **songs_classification**.


Mam więc dwa różne wektory X. Jeden przetrzymuje 26 cech, drugi zawiera 2 cechy otrzymane z redukcji wymiarów algorytmem LDA
Podzieliłem dane na zbiór testowy i treningowy. Zaimportowałem 6 klasyfikatorów – K Najbliższych sąsiadów, Drzewo Decyzyjne, Las Losowy, Maszynę Wektorów Stałych, Regresje Logistyczną i sieć neuronową XGBoost.
Następnie, by oszczędzić czas, w jednej pętli przeprowadziłem klasyfikację dla jednego zbioru testowego i treningowego na każdym z sześciu klasyfikatorów, by uzyskać macierz pomyłek dla każdego z trzech zbiorów danych i każdego z dwóch rodzajów macierzy wejściowej X.
Wyniki pojedynczej klasyfikacji dla każdego rodzaju pliku i macierzy wyjściowej prezentują się w sposób następujący:
![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/fbaceca7-2966-465a-9cb9-f91b67430261)
Macierz pomyłek, dla pojedynczej klasyfikacji, na każdym klasyfikatorze, dla każdego z trzech zbiorów danych, przeprowadzonych na dwóch różnych macierzach wejściowych X (26 cech i 2 cech zredukowanych algorytmem LDA)








**5. Prezentacja wyników klasyfikacji**


Jedna klasyfikacja nie jest jednak miarodajna, dlatego zdecydowałem się przeprowadzić 10 prób walidacji krzyżowej. Dopiero dla uśrednionej wartości skuteczności 10 klasyfikacji dla danego klasyfikatora, mogłem dojść do bardziej wiarygodnych wyników. Jakoś klasyfikacji zwizualizowałem za pomocą wykresów

KLASYFIKACJA DLA ZBIORU DANYCH: C:\Users\akr\wrapped_dataset_226.xlsx

Próba dla wektora X o 26 cechach


Wynik dla klasyfikatora KNN usredniony 10 prób krzyżowych: 0.7061904761904761

Wynik dla klasyfikatora Decision Tree usredniony 10 prób krzyżowych: 0.54

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/e665d773-249e-4421-a3f2-53c6d64420ba)

Wynik dla klasyfikatora Random Forest usredniony 10 prób krzyżowych: 0.6869047619047619

Wynik dla klasyfikatora SVC usredniony 10 prób krzyżowych: 0.7014285714285713

Wynik dla klasyfikatora Logistic Regression usredniony 10 prób krzyżowych: 0.7016666666666667

**Wynik dla klasyfikatora XGBoost usredniony 10 prób krzyżowych: 0.7071428571428572**

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/f1a7a244-bcac-48a5-af5d-5833d85ba7d0)

--------------

Próba dla Wektor X 2 cech po alg. LDA


Wynik dla klasyfikatora KNN usredniony 10 prób krzyżowych: 0.6461904761904762

Wynik dla klasyfikatora Decision Tree usredniony 10 prób krzyżowych: 0.5630952380952381

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/bf577026-7695-44dd-be07-ebaa1fc209c9)

Wynik dla klasyfikatora Random Forest usredniony 10 prób krzyżowych: 0.6326190476190476

Wynik dla klasyfikatora SVC usredniony 10 prób krzyżowych: 0.6666666666666667

**Wynik dla klasyfikatora Logistic Regression usredniony 10 prób krzyżowych: 0.7011904761904763**

Wynik dla klasyfikatora XGBoost usredniony 10 prób krzyżowych: 0.618095238095238

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/69c17278-cb6c-422f-beef-24e2456556b1)

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

KLASYFIKACJA DLA ZBIORU DANYCH: C:\Users\akr\wrapped_dataset_20s_226.xlsx

Próba dla 26D

Wynik dla klasyfikatora KNN usredniony 10 prób krzyżowych: 0.4616666666666666

Wynik dla klasyfikatora Decision Tree usredniony 10 prób krzyżowych: 0.43666666666666665

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/ced5af70-6528-4bae-8b70-b132b90fd6a0)

**Wynik dla klasyfikatora Random Forest usredniony 10 prób krzyżowych: 0.5161904761904761**

Wynik dla klasyfikatora SVC usredniony 10 prób krzyżowych: 0.4809523809523809

Wynik dla klasyfikatora Logistic Regression usredniony 10 prób krzyżowych: 0.4754761904761904

Wynik dla klasyfikatora XGBoost usredniony 10 prób krzyżowych: 0.49499999999999994

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/bb572162-597f-49fd-a50c-0255aa55df8a)

--------------

Próba dla 2D LDA

Wynik dla klasyfikatora KNN usredniony 10 prób krzyżowych: 0.5349999999999999

Wynik dla klasyfikatora Decision Tree usredniony 10 prób krzyżowych: 0.5397619047619048

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/ee802a52-3353-4406-bf6b-8c0378b70dbb)

Wynik dla klasyfikatora Random Forest usredniony 10 prób krzyżowych: 0.5692857142857143

Wynik dla klasyfikatora SVC usredniony 10 prób krzyżowych: 0.6126190476190475

**Wynik dla klasyfikatora Logistic Regression usredniony 10 prób krzyżowych: 0.6371428571428571**

Wynik dla klasyfikatora XGBoost usredniony 10 prób krzyżowych: 0.5685714285714286

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/cef864d9-18a9-4f17-b8cb-af2f2b75d72a)

--------------

KLASYFIKACJA DLA ZBIORU DANYCH: C:\Users\akr\wrapped_dataset_vocal_226.xlsx

Próba dla 26D

Wynik dla klasyfikatora KNN usredniony 10 prób krzyżowych: 0.638095238095238

Wynik dla klasyfikatora Decision Tree usredniony 10 prób krzyżowych: 0.47619047619047616

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/f7e17475-cdd2-47bc-a73b-ea38eec3b35a)

Wynik dla klasyfikatora Random Forest usredniony 10 prób krzyżowych: 0.6721428571428572

Wynik dla klasyfikatora SVC usredniony 10 prób krzyżowych: 0.6483333333333333

Wynik dla klasyfikatora Logistic Regression usredniony 10 prób krzyżowych: 0.6721428571428572

**Wynik dla klasyfikatora XGBoost usredniony 10 prób krzyżowych: 0.6864285714285713**

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/053dc552-84b4-4273-b692-f8eeed57050b)

--------------

Próba dla 2D LDA

Wynik dla klasyfikatora KNN usredniony 10 prób krzyżowych: 0.6711904761904762

Wynik dla klasyfikatora Decision Tree usredniony 10 prób krzyżowych: 0.6719047619047619

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/93375a43-bb31-49ec-a119-3dbe92262ddc)

Wynik dla klasyfikatora Random Forest usredniony 10 prób krzyżowych: 0.6866666666666668

**Wynik dla klasyfikatora SVC usredniony 10 prób krzyżowych: 0.7259523809523809**

Wynik dla klasyfikatora Logistic Regression usredniony 10 prób krzyżowych: 0.7211904761904762

Wynik dla klasyfikatora XGBoost usredniony 10 prób krzyżowych: 0.6573809523809524

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/8a341ee0-ddc4-4d43-8ac2-f1b270e4960c)

--------------


**6. Analiza wyników i wnioski**


1.	Najlepiej sprawdził się klasyfikator wektorów nośnych dla zbioru danych wyodrębnionej ścieżki wokalnej, przy redukcji macierzy wejściowej X do dwóch wymiarów algorytmem LDA, osiągając skutecznośc 72,59% klasyfikacji, przy 10 próbach walidacji krzyżowej 
2.	Dla wielowymiarowej macierzy wejściowej X jakość klasyfikacji dla 10 prób walidacji krzyżwoej jest zazwyczaj  gorsza niż dla zredukowanej do dwóch wymiarów macierzy wejściowej X
3.	Najgorzej sprawiła się klasyfikacja dla przedziału od 10-tej do 30-tej sekundy utworów, w której wartości na każdym z klasyfikatorów dla 10 walidacji krzyżowych były średnio najniższe
4.	Dla macierzy wejściowej X o 26 wymiarach przeważnie najlepiej sprawdzał się klasyfikator lasu losowego, a dla macierzy wejściowej X o 2 wymiarach przeważnie najlepiej sprawdzały się klasyfikatory regresji logistycznej i maszyny wektorów nośnych
5.	Dla prawie każdego zbioru danych (oprócz zbioru danych 20 sekundowego wycinku dla X o 26 wymiarach) najgorzej sprawdzał się klasyfikator drzewa decyzyjnego
6.	Przy pojedynczej klasyfikacji łatwo zauważyć, że etykieta 3 (Radiohead) najczęściej mylona była z etykietą 2 (La femme). Może to wskazywać na istnienie faktycznej konotacji Radiohead z La Femme na płaszczyźnie elektronicznego brzmienia.
7.	Z pewnością mogłem poszerzyć zbiór danych o całą dyskografię każdego zespołu i artysty, by zgromadzić więcej przykładów, co mogłoby wpłynąć na późniejszą jakość klasyfikacji.
8.	Zamiast prób jedynie dla wielowymiarowego wektora X, mogłem przeprowadzić klasyfikację dla pośrednich mniejszych rozmiarów wektora cech wejściowych X, by sprawdzić, jak klasyfikator radzi sobie z mniej złożonym zbiorem danych wejściowych.


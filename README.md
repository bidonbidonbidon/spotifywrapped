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
Kod służący do wizualizacji wydobycia cech i zapisania ich w plikach csv został umieszczony w zakładce **creating_dataset**.

**b)	Ostatnie kroki przed klasyfikacją**
Zauważyłem, że podczas analizy unikatowych nazw zespołów jest ich więcej niż planowałem. Wynikało to z tego, że niektóre piosenki zespołu Coals były koprodukcjami z innymi artystami. Zamieniłem, więc wszystkie wyjątki, tak, by różnych typów etykiet faktycznie było 5. Ponadto zmieniłem wartości etykiet tekstowych na liczbowe, by zmapowany wektor wyjściowy nadawał się do nasycenia nim klasyfikatora.
 
Przeskalowałem wszystkie dane tak, by były one ustandaryzowane, co nie zmieniło ich wzajemnych zależności, a mogło okazać się przydatne przy potencjalnej redukcji wymiarów.
Zauważyłem, że duża liczba cech (jest ich aż 26) może w przyszłości wpłynąć na dokładność klasyfikacji, więc uznałem, że prewencyjnie stworzę odpowiadający zbiorowi danych zestaw 2 cech, które zostały otrzymane po zredukowaniu wymiarowości macierzy cech X, algorytmem LDA, do 2 cech o największej wariancji.
Kod służący do opracowania zbioru danych umieszczony w zakładce **data_preprocessing**.
 
**4. Klasyfikacja**


Mam więc dwa różne wektory X. Jeden przetrzymuje 26 cech, drugi zawiera 2 cechy otrzymane z redukcji wymiarów algorytmem LDA
Podzieliłem dane na zbiór testowy i treningowy. Zaimportowałem 6 klasyfikatorów – K Najbliższych sąsiadów, Drzewo Decyzyjne, Las Losowy, Maszynę Wektorów Stałych, Regresje Logistyczną i sieć neuronową XGBoost.
Następnie, by oszczędzić czas, w jednej pętli przeprowadziłem klasyfikację dla jednego zbioru testowego i treningowego na każdym z sześciu klasyfikatorów, by uzyskać macierz pomyłek dla każdego z trzech zbiorów danych i każdego z dwóch rodzajów macierzy wejściowej X.
Wyniki pojedynczej klasyfikacji dla każdego rodzaju pliku i macierzy wyjściowej prezentują się w sposób następujący:
![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/fbaceca7-2966-465a-9cb9-f91b67430261)
Macierz pomyłek, dla pojedynczej klasyfikacji, na każdym klasyfikatorze, dla każdego z trzech zbiorów danych, przeprowadzonych na dwóch różnych macierzach wejściowych X (26 cech i 2 cech zredukowanych algorytmem LDA)








**5. Prezentacja wyników klasyfikacji**


Jedna klasyfikacja nie jest jednak miarodajna, dlatego zdecydowałem się przeprowadzić 10 prób walidacji krzyżowej. Dopiero dla uśrednionej wartości skuteczności 10 klasyfikacji dla danego klasyfikatora, mogłem dojść do bardziej wiarygodnych wyników. Jakoś klasyfikacji zwizualizowałem za pomocą wykresów


KLASYFIKACJA DLA ZBIORU DANYCH: C:\Users\akr\wrapped_dataset_226.xlsx
Próba dla Wektor X 26 cech


Wynik dla klasyfikatora knn usredniony 10 prób krzyżowych: 0.7114285714285714

Wynik dla klasyfikatora dtc usredniony 10 prób krzyżowych: 0.5204761904761904

**Wynik dla klasyfikatora rfc usredniony 10 prób krzyżowych: 0.7359523809523809**

Wynik dla klasyfikatora svc usredniony 10 prób krzyżowych: 0.7069047619047619

Wynik dla klasyfikatora logreg usredniony 10 prób krzyżowych: 0.7121428571428571

Wynik dla klasyfikatora xgb usredniony 10 prób krzyżowych: 0.6916666666666667

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/b8a6ebb9-042f-4871-8b9e-08870ee79ee8)



Próba dla Wektor X 2 cech po alg. LDA


Wynik dla klasyfikatora knn usredniony 10 prób krzyżowych: 0.6321428571428571

Wynik dla klasyfikatora dtc usredniony 10 prób krzyżowych: 0.6073809523809524 

Wynik dla klasyfikatora rfc usredniony 10 prób krzyżowych: 0.6323809523809524

Wynik dla klasyfikatora svc usredniony 10 prób krzyżowych: 0.6814285714285715

**Wynik dla klasyfikatora logreg usredniony 10 prób krzyżowych: 0.6964285714285714**

Wynik dla klasyfikatora xgb usredniony 10 prób krzyżowych: 0.6133333333333333

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/fabbe0e4-1fed-4c83-88d7-815fa4daa49f)



KLASYFIKACJA DLA ZBIORU DANYCH: C:\Users\akr\wrapped_dataset_20s_226.xlsx
Próba dla Wektor X 26 cech

Wynik dla klasyfikatora knn usredniony 10 prób krzyżowych: 0.5245238095238095

Wynik dla klasyfikatora dtc usredniony 10 prób krzyżowych: 0.41119047619047616

**Wynik dla klasyfikatora rfc usredniony 10 prób krzyżowych: 0.5388095238095238**

Wynik dla klasyfikatora svc usredniony 10 prób krzyżowych: 0.4988095238095238

Wynik dla klasyfikatora logreg usredniony 10 prób krzyżowych: 0.49952380952380954

Wynik dla klasyfikatora xgb usredniony 10 prób krzyżowych: 0.5142857142857143

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/c2b4f922-2b4f-4acc-bfd6-cfcbdb57e1e2)



Próba dla Wektor X 2 cech po alg. LDA

Wynik dla klasyfikatora knn usredniony 10 prób krzyżowych: 0.5335714285714286

Wynik dla klasyfikatora dtc usredniony 10 prób krzyżowych: 0.5147619047619048

Wynik dla klasyfikatora rfc usredniony 10 prób krzyżowych: 0.5397619047619047

**Wynik dla klasyfikatora svc usredniony 10 prób krzyżowych: 0.6314285714285715**

Wynik dla klasyfikatora logreg usredniony 10 prób krzyżowych: 0.621904761904762

Wynik dla klasyfikatora xgb usredniony 10 prób krzyżowych: 0.5390476190476192

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/f426851a-c285-4e96-86a1-b43023ed26fa)


KLASYFIKACJA DLA ZBIORU DANYCH: C:\Users\akr\wrapped_dataset_vocal_226.xlsx
Próba dla Wektor X 26 cech

**Wynik dla klasyfikatora knn usredniony 10 prób krzyżowych: 0.6766666666666666**

Wynik dla klasyfikatora dtc usredniony 10 prób krzyżowych: 0.5011904761904762

Wynik dla klasyfikatora rfc usredniony 10 prób krzyżowych: 0.6421428571428571

Wynik dla klasyfikatora svc usredniony 10 prób krzyżowych: 0.6440476190476191

Wynik dla klasyfikatora logreg usredniony 10 prób krzyżowych: 0.6673809523809524

Wynik dla klasyfikatora xgb usredniony 10 prób krzyżowych: 0.6673809523809523

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/fcdf3c35-c3bc-4f25-88a3-1d3de8d6d536)


Próba dla Wektor X 2 cech po alg. LDA

Wynik dla klasyfikatora knn usredniony 10 prób krzyżowych: 0.6828571428571429

Wynik dla klasyfikatora dtc usredniony 10 prób krzyżowych: 0.6445238095238095

Wynik dla klasyfikatora rfc usredniony 10 prób krzyżowych: 0.6880952380952381

Wynik dla klasyfikatora svc usredniony 10 prób krzyżowych: 0.7269047619047619

**Wynik dla klasyfikatora logreg usredniony 10 prób krzyżowych: 0.7414285714285713**

Wynik dla klasyfikatora xgb usredniony 10 prób krzyżowych: 0.6480952380952381

![image](https://github.com/bidonbidonbidon/spotifywrapped/assets/134869902/bd170278-05fc-4255-a563-9e27f3c7bedb)


**6. Analiza wyników i wnioski**


1.	Najlepiej sprawdził się klasyfikator regresji logistycznej dla zbioru danych wyodrębnionej ścieżki wokalnej, przy redukcji macierzy wejściowej X do dwóch wymiarów algorytmem LDA, osiągając skutecznośc 74,14% klasyfikacji, przy 10 próbach walidacji krzyżowej 
2.	Dla wielowymiarowej macierzy wejściowej X wyniki klasyfikatorów są bardziej zróżnicowane niż dla zredukowanej do dwóch wymiarów macierzy wejściowej X
3.	Najgorzej sprawiła się klasyfikacja dla przedziału od 10-tej do 30-tej sekundy utworów, w której wartości na każdym z klasyfikatorów dla 10 walidacji krzyżowych były średnio najniższe
4.	Dla macierzy wejściowej X o 26 wymiarach przeważnie najlepiej sprawdzał się klasyfikator lasu losowego, a dla macierzy wejściowej X o 2 wymiarach przeważnie najlepiej sprawdzał się klasyfikator regresji logistycznej
5.	Dla prawie każdego zbioru danych najgorzej sprawdzał się klasyfikator drzewa decyzyjnego
6.	Przy pojedynczej klasyfikacji łatwo zauważyć, że etykieta 3 (Radiohead) najczęściej mylona była z etykietą 2 (La femme). Może to wskazywać na istnienie faktycznej konotacji Radiohead z La Femme na płaszczyźnie elektronicznego brzmienia.
7.	Z pewnością mogłem poszerzyć zbiór danych o całą dyskografię każdego zespołu i artysty, by zgromadzić więcej przykładów, co mogłoby wpłynąć na późniejszą jakość klasyfikacji.
8.	Zamiast prób jedynie dla wielowymiarowego wektora X, mogłem przeprowadzić klasyfikację dla pośrednich mniejszych rozmiarów wektora cech wejściowych X, by sprawdzić, jak klasyfikator radzi sobie z mniej złożonym zbiorem danych wejściowych.


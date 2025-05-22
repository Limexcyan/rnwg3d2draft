# Wykrywanie uszkodzeń pojazdów

## Prowadzenie i cel projektu

Celem projektu jest zapoznanie czytelników z problemem wykrywania uszkodzeń pojazdów w warunkach autonomicznej inspekcji. Do projektu wprowadza [prezentacja](https://docs.google.com/presentation/d/1P974FfTYzp1ZDD83jIs0dkoAGUXZAxtcDl8DwjqhLTI/edit?usp=sharing) przedstawiająca założenia i główne problemy automatycznej inspekcji pojazdów. Pliki `tires.md`, `body.md` oraz `undercarriage.md` przedstawiają bardziej szczegółowo rodzaje uszkodzeń oraz problemy z ich wykrywaniem przy pomocy wizji komputerowej dla odpowiednio, opon, nadwozia oraz karoserii. Ich zawartość rozszerza wiedzę dotyczącą inspekcji pojazdów, jednakże zapoznanie z nimi nie jest konieczne do analizowania kodu i modeli będących kolejnymi częściami projektu.

## Przekształcenia geometryczne obrazów

### Opis problemu

Jednym z wyzwań związanych z wykrywaniem uszkodzeń pojazdów jest ograniczona aparatura, brak ustandaryzowanych rozmiarów pojazdów oraz sposób przeprowadzania inspekcji. 
Dlatego pomocna staje się znajomość geometrycznych transformacji pojazdu, które szerzej omówione są w pliku `geo_im_trans.ipynb`. Notebook zawiera wprowadzenie do przekształceń przy użyciu biblioteki `cv2`. 
Omówione zostaje przeskalowanie, przekształcenie afiniczne, przekształcenie perspektywy oraz obroty obrazów przy pomocy wspomnianej biblioteki. 
Na koniec zostaje omówiony problem zamiany układu współrzędnych obrazu ze współrzędnych kartezjańskich na biegunowe. 
Notatnik zawiera jedno zadanie, którego celem jest przekształcenie bocznego obrazu opony przy użyciu poznanych przekształceń tak, aby ułatwiał odczytywanie informacji na niej zawartych.
Zainteresowany czytelnik może rozszerzyć rozwiązanie do zaprojektowania algorytmu OCR (Optical Character Recognition), aby automatycznie uzyskiwać informacje o danych technicznych opony w surowej formie.

### Instalacja środowiska

Do korzystania z `geo_im_trans.ipynb` wymagane jest stworzenie środowiska. Dependencje używane w notebooku znajdują się w pliku `requirements.txt`.
Przykładowa instalacja środowiska za pomocą condy znajduje się poniżej.

```bash
conda create --prefix env python=3.10 ipykernel
conda activate ./env
pip install -r requirements.txt
```

# Modele wykrywania uszkodzeń

W związku z mnogością potencjalnych uszkodzeń pojazdów (co zostało pokazane chociażby w `tires.md`, `body.md` oraz `undercarriage.md`) jak również mocno ograniczonym dostępem do zbiorów danych, część modelowa projektu została zawężona do wytrenowania dwóch klasyfikatorów. Pozyskanie danych zapewniających informacje z powtarzalnie odbywających się inspekcji pojazdów, na przykład flot samochodów dostawczych, niestety wykracza poza budżet projektu.

## Klasyfikator prawidłowego ciśnienia w oponach

### Informacje o modelu

Pierwszy omawiany model służy do rozpoznawania klasy zdjęcia z pośród trzech możliwych etykiet: opona z prawidłowym ciśnieniem powietrza, opona z niewystarczającym ciśnieniem powietrza oraz zdjęcie, które nie przedstawia opony. 
Klasyfikator oparty na sieci `ResNet-18` został wytrenowany na podstawie zbioru danych [Full vs Flat Tire Images](https://www.kaggle.com/datasets/rhammell/full-vs-flat-tire-images) znajdujących się na platformie kaggle.

### Zbiór danych

Ten zbiór danych składa się z obrazów zarejestrowanych za pomocą kamery OpenMV Cam H7 Plus. Zawiera on 900 obrazów w odcieniach szarości o rozdzielczości 240x240 pikseli, oznaczonych etykietami: `full`, `flat` lub `no-tire`. Obrazy zostały wykonane pod różnymi kątami i z różnych odległości, a także przy różnych kątach obrotu opony. `tire-dataset` zawiera cały zbiór danych zapisany w formacie .jpg. Znajdują się w nim trzy foldery nazwane zgodnie z etykietami klas – `full`, `flat`, `no-tire` – zawierające obrazy przypisane do danej klasy.

Klasa `full` zawiera 300 obrazów opon z odpowiednim ciśnieniem powietrza (około 45 PSI). Obrazy tej klasy są wycentrowane na pojedynczej oponie samochodowej z marginesem 6-12 cali wokół niej. Zdjęcia wykonano pod różnymi kątami i z różnych odległości, a także przy różnych ustawieniach rotacji opony. Klasa `flat` zawiera 300 obrazów opon z niskim ciśnieniem powietrza (około 10 PSI). Obrazy tej klasy są wycentrowane na pojedynczej oponie samochodowej z marginesem 6-12 cali wokół niej. Zdjęcia wykonano pod różnymi kątami i z różnych odległości, a także przy różnych ustawieniach rotacji opony. Klasa `no-tire` zawiera 300 obrazów przedstawiających obiekty inne niż opony. Obejmuje to zdjęcia boku samochodu, tła pustego garażu oraz częściowe ujęcia opon.

### Kod

Kod źródłowy modelu znajduje się na stronie [ResNet-18 for flat tire detection](https://www.kaggle.com/code/ukaszgorczyca/notebooka782d36f54).

## Model sprawdzający, czy pojazd jest uszkodzony

### Informacje o modelu

Drugi model został wytrenowany na zdjęciach mających bardzo ogólne etykiety: pojazd uszkodzony, pojazd bez uszkodzeń. Jest to klasyfikator oparty na sieci `YOLOv8-cls` wytrenowany na podstawie zbioru danych [Car damage detection](https://www.kaggle.com/datasets/anujms/car-damage-detection) znajdujących się na platformie kaggle.

### Zbiór danych

Ten zbiór danych składa się ze zdjęć pojazdów. Zawiera on 2300 obrazów o różnej rozdzielczości, oznaczonych etykietami: `00-damage` lub `01-whole`. Obrazy zostały wykonane pod różnymi kątami i z różnych odległości, obejmują różne perspektywy oraz fragmenty zarejestrowanych pojazdów. Zbiór `Car damage detection` zawiera zdjęcia zapisane w formacie .jpg oraz .jpeg. Znajdują się w nim dwa foldery `training` oraz `validation` (w załączonym notebooku zostaje wykorzystany tylko katalog training, wewnątrz którego następuje train-test split wywoływany automatycznie przez funkcjonalności biblioteki `ultralytics`). Każdy z folderów zawiera dwa podfoldery nazwane zgodnie z etykietami klas, czyli  `00-damage` oraz `01-whole` – zawierające obrazy przypisane do danej klasy.

W folderze `training`, klasa `00-damage` zawiera 920 obrazów uszkodzonych pojazdów, a klasa `01-whole` zawiera 920 obrazów prezentujących nieuszkodzone pojazdy. Zdjęcia wykonano pod różnymi kątami i z różnych odległości, niektóre pliki zawierają całą figurę pojazdu, inne fragmenty, a jeszcze inne zawierają też okolicę w której znajduje się pojazd.

### Kod

Kod źródłowy modelu znajduje się na stronie [YOLOv8 for cars damage detection](https://www.kaggle.com/code/ukaszgorczyca/notebook1c851a9f24)).

## Podsumowanie i możliwe rozszerzenia.

Projekt pozwolił na zaznajomienie się z szerokim zagadnieniem autonomicznej inspekcji pojazdów w celu wykrywania ich uszkodzeń. 
Omówione przeształcenia geometryczne pojazdów stanowiły część preprocesingu danych z inspekcji, natomiast przedstawione modele skupiały się na rozwiązywaniu szczególnych problemów związanych z oponami i z generalnym stanem pojazdów.

Tematyka pozostawia duże pole do rozszerzenia architektury modeli. 
Spojrzenie na jeden problem inspekcji pojazdów jak na wiele problemów wykrywania poszczególnych typów uszkodzeń każe się zastanowić, czy nie warto rozważać tego typu zagadnień metodami continual learningowymi.
Stworzenie modelu uczącego się po kolei jak roznawać różne typy uszkodzeń, a przy tym nie zapominającego jak rozwiązywać poprzednie problemy, stanowi duże wyzwanie. 
Warto jednak jest mu się przyjrzeć, zwłaszacza ze względu na rozwijane obecnie metody continual learningowe wykorzystujące hypersieci między innymi do radzenia sobie z katastroficznym zapominaniem.

Uwagę zwraca również fakt, że do dogłębniejszej analizy problemu konieczne jest uzyskanie wysokiej jakości danych zawierających duże zbiory zdjęć z inspekcji pojazdów. 
Podczas przygotowania projektu doskwierał jednak brak odpowiednio skrojonych do zadania danych i bardzo możliwe, że w momencie jego tworzenia takie dane są tylko i wyłącznie posiadane przez prywatne firmy.

Przy próbie stworzenia narzędzi do pełnej, regularnej inspekcji pojazdów warstwa techniczna mogłaby zostać rozszerzona o bazę danych z historią inspekcji wraz z interfejsem graficznym umożliwiającym ich podgląd oraz powrót do starszych przeglądów.
Taki kierunek rozwoju wymagałby również implementacji bardzo dokładnego algorytmu OCR do odczytywania numerów rejestracyjnych pojazdów, aby w bazie nie miały miejsca pomyłki dotyczące stanu techniczego konkretnych pojazdów z powodu odczytania złego numeru.

### Autor
Łukasz Gorczyca

## Źródła

1. [Przewodnik po modelu YOLOv8](https://medium.com/data-science/the-comprehensive-guide-to-training-and-running-yolov8-models-on-custom-datasets-22946da259c3)
2. [Repozytorium projektu ultralytics](https://github.com/ultralytics/ultralytics)
3. [Rozwiązania biznesowe inspekcji pojazdów](https://www.uveye.com/)
4. [Wykrywanie uszkodzeń przy pomocy modelu Mask-RCNN - repozytorium projektu](https://github.com/basel-ay/Automated-Car-Damage-Detection)
5. [Dokumentacja biblioteki cv2](https://huningxin.github.io/opencv_docs/da/d54/group__imgproc__transform.html)
6. [Poradnik obsługi biblioteki cv2](https://theailearner.com/image-processing/)
7. [Artykuł o implementacji zamiany układu zmiennych obrazu](https://medium.com/@sharadsiddanagoudar/converting-cartesian-image-to-polar-image-ab8522b72b50)
8. [Blog o metodach wykrywania uszkodzeń, część I](https://inspektlabs.com/blog/car-damage-detection-using-ai-methodology-and-approach-for-high-accuracy/)
9. [Blog o metodach wykrywania uszkodzeń, część II](https://inspektlabs.com/blog/ai-car-damage-detection-how-it-works-part-2/)
10. [Wykrywanie uszkodzeń karoserii przy pomocy modelu YOLO](https://www.researchgate.net/publication/370536558_Detects_Damage_Car_Body_using_YOLO_Deep_Learning_Algorithm)
11. Zbiory prywatne

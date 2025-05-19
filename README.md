# Wykrywanie uszkodzeń pojazdów

### Pierwszą składową projektu jest [prezentacja](https://docs.google.com/presentation/d/1P974FfTYzp1ZDD83jIs0dkoAGUXZAxtcDl8DwjqhLTI/edit?usp=sharing) przedstawiająca założenia i główne problemy automatycznej inspekcji pojazdów. 

### Pliki `tires.md`, `body.md` oraz `undercarriage.md` przedstawiają bardziej szczegółowo rodzaje uszkodzeń oraz problemy z ich wykrywaniem przy pomocy wizji komputerowej dla odpowiednio, opon, nadwozia oraz karoserii.

### Notebook `geo_im_trans.ipynb` zawiera poradnik dotyczący geometrycznych przekształceń obrazów, opisujący metody biblioteki `cv2` przydatne w analizie ukszodzeń pojazdów. 
Na podstawie zawartości tego notebooka powstało pierwsze zadanie z pracy własnej, znajdujące się w pliku `homework.ipynb`. 
Do korzystania z `geo_im_trans.ipynb` wymagane jest stworzenie środowiska. Dependencje używane w notebooku znajdują się w pliku `requirements.txt`.

#### Przykładowa instalacja środowiska za pomocą condy

```bash
conda create --prefix env python=3.10 ipykernel
conda activate ./env
pip install -r requirements.txt
```

### W związku z mnogością potencjalnych uszkodzeń pojazdów oraz mocno ograniczonym dostępem do zbiorów danych zapewniających powtarzalne odbywanie się inspekcji, część modelowa projektu została zawężona do wytrenowania dwóch klasyfikatorów.

Pierwszy model służy do rozpoznawania klasy zdjęcia z pośród trzech możliwych etykiet: opona z prawidłowym ciśnieniem powietrza, opona z niewystarczającym ciśnieniem powietrza oraz zdjęcie, które nie przedstawia opony. 
Klasyfikator został wytrenowany na podstawie zbioru danych znajdujących się na platformie kaggle. Kod źródłowy wraz z podpiętym zbiorem danych znajduje się na stronie [kaggle notebook](https://www.kaggle.com/code/ukaszgorczyca/notebooka782d36f54).

Drugi model został wytrenowany na zdjęciach mających bardzo ogólne etykiety: pojazd uszkodzony, pojazd bez uszkodzeń. 
Kod źródłowy wraz z podpiętym zbiorem danych treningowych znajduje się na stronie [kaggle notebook](https://www.kaggle.com/code/ukaszgorczyca/notebooka782d36f54).

### Autor
Łukasz Gorczyca

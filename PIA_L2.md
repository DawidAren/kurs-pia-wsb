# Wdrażanie aplikacji w Azure App Service i GitHub Actions
W tej części kursu nauczysz się czym jest usługa Azure App Service i jak wdrażać w niej aplikacje korzystając z funkcji GitHub Actions.

## Wprowadzenie
Usługi hostowania witryn oraz aplikacji internetowych w modelu PaaS (ang. Platform-as-a-Service) charakteryzują się łatwością wdrażania i testowania nowych wersji, bezpieczeństwem oraz wysoką niezawodnością. Platforma Azure dostarcza dedykowane rozwiązanie do hostowania witryn oraz aplikacji internetowych, działające w modelu PaaS. Azure App Service to usługa oparta na protokole HTTP do hostowania aplikacji internetowych, interfejsów API REST i zapleczy (ang. back-ends) mobilnych. Cechuje się wysoką dostępnością, oferuje automatyczne skalowanie oraz mechanizmy równoważenia obciążenia.
Podczas korzystania z usługi Azure App Service, klient nie zajmuje się utrzymaniem infrastruktury ani środowiska uruchomieniowego – te zadania spoczywają na dostawcy usługi. Najważniejsze cechy usługi Azure App Service to:

* Wsparcie dla technologii .NET, .NET Core, Java, Ruby, Node.js, PHP, Python
* Wsparcie dla systemów operacyjnych Windows oraz Linux
* Wsparcie dla ciągłego wdrażania (ang. continuous deployment)
* Globalna skala i gwarantowane SLA na poziomie 99,95%
* Możliwość użycia niestandardowych domen oraz włączenia protokołu HTTPS
* Mechanizmy uwierzytelniania z użyciem Azure AD, Google, Facebook, Twitter
* Integracja z IDE Visual Studio oraz edytorem Visual Studio Code
* Wsparcie dla tworzenia i udostępniania interfejsów RESTful
* Uruchamianie kodu bezserwerowego (ang. serverless)

### Azure App Service Plan
Plan usługi App Service definiuje zestaw zasobów obliczeniowych dla aplikacji internetowej używanych podczas jej działania. Aplikacje w usłudze App Service uruchamiane są w ramach planu usługi App Service. Po utworzeniu planu usługi App Service w określonym regionie, zestaw zasobów obliczeniowych jest przydzielany dla tego planu w tym regionie. Plan usługi definiuje:

* Region (Europa Zachodnia, Europa Północna, itd.)
* Liczba wystąpień maszyn wirtualnych
* Rozmiar wystąpień maszyn wirtualnych (mały, średni, duży)
* Warstwa cenowa (bezpłatna, współdzielona, podstawowa, Premium, PremiumV2, izolowana)

Warstwa cenowa planu określa do jakich funkcji App Service klient ma dostęp i jaki będzie ich koszt. Istnieje kilka warstw cenowych:

* **Współdzielone (Free, Shared)**: Uruchamiają aplikacje na tej samej maszynie wirtualnej platformy Azure co inne aplikacje usługi App Service, w tym aplikacje innych klientów. W tej warstwie aplikacje mają przydzielony limit czasu procesora, a zasoby nie mogą skalować się w poziomie (liczba wystąpień to zawsze 1). Ze względu na działanie aplikacji na współdzielonych zasobach, warstwa ta zalecana jest do wykorzystania w procesie rozwoju i testowania aplikacji.
* **Dedykowane (Basic, Standard, Premium, PremiumV2)**: Uruchamiają aplikacje na dedykowanych maszynach wirtualnych platformy Azure. Wyłącznie aplikacje w obrębie tego samego planu usługi App Service współdzielą zasoby. Im wyższa warstwa cenowa tym więcej wystąpień maszyn wirtualnych jest dostępnych w ramach skalowania poziomego. W tej warstwie klienci zyskują również dostęp do dodatkowych funkcji takich jak miejsca wdrożenia czy wdrażanie kontenerów.
* **Izolowane (Isolated)**: Uruchamiają aplikacje na dedykowanych maszynach wirtualnych platformy Azure oraz w dedykowanej i izolowanej sieci. Zapewnienie izolacji sieci oraz zasobów obliczeniowych daje największe możliwości skalowania i nawyższą wydajność działania.

W warstwach cenowych **Free** i **Shared**, aplikacja otrzymuje przydział czasu procesora, w którym może działać. Aplikacje działające w ramach tych planów nie mogą być skalowane w poziomie. W przypadku pozostałych planów, aplikacja działa na wszystkich wystąpieniach maszyn wirtualnych skonfigurowanych w tym planie. W przypadku uruchamiania kilku aplikacji w ramach jednego planu, wszystkie współdzielą te same wystąpienia maszyn wirtualnych. Ta sama zasada dotyczy wykorzystania miejsc wdrożenia. Logowanie, wykonywanie kopii zapasowych oraz uruchamianie zadań (ang. WebJobs), również jest wykonywane na wystąpieniach maszyn wirtualnych w ramach tego samego planu.

### Praca z planami usługi App Service w wierszu poleceń
Do zarządzania planami usługi App Service służy polecenie `az appservice`. Do utworzenia nowego planu usługi App Service służy polecenie `az appservice plan create`:

```sh
az appservice plan create \
  --resource-group <nazwa-grupy-zasobów> \
  --name <nazwa-planu> \
  --location <lokalizacja> \
  --sku <warstwa-cenowa>
```

Plan oparty na systemie operacyjnym Linux tworzy się dodając argument `--is-linux`.

Listę dostępnych lokalizacji dla wybranej warstwy cenowej można wyświetlić za pomocą polecenia `az appservice list-locations`:

```sh
az appservice list-locations \
  --sku <warstwa-cenowa>
```

Istnieją dwa sposoby skalowania planu usługi App Service:

* **Skalowanie pionowe**, zwane również skalowaniem w górę (ang. scale up, scale down), oznacza zmianę wydajności maszyny wirtualnej. W kontekście planu usługi App Service oznacza to zmianę warstwy cenowej na niższą lub wyższą.
* **Skalowanie poziome**, zwane również skalowaniem wszerz (ang. scale in, scale out), oznacza dodanie lub usunięcie maszyn wirtualnych. W kontekście planu usługi App Service oznacza to uruchomienie dodatkowych maszyn na których będzie działać aplikacja.

Do skalowania poziomo służy polecenie `az appservice plan update`:

```sh
az appservice plan update \
  --resource-group <nazwa-grupy-zasobów> \
  --name <nazwa-planu> \
  --sku <warstwa-cenowa>
```

Do skalowania pionowego służy to samo polecenie, ale z innym argumentem:

```sh
az appservice plan update \
  --resource-group <nazwa-grupy-zasobów> \
  --name <nazwa-planu> \
  --number-of-workers <liczba-maszyn>
```

Plan usługi App Service usuwa się poleceniem `az appservice plan delete`:

```sh
az appservice plan delete \
  --resource-group <nazwa-grupy-zasobów> \
  --name <nazwa-planu>
```

### Praca z aplikacjami usługi Azure App Service w wierszu poleceń
Do zarządzania aplikacjami w usłudze App Service służy polecenie `az webapp`. Aby utworzyć nową aplikację w ramach istniejącego planu należy skorzystać z polecenia `az webapp create`:

```sh
az webapp create \
  --resource-group <nazwa-grupy-zasobów> \
  --plan <nazwa-planu> \
  --name <nazwa-aplikacji> \
  --runtime <środowisko-uruchomieniowe>
```

Należy pamiętać o tym, że nazwa aplikacji musi być unikalna globalnie.

Listę środowisk uruchomieniowych pobiera się poleceniem `az webapp list-runtimes`. W przypadku planu opartego o system operacyjny Linux należy dodać argument `--linux`.

Usługa App Service wspiera kilka scenariuszy wdrożenia. Najczęstszym jest wykorzystanie repozytorium git. Aby móc skorzystać z tej opcji należy skonfigurować użytkownika wdrożenia. Robi się to za pomocą polecenia `az webapp deployment user set`:

```sh
az webapp deployment user set \
  --user-name <nazwa-użytkownika> \
  --password <hasło>
```

Aby móc wypchnąć kod i wdrożyć aplikację należy najpierw pobrać adres repozytorium poleceniem `az webapp deployment source`:

```sh
az webapp deployment source config-local-git \
  --resource-group <nazwa-grupy-zasobów> \
  --name <nazwa-aplikacji> \
  --query url
```
Powyższe polecenie wyświetli adres URL repozytorium wdrożenia. Posiadając adres można skonfigurować lokalne repozytorium git:

```sh
git remote add azure <url-repozytorium-wdrożenia>
```

Aby wdrożyć aplikację należy wypchnąć kod do tego repozytorium:

```sh
git push azure master
```

Wypchnięcie kodu spowoduje uruchomienie procesu wdrożenia aplikacji. Za całość po stronie platformy Azure odpowiedzialna jest usługa o nazwie Kudu, która dodatkowo udostępnia API do zarządzania ustawieniami aplikacji, przeglądania plików, monitorowania procesów, zarządzania wersjami środowisk uruchomieniowych oraz uruchamiania zadań, konfiguracji tzw. WebHooks i uruchamiania zadań (ang. WebJobs).

Po zakończeniu wdrożenia aplikacja dostępna jest pod adresem `http://<nazwa-aplikacji>.azurewebsites.net`. Aplikację usuwa się poleceniem `az webapp delete`:

```sh
az webapp delete \
  --resource-group <nazwa-grupy-zasobów> \
  --name <nazwa-aplikacji>
```

Usunięcie aplikacji nie powoduje usunięcia planu.

Dodatkowe materiały:

* [Dokumentacja Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/overview)

### Ciągłe wdrażanie aplikacji z użyciem GitHub Actions

**Ciągła integracja** (ang. continuous integration) to proces w którym programiści tworzący oprogramowanie na gałęziach deweloperskich, integrują swoje zmiany do gałęzi głównej tak często jak to tylko możliwe. Wszystkie zmiany są sprawdzane pod kątem poprawności poprzez uruchomienie testów automatycznych. Ciągłe wdrażanie kładzie duży nacisk na automatyzację testowania.

**Ciągłe dostarczanie** (ang. continuous delivery) to rozszerzenie procesu ciągłej integracji, które polega na weryfikacji czy zmiany w oprogramowaniu mogą zostać dostarczone do klientów końcowych bez błędów. Do automatyzacji testowania dodany zostaje proces przygotowania oprogramowania do wdrożenia w taki sposób, że udostępnienie nowej wersji możliwe jest do wykonania w sposób prawie natychmiastowy.

**Ciągłe wdrażanie** (ang. continuous deployment) to rozszerzenie procesu ciągłego dostarczania, w którym końcowi klienci automatycznie otrzymują nowe wersje oprogramowania. Każda zmiana, która przejdzie przez wszystkie etapy produkcji zostaje udostępniona końcowym klientom. Cały proces odbywa się bez udziału człowieka.

Wszystkie trzy procesy pozwalają skrócić czas potrzebny do udostępnienia nowych funkcji oprogramowania oraz prowadzą do zmniejszenia liczby błędów. We wszystkich procesach położony jest duży nacisk na automatyzację każdego etapu co wymaga użycia nowych narzędzi w organizacji.

Serwis GitHub udostępnia funkcjonalność o nazwie GitHub Actions, która umożliwia wprowadzenie procesów ciągłej integracji, dostarczania i wdrażania dla kodu hostowanego w repozytoriach serwisu GitHub. Podstawowe pojęcia związane z usługą GitHub Actions:
* **Akcja** (ang. action): Pojedyncze zadanie, które w połączeniu z innymi zadaniami tworzy zadanie nadrzędne. Zadania są najmniejszymi jednostkami w pliku przepływu. GitHub umożliwia tworzenie własnych akcji oraz wykorzystywanie akcji stworzonych przez innych użytkowników serwisu.
*	**Artefakt** (ang. artifact): Pliki utworzone podczas procesu budowania i testowania kodu. Przykładem artefaktu mogą być pliki binarne po kompilacji kodu, wyniki testów, zrzuty ekranu czy też dzienniki zdarzeń.
*	**Zdarzenie** (ang. event): Aktywność wyzwalająca uruchomienie przepływu. Przykładem zdarzenia jest zatwierdzenie przez użytkownika zmian w pliku i wypchnięcie go do repozytorium w serwisie GitHub.
* **Runner**: GitHub udostępnia maszyny z systemami Linux, Windows i macOS do uruchamiania zadań. Na potrzeby wykonania zadań aprowizowane są nowe maszyny wirtualne aby zapewnić czyste środowisko pracy.
* **Zadanie** (ang. job): Zestaw akcji do wykonania na maszynie wirtualnej. Zadania definiowane są w pliku przepływu i mogą być wykonywane sekwencyjnie lub równolegle w zależności od potrzeb.
* **Przepływ** (ang. workflow): Konfigurowalny i automatyczny proces zdefiniowany w repozytorium do budowania, testowania, pakowania i wdrażania projektów. Na przepływ składa się jedno lub więcej zadań.
* **Plik przepływu** (ang. workflow file): Plik w formacie YAML opisujący przepływ. Przechowywany wraz z kodem repozytorium w katalogu .github/workflows.

Poniższy przykład prezentuje plik przepływu opisujący jedno zadanie wykonujące akcję dostarczoną przez programistów platformy GitHub:

```yaml
on: [push]
jobs:
  build:
    name: Greeting
    runs-on: ubuntu-latest
    steps:
      https://github.com/actions/hello-world-javascript-action
      - name: Hello world
        uses: actions/hello-world-javascript-action@v1
        with:
          who-to-greet: 'Mona the Octocat'
        id: hello
      - name: Echo the greeting's time
        run: echo 'The time was ${{ steps.hello.outputs.time }}.'
```

Zdarzenie na które zostanie uruchomione zadanie znajduje się w linii 1 – wypchnięcie kodu do którejkolwiek z gałęzi w repozytorium w serwisie GitHub spowoduje uruchomienie zadań opisanych niżej. Sekcja opisująca zadania rozpoczyna się w linii 2. Poniżej, w linii 3, rozpoczyna się opis pierwszego zadania. W linii 4 nadana zostaje mu nazwa `Greeting`. W linii 5 wybrany zostaje system operacyjny na którym zadanie będzie wykonane. W tym przypadku jest to najnowsza wersja Ubuntu. Dalej od linii 6 opisane są kroki zadania.

### Łączenie usługi App Service z GitHub Actions
Dzięki akcjom przygotowanym przez firmę Microsoft możliwe jest wdrażanie kodu w usłudze App Service korzystając z GitHub Actions. Pierwszym krokiem do integracji usług jest utworzenie pliku przepływu w katalogu `.github/workflows` i umieszczenie go razem z kodem w repozytorium.

Poniższy przykład opisuje przepływ, który zostanie uruchomiony na każde wypchnięcie kodu do gałęzi `master` w repozytorium w serwisie GitHub:

```yaml
on:
  push:
    branches:
      - master

env:
  AZURE_WEBAPP_NAME: <nazwa-aplikacji>
  AZURE_WEBAPP_PACKAGE_PATH: '.'
  NODE_VERSION: '12.9'

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
    - name: Install dependencies
      run: |
        npm install
    - name: 'Deploy to Azure WebApp'
      uses: azure/webapps-deploy@v1
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

Powyższy przepływ składa się z jednego zadania na które składają się trzy kroki:

* Instalacja środowiska uruchomieniowego Node.js na maszynie wirtualnej
* Zainstalowanie zależności projektu poleceniem `npm install`
* Wdrożenie aplikacji w usłudze Azure App Service

Dodatkowe materiały:

* [Dokumentacja GitHub Actions](https://help.github.com/en/actions)

## Zadanie 1
Skonfiguruj automatyczne wdrażanie aplikacji za pomocą funkcji GitHub Actions w usłudze Azure App Service.

**Wartości w nawiasach ostrych, na przykład `<nazwa-aplikacji>`, należy zamienić na nazwy własne, na przykład `moja-aplikacja`.**

1. Otwórz wiersz poleceń i zaloguj się na swoje konto Azure:

```sh
az login
```

2. Utwórz grupę zasobów:

```sh
az group create --name <grupa-zasobów> --location westeurope
```

3. Utwórz plan aplikacji:

```sh
az appservice plan create --resource-group <grupa-zasobów> --name <nazwa-planu> --is-linux --sku FREE
```

4. Utwórz aplikację:

```sh
az webapp create --resource-group <grupa-zasobów> --plan <nazwa-planu> --name <nazwa-aplikacji> --runtime "NODE|12.9"
```

5. Przejdź do zasobu aplikacji w Azure Portal a następnie do zakładki **Omówienie** i pobierz jej profil publikowania klikając przycisk **Pobierz profil publikowania**:

![](obrazy/L2_Z1_Profil.png)

**Aplikacja w Azure Portal może pojawić się z opóźnieniem.**

6. Przejdź do zakładki **Settings > Secrets** na stronie repozytorium GitHub:

![](./obrazy/L2_Z1_1.png)

7. Dodaj nowy sekret o nazwie `AZURE_WEBAPP_NAME`. Jako wartość podaj nazwę swojej aplikacji utworzonej w kroku nr 4 (argument `<nazwa-aplikacji>`).

8. Dodaj nowy sekret o nazwie `AZURE_WEBAPP_PUBLISH_PROFILE`. Jako wartość podaj profil publikowania pobrany w kroku nr 5 (otwórz plik i wklej tutaj jego zawartość).

9. W katalogu projektu utwórz plik `.github/workflows/azure.yml` i dodaj następującą treść:

```yaml
on: push

jobs:
  build-and-deploy:
    name: Build and deploy
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout GitHub Action' 
      uses: actions/checkout@master
    
    - name: Setup Node
      uses: actions/setup-node@v1
      with:
        node-version: '12.9'

    - name: 'Install dependencies'
      run: |
        npm install
       
    - name: 'Deploy to Azure WebApp'
      uses: azure/webapps-deploy@v1
      with: 
        app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

10. Dodaj pliki:

```sh
git add --all
```

11. Zatwierdź zmiany w repozytorium:

```sh
git commit -m "Dodano integrację z GitHub Actions"
```

12. Wypchnij zmiany do repozytorium w serwisie GitHub

```sh
git push origin master
```

13. Przejdź do zakładki **Actions** na stronie repozytorium GitHub i poczekaj aż akcja wdrożenia zakończy się powodzeniem:

![](obrazy/L2_Z1_akcja.png)

Następnie otwórz aplikację w przeglądarce przechodząc pod adres `http://<nazwa-aplikacji>.azurewebsites.net` i sprawdź czy działa tak samo jak uruchomiona lokalnie.

## Zadanie 2
Utwórz nową gałąź o nazwie `test` z gałęzi `master`. W pliku `.github/workflows/azure.yml` dodaj nowy krok o nazwie `Run tests`, który uruchomi testy za pomocą skryptu `test` zdefiniowanym w pliku `package.json`. Testy powinny zostać wykonane wyłącznie po wypchnięciu zmian do gałęzi `test`. Wszystkie zmiany zatwierdź w repozytorium z komunikatem "Dodano uruchamianie testów" i wypchnij do serwisu GitHub. Sprawdź czy odpowiednia akcja została uruchomiona i czy zakończyła się błędem.

## Zadanie 3
Utwórz nową gałąź o nazwie `staging` z gałęzi `master`. Zmodyfikuj plik `.github/workflows/azure.yml` w taki sposób aby wdrożenie zostało przeprowadzone do miejsca wdrożenia o nazwie `staging`. Wdrożenie powinno zostać wykonane wyłącznie po wypchnięciu kodu gałęzi `staging`. Miejsce wdrożenia utwórz w aplikacji usługi App Service na potrzeby tego zadania a następnie je usuń. Wszystkie zmiany zatwierdź w repozytorium z komunikatem "Skonfigurowano miejsce wdrożenia" i wypchnij do serwisu GitHub. Sprawdź czy odpowiednia akcja została uruchomiona i czy wdrożenie się powiodło.

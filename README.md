# Programowanie i architektura aplikacji w chmurze

## Wprowadzenie
Podczas tego kursu zaimplementujesz oraz wdrożysz prostą aplikację internetową do tworzenia listy zadań, tzw. todo list. Po skończeniu aplikacja będzie umożliwiać dodawanie oraz usuwanie zadań, zmianę ich statusu oraz wysyłanie przypomnień na wskazany adres email. Aplikacja będzie działać na platformie Azure, a dostęp do niej możliwy będzie za pośrednictwem internetu.

## Zaliczenie
Repozytorium kursu zawiera 8 list zadań. Na każdej liście znajdują się 3 zadania: jedno zadanie na zaliczenie (1) oraz dwa zadania dodatkowe na wyższą ocenę (2 i 3).

Repozytorium projektu w serwisie GitHub, które utworzysz podczas zajęć będzie podstawą do wystawienia oceny. Nie ma potrzeby wysyłania kodu źródłowego projektu przez email lub platformę Moodle do prowadzącego. Termin wykonania wszystkich zadań przypada na dzień, w którym odbędą się ostatnie zajęcia.

## Rozpoczynanie pracy
Do napisania oraz pzetestowania działania aplikacji będziesz potrzebował komputera z systemem operacyjnym Windows, Linux albo macOS oraz następujący narzędzi: Git, Node.js, Azure CLI, Azure Functions Core Tools oraz edytor tekstu.

### Git
Git to system kontroli wersji umożliwiający zapisywanie i przeglądanie historii zmian kodu źródłowego. Używać go będziesz do zapisywania swojej pracy i udostępniania jej w serwisie GitHub. Instrukcja instalacji dla najpopularniejszych systemów operacyjnych znajduje się na [oficjalnej stronie projektu](https://git-scm.com/downloads).

### Node.js
Node.js to środowisko uruchomieniowe dla języka JavaScript. Wykorzystasz je do implementacji części serwerowej aplikacji. W przypadku systemu operacyjnego Windows najłatwiejszą metodą instalacji jest skorzystanie z instalatora dostępnego na [oficjalnej stronie projektu](https://nodejs.org/en/download/). W przypadku dystrybucji Linux oraz macOS rekomendowanym przeze mnie sposobem instalacji jest skorzystanie z narzędzia NVM (ang. Node Version Manager). Instrukcję znajdziesz na [oficjalnej stronie projektu](https://github.com/nvm-sh/nvm#installing-and-updating).

### Azure CLI
Azure CLI (ang. Azure Command Line Interface) to zestaw poleceń do zarządzania zasobami platformy Azure. Instrukcję instalacji znajdziesz na [oficjalnych stronach dokumentacji](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).

### Azure Functions Core Tools
Azure Functions Core Tools to zestaw narzędzi umożliwiających lokalne tworzenie i testowanie funcji usługi Azure Functions. Instrukcję instalacji znajdziesz na [oficjalnych stronach dokumentacji](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#v2).

### Edytor tekstu
Możesz korzystać z dowolnego edytora tekstu do edycji kodu źródłowego. Moją rekomendacją jest Visual Studio Code, który możesz pobrać z [oficjalnej strony projektu](https://code.visualstudio.com/).

## Tworzenie projektu

### Zakładanie repozytorium
Przejdź na [stronę tworzenia repozytorium](https://github.com/new) w serwisie GitHub. Jeżeli nie posiadasz konta w serwisie GitHub, możesz je założyć za darmo.

W polu *Repository name* wpisz *todo-list*. Ustaw widoczność repozytorium na *Public*. Z listy *Add .gitignore* wybierz *Node*. Z listy *Add a license* wybierz *MIT License*. Kliknij przycisk *Create repository*.

**Adres URL do strony repozytorium w serwisie GitHub wyślij na adres email prowadzącego.**

### Klonowanie repozytorium
Ze strony głównej repozytorium pobierz adres URL klonowania klikając przycisk *Clone or download* i kopiując adres z pola tekstowego. Następnie otwórz terminal i sklonuj repozytorium (zmień `<adres-repozytorium>` na adres URL klonowania repozytorium):

```sh
git clone <adres-repozytorium>
```

W katalogu bieżącym zostanie utworzony katalog o nazwie `todo-list`, z taką samą zawartością jak w repozytorium w serwisie GitHub.

### Konfiguracja projektu
Otwórz terminal i wykonaj następujące polecenie w katalogu `todo-list`:

```sh
npm init -y
```

W katalogu projektu zostanie utworzony plik `pacakge.json`. Zawiera on ustawienia projektu, w tym jego nazwę, wersję, link do repozytorium i inne. W przyszłości znajdą się tutaj również zależności do różnych bibliotek. Plik ten powinien znaleźć się w repozytorium projektu w serwisie GitHub. Aby to zrobić, wykonaj następujące kroki:

Dodaj zmiany:

```sh
git add --all
```

Zatwierdź zmiany w lokalnym repozytorium:

```sh
git commit -m "Utworzono projekt"
```

Wypchnij zmiany do repozytorium w serwisie GitHub:

```sh
git push origin master
```

Zostaniesz poproszony o podanie nazwy użytkownika oraz hasła w serwisie GitHub. Przejdź na stronę główną repozytorium w serwisie GitHub i sprawdź czy znajduje się tam plik `package.json`.

Jeżeli wszystkie kroki wykonałeś poprawnie, możesz przejść do pracy nad [pierwszą listą zadań](PIA_L1.md).
# Instrukcja AI

## Szybki kontekst
- Standardowy kod aplikacji znajduje się w [../../corecode](../../corecode) (warstwa Base). Customizacje rozwijamy w [../../DEV/workspace](../../DEV/workspace) (warstwa Cust), a instrukcje w tym repo ([.](.)).
- Dokumentacja produktowa jest poza repozytorium w [../../TechDocs](../../TechDocs); zawsze używaj wersji odpowiadającej aktywnemu checkoutowi CoreCode.
- Stos technologiczny: Marble DSL (entity/projection/client/fragment), PL/SQL (logika biznesowa i API), projekcje REST oraz klient Aurena.
- Buildy, deploy i debug realizujemy w IFS Developer Studio 18 z użyciem build places DEV (Build Place), CFG i TRN (Use Places).
- Wszelkie odpowiedzi formułuj po polsku, z odnośnikami do konkretnych plików/ścieżek w workspace.

## Przegląd projektu
- Repo wspiera AI podczas rozszerzania IFS Cloud (logika PL/SQL, projekcje, klient Aurena) z zachowaniem layering.
- Marble i modele wizualne generują obiekty DB i boilerplate – nie usuwaj wygenerowanych sekcji ręcznie.
- Warstwa Base w [../../corecode](../../corecode) jest tylko do odczytu; wszystkie customizacje trafiają do `*-Cust.*` w [../../DEV/workspace](../../DEV/workspace).
- Workspace jest w pełni zarządzany przez IFS DevOps, więc nie umieszczaj tam plików GitHub (dokumentacja, workflowy) – trzymaj je w repo lub `.github/`.

## Architektura i stos
- **Backend (DB)**: PL/SQL pakiety `<MODULE>_<ENTITY>_API`, widoki z filtrowaniem bezpieczeństwa, procedury `Insert___/Update___/Delete___`.
- **Warstwa API**: pliki `.projection` mapują widoki na encje REST, eksponują akcje/funkcje i structure selectors.
- **Frontend**: Aurena (`.client` lub designer), fragmenty UI, polecenia MVVM (`command`, `selector`, `list`, `card`, `navigate`).
- **Deployment**: IFS Developer Studio 18 + Delivery Manager / Ant (`../../DEV/build.xml`, `../../DEV/nbproject/ant-deploy.xml`).

## Zasady ogólne i nazewnictwo
- Nie zmieniaj plików Base; rozszerzenia twórz jako `*-Cust.*` w [../../DEV/workspace](../../DEV/workspace), odwzorowując strukturę entity/projection/client/fragment/overview.
- Fragmenty (selektory, LOV) utrzymuj jako współdzielone – dołączaj je zamiast powielać zapytania.
- **Prefiksy artefaktów**:
  - Komponenty NSITN, NSDEV, NSMES: nowe pliki `entity`, `projection`, `client` zaczynaj od `Ns` (np. `NsPriceList-Cust.projection`). Wewnętrzne obiekty mogą zachować nazwy core.
  - Pozostałe komponenty: wszystkie nowe pliki i obiekty oznaczaj prefiksem `C` (np. `COrderAddon.client`, `CNewAttr`).
- Przy rozszerzaniu encji wstawiaj atrybuty `C*` (lub `Ns*` w modułach zaczynających się od `ns`).
- Pakiety PL/SQL stosują standard `Check_*`, `Unpack_*`, `Insert___/Update___/Delete___`, `Client_SYS.Add_To_Attr()` oraz `Error_SYS.Record_General()`.

## Workflow deweloperski
- Pracuj na środowisku DEV (Build Place). PL/SQL pisz w SQL Developer/VS Code, projekcje w Solution Manager lub bezpośrednio w Cust.
- Po każdej zmianie Marble uruchom generację (`Build > Generate`) i przejrzyj logi w `../../DEV/build/log`.
- UI testuj w Aurena z włączonym trybem debug, korzystając z ról odpowiadających docelowemu użytkownikowi.
- Przed promocją na CFG/TRN zadbaj o pakiet testowy (smoke + regresja modułowa) w dedykowanym build place.

## Standardy PL/SQL
- Buduj ciągi atrybutów przez `Client_SYS.Add_To_Attr()`.
- Błędy raportuj `Error_SYS.Record_General()` i rozdzielaj walidacje (`Check_*`) od logiki biznesowej.
- Zawsze ustaw kontekst firmy (`User_Finance_API.Get_Default_Company()`) i sprawdzaj uprawnienia użytkownika.
- Nie twórz własnego cachingu – korzystaj z wbudowanych mechanizmów IFS.

## Wzorce projekcji
- Utrzymuj `keys`, `etag` oraz `ludependencies` dla kontroli współbieżności.
- Mapuj widoki/encje na REST, eksponuj akcje/funkcje powiązane z pakietami PL/SQL.
- Structure selectors wykorzystuj do optymalizacji odczytów w UI.
- LOV/asocjacje dokumentuj; `@DynamicComponentDependency` dodawaj przy odwołaniach do NSINT lub innych komponentów.

## Wzorce klienta Aurena
- Rozszerzenia utrzymuj w `*-Cust.client`, kopiując układ grup/zakładek Base.
- `command` mapuj na operacje projekcji; `selector` ogranicza dane ładowane na stronę.
- Korzystaj z widoków `list`/`card` adekwatnie do scenariusza oraz `navigate` dla przejść.
- Fragmenty UI i LOV trzymaj w Cust i współdziel pomiędzy stronami.

## Dodawanie pól do encji core (np. SalesPriceList, CustomerOrder)
1. Uaktualnij `*-Cust.entity` w [../../DEV/workspace](../../DEV/workspace), dodając atrybut (`C*` lub `Ns*`) i relacje.
2. W `*-Cust.projection` wystaw pole, dodaj LOV/asocjacje i wymagane `@DynamicComponentDependency`.
3. W `*-Cust.client` dodaj kontrolkę w odpowiedniej grupie, korzystając z istniejących fragmentów (np. `NsLabelActiveSelector`).
4. Gdy tworzysz nowy lookup, zapewnij pełny zestaw Cust (encja, projection handler, fragment selektora, strona klienta, wpis w nawigatorze, opcjonalny overview).
5. Wygeneruj metadane i przetestuj zarówno projekcję (REST), jak i UI Aurena.

## Wzorzec lookup/kategorii (np. NsPriceListCategory)
- Dane referencyjne przechowuj w [../../DEV/workspace/nsint/model/nsint](../../DEV/workspace/nsint/model/nsint).
- Odwołania z ORDER i innych komponentów rób poprzez asocjacje/LOV oraz dynamic dependencies.
- Dodawaj wpisy w `NsintNavigator`, grupując je z pokrewnymi stronami (np. `NsLabelsNav`).

## Testowanie
- PL/SQL: testy jednostkowe (PL/SQL Test) + walidacja transakcji i obsługi błędów.
- Integracja: wywołania projekcji (CRUD, akcje, funkcje) oraz structure selectors.
- UI: Aurena z różnymi rolami, w tym tryb debug.
- Wydajność: używaj generatorów danych IFS na dużych wolumenach.

## Typowe pułapki
- Nie modyfikuj bezpośrednio standardowych obiektów – zawsze używaj warstwy Cust/overlay.
- Sprawdzaj uprawnienia i kontekst wielofirmowy w każdym zapytaniu.
- Uwzględniaj `ludependencies` i `etag`, aby uniknąć kolizji.
- Korzystaj z wbudowanego cachingu zamiast implementować własny.

## TechDocs
- Przechowywane lokalnie (np. `c:/IFS Cloud/TechDocs/25R1`, `25R2`, `26R1`). Wybieraj wersję pasującą do aktywnego CoreCode.
- Przed implementacją sprawdź odpowiedni temat (np. „010 add edit persistent attribute”, sekcje projekcji/Aurena).
- Nie kopiuj dużych fragmentów TechDocs; streszczaj i odsyłaj do tytułu. W razie konfliktu TechDocs jest źródłem prawdy.

## CoreCode
- Checkouty referencyjne znajdują się w `c:/IFS Cloud/corecode/<moduły>` (np. order, invent, fndbas, purch...).
- Traktuj je jako read-only; edytuj tylko odpowiadające im pliki `*-Cust.*` w workspace.
- Nie łącz workflow Git tego repo z DevOps CoreCode (oddzielne systemy wersjonowania).

## Struktura projektu
- Struktura obejmuje warstwę DB (PL/SQL), projekcje, klienta Aurena oraz skrypty build/deploy. Dodając nowe katalogi, kieruj się podziałem funkcjonalnym modułów IFS.

## Moduły w workspace
- [../../DEV/workspace/order](../../DEV/workspace/order): sprzedaż i zamówienia klienta (np. CustomerOrder).
- [../../DEV/workspace/invent](../../DEV/workspace/invent): gospodarka magazynowa i zapasy.
- [../../DEV/workspace/cost](../../DEV/workspace/cost): kalkulacje kosztowe i controlling.
- [../../DEV/workspace/cussch](../../DEV/workspace/cussch): obsługa serwisowa i zgłoszenia.
- [../../DEV/workspace/docman](../../DEV/workspace/docman): zarządzanie dokumentami i linkami.
- [../../DEV/workspace/disord](../../DEV/workspace/disord): dystrybucja zamówień i fulfillment.
- [../../DEV/workspace/fndbas](../../DEV/workspace/fndbas): fundamenty (security, messaging, background plans).
- [../../DEV/workspace/nsdev](../../DEV/workspace/nsdev): moduły Ns dla rozwiązań developerskich.
- [../../DEV/workspace/nsint](../../DEV/workspace/nsint): encje słownikowe/lookup wykorzystywane w innych komponentach.
- [../../DEV/workspace/nsmes](../../DEV/workspace/nsmes): integracje MES i terminale shop-floor.
- [../../DEV/workspace/dop](../../DEV/workspace/dop): produkcja dyskretna i planowanie operacji.
- [../../DEV/workspace/fnddev](../../DEV/workspace/fnddev): narzędzia developerskie i rozszerzenia platformowe.

## Deployment i dostawy
1. **Generacja metadanych**: w IFS Developer Studio 18 otwórz warstwę z [../../DEV/workspace/layers.ini](../../DEV/workspace/layers.ini) i wybierz `Build > Generate`. Artefakty trafią do [../../DEV/build/gen](../../DEV/build/gen); logi znajdują się w [../../DEV/build/log](../../DEV/build/log).
2. **Pakietowanie**: użyj `Delivery > Create Delivery` lub Ant (`ant -f ../../DEV/build.xml delivery -Ddeliver.id=NsAddon_25R1_001 -Ddeliver.dir=../../DEV/build/deliveries`). Zip znajdziesz w [../../DEV/build/deliveries](../../DEV/build/deliveries).
3. **Publikacja**: w Delivery Manager wgraj paczkę do DEV (np. `DEV_BP01`), wykonaj smoke/regresję, następnie promuj bez zmian do CFG (`CFG_BP01`), TRN (`TRN_BP01`) i dalszych miejsc (QA/UAT/PRD) według planu release.
4. **Kontrola wersji**: trzymaj spójny identyfikator paczki (np. `NsAddon_25R1_001`) i changelog w repo. Jeden build = konkretny zestaw user stories/CR.
5. **Rollback**: archiwizuj poprzednie paczki w [../../DEV/build/deliveries/backup](../../DEV/build/deliveries/backup) wraz z instrukcją przywrócenia przed wdrożeniem na CFG/TRN.

### Build places i promocje
- **DEV** (Build Place): środowisko iteracyjne; po większych zmianach wykonuj `Build > Clean`.
- **CFG** (Use Place): tylko zatwierdzone pakiety; służy do walidacji konfiguracji.
- **TRN** (Use Place): szkolenia/UAT. Dodatkowe miejsca (QA, PREPROD, PROD) używają tej samej paczki, zmienia się jedynie docelowy place.
- Każdy place ma własne repo metadanych – nigdy nie kopiuj ręcznie plików między miejscami.

### Automatyzacja Ant
- `ant -f ../../DEV/build.xml generate` – generuje metadane (korzysta z `workspace/layers.ini`).
- `ant -f ../../DEV/build.xml delivery -Ddeliver.id=<ID> -Ddeliver.dir=../../DEV/build/deliveries` – tworzy paczkę Cust.
- Skrypt [../../DEV/nbproject/ant-deploy.xml](../../DEV/nbproject/ant-deploy.xml) oferuje cele `run`, `deploy`, `redeploy`; konfiguruj poprzez właściwości (`-Difs.workspace=...`).
- `ant -f ../../DEV/build.xml clean` usuwa `build/gen` i `build/merge` – używaj przed pełnym rebuildem.

## Zasoby
- IFS Cloud Tech Docs: https://docs.ifs.com/
- IFS Community: https://community.ifs.com/
- Standardowe prefiksy modułów (ORDER, INVENT, ACCRUL itd.) i zasada `C`/`Ns` obowiązują dla wszystkich nowych artefaktów.

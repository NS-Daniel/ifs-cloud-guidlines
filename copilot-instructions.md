# Instrukcje GitHub Copilot dla IFS Cloud

## Cel
Ten dokument zawiera wytyczne i instrukcje dla GitHub Copilot dotyczące rozwoju aplikacji IFS Cloud.

## Ogólne zasady

### Konwencje nazewnictwa
- Używaj CamelCase dla nazw klas i typów
- Używaj snake_case dla nazw zmiennych i funkcji w PL/SQL
- Używaj camelCase dla JavaScript/TypeScript
- Prefiksy: `API_` dla publicznych API, `IMPL_` dla implementacji

### Struktura kodu
- Zachowaj modularność kodu
- Stosuj wzorce projektowe zgodne z architekturą IFS Cloud
- Dokumentuj złożone funkcje i procedury
- Używaj komentarzy w języku angielskim dla kodu źródłowego

### Standardy jakości
- Zawsze sprawdzaj poprawność składni SQL
- Stosuj odpowiednią obsługę wyjątków
- Waliduj dane wejściowe
- Optymalizuj zapytania SQL

## Specyficzne wytyczne IFS Cloud

### Praca z PL/SQL
```sql
-- Przykład struktury procedury
PROCEDURE Process_Order (
   order_id_ IN NUMBER,
   result_   OUT VARCHAR2
) IS
   -- Deklaracje zmiennych
BEGIN
   -- Implementacja
   -- Obsługa błędów
EXCEPTION
   WHEN OTHERS THEN
      Error_SYS.Record_General(lu_name_, 'ERROR: :P1', SQLERRM);
      RAISE;
END Process_Order;
```

### Praca z API
- Używaj standardowych metadanych IFS
- Stosuj wersjonowanie API
- Dokumentuj endpointy i parametry
- Implementuj odpowiednią autoryzację

### Bezpieczeństwo
- Nigdy nie przechowuj haseł w plain text
- Używaj bind variables w SQL
- Waliduj wszystkie dane użytkownika
- Stosuj zasadę najmniejszych uprawnień (principle of least privilege)

### Testy
- Pisz testy jednostkowe dla logiki biznesowej
- Testuj przypadki brzegowe
- Sprawdzaj obsługę błędów
- Dokumentuj scenariusze testowe

## Best Practices

1. **Wydajność**: Optymalizuj zapytania, używaj indeksów odpowiednio
2. **Czytelność**: Kod powinien być samowytłumaczający się
3. **Konserwacja**: Ułatwiaj przyszłe modyfikacje
4. **Dokumentacja**: Aktualizuj dokumentację wraz z kodem
5. **Review**: Kod powinien być gotowy do code review

## Zasoby

- Dokumentacja IFS Cloud
- Standardy kodowania firmy
- Przewodniki architektoniczne

## Kontakt

W razie pytań lub wątpliwości, skontaktuj się z zespołem architektonicznym.

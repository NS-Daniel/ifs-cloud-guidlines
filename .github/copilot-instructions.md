# Instrukcje GitHub Copilot dla IFS Cloud Development

## Wprowadzenie
Ten dokument zawiera szczegółowe instrukcje dla GitHub Copilot i innych narzędzi AI dotyczące rozwoju aplikacji IFS Cloud. Copilot powinien stosować się do tych wytycznych przy generowaniu, modyfikowaniu i analizowaniu kodu.

## Architektura IFS Cloud

### Struktura aplikacji
- IFS Cloud jest systemem ERP opartym na architekturze trójwarstwowej
- Warstwa bazodanowa: Oracle Database z PL/SQL
- Warstwa logiki biznesowej: Java/Spring Boot
- Warstwa prezentacji: React/TypeScript z IFS Aurena Framework

### Moduły i komponenty
- Każdy moduł biznesowy ma własną przestrzeń nazw
- Komponenty są organizowane według wzorca MVC
- Używaj Logical Unit (LU) jako podstawowej jednostki logiki biznesowej

## Standardy kodowania PL/SQL

### Konwencje nazewnictwa
```sql
-- Nazwy zmiennych: małe litery + underscore, końcówka _
order_id_        NUMBER;
customer_name_   VARCHAR2(100);
is_valid_        BOOLEAN;

-- Nazwy procedur i funkcji: CamelCase z prefiksem
PROCEDURE Calculate_Order_Total
FUNCTION Get_Customer_Info

-- Nazwy pakietów: UPPER_CASE z suffiksem _API lub _RPI
ORDER_API
CUSTOMER_INFO_RPI

-- Nazwy tabel: UPPER_CASE
CUSTOMER_ORDER
ORDER_LINE
```

### Struktura procedury
```sql
PROCEDURE Process_Customer_Order (
   order_id_     IN NUMBER,
   customer_id_  IN NUMBER,
   status_       OUT VARCHAR2,
   error_msg_    OUT VARCHAR2
)
IS
   -- Zmienne lokalne z końcówką _
   total_amount_    NUMBER;
   order_status_    VARCHAR2(20);
   
   -- Kursory
   CURSOR get_order_lines IS
      SELECT line_no, part_no, qty
      FROM order_line_tab
      WHERE order_id = order_id_;
      
BEGIN
   -- Walidacja parametrów wejściowych
   IF order_id_ IS NULL THEN
      Error_SYS.Record_General(lu_name_, 'ORDERIDNULL: Order ID cannot be null');
   END IF;
   
   -- Logika biznesowa
   FOR line_rec IN get_order_lines LOOP
      -- Przetwarzanie
      NULL;
   END LOOP;
   
   -- Ustawienie wyniku
   status_ := 'SUCCESS';
   
EXCEPTION
   WHEN OTHERS THEN
      error_msg_ := SQLERRM;
      Error_SYS.Record_General(lu_name_, 'PROCESSERR: Error processing order :P1', order_id_);
      RAISE;
END Process_Customer_Order;
```

### Obsługa błędów w PL/SQL
```sql
-- Używaj Error_SYS dla standardowych błędów
Error_SYS.Record_General(lu_name_, 'ERRORCODE: Message :P1 :P2', param1_, param2_);

-- Dla błędów aplikacyjnych
Error_SYS.Appl_General(lu_name_, 'APPERROR: Application error message');

-- Dla błędów rekordów
Error_SYS.Record_Not_Exist(lu_name_, 'NOTEXIST: Record :P1 does not exist', key_);
```

### Zapytania SQL
```sql
-- ZAWSZE używaj bind variables (parametrów) zamiast konkatenacji stringów
-- DOBRZE:
SELECT order_id, customer_id 
FROM customer_order_tab
WHERE order_no = order_no_
  AND customer_id = customer_id_;

-- BAD (SQL injection!):
-- EXECUTE IMMEDIATE 'SELECT * FROM table WHERE id = ' || user_input;

-- Używaj EXISTS zamiast COUNT gdy sprawdzasz istnienie
-- DOBRZE:
IF EXISTS (SELECT 1 FROM customer_order_tab WHERE order_id = order_id_) THEN
   -- ...
END IF;

-- BAD:
-- IF (SELECT COUNT(*) FROM customer_order_tab WHERE order_id = order_id_) > 0 THEN
```

## Standardy kodowania Java/Spring Boot

### Konwencje nazewnictwa
```java
// Klasy: PascalCase
public class CustomerOrderService

// Metody: camelCase
public void processOrder()

// Zmienne: camelCase
private String customerName;

// Stałe: UPPER_SNAKE_CASE
private static final int MAX_RETRY_COUNT = 3;

// Interfejsy: prefiks I (opcjonalnie) lub bez prefiksu
public interface OrderRepository
public interface IOrderService
```

### Struktura serwisu
```java
@Service
@Slf4j
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final CustomerService customerService;
    
    @Autowired
    public OrderService(OrderRepository orderRepository, 
                       CustomerService customerService) {
        this.orderRepository = orderRepository;
        this.customerService = customerService;
    }
    
    @Transactional
    public OrderDTO processOrder(Long orderId) {
        log.debug("Processing order: {}", orderId);
        
        // Walidacja
        validateOrderId(orderId);
        
        // Logika biznesowa
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
            
        // Przetwarzanie
        order.setStatus(OrderStatus.PROCESSED);
        orderRepository.save(order);
        
        log.info("Order {} processed successfully", orderId);
        return OrderMapper.toDTO(order);
    }
    
    private void validateOrderId(Long orderId) {
        if (orderId == null || orderId <= 0) {
            throw new IllegalArgumentException("Invalid order ID: " + orderId);
        }
    }
}
```

### Obsługa wyjątków
```java
// Niestandardowe wyjątki
public class OrderNotFoundException extends RuntimeException {
    public OrderNotFoundException(Long orderId) {
        super("Order not found: " + orderId);
    }
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleOrderNotFound(OrderNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

## Standardy kodowania TypeScript/React

### Konwencje nazewnictwa
```typescript
// Komponenty: PascalCase
export const CustomerOrderList: React.FC = () => {

// Hooki: camelCase z prefiksem use
const useOrderData = () => {

// Interfejsy: PascalCase z prefiksem I (opcjonalnie)
interface IOrderProps {
  orderId: number;
  customerName: string;
}

// Typy: PascalCase
type OrderStatus = 'NEW' | 'PROCESSING' | 'COMPLETED' | 'CANCELLED';

// Zmienne i funkcje: camelCase
const orderTotal = calculateTotal();
```

### Struktura komponentu
```typescript
import React, { useState, useEffect } from 'react';
import { OrderService } from '@/services/OrderService';
import { IOrder } from '@/types/Order';

interface OrderListProps {
  customerId: number;
  onOrderSelect?: (order: IOrder) => void;
}

export const OrderList: React.FC<OrderListProps> = ({ 
  customerId, 
  onOrderSelect 
}) => {
  const [orders, setOrders] = useState<IOrder[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchOrders = async () => {
      try {
        setLoading(true);
        const data = await OrderService.getOrdersByCustomer(customerId);
        setOrders(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    fetchOrders();
  }, [customerId]);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;

  return (
    <div className="order-list">
      {orders.map(order => (
        <OrderItem 
          key={order.id}
          order={order}
          onClick={() => onOrderSelect?.(order)}
        />
      ))}
    </div>
  );
};
```

## API i REST Endpoints

### Struktura URL
```
GET    /api/v1/orders              - Lista zamówień
GET    /api/v1/orders/{id}         - Szczegóły zamówienia
POST   /api/v1/orders              - Utworzenie zamówienia
PUT    /api/v1/orders/{id}         - Aktualizacja zamówienia
DELETE /api/v1/orders/{id}         - Usunięcie zamówienia
PATCH  /api/v1/orders/{id}/status  - Częściowa aktualizacja
```

### Odpowiedzi API
```json
// Sukces (200 OK)
{
  "status": "success",
  "data": {
    "orderId": 12345,
    "customerName": "John Doe"
  }
}

// Błąd (400 Bad Request)
{
  "status": "error",
  "message": "Invalid order ID",
  "code": "INVALID_ORDER_ID",
  "timestamp": "2024-01-15T10:30:00Z"
}

// Lista z paginacją (200 OK)
{
  "status": "success",
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 150,
    "totalPages": 8
  }
}
```

## Bezpieczeństwo

### Walidacja danych wejściowych
```java
// ZAWSZE waliduj dane wejściowe
public void processOrder(OrderDTO orderDTO) {
    // Sprawdź null
    Objects.requireNonNull(orderDTO, "Order cannot be null");
    
    // Waliduj format
    if (!isValidOrderNumber(orderDTO.getOrderNumber())) {
        throw new ValidationException("Invalid order number format");
    }
    
    // Sanityzuj dane
    String sanitizedComment = sanitizeInput(orderDTO.getComment());
}
```

### SQL Injection Prevention
```sql
-- ZAWSZE używaj bind variables
SELECT * FROM customer_order_tab 
WHERE order_no = :order_no  -- DOBRZE
  AND customer_id = :customer_id;

-- NEVER concatenate strings in SQL
-- BAD: 'SELECT * FROM table WHERE id = ' || user_input
```

### Autoryzacja i uprawnienia
```java
// Sprawdź uprawnienia przed operacją
@PreAuthorize("hasPermission(#orderId, 'Order', 'WRITE')")
public void updateOrder(Long orderId, OrderDTO orderDTO) {
    // Logika aktualizacji
}

// Loguj operacje wrażliwe
@PostAuthorize("returnObject != null")
public Order getOrder(Long orderId) {
    auditLog.log("Order accessed", orderId, currentUser);
    return orderRepository.findById(orderId);
}
```

## Testy

### Testy jednostkowe Java
```java
@Test
@DisplayName("Should process order successfully")
void testProcessOrderSuccess() {
    // Given
    Long orderId = 123L;
    Order order = createTestOrder(orderId);
    when(orderRepository.findById(orderId)).thenReturn(Optional.of(order));
    
    // When
    OrderDTO result = orderService.processOrder(orderId);
    
    // Then
    assertNotNull(result);
    assertEquals(OrderStatus.PROCESSED, result.getStatus());
    verify(orderRepository).save(any(Order.class));
}

@Test
@DisplayName("Should throw exception when order not found")
void testProcessOrderNotFound() {
    // Given
    Long orderId = 999L;
    when(orderRepository.findById(orderId)).thenReturn(Optional.empty());
    
    // When & Then
    assertThrows(OrderNotFoundException.class, 
                () -> orderService.processOrder(orderId));
}
```

### Testy jednostkowe TypeScript/Jest
```typescript
describe('OrderService', () => {
  it('should fetch orders successfully', async () => {
    // Arrange
    const mockOrders = [
      { id: 1, customerName: 'John' },
      { id: 2, customerName: 'Jane' }
    ];
    jest.spyOn(api, 'get').mockResolvedValue({ data: mockOrders });
    
    // Act
    const result = await OrderService.getOrders();
    
    // Assert
    expect(result).toEqual(mockOrders);
    expect(api.get).toHaveBeenCalledWith('/api/v1/orders');
  });
});
```

## Performance i Optymalizacja

### Optymalizacja zapytań SQL
```sql
-- Używaj indeksów
CREATE INDEX order_customer_idx ON customer_order_tab(customer_id);

-- Unikaj SELECT *
SELECT order_id, order_no, customer_id  -- Wybieraj tylko potrzebne kolumny
FROM customer_order_tab;

-- Używaj JOIN zamiast subqueries gdy możliwe
SELECT o.order_id, c.customer_name
FROM customer_order_tab o
JOIN customer_info_tab c ON o.customer_id = c.customer_id;
```

### Caching
```java
// Cache wyników
@Cacheable(value = "orders", key = "#orderId")
public Order getOrder(Long orderId) {
    return orderRepository.findById(orderId)
        .orElseThrow(() -> new OrderNotFoundException(orderId));
}

// Invalidacja cache
@CacheEvict(value = "orders", key = "#order.id")
public void updateOrder(Order order) {
    orderRepository.save(order);
}
```

## Dokumentacja kodu

### JavaDoc
```java
/**
 * Processes a customer order and updates its status.
 * 
 * @param orderId the unique identifier of the order to process
 * @return the processed order as a DTO
 * @throws OrderNotFoundException if the order does not exist
 * @throws ValidationException if the order data is invalid
 */
@Transactional
public OrderDTO processOrder(Long orderId) {
    // ...
}
```

### TSDoc
```typescript
/**
 * Fetches all orders for a specific customer.
 * 
 * @param customerId - The unique identifier of the customer
 * @returns A promise that resolves to an array of orders
 * @throws {ApiError} When the API request fails
 * 
 * @example
 * ```typescript
 * const orders = await OrderService.getOrdersByCustomer(123);
 * ```
 */
export async function getOrdersByCustomer(customerId: number): Promise<IOrder[]> {
  // ...
}
```

## Best Practices - Podsumowanie

### DO (Rób):
- ✅ Używaj bind variables w SQL
- ✅ Waliduj wszystkie dane wejściowe
- ✅ Loguj ważne operacje
- ✅ Pisz testy dla logiki biznesowej
- ✅ Obsługuj błędy gracefully
- ✅ Używaj transactions dla operacji wieloetapowych
- ✅ Dokumentuj złożoną logikę
- ✅ Stosuj principle of least privilege
- ✅ Używaj constów dla wartości magicznych
- ✅ Kod powinien być self-documenting

### DON'T (Nie rób):
- ❌ Nie konkatenuj stringów w SQL
- ❌ Nie ignoruj wyjątków
- ❌ Nie przechowuj haseł w plain text
- ❌ Nie używaj SELECT *
- ❌ Nie commituj wrażliwych danych
- ❌ Nie używaj global variables
- ❌ Nie pomiń walidacji danych
- ❌ Nie zapomnij o logowaniu błędów
- ❌ Nie używaj deprecated API
- ❌ Nie duplikuj kodu

## Narzędzia i środowisko

### Wymagane narzędzia
- Oracle SQL Developer dla PL/SQL
- IntelliJ IDEA / VS Code dla Java/TypeScript
- Maven/Gradle dla budowania Java
- npm/yarn dla TypeScript/React
- Git dla kontroli wersji
- SonarQube dla analizy jakości kodu

### Linting i formatowanie
- Checkstyle dla Java
- ESLint + Prettier dla TypeScript
- SQL Developer Formatter dla PL/SQL

## Zasoby i dokumentacja

- IFS Cloud Documentation - sprawdź wewnętrzną dokumentację firmową
- IFS Developer Guide - dostępny w portalu deweloperskim IFS
- [Oracle PL/SQL Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/)
- [Spring Boot Reference](https://spring.io/projects/spring-boot)
- [React Documentation](https://react.dev/)

---

**Uwaga**: Ten dokument powinien być aktualizowany wraz z ewolucją standardów projektu.

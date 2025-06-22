# Sử dụng JUnit 5 để Kiểm thử Ứng dụng Java

## 1. Giới thiệu
JUnit 5 là phiên bản mới nhất của framework kiểm thử đơn vị (unit testing) trong Java.Nó giúp viết các bài kiểm thử dễ đọc, dễ bảo trì và hiệu quả. Tài liệu này hướng dẫn cách sử dụng JUnit 5 từ cơ bản đến nâng cao, bao gồm tất cả các annotation phổ biến và các kỹ thuật kiểm thử thực tế.

## 2. Thiết lập dự án
### 2.1. Yêu cầu
- Java JDK 8 trở lên (khuyến nghị JDK 11+).
- Maven hoặc Gradle để quản lý thư viện.
- IDE (IntelliJ, Eclipse, VS Code, v.v.).

### 2.2. Thêm dependency
Thêm JUnit 5 vào file `pom.xml` (nếu dùng Maven):

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Nếu dùng Gradle, thêm vào `build.gradle`:

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.11.0'
}
```

**Lưu ý**: `junit-jupiter` bao gồm cả `junit-jupiter-api` (API để viết test) và `junit-jupiter-engine` (engine để chạy test).

### 2.3. Cấu hình cơ bản
Nhập các package cần thiết trong class kiểm thử:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
```

### 2.4. Cấu trúc thư mục dự án test
Một dự án kiểm thử JUnit 5 nên có cấu trúc thư mục rõ ràng để tổ chức mã nguồn và tài nguyên kiểm thử. Dưới đây là cấu trúc thư mục tiêu chuẩn cho dự án Java sử dụng Maven hoặc Gradle:

```
project-root/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/app/    # Mã nguồn ứng dụng
│   │   └── resources/              # Tài nguyên ứng dụng (properties, XML, v.v.)
│   └── test/
│       ├── java/
│       │   └── com/example/app/    # Mã nguồn test (test classes)
│       └── resources/              # Tài nguyên test (JSON, CSV, test data, v.v.)
├── pom.xml                         # File cấu hình Maven (hoặc build.gradle cho Gradle)
└── README.md                       # Tài liệu dự án
```

#### 2.4.1. Giải thích cấu trúc
- **src/main/java**: Chứa mã nguồn chính của ứng dụng, ví dụ: các class như `Calculator`, `UserService`.
- **src/main/resources**: Chứa các file tài nguyên như `application.properties` hoặc `log4j2.xml`.
- **src/test/java**: Chứa các class kiểm thử, thường phản ánh cấu trúc package của `src/main/java`. Ví dụ: Nếu `src/main/java/com/example/app/Calculator.java` tồn tại, thì test class sẽ là `src/test/java/com/example/app/CalculatorTest.java`.
- **src/test/resources**: Chứa các file tài nguyên dùng cho kiểm thử, như:
  - File JSON hoặc CSV cho test tham số hóa.
  - File cấu hình riêng cho test (ví dụ: `application-test.properties`).
  - File schema để xác minh dữ liệu.

#### 2.4.2. Ví dụ cấu trúc thực tế
Dự án kiểm thử cho ứng dụng `Calculator`:

```
project-root/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/calc/
│   │   │       └── Calculator.java
│   │   └── resources/
│   │       └── log4j2.xml
│   └── test/
│       ├── java/
│       │   └── com/example/calc/
│       │       ├── CalculatorTest.java
│       │       └── CalculatorIntegrationTest.java
│       └── resources/
│           ├── test-data.csv
│           └── user-schema.json
├── pom.xml
└── README.md
```

#### 2.4.3. Lưu ý khi tổ chức thư mục
- **Phản ánh cấu trúc package**: Giữ cấu trúc package trong `src/test/java` tương tự `src/main/java` để dễ tìm kiếm.
- **Tách biệt test unit và integration**: Đặt test unit (nhanh, không phụ thuộc) và test tích hợp (chậm, phụ thuộc DB, API) vào các package riêng, ví dụ: `com.example.calc.unit` và `com.example.calc.integration`.
- **Quản lý tài nguyên test**: Đặt các file dữ liệu test (CSV, JSON) trong `src/test/resources` và sử dụng `ClassLoader` để đọc chúng.
- **Sử dụng công cụ build**: Cấu hình Maven/Gradle để chỉ chạy test trong `src/test/java` và bỏ qua các thư mục khác.

## 3. Cấu trúc cơ bản của một test case
Một test case JUnit 5 thường bao gồm:
- **Annotation**: Đánh dấu phương thức hoặc class kiểm thử (ví dụ: `@Test`).
- **Assertions**: Kiểm tra kết quả mong đợi (ví dụ: `assertEquals`).
- **Setup/Teardown**: Chuẩn bị và dọn dẹp dữ liệu kiểm thử.

Ví dụ cơ bản:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {

    @Test
    void testAddition() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertEquals(5, result, "2 + 3 should equal 5");
    }
}

class Calculator {
    int add(int a, int b) {
        return a + b;
    }
}
```

## 4. Các Annotation phổ biến trong JUnit 5
JUnit 5 cung cấp nhiều annotation để hỗ trợ viết và quản lý test case. Dưới đây là danh sách các annotation phổ biến, kèm ví dụ chi tiết.

### 4.1. `@Test`
Đánh dấu một phương thức là test case.

```java
@Test
void testMultiplication() {
    Calculator calc = new Calculator();
    assertEquals(6, calc.multiply(2, 3), "2 * 3 should equal 6");
}
```

### 4.2. `@BeforeEach`
Chạy trước mỗi test case để thiết lập trạng thái ban đầu.

```java
private Calculator calc;

@BeforeEach
void setUp() {
    calc = new Calculator();
}

@Test
void testAddition() {
    assertEquals(5, calc.add(2, 3));
}
```

### 4.3. `@AfterEach`
Chạy sau mỗi test case để dọn dẹp.

```java
@AfterEach
void tearDown() {
    calc = null; // Giải phóng tài nguyên
}
```

### 4.4. `@BeforeAll`
Chạy một lần duy nhất trước tất cả test case trong class (phải là phương thức `static`).

```java
private static DatabaseConnection db;

@BeforeAll
static void initAll() {
    db = new DatabaseConnection();
}
```

### 4.5. `@AfterAll`
Chạy một lần duy nhất sau tất cả test case (phải là phương thức `static`).

```java
@AfterAll
static void tearDownAll() {
    db.close();
}
```

### 4.6. `@DisplayName`
Cung cấp tên hiển thị tùy chỉnh cho test case hoặc class.

```java
@Test
@DisplayName("Kiểm tra phép trừ")
void testSubtraction() {
    assertEquals(1, calc.subtract(3, 2), "3 - 2 should equal 1");
}
```

### 4.7. `@Disabled`
Tạm thời vô hiệu hóa test case hoặc class.

```java
@Test
@Disabled("Tạm thời bỏ qua do lỗi đã biết")
void testDivision() {
    assertEquals(2, calc.divide(4, 2));
}
```

### 4.8. `@ParameterizedTest` và `@ValueSource`
Chạy test với nhiều giá trị đầu vào.

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3})
void testIsPositive(int number) {
    assertTrue(calc.isPositive(number), number + " should be positive");
}
```

### 4.9. `@CsvSource` và `@CsvFileSource`
Sử dụng dữ liệu từ chuỗi CSV hoặc file CSV cho test tham số hóa.

```java
@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "0, 0, 0",
    "-1, 1, 0"
})
void testAdditionWithCsv(int a, int b, int expected) {
    assertEquals(expected, calc.add(a, b));
}
```

Sử dụng file CSV (`src/test/resources/test-data.csv`):

```csv
a,b,expected
2,3,5
0,0,0
-1,1,0
```

```java
@ParameterizedTest
@CsvFileSource(resources = "/test-data.csv")
void testAdditionWithCsvFile(int a, int b, int expected) {
    assertEquals(expected, calc.add(a, b));
}
```

### 4.10. `@MethodSource`
Cung cấp dữ liệu từ phương thức tĩnh cho test tham số hóa.

```java
@ParameterizedTest
@MethodSource("provideTestData")
void testAdditionWithMethodSource(int a, int b, int expected) {
    assertEquals(expected, calc.add(a, b));
}

static Stream<Arguments> provideTestData() {
    return Stream.of(
        Arguments.of(2, 3, 5),
        Arguments.of(0, 0, 0),
        Arguments.of(-1, 1, 0)
    );
}
```

### 4.11. `@EnumSource`
Sử dụng các giá trị của enum làm đầu vào.

```java
enum Operation { ADD, SUBTRACT }

@ParameterizedTest
@EnumSource(Operation.class)
void testOperations(Operation op) {
    assertNotNull(op, "Operation should not be null");
}
```

### 4.12. `@RepeatedTest`
Lặp lại test case một số lần nhất định.

```java
@RepeatedTest(value = 3, name = "{displayName} {currentRepetition}/{totalRepetitions}")
@DisplayName("Kiểm tra phép chia lặp lại")
void testDivisionRepeated(RepetitionInfo repetitionInfo) {
    assertEquals(2, calc.divide(4, 2));
}
```

### 4.13. `@TestFactory`
Tạo test case động (dynamic tests).

```java
@TestFactory
Collection<DynamicTest> dynamicTests() {
    return Arrays.asList(
        DynamicTest.dynamicTest("Test cộng", () -> assertEquals(5, calc.add(2, 3))),
        DynamicTest.dynamicTest("Test trừ", () -> assertEquals(1, calc.subtract(3, 2)))
    );
}
```

### 4.14. `@Nested`
Nhóm các test case liên quan trong một class lồng nhau.

```java
@Nested
@DisplayName("Kiểm tra các phép toán số học")
class ArithmeticTests {
    @Test
    void testAddition() {
        assertEquals(5, calc.add(2, 3));
    }

    @Test
    void testSubtraction() {
        assertEquals(1, calc.subtract(3, 2));
    }
}
```

### 4.15. `@Tag`
Gắn thẻ cho test case để chạy có chọn lọc.

```java
@Test
@Tag("slow")
void testSlowOperation() {
    // Test chạy lâu
}
```

Chạy test với thẻ cụ thể (Maven):

```bash
mvn test -Dgroups="slow"
```

### 4.16. `@Timeout`
Đặt thời gian tối đa cho test case.

```java
@Test
@Timeout(value = 1, unit = TimeUnit.SECONDS)
void testWithTimeout() throws InterruptedException {
    Thread.sleep(500); // Đảm bảo dưới 1 giây
    assertTrue(true);
}
```

### 4.17. `@ExtendWith`
Tích hợp extension tùy chỉnh (ví dụ: Mockito, Spring).

```java
@ExtendWith(MockitoExtension.class)
class ServiceTest {
    @Mock
    private Repository repo;

    @Test
    void testService() {
        when(repo.findById(1)).thenReturn("Data");
        assertEquals("Data", repo.findById(1));
    }
}
```

### 4.18. `@TestInstance`
Quy định vòng đời của instance test class (mặc định là `PER_METHOD`, có thể đổi thành `PER_CLASS`).

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class PerClassTest {
    private int counter = 0;

    @Test
    void testIncrement() {
        counter++;
        assertEquals(1, counter);
    }

    @Test
    void testIncrementAgain() {
        counter++;
        assertEquals(2, counter); // Dùng cùng instance
    }
}
```

### 4.19. `@Order`
Annotation `@Order` được sử dụng để chỉ định thứ tự thực thi của các test case hoặc các phương thức setup/teardown trong một class. Mặc định, JUnit 5 không đảm bảo thứ tự thực thi test, nhưng `@Order` cho phép kiểm soát điều này khi cần.

#### 4.19.1. Sử dụng `@Order` cho Test Case
Kết hợp với `@TestMethodOrder` và `MethodOrderer.OrderAnnotation` để sắp xếp các test case:

```java
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrderedTest {

    @Test
    @Order(1)
    void testFirst() {
        assertTrue(true, "This test runs first");
    }

    @Test
    @Order(2)
    void testSecond() {
        assertTrue(true, "This test runs second");
    }

    @Test
    @Order(3)
    void testThird() {
        assertTrue(true, "This test runs third");
    }
}
```

#### 4.19.2. Sử dụng `@Order` cho Setup/Teardown
Kiểm soát thứ tự các phương thức `@BeforeEach`, `@AfterEach`, `@BeforeAll`, hoặc `@AfterAll`:

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class SetupOrderTest {
    @BeforeEach
    @Order(1)
    void setupFirst() {
        System.out.println("First setup");
    }

    @BeforeEach
    @Order(2)
    void setupSecond() {
        System.out.println("Second setup");
    }

    @Test
    void test() {
        assertTrue(true);
    }
}
```


#### 4.19.3. Lưu ý khi sử dụng `@Order`
- **Cần `@TestMethodOrder`**: Annotation `@Order` chỉ hoạt động khi class được đánh dấu với `@TestMethodOrder`.
- **Giá trị số**: Giá trị của `@Order` là số nguyên, số nhỏ hơn được thực thi trước (ví dụ: `@Order(1)` chạy trước `@Order(2)`).
- **Không khuyến khích phụ thuộc thứ tự**: Thiết kế test nên độc lập với nhau để đảm bảo tính mạnh mẽ. Chỉ sử dụng `@Order` khi thực sự cần (ví dụ: kiểm thử trạng thái tuần tự hoặc tài nguyên chia sẻ).
- **Hiệu suất**: Sắp xếp thứ tự có thể làm tăng thời gian thiết lập nếu các test phụ thuộc vào nhau.

## 5. Assertions trong JUnit 5
JUnit 5 cung cấp nhiều phương thức assertion để kiểm tra kết quả. Các assertion nên đi kèm thông báo thất bại (failure message) để cung cấp thông tin chi tiết khi test thất bại, giúp debug dễ dàng hơn.

### 5.1. Assertions cơ bản
Các phương thức assertion cơ bản bao gồm:
- `assertEquals(expected, actual)`: Kiểm tra hai giá trị bằng nhau.
- `assertNotEquals(unexpected, actual)`: Kiểm tra hai giá trị khác nhau.
- `assertTrue(condition)`: Kiểm tra điều kiện đúng.
- `assertFalse(condition)`: Kiểm tra điều kiện sai.
- `assertNull(object)`: Kiểm tra đối tượng là null.
- `assertNotNull(object)`: Kiểm tra đối tượng không null.

#### 5.1.1. Sử dụng thông báo thất bại tĩnh
Thêm thông điệp tĩnh vào assertion để mô tả lỗi khi test thất bại:

```java
@Test
void testBasicAssertions() {
    Calculator calc = new Calculator();
    int result = calc.add(2, 3);
    assertEquals(5, result, "2 + 3 should equal 5");
    assertTrue(calc.isPositive(1), "Number 1 should be positive");
    assertNull(calc.getNullValue(), "Value should be null");
}
```

#### 5.1.2. Sử dụng thông báo thất bại động
Tạo thông điệp động dựa trên dữ liệu test:

```java
@Test
void testDynamicFailureMessage() {
    Calculator calc = new Calculator();
    int a = 2, b = 4;
    int result = calc.add(a, b);
    assertEquals(6, result, String.format("%d + %d should equal 6", a, b));
}
```

#### 5.1.3. Sử dụng Supplier<String> cho thông báo thất bại
Sử dụng `Supplier<String>` để trì hoãn việc tạo thông điệp, tối ưu hiệu suất khi thông điệp phức tạp:

```java
@Test
void testSupplierFailureMessage() {
    Calculator calc = new Calculator();
    int result = calc.add(2, 3);
    assertEquals(5, result, () -> "Complex calculation for 2 + 3 failed, expected 5 but got " + result);
}
```

#### 5.1.4. Lưu ý khi sử dụng thông báo thất bại
- **Thông điệp rõ ràng**: Viết thông điệp mô tả chính xác kỳ vọng và lỗi (ví dụ: "Expected sum of 2 and 3 to be 5, but got 6").
- **Tối ưu hiệu suất**: Sử dụng `Supplier<String>` cho các thông điệp yêu cầu tính toán nặng (ví dụ: xử lý chuỗi lớn, truy vấn cơ sở dữ liệu).
- **Định dạng nhất quán**: Sử dụng định dạng thông điệp thống nhất trong dự án để dễ đọc báo cáo test.

#### 5.1.5. Tips viết thông báo thất bại hiệu quả
Để viết thông báo thất bại hiệu quả, hãy tuân theo các mẹo sau:

1. **Mô tả rõ ràng kỳ vọng và thực tế**:
   - Bao gồm giá trị mong đợi (expected) và giá trị thực tế (actual) trong thông điệp.
   - Ví dụ: `assertEquals(5, result, "Expected sum to be 5, but got " + result)`.

2. **Liên kết với ngữ cảnh test**:
   - Đề cập đến hành vi hoặc chức năng đang kiểm tra.
   - Ví dụ: `assertTrue(user.isActive(), "User with ID " + user.getId() + " should be active")`.

3. **Sử dụng định dạng dễ đọc**:
   - Sử dụng `String.format` hoặc Text Blocks (Java 15+) cho thông điệp phức tạp.
   - Ví dụ:
     ```java
     @Test
     void testFormattedMessage() {
         int a = 2, b = 3;
         int result = calc.add(a, b);
         String message = """
             Sum of %d and %d should be 5, but got %d
             """.formatted(a, b, result);
         assertEquals(5, result, message);
     }
     ```

4. **Tối ưu hiệu suất với Supplier**:
   - Sử dụng `Supplier<String>` cho thông điệp yêu cầu tính toán nặng để chỉ tạo chuỗi khi test thất bại.
   - Ví dụ:
     ```java
     @Test
     void testHeavyCalculationMessage() {
         int result = calc.complexOperation(100);
         assertEquals(1000, result, () -> "Complex operation failed after processing 100 items, expected 1000 but got " + result);
     }
     ```

5. **Giữ thông điệp ngắn gọn nhưng đủ ý**:
   - Tránh thông điệp quá dài, chỉ tập trung vào thông tin cần thiết.
   - Ví dụ: Thay vì `"The test for addition of two numbers in the Calculator class failed because the result was incorrect"`, dùng `"2 + 3 should equal 5, but got " + result`.

6. **Sử dụng thông điệp nhất quán trong dự án**:
   - Định nghĩa một mẫu thông điệp chung (ví dụ: `[Test Name]: Expected [value], but got [value]`) để dễ đọc báo cáo.
   - Ví dụ: `"AdditionTest: Expected 5, but got " + result`.

7. **Kết hợp với logging**:
   - Sử dụng `TestReporter` để ghi log bổ sung, hỗ trợ debug mà không làm lộn xộn thông điệp assertion.
   - Ví dụ:
     ```java
     @Test
     void testWithReporter(TestReporter testReporter) {
         int result = calc.add(2, 3);
         testReporter.publishEntry("Input values", "2 and 3");
         assertEquals(5, result, "2 + 3 should equal 5");
     }
     ```

8. **Kiểm tra thông điệp trong CI/CD**:
   - Đảm bảo thông điệp xuất hiện rõ ràng trong báo cáo test (JUnit, Maven Surefire, Gradle) để hỗ trợ nhóm phát triển.

### 5.2. Assertions nâng cao
Các assertion nâng cao hỗ trợ kiểm tra các trường hợp phức tạp hơn.

#### 5.2.1. assertThrows
Kiểm tra ngoại lệ được ném, với thông báo thất bại:

```java
@Test
void testDivisionByZero() {
    Calculator calc = new Calculator();
    assertThrows(ArithmeticException.class, () -> calc.divide(1, 0), "Division by zero should throw ArithmeticException");
}
```

#### 5.2.2. assertTimeout
Kiểm tra thời gian thực thi, với thông báo thất bại:

```java
@Test
void testTimeout() {
    assertTimeout(Duration.ofSeconds(1), () -> {
        Thread.sleep(500);
    }, "Operation should complete within 1 second");
}
```

#### 5.2.3. assertAll
Nhóm nhiều assertion, báo lỗi tất cả nếu có, với thông báo thất bại riêng cho mỗi assertion:

```java
@Test
void testMultipleAssertions() {
    User user = new User(1, "John", 30, "john@example.com");
    assertAll("user properties",
        () -> assertEquals("John", user.getName(), "User name should be John"),
        () -> assertEquals(30, user.getAge(), "User age should be 30"),
        () -> assertEquals("john@example.com", user.getEmail(), "User email should be john@example.com")
    );
}
```

#### 5.2.4. Sử dụng Supplier<String> trong Assertions nâng cao
Tương tự assertions cơ bản, các assertion nâng cao cũng hỗ trợ `Supplier<String>`:

```java
@Test
void testThrowsWithSupplier() {
    Calculator calc = new Calculator();
    int divisor = 0;
    assertThrows(ArithmeticException.class, () -> calc.divide(1, divisor),
        () -> String.format("Division by %d should throw ArithmeticException", divisor));
}
```

## 6. Kiểm thử nâng cao
### 6.1. Kiểm thử với Dependency Injection
JUnit 5 hỗ trợ tiêm tham số vào test method.

```java
@Test
void testWithParameters(TestInfo testInfo, TestReporter testReporter) {
    testReporter.publishEntry("Running " + testInfo.getDisplayName());
    assertTrue(true, "Test should pass");
}
```

### 6.2. Kiểm thử với Temporary Files
Sử dụng `TempDir` để tạo thư mục tạm.

```java
@Test
void testWithTempDir(@TempDir Path tempDir) throws IOException {
    Path file = tempDir.resolve("test.txt");
    Files.writeString(file, "Hello");
    assertEquals("Hello", Files.readString(file), "File content should be 'Hello'");
}
```

### 6.3. Tích hợp với Mockito
Kết hợp JUnit 5 và Mockito để mock đối tượng.

```java
@ExtendWith(MockitoExtension.class)
class ServiceTest {
    @Mock
    private Repository repo;
    @InjectMocks
    private Service service;

    @Test
    void testService() {
        when(repo.findById(1)).thenReturn("Data");
        assertEquals("Data", service.getData(1), "Service should return mocked data");
    }
}
```

### 6.4. Tích hợp với Spring Boot
Kiểm thử Spring Boot ứng dụng:

```java
@SpringBootTest
@ExtendWith(SpringExtension.class)
class ApplicationTest {
    @Autowired
    private UserService userService;

    @Test
    void testUserService() {
        assertNotNull(userService.findById(1), "User service should return non-null user");
    }
}
```

### 6.5. Logging trong test
Logging là một kỹ thuật quan trọng trong kiểm thử để ghi lại thông tin về quá trình thực thi test, hỗ trợ debug và phân tích lỗi. JUnit 5 cung cấp các cách tích hợp logging, từ sử dụng thư viện logging (như SLF4J, Log4j2) đến các công cụ tích hợp như `TestReporter`.

#### 6.5.1. Sử dụng SLF4J với Logback
Thêm dependency SLF4J và Logback vào `pom.xml`:

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.16</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.8</version>
    <scope>test</scope>
</dependency>
```

Cấu hình Logback trong `src/test/resources/logback-test.xml`:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="debug">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

Sử dụng logging trong test:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

class CalculatorTest {
    private static final Logger logger = LoggerFactory.getLogger(CalculatorTest.class);
    private Calculator calc;

    @BeforeEach
    void setUp() {
        calc = new Calculator();
        logger.info("Initializing Calculator for test");
    }

    @Test
    void testAddition() {
        logger.debug("Testing addition with inputs 2 and 3");
        int result = calc.add(2, 3);
        assertEquals(5, result, "2 + 3 should equal 5");
        logger.info("Test passed with result: {}", result);
    }
}
```

#### 6.5.2. Sử dụng TestReporter
JUnit 5 cung cấp `TestReporter` để ghi log thông tin test mà không cần thư viện logging bên ngoài:

```java
@Test
void testWithTestReporter(TestReporter testReporter) {
    Calculator calc = new Calculator();
    int a = 2, b = 3;
    testReporter.publishEntry("Test Inputs", String.format("a=%d, b=%d", a, b));
    int result = calc.add(a, b);
    assertEquals(5, result, "2 + 3 should equal 5");
    testReporter.publishEntry("Test Result", String.valueOf(result));
}
```

#### 6.5.3. Logging khi test thất bại
Ghi log chi tiết khi test thất bại bằng cách kết hợp với `assertThrows`:

```java
@Test
void testDivisionByZero(TestReporter testReporter) {
    Calculator calc = new Calculator();
    testReporter.publishEntry("Testing division by zero");
    assertThrows(ArithmeticException.class, () -> {
        try {
            calc.divide(1, 0);
        } catch (ArithmeticException e) {
            testReporter.publishEntry("Exception Caught", e.getMessage());
            throw e;
        }
    }, "Division by zero should throw ArithmeticException");
}
```

#### 6.5.4. Lưu ý khi sử dụng logging
- **Chọn mức độ log phù hợp**:
  - Sử dụng `DEBUG` cho thông tin chi tiết (ví dụ: giá trị đầu vào).
  - Sử dụng `INFO` cho thông tin chính (ví dụ: test bắt đầu/kết thúc).
  - Sử dụng `ERROR` cho lỗi nghiêm trọng.
- **Tối ưu hiệu suất**:
  - Tránh ghi log quá nhiều trong các test chạy thường xuyên để giảm thời gian thực thi.
  - Sử dụng tham số hóa trong log (ví dụ: `logger.info("Result: {}", result)` thay vì `logger.info("Result: " + result)`).
- **Tích hợp với CI/CD**:
  - Đảm bảo log xuất hiện trong báo cáo build (Maven Surefire, Gradle) bằng cách cấu hình appender phù hợp.
- **Tách biệt log test và ứng dụng**:
  - Sử dụng file cấu hình riêng cho test (ví dụ: `logback-test.xml`) để tránh xung đột với ứng dụng.
- **Sử dụng TestReporter khi cần đơn giản**:
  - `TestReporter` phù hợp cho các dự án không muốn thêm thư viện logging, nhưng ít linh hoạt hơn SLF4J.

## 7. Best Practices
- **Tách biệt test**: Mỗi test case kiểm tra một chức năng duy nhất.
- **Sử dụng `@Nested`**: Nhóm các test liên quan để dễ đọc.
- **Tái sử dụng code**: Sử dụng `@BeforeEach`, `@BeforeAll` để tránh lặp code.
- **Kiểm tra ngoại lệ**: Sử dụng `assertThrows` thay vì try-catch.
- **Chạy test có chọn lọc**: Sử dụng `@Tag` để phân loại test (unit, integration, slow, v.v.).
- **Báo cáo lỗi chi tiết**: Luôn thêm thông báo thất bại vào assertion để hỗ trợ debug, tuân theo các mẹo viết thông báo hiệu quả.
- **Tối ưu thông báo thất bại**: Sử dụng `Supplier<String>` cho các thông điệp phức tạp để tránh tạo chuỗi không cần thiết khi test thành công.
- **Kiểm soát thứ tự test cẩn thận**: Chỉ sử dụng `@Order` khi cần thiết và đảm bảo các test không phụ thuộc lẫn nhau.

### 7.1. Cách đặt tên test method
Đặt tên test method rõ ràng và mô tả đúng hành vi được kiểm tra là yếu tố quan trọng để test dễ hiểu và bảo trì. Dưới đây là các hướng dẫn và ví dụ:

#### 7.1.1. Quy tắc đặt tên
- **Mô tả hành vi**: Tên method nên nêu rõ chức năng hoặc kịch bản được kiểm tra, thường theo cấu trúc `[Hành vi]_[Kịch bản]_[Kết quả mong đợi]`.
- **Sử dụng camelCase**: Theo chuẩn Java, tên method sử dụng camelCase, tránh dấu gạch dưới.
- **Tránh tên chung chung**: Không sử dụng tên như `test1`, `testMethod`, hoặc `runTest`.
- **Kết hợp với `@DisplayName`**: Nếu tên method quá dài, sử dụng `@DisplayName` để cung cấp mô tả chi tiết hơn.

#### 7.1.2. Các mẫu đặt tên phổ biến
1. **should[Action]When[Condition]**:
   - Ví dụ: `shouldReturnSumWhenAddingTwoNumbers`
   - Mô tả: Kiểm tra hành vi (return sum) trong điều kiện cụ thể (adding two numbers).

2. **given[Condition]When[Action]Then[Result]**:
   - Ví dụ: `givenValidInputWhenAddingThenReturnsCorrectSum`
   - Mô tả: Phản ánh mô hình Given-When-Then của BDD (Behavior-Driven Development).

3. **test[Feature]With[Condition]**:
   - Ví dụ: `testDivisionWithZeroDenominatorThrowsException`
   - Mô tả: Kiểm tra tính năng (division) với điều kiện cụ thể (zero denominator).

#### 7.1.3. Ví dụ thực tế
```java
class CalculatorTest {
    private Calculator calc;

    @BeforeEach
    void setUp() {
        calc = new Calculator();
    }

    @Test
    @DisplayName("Should return correct sum when adding two positive numbers")
    void shouldReturnSumWhenAddingTwoPositiveNumbers() {
        assertEquals(5, calc.add(2, 3), "2 + 3 should equal 5");
    }

    @Test
    void givenZeroDenominatorWhenDividingThenThrowsException() {
        assertThrows(ArithmeticException.class, () -> calc.divide(1, 0),
            "Division by zero should throw ArithmeticException");
    }

    @Test
    void testMultiplicationWithNegativeNumbers() {
        assertEquals(-6, calc.multiply(-2, 3), "-2 * 3 should equal -6");
    }
}
```

#### 7.1.4. Lưu ý khi đặt tên
- **Độ dài hợp lý**: Tên method nên dài đủ để mô tả nhưng không quá rườm rà. Nếu quá dài, sử dụng `@DisplayName`.
- **Nhất quán trong dự án**: Thống nhất một mẫu đặt tên (ví dụ: `shouldXWhenY`) để dễ đọc và tìm kiếm.
- **Phản ánh kịch bản thực tế**: Đặt tên dựa trên yêu cầu nghiệp vụ hoặc kịch bản người dùng, ví dụ: `shouldRejectInvalidUserWhenRegistering`.
- **Tránh lặp lại tên class**: Không cần lặp lại tên class trong tên method (ví dụ: `CalculatorTest.testCalculatorAdd` là thừa, chỉ cần `testAdd`).
- **Kiểm tra báo cáo test**: Đảm bảo tên method xuất hiện rõ ràng trong báo cáo CI/CD (Maven Surefire, Gradle).


# Sử dụng Rest Assured để Kiểm thử API RESTful

## 1. Giới thiệu

Rest Assured là một thư viện Java để kiểm thử API RESTful. Nó cung cấp cú pháp đơn giản, dễ đọc để gửi các HTTP request và xác minh response, cho phép tester viết các kịch bản kiểm thử tự động để xác minh phản hồi của API, như trạng thái HTTP, nội dung JSON/XML, hay header. Tài liệu này hướng dẫn cách sử dụng Rest Assured để kiểm thử API, bao gồm các thiết lập quan trọng, tạo body request JSON, gửi request và kiểm tra kết quả.

## 2. Thiết lập dự án

### 2.1. Yêu cầu

- Java JDK 8 trở lên (JDK 15+ để sử dụng Text Blocks).
- Maven hoặc Gradle để quản lý thư viện.
- IDE (IntelliJ, Eclipse, v.v.).

### 2.2. Thêm dependency

Thêm Rest Assured vào file `pom.xml` (nếu dùng Maven):

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.5.0</version>
    <scope>test</scope>
</dependency>
```

### 2.3. Cấu hình cơ bản

Import các package cần thiết trong class test:

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import static io.restassured.RestAssured.given;
```

## 3. Các thiết lập quan trọng

Để đảm bảo quá trình kiểm thử API hiệu quả, Rest Assured cung cấp các thiết lập cơ bản như baseURI, header chung. Các thiết lập này giúp giảm lặp code, tái sử dụng code tốt hơn.

### 3.1. Thiết lập BaseURI, Port, và BasePath

Cấu hình `baseUri`, `port`, và `basePath` để xác định URL cơ sở cho tất cả các request. 'basePath' là phần cố định ở đầu của đường dẫn (path) trong URL mà mọi request sẽ sử dụng mặc định.

#### 3.1.1. Thiết lập toàn cục

Cấu hình mặc định cho toàn bộ ứng dụng:

```java
@BeforeAll
public static void setup() {
    RestAssured.baseURI = "https://api.example.com";
    RestAssured.port = 443; // Cổng mặc định cho HTTPS
    RestAssured.basePath = "/v1";
}
// Cách viết ngắn gọn hơn
@BeforeAll
public static void setup() {
    RestAssured.baseURI = "https://api.example.com:443/v1";
}
```

#### 3.1.2. Sử dụng cấu hình toàn cục trong test case

Khi cấu hình toàn cục đã được thiết lập, các test case sẽ tự động sử dụng nó mà không cần khai báo lại. Request URL cuối cùng sẽ là `https://api.example.com:443/v1/users/1`.

```java
@Test
public void testGetUserWithBaseConfig() {
    given()
        // Không cần chỉ định baseURI ở đây
    .when()
        .get("/users/1") // Chỉ cần cung cấp endpoint tương đối
    .then()
        .statusCode(200)
        .body("name", equalTo("John Doe"));
}
```

#### 3.1.3. Ghi đè cấu hình trong một test case cụ thể

Trong trường hợp một test case cần gọi đến một API endpoint khác với cấu hình toàn cục (ví dụ: một dịch vụ khác hoặc một phiên bản API cũ hơn), có thể ghi đè baseURI bằng phương thức `.baseUri()` trong request.
Thiết lập này chỉ có hiệu lực cho request đó và không ảnh hưởng đến các test case khác.

```java
@Test
public void testEndpointFromAnotherService() {
    given()
        // Ghi đè baseURI toàn cục cho request này
        .baseUri("https://api.another-service.com")
        // Port và basePath cũng có thể được ghi đè nếu cần
        .port(8080)
        .basePath("/api/v2")
    .when()
        .get("/health-check")
    .then()
        .statusCode(200);
}
```

#### 3.1.4. Lưu ý

- `baseURI`: Địa chỉ cơ sở của API (ví dụ: `https://api.example.com`).
- `port`: Cổng của server (mặc định: 80 cho HTTP, 443 cho HTTPS).
- `basePath`: Đường dẫn chung cho các endpoint (ví dụ:orbet `/v1` hoặc `/api`).
- Có thể ghi đè trong từng test case bằng `.baseUri()`, `.port()`, hoặc `.basePath()`.

### 3.2. Thiết lập Header Chung

Để áp dụng header chung cho tất cả các request, sử dụng `RequestSpecification` hoặc cấu hình toàn cục.

#### 3.2.1. Thiết lập header chung toàn cục

```java
@BeforeAll
public static void setup() {
    RestAssured.requestSpecification = new RequestSpecBuilder()
        .setBaseUri("https://api.example.com")
        .setPort(443)
        .setBasePath("/v1")
        .addHeader("Authorization", "Bearer " + getJwtToken())
        .addHeader("Content-Type", "application/json")
        .addHeader("Accept", "application/json")
        .build();
}
```

#### 3.2.2. Sử dụng nhiều header trong một lần

```java
@Test
public void testMultipleHeaders() {
    given()
        .headers(
            "Authorization", "Bearer " + getJwtToken(),
            "Content-Type", "application/json",
            "Custom-Header", "custom-value"
        )
        .when()
        .get("/users/1")
        .then()
        .statusCode(200);
}
```

#### 3.2.3. Lưu ý

- **Ưu điểm**: Giảm lặp code, đảm bảo tính nhất quán cho các header như `Authorization` hoặc `Content-Type`.
- **Nhược điểm**: Header chung áp dụng cho tất cả request, cần cẩn thận khi một số endpoint yêu cầu header khác.
- **Ghi đè**: Có thể ghi đè header trong từng test case bằng `.header()` hoặc `.headers()`.

### 3.3. Thiết lập Query Parameters

Query parameters được sử dụng để truyền tham số qua URL (ví dụ: `/users?role=admin&status=active`).

#### 3.3.1. Thêm từng query parameter

```java
@Test
public void testQueryParams() {
    given()
        .baseUri("https://api.example.com")
        .queryParam("role", "admin")
        .queryParam("status", "active")
        .when()
        .get("/users")
        .then()
        .statusCode(200);
}
```

#### 3.3.2. Thêm nhiều query parameters

Sử dụng `queryParams` để thêm nhiều tham số cùng lúc:

```java
@Test
public void testMultipleQueryParams() {
    given()
        .baseUri("https://api.example.com")
        .queryParams(
            "role", "admin",
            "status", "active",
            "page", "1"
        )
        .when()
        .get("/users")
        .then()
        .statusCode(200);
}
```

#### 3.3.3. Sử dụng Map cho query parameters

```java
@Test
public void testQueryParamsWithMap() {
    Map<String, String> queryParams = new HashMap<>();
    queryParams.put("role", "admin");
    queryParams.put("status", "active");

    given()
        .baseUri("https://api.example.com")
        .queryParams(queryParams)
        .when()
        .get("/users")
        .then()
        .statusCode(200);
}
```

#### 3.3.4. Lưu ý

- Query parameters được tự động mã hóa (URL-encoded) bởi Rest Assured.
- Đảm bảo các giá trị query params hợp lệ để tránh lỗi 400 (Bad Request).

### 3.4. Thiết lập Path Parameters

Path parameters được sử dụng để thay thế các phần động trong URL (ví dụ: `/users/{id}`).

#### 3.4.1. Thêm path parameter

```java
@Test
public void testPathParam() {
    given()
        .baseUri("https://api.example.com")
        .pathParam("id", 1)
        .when()
        .get("/users/{id}")
        .then()
        .statusCode(200)
        .body("id", equalTo(1));
}
```

#### 3.4.2. Thêm nhiều path parameters

```java
@Test
public void testMultiplePathParams() {
    given()
        .baseUri("https://api.example.com")
        .pathParam("userId", 1)
        .pathParam("orderId", 100)
        .when()
        .get("/users/{userId}/orders/{orderId}")
        .then()
        .statusCode(200);
}
```

#### 3.4.3. Sử dụng Map cho path parameters

```java
@Test
public void testPathParamsWithMap() {
    Map<String, Object> pathParams = new HashMap<>();
    pathParams.put("userId", 1);
    pathParams.put("orderId", 100);

    given()
        .baseUri("https://api.example.com")
        .pathParams(pathParams)
        .when()
        .get("/users/{userId}/orders/{orderId}")
        .then()
        .statusCode(200);
}
```

#### 3ерше

System: 3.4.4. Lưu ý

- Path parameters phải khớp với các placeholder trong URL (ví dụ: `{id}`).
- Sử dụng `pathParams` để quản lý nhiều tham số, đặc biệt trong các endpoint phức tạp.
- Đảm bảo giá trị path params không chứa ký tự không hợp lệ (như `/`).

### 3.5. Thiết lập Timeout

Cấu hình thời gian chờ cho request:

```java
@Test
public void testWithTimeout() {
    given()
        .baseUri("https://api.example.com")
        .config(RestAssured.config()
            .httpClient(httpClientConfig()
                .setParam("connectionTimeout", 5000)
                .setParam("socketTimeout", 10000)))
        .when()
        .get("/users")
        .then()
        .statusCode(200);
}
```

### 3.6. Bật Logging toàn diện

Ghi log request và response để debug:

```java
@Test
public void testWithLogging() {
    given()
        .baseUri("https://api.example.com")
        .log().all()
        .when()
        .get("/users")
        .then()
        .log().all()
        .statusCode(200);
}
```

### 3.7. Lưu ý

- **Tái sử dụng cấu hình**: Sử dụng `RequestSpecification` để lưu các thiết lập như timeout.
- **Môi trường test**: Đảm bảo các thiết lập phù hợp với môi trường kiểm thử.
- **Logging có chọn lọc**: Chỉ bật log khi cần để tránh làm chậm test hoặc lộ thông tin nhạy cảm.

## 4. Tạo Body Request JSON với Text Blocks

Text Blocks (tính năng từ Java 15) giúp viết chuỗi JSON dễ đọc và gọn gàng hơn so với chuỗi thông thường. Đây là cách tạo body request JSON bằng Text Blocks trong Rest Assured.

### 4.1. Cú pháp cơ bản

Sử dụng `"""` để định nghĩa Text Block. Ví dụ:

```java
String requestBody = """
    {
        "name": "Jane Doe",
        "email": "jane@example.com",
        "age": 30
    }
    """;
```

### 4.2. Sử dụng trong Rest Assured

Áp dụng Text Block vào request POST:

```java
@Test
public void testCreateUserWithTextBlock() {
    String requestBody = """
        {
            "name": "Jane Doe",
            "email": "jane@example.com",
            "age": 30
        }
        """;

    given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .body(requestBody)
        .when()
        .post("/users")
        .then()
        .statusCode(201)
        .body("email", equalTo("jane@example.com"));
}
```

### 4.3. Lợi ích của Text Blocks

- **Dễ đọc**: JSON được định dạng tự nhiên, không cần thêm ký tự thoát (`\`) hoặc nối chuỗi.
- **Bảo trì dễ dàng**: Dễ chỉnh sửa cấu trúc JSON mà không lo lỗi cú pháp.
- **Tích hợp biến**: Có thể chèn biến vào Text Block bằng cách sử dụng `String.format` hoặc concatenation.

### 4.4. Chèn biến vào Text Block

Để thêm giá trị động vào JSON:

```java
@Test
public void testCreateUserWithDynamicTextBlock() {
    String name = "John Doe";
    int age = 25;

    String requestBody = """
        {
            "name": "%s",
            "email": "john@example.com",
            "age": %d
        }
        """.formatted(name, age);

    given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .body(requestBody)
        .when()
        .post("/users")
        .then()
        .statusCode(201);
}
```

### 4.5. Lưu ý

- **Yêu cầu Java phiên bản**: Text Blocks chỉ hỗ trợ từ Java 15 trở lên. Nếu dùng phiên bản thấp hơn, bạn cần sử dụng chuỗi thông thường hoặc các thư viện như Jackson/Gson để tạo JSON.
- **Định dạng**: Đảm bảo JSON trong Text Block đúng cú pháp để tránh lỗi khi gửi request.

## 5. Cấu trúc cơ bản của một test case

Rest Assured sử dụng mô hình **Given-When-Then**:

- **Given**: Thiết lập request (header, body, query param, v.v.).
- **When**: Gửi request (GET, POST, PUT, v.v.).
- **Then**: Xác minh response (status code, body, header, v.v.).

Ví dụ cơ bản:

```java
@Test
public void testGetUser() {
    given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .body("name", equalTo("John Doe"));
}
```

## 6. Các phương thức HTTP phổ biến

### 6.1. GET: Lấy dữ liệu

Kiểm thử API lấy thông tin người dùng:

```java
@Test
public void testGetUser() {
    Response response = given()
        .baseUri("https://api.example.com")
        .queryParam("id", 1)
        .when()
        .get("/users")
        .then()
        .statusCode(200)
        .extract().response();

    // Kiểm tra response body
    assertEquals("John Doe", response.jsonPath().getString("name"));
}
```

### 6.2. POST: Tạo tài nguyên

Kiểm thử API tạo người dùng mới:

```java
@Test
public void testCreateUser() {
    String requestBody = """
        {
            "name": "Jane Doe",
            "email": "jane@example.com"
        }
        """;

    given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .body(requestBody)
        .when()
        .post("/users")
        .then()
        .statusCode(201)
        .body("email", equalTo("jane@example.com"));
}
```

### 6.3. PUT: Cập nhật tài nguyên

Kiểm thử API cập nhật thông tin người dùng:

```java
@Test
public void testUpdateUser() {
    String requestBody = """
        {
            "name": "Jane Smith"
        }
        """;

    given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .body(requestBody)
        .when()
        .put("/users/1")
        .then()
        .statusCode(200)
        .body("name", equalTo("Jane Smith"));
}
```

### 6.4. DELETE: Xóa tài nguyên

Kiểm thử API xóa người dùng:

```java
@Test
public void testDeleteUser() {
    given()
        .baseUri("https://api.example.com")
        .when()
        .delete("/users/1")
        .then()
        .statusCode(204);
}
```

## 7. Kiểm tra Response

### 7.1. Kiểm tra Status Code

```java
.then().statusCode(200); // Kiểm tra mã trạng thái 200 OK
```

### 7.2. Kiểm tra Response Body

Sử dụng `jsonPath` để truy xuất dữ liệu trong JSON response:

```java
.then()
    .body("data.id", equalTo(1))
    .body("data.name", equalTo("John Doe"));
```

### 7.3. Kiểm tra Header

```java
.then()
    .header("Content-Type", equalTo("application/json; charset=utf-8"));
```

### 7.4. Kiểm tra Schema

Xác minh cấu trúc JSON response bằng JSON Schema:

```java
@Test
public void testSchemaValidation() {
    given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .body(matchesJsonSchemaInClasspath("user-schema.json"));
}
```

File `user-schema.json` (đặt trong `src/test/resources`):

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": { "type": "integer" },
    "name": { "type": "string" },
    "email": { "type": "string", "format": "email" }
  },
  "required": ["id", "name", "email"]
}
```

## 8. Xác thực (Authentication)

### 8.1. Basic Authentication

```java
given()
    .auth().basic("username", "password")
    .when()
    .get("/secured-endpoint")
    .then()
    .statusCode(200);
```

### 8.2. OAuth 2.0

```java
given()
    .auth().oauth2("access_token")
    .when()
    .get("/secured-endpoint")
    .then()
    .statusCode(200);
```

### 8.3. JWT Authentication

JSON Web Token (JWT) thường được sử dụng để xác thực API. JWT được gửi trong header `Authorization` với định dạng `Bearer <token>`.

#### 8.3.1. Gửi request với JWT

```java
@Test
public void testApiWithJwt() {
    String jwtToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c";

    given()
        .baseUri("https://api.example.com")
        .header("Authorization", "Bearer " + jwtToken)
        .when()
        .get("/protected-endpoint")
        .then()
        .statusCode(200)
        .body("message", equalTo("Access granted"));
}
```

#### 8.3.2. Lấy JWT từ API đăng nhập

Thông thường, JWT được lấy từ endpoint đăng nhập trước khi sử dụng:

```java
@Test
public void testLoginAndUseJwt() {
    String loginBody = """
        {
            "username": "user",
            "password": "pass"
        }
        """;

    // Gửi request đăng nhập để lấy JWT
    Response loginResponse = given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .body(loginBody)
        .when()
        .post("/login")
        .then()
        .statusCode(200)
        .extract().response();

    // Trích xuất JWT từ response
    String jwtToken = loginResponse.jsonPath().getString("token");

    // Sử dụng JWT cho request tiếp theo
    given()
        .baseUri("https://api.example.com")
        .header("Authorization", "Bearer " + jwtToken)
        .when()
        .get("/protected-endpoint")
        .then()
        .statusCode(200)
        .body("message", equalTo("Access granted"));
}
```

#### 8.3.3. Lưu ý khi sử dụng JWT

- **Lưu trữ JWT an toàn**: Trong kiểm thử, đảm bảo JWT không bị lộ trong log hoặc mã nguồn.
- **Xử lý token hết hạn**: Thêm logic làm mới token nếu API trả về lỗi 401 (Unauthorized).
- **Kiểm tra token hợp lệ**: Xác minh cấu trúc và chữ ký của JWT nếu cần (sử dụng thư viện như `jjwt`).

## 9. Tùy chỉnh Request

### 9.1. Thêm Query Parameters

```java
given()
    .queryParam("key", "value")
    .when()
    .get("/endpoint");
```

### 9.2. Thêm Headers

```java
given()
    .header("Authorization", "Bearer token")
    .header("Custom-Header", "value")
    .when()
    .get("/endpoint");
```

### 9.3. Sử dụng Request Specification

Tái sử dụng cấu hình request:

```java
RequestSpecification requestSpec = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .addHeader("Content-Type", "application/json")
    .build();

given()
    .spec(requestSpec)
    .body(requestBody)
    .when()
    .post("/users")
    .then()
    .statusCode(201);
```

## 10. Xử lý Response

Rest Assured cung cấp nhiều cách để trích xuất và xử lý response từ API. Dưới đây là các phương pháp khác nhau để kiểm tra giá trị body của response, bao gồm các ví dụ minh họa, ưu/nhược điểm của từng phương pháp, và các hàm xử lý phổ biến.

### 10.1. Kiểm tra trực tiếp trong `.then()`

Phương pháp này sử dụng các matcher của Rest Assured (như `equalTo`, `containsString`) để kiểm tra body response ngay trong chuỗi `.then()`. Đây là cách phổ biến và ngắn gọn nhất.

#### 10.1.1. Ví dụ

```java
@Test
public void testCheckBodyInThen() {
    given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .body("name", equalTo("John Doe"))
        .body("email", equalTo("john@example.com"))
        .body("address.city", containsString("York"));
}
```

#### 10.1.2. Ưu điểm

- **Ngắn gọn**: Kiểm tra trực tiếp trong `.then()` giúp code dễ đọc, phù hợp với các test case đơn giản.
- **Tích hợp tốt với Given-When-Then**: Tuân thủ mô hình BDD, dễ hiểu cho cả tester và developer.
- **Hỗ trợ nhiều matcher**: Rest Assured cung cấp các matcher như `equalTo`, `containsString`, `hasItems` để kiểm tra linh hoạt.

#### 10.1.3. Nhược điểm

- **Khó xử lý logic phức tạp**: Nếu cần kiểm tra nhiều điều kiện hoặc xử lý dữ liệu phức tạp, việc nhúng logic vào `.then()` làm code khó bảo trì.
- **Khó tái sử dụng**: Các kiểm tra trong `.then()` thường không thể tái sử dụng ở các test case khác.
- **Khó debug**: Nếu kiểm tra thất bại, thông báo lỗi có thể không đủ chi tiết.

#### 10.1.4. Khi nào nên dùng

- Dùng cho các test case đơn giản, chỉ cần kiểm tra một vài trường cụ thể.
- Phù hợp khi không cần logic xử lý phức tạp hoặc lưu trữ dữ liệu response.

#### 10.1.5. Các hàm xử lý phổ biến

- `equalTo(value)`: Kiểm tra giá trị chính xác.
- `containsString(substring)`: Kiểm tra chuỗi có chứa một chuỗi con.
- `hasItems(items)`: Kiểm tra danh sách chứa các phần tử cụ thể.
- `notNullValue()`: Kiểm tra giá trị không null.
- `isEmpty()`: Kiểm tra chuỗi hoặc danh sách rỗng.

### 10.2. Sử dụng Assertion với Response trích xuất

Phương pháp này trích xuất `Response` object và sử dụng các assertion của framework kiểm thử (như JUnit/TestNG) để kiểm tra body. Điều này cho phép áp dụng logic xử lý phức tạp hơn.

#### 10.2.1. Ví dụ

```java
@Test
public void testCheckBodyWithAssertion() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    // Sử dụng JUnit assertions
    Assertions.assertEquals("John Doe", response.jsonPath().getString("name"));
    Assertions.assertEquals("john@example.com", response.jsonPath().getString("email"));
    Assertions.assertTrue(response.jsonPath().getString("address.city").contains("York"));
}
```

#### 10.2.2. Ưu điểm

- **Linh hoạt**: Có thể áp dụng logic kiểm tra phức tạp, như so sánh điều kiện, vòng lặp, hoặc xử lý dữ liệu response.
- **Dễ debug**: Assertions của JUnit/TestNG thường cung cấp thông báo lỗi chi tiết hơn.
- **Tích hợp với framework kiểm thử**: Dễ dàng sử dụng với các công cụ như JUnit, TestNG.

#### 10.2.3. Nhược điểm

- **Code dài hơn**: Yêu cầu trích xuất `Response` và viết các assertion riêng, làm tăng số dòng code.
- **Tách biệt với Given-When-Then**: Làm mất đi tính liền mạch của mô hình Given-When-Then.

#### 10.2.4. Khi nào nên dùng

- Dùng khi cần kiểm tra phức tạp, ví dụ: xử lý danh sách, kiểm tra điều kiện logic, hoặc kết hợp nhiều giá trị.
- Phù hợp khi cần tích hợp với các framework kiểm thử như JUnit hoặc TestNG.

#### 10.2.5. Các hàm xử lý phổ biến

- `response.jsonPath().getString(path)`: Trích xuất giá trị chuỗi từ JSON.
- `response.jsonPath().getInt(path)`: Trích xuất giá trị số nguyên.
- `response.jsonPath().getList(path)`: Trích xuất danh sách từ JSON.
- `response.asString()`: Lấy toàn bộ body dưới dạng chuỗi.
- `response.jsonPath().getMap(path)`: Trích xuất đối tượng JSON dưới dạng Map.

### 10.3. Deserialize Response thành Object và Assertion

Phương pháp này chuyển đổi response thành đối tượng Java (POJO) và sử dụng assertion để kiểm tra các thuộc tính của đối tượng. Điều này rất hữu ích khi làm việc với dữ liệu có cấu trúc rõ ràng.

#### 10.3.1. Ví dụ

```java
public class User {
    private int id;
    private String name;
    private String email;
    private Address address;

    public static class Address {
        private String city;
        // Getters và Setters
    }
    // Getters và Setters
}

@Test
public void testDeserializeAndAssert() {
    User user = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().as(User.class);

    // Sử dụng JUnit assertions
    Assertions.assertEquals("John Doe", user.getName());
    Assertions.assertEquals("john@example.com", user.getEmail());
    Assertions.assertTrue(user.getAddress().getCity().contains("York"));
}
```

#### 10.3.2. Ưu điểm

- **Dễ bảo trì**: Sử dụng POJO giúp code dễ đọc và dễ bảo trì, đặc biệt với các response phức tạp.
- **Kiểu an toàn**: Kiểm tra kiểu dữ liệu ngay từ khi deserialize, giảm nguy cơ lỗi runtime.
- **Tái sử dụng**: POJO có thể được sử dụng lại ở nhiều test case.

#### 10.3.3. Nhược điểm

- **Cần định nghĩa POJO**: Yêu cầu tạo các class Java tương ứng với cấu trúc response, tăng công việc ban đầu.
- **Phụ thuộc vào cấu trúc response**: Nếu API thay đổi cấu trúc, POJO cần được cập nhật.

#### 10.3.4. Khi nào nên dùng

- Dùng khi response có cấu trúc rõ ràng và cần kiểm tra nhiều trường.
- Phù hợp khi làm việc với dữ liệu phức tạp và muốn đảm bảo kiểu an toàn.

#### 10.3.5. Các hàm xử lý phổ biến

- `response.as(class)`: Chuyển đổi response thành đối tượng POJO.
- `response.jsonPath().getObject(path, class)`: Trích xuất một phần JSON thành POJO.
- `Assertions.assertEquals(expected, actual)`: So sánh giá trị của thuộc tính POJO.
- `Assertions.assertNotNull(value)`: Kiểm tra giá trị không null.
- `Assertions.assertTrue(condition)`: Kiểm tra điều kiện logic.

### 10.4. Sử dụng JsonPath với Logic Tùy chỉnh

Phương pháp này sử dụng `JsonPath` để trích xuất dữ liệu và áp dụng logic tùy chỉnh (như vòng lặp, lọc, hoặc xử lý dữ liệu phức tạp) trước khi kiểm tra.

#### 10.4.1. Ví dụ

```java
@Test
public void testJsonPathWithCustomLogic() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users")
        .then()
        .statusCode(200)
        .extract().response();

    // Tìm người dùng với email cụ thể
    Map<String, Object> user = response.jsonPath()
        .getList("data")
        .stream()
        .filter(u -> u.get("email").equals("john@example.com"))
        .findFirst()
        .orElse(null);

    Assertions.assertNotNull(user, "User with email john@example.com should exist");
    Assertions.assertEquals("John Doe", user.get("name"));
}
```

#### 10.4.2. Ưu điểm

- **Linh hoạt cao**: Cho phép xử lý dữ liệu phức tạp, như lọc, tìm kiếm, hoặc tính toán trên response.
- **Tích hợp với assertion**: Có thể kết hợp với JUnit/TestNG để kiểm tra kết quả.
- **Hữu ích cho danh sách lớn**: Phù hợp khi làm việc với mảng JSON lớn hoặc cần tìm kiếm dữ liệu cụ thể.

#### 10.4.3. Nhược điểm

- **Code phức tạp hơn**: Yêu cầu viết thêm logic xử lý, làm tăng độ phức tạp của code.
- **Hiệu suất**: Xử lý danh sách lớn hoặc logic phức tạp có thể ảnh hưởng đến hiệu suất test.

#### 10.4.4. Khi nào nên dùng

- Dùng khi response là danh sách lớn hoặc cần xử lý logic phức tạp, như tìm kiếm, lọc, hoặc tính toán dữ liệu.

#### 10.4.5. Các hàm xử lý phổ biến

- `response.jsonPath().getList(path)`: Trích xuất danh sách từ JSON.
- `response.jsonPath().getMap(path)`: Trích xuất đối tượng JSON dưới dạng Map.
- `Stream.filter(predicate)`: Lọc danh sách theo điều kiện.
- `Stream.findFirst()`: Lấy phần tử đầu tiên khớp điều kiện.
- `Assertions.assertNotNull(value)`: Kiểm tra giá trị không null.

### 10.5. Sử dụng JsonPath để kiểm tra danh sách

Phương pháp này sử dụng `JsonPath` để trích xuất và kiểm tra danh sách trong response.

#### 10.5.1. Ví dụ

```java
@Test
public void testJsonPathList() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users")
        .then()
        .statusCode(200)
        .extract().response();

    // Lấy danh sách tên người dùng
    List<String> names = response.jsonPath().getList("data.name");
    Assertions.assertTrue(names.contains("John Doe"), "List should contain John Doe");
    Assertions.assertEquals(3, names.size(), "List should have 3 users");
}
```

#### 10.5.2. Ưu điểm

- **Đơn giản với danh sách**: Dễ dàng trích xuất và kiểm tra các mảng JSON.
- **Tích hợp với assertion**: Có thể sử dụng assertion để kiểm tra kích thước, nội dung, hoặc các điều kiện khác.

#### 10.5.3. Nhược điểm

- **Giới hạn với logic phức tạp**: Nếu cần xử lý phức tạp hơn (như lọc theo nhiều điều kiện), cần kết hợp với logic tùy chỉnh.
- **Phụ thuộc JsonPath**: Chỉ hoạt động tốt với JSON, không phù hợp cho XML hoặc các định dạng khác.

#### 10.5.4. Khi nào nên dùng

- Dùng khi response trả về danh sách và cần kiểm tra các thuộc tính cụ thể của danh sách.

#### 10.5.5. Các hàm xử lý phổ biến

- `response.jsonPath().getList(path)`: Trích xuất danh sách từ JSON.
- `Assertions.assertTrue(condition)`: Kiểm tra điều kiện logic.
- `Assertions.assertEquals(expected, actual)`: So sánh kích thước hoặc giá trị danh sách.
- `names.contains(value)`: Kiểm tra danh sách chứa giá trị cụ thể.
- `names.size()`: Lấy kích thước danh sách.

### 10.6. Kiểm tra Response Header

Trích xuất và kiểm tra header:

```java
@Test
public void testResponseHeader() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    String contentType = response.getHeader("Content-Type");
    Assertions.assertEquals("application/json; charset=utf-8", contentType);
}
```

#### 10.6.1. Các hàm xử lý phổ biến

- `response.getHeader(name)`: Trích xuất giá trị của header cụ thể.
- `response.getHeaders()`: Lấy toàn bộ danh sách header.
- `Assertions.assertEquals(expected, actual)`: So sánh giá trị header.
- `Assertions.assertNotNull(header)`: Kiểm tra header tồn tại.

### 10.7. Kiểm tra Response Time

Kiểm tra thời gian phản hồi của API:

```java
@Test
public void testResponseTime() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    long responseTime = response.getTimeIn(TimeUnit.MILLISECONDS);
    Assertions.assertTrue(responseTime < 1000, "Response time should be less than 1 second");
}
```

#### 10.7.1. Các hàm xử lý phổ biến

- `response.getTimeIn(TimeUnit)`: Lấy thời gian phản hồi theo đơn vị thời gian.
- `Assertions.assertTrue(condition)`: Kiểm tra điều kiện thời gian.
- `Assertions.assertTrue(responseTime < maxTime)`: Kiểm tra thời gian nhỏ hơn giá trị tối đa.

### 10.8. Lưu Response vào File

Lưu response để phân tích sau:

```java
@Test
public void testSaveResponseToFile() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    // Lưu response thành file
    try {
        Files.writeString(Paths.get("response.json"), response.asString());
    } catch (IOException e) {
        Assertions.fail("Failed to save response to file: " + e.getMessage());
    }
}
```

#### 10.8.1. Các hàm xử lý phổ biến

- `response.asString()`: Lấy nội dung response dưới dạng chuỗi.
- `Files.writeString(path, string)`: Ghi chuỗi vào file.
- `Assertions.fail(message)`: Báo lỗi nếu lưu file thất bại.

### 10.9. Lưu ý khi xử lý Response

- **Kiểm tra Content-Type**: Đảm bảo response đúng định dạng (JSON, XML, v.v.) trước khi xử lý.
- **Xử lý lỗi**: Kiểm tra status code và nội dung lỗi nếu API trả về lỗi (400, 500, v.v.).
- **Logging**: Sử dụng `.log().all()` để ghi log response khi debug.
- **Hiệu suất**: Khi xử lý response lớn, tối ưu hóa bằng cách chỉ trích xuất dữ liệu cần thiết.

## 11. Best Practices

- **Tái sử dụng cấu hình**: Sử dụng `RequestSpecification` và `ResponseSpecification` để tránh lặp code.
- **Kiểm tra schema**: Sử dụng JSON Schema để đảm bảo response đúng định dạng.
- **Tổ chức test case**: Sử dụng framework như JUnit/TestNG để quản lý test.
- **Logging**: Bật logging để debug dễ dàng:

```java
given()
    .log().all() // Log request
    .when()
    .get("/users")
    .then()
    .log().all() // Log response
    .statusCode(200);
```
## 12. Cách đặt tên test case cho API test

Việc đặt tên test case một cách rõ ràng và nhất quán là rất quan trọng để đảm bảo tính dễ đọc, dễ bảo trì và dễ hiểu cho các thành viên trong nhóm. Một tên test case tốt nên phản ánh mục đích của bài kiểm thử, hành vi được kiểm tra, và kết quả mong đợi. 

### 12.1. Quy ước đặt tên test case

Một tên test case tốt thường tuân theo các nguyên tắc sau:

- **Rõ ràng và mô tả**: Tên test case nên mô tả chính xác hành vi hoặc chức năng được kiểm tra.
- **Ngắn gọn nhưng đủ thông tin**: Tránh tên quá dài, nhưng vẫn cung cấp đủ ngữ cảnh.
- **Tuân theo mẫu nhất quán**: Sử dụng một mẫu đặt tên thống nhất trong toàn bộ dự án để dễ tìm kiếm và quản lý.
- **Tránh thông tin dư thừa**: Không lặp lại thông tin đã được biểu thị trong tên class hoặc package.

### 12.2. Các mẫu đặt tên phổ biến

Dưới đây là một số mẫu đặt tên phổ biến, ưu tiên mô tả kết quả theo ngữ cảnh thay vì chỉ dựa vào mã trạng thái HTTP.

#### 12.2.1. Mẫu `test[Chức năng][Tình huống][Kết quả ngữ cảnh]`

Mẫu này tập trung vào chức năng, tình huống cụ thể và kết quả mong đợi được diễn đạt bằng ngôn ngữ tự nhiên.

- **Cú pháp**: `test[Chức năng][Tình huống][Kết quả ngữ cảnh]`
- **Ví dụ**:
  ```java
  @Test
  public void testGetUserByIdValidSucceedsWithUserData() {
      given()
          .baseUri("https://api.example.com")
          .pathParam("id", 1)
          .when()
          .get("/users/{id}")
          .then()
          .statusCode(200)
          .body("id", equalTo(1));
  }

  @Test
  public void testCreateUserWithValidDataSucceedsWithCreation() {
      String requestBody = "{\"name\": \"Jane Doe\", \"email\": \"jane@example.com\"}";
      given()
          .baseUri("https://api.example.com")
          .contentType(ContentType.JSON)
          .body(requestBody)
          .when()
          .post("/users")
          .then()
          .statusCode(201)
          .body("email", equalTo("jane@example.com"));
  }
  ```

- **Ưu điểm**:
  - Rất rõ ràng, dễ hiểu mục đích của test case.
  - Phản ánh cả input (tình huống) và output (kết quả) một cách tự nhiên.
  - Tập trung vào "cái gì" được trả về, không chỉ là "mã trạng thái".
- **Nhược điểm**:
  - Tên có thể dài nếu tình huống hoặc kết quả phức tạp.
- **Khi nào nên dùng**: Khi muốn nhấn mạnh hành vi cụ thể và kết quả mong đợi một cách tự nhiên, dễ hiểu. Đây là mẫu được khuyến khích sử dụng.

#### 12.2.2. Mẫu `test[HTTP Method][Resource][Tình huống]`

Mẫu này tập trung vào phương thức HTTP và tài nguyên, với kết quả được ngụ ý qua ngữ cảnh của tình huống (thường là trường hợp lỗi).

- **Cú pháp**: `test[HTTP Method][Resource][Tình huống]`
- **Ví dụ**:
  ```java
  @Test
  public void testGetUserWithInvalidIdFailsWithNotFound() {
      given()
          .baseUri("https://api.example.com")
          .pathParam("id", 999)
          .when()
          .get("/users/{id}")
          .then()
          .statusCode(404);
  }

  @Test
  public void testPostUserWithMissingEmailFailsWithBadRequest() {
      String requestBody = "{\"name\": \"Jane Doe\"}";
      given()
          .baseUri("https://api.example.com")
          .contentType(ContentType.JSON)
          .body(requestBody)
          .when()
          .post("/users")
          .then()
          .statusCode(400);
  }
  ```

- **Ưu điểm**:
  - Nhấn mạnh phương thức HTTP và tài nguyên, giúp dễ phân loại test case theo endpoint.
  - Kết quả được ngụ ý qua ngữ cảnh (FailsWithNotFound), tránh việc chỉ dựa vào mã trạng thái.
- **Nhược điểm**:
  - Có thể thiếu thông tin chi tiết về kết quả nếu ngữ cảnh không đủ rõ ràng.
- **Khi nào nên dùng**: Khi cần tổ chức test case theo phương thức HTTP hoặc tài nguyên, đặc biệt cho các trường hợp lỗi.

#### 12.2.3. Mẫu `should[Action][Result]` (Phong cách BDD)

Mẫu này sử dụng phong cách BDD (Behavior-Driven Development), tập trung vào hành động và kết quả mong đợi bằng ngôn ngữ tự nhiên.

- **Cú pháp**: `should[Action][Result]`
- **Ví dụ**:
  ```java
  @Test
  public void shouldRetrieveUserWhenIdIsValid() {
      given()
          .baseUri("https://api.example.com")
          .pathParam("id", 1)
          .when()
          .get("/users/{id}")
          .then()
          .statusCode(200)
          .body("name", equalTo("John Doe"));
  }

  @Test
  public void shouldFailWhenCreatingUserWithInvalidData() {
      String requestBody = "{\"name\": \"\", \"email\": \"invalid-email\"}";
      given()
          .baseUri("https://api.example.com")
          .contentType(ContentType.JSON)
          .body(requestBody)
          .when()
          .post("/users")
          .then()
          .statusCode(400);
  }
  ```

- **Ưu điểm**:
  - Tuân thủ phong cách BDD, dễ đọc và thân thiện với cả các bên liên quan không rành kỹ thuật.
  - Tập trung vào hành vi (behavior) của API.
- **Nhược điểm**:
  - Có thể không rõ ràng về tài nguyên hoặc phương thức HTTP nếu không có ngữ cảnh từ tên class.
- **Khi nào nên dùng**: Khi dự án áp dụng BDD hoặc cần tên test case dễ hiểu cho cả đội phát triển và đội nghiệp vụ.

### 12.3. Lưu ý khi đặt tên test case

- **Sử dụng CamelCase**: Tuân thủ quy ước Java, ví dụ: `testGetUserByIdValidSucceeds`.
- **Tránh sử dụng số thứ tự**: Không đặt tên như `test1`, `test2` vì không cung cấp thông tin.
- **Bao gồm ngữ cảnh cụ thể**: Nếu test liên quan đến điều kiện đặc biệt (ví dụ: `InvalidId`, `MissingEmail`), hãy đưa vào tên để rõ ràng.
- **Nhất quán trong dự án**: Chọn một mẫu và đảm bảo toàn đội sử dụng cùng mẫu đó để dễ quản lý.
- **Mô tả kết quả mong đợi**: Thay vì chỉ ghi mã trạng thái, hãy dùng các từ mô tả hành vi như `Succeeds`, `FailsWithNotFound`, `IsCreated` để làm rõ kết quả.

### 12.4. Ví dụ tổ chức test case

Dưới đây là ví dụ về cách tổ chức và đặt tên test case trong một class kiểm thử, áp dụng quy ước hướng hành vi.

```java
// File: UserApiTests.java
public class UserApiTests {

    @BeforeAll
    public static void setup() {
        RestAssured.baseURI = "https://api.example.com";
        RestAssured.basePath = "/v1";
    }

    @Test
    public void testGetUserByIdValidSucceeds() {
        given()
            .pathParam("id", 1)
            .when()
            .get("/users/{id}")
            .then()
            .statusCode(200)
            .body("name", equalTo("John Doe"));
    }

    @Test
    public void testGetUserByIdInvalidFailsWithNotFound() {
        given()
            .pathParam("id", 999)
            .when()
            .get("/users/{id}")
            .then()
            .statusCode(404);
    }

    @Test
    public void testPostUserWithValidDataSucceedsWithCreation() {
        String requestBody = "{\"name\": \"Jane Doe\", \"email\": \"jane@example.com\"}";
        given()
            .contentType(ContentType.JSON)
            .body(requestBody)
            .when()
            .post("/users")
            .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }

    @Test
    public void testPostUserWithMissingEmailFailsWithBadRequest() {
        String requestBody = "{\"name\": \"Jane Doe\"}";
        given()
            .contentType(ContentType.JSON)
            .body(requestBody)
            .when()
            .post("/users")
            .then()
            .statusCode(400);
    }
}
```

### 12.5. Best Practices

- **Tổ chức theo tài nguyên**: Nhóm các test case theo endpoint hoặc tài nguyên trong các class riêng biệt (ví dụ: `UserApiTests`, `OrderApiTests`).
- **Kiểm tra cả trường hợp tích cực và tiêu cực**: Đảm bảo bao gồm các test case cho cả dữ liệu hợp lệ và không hợp lệ.
- **Kết hợp Tên phương thức và @DisplayName để có hiệu quả tối ưu**: Đây là phương pháp được khuyến khích mạnh mẽ để đạt được sự cân bằng giữa code dễ đọc và báo cáo chi tiết.
  - **Tên phương thức (Method Name)**: Sử dụng quy ước hướng hành vi, tập trung vào ngữ cảnh (ví dụ: `...FailsWithNotFound`). Tên này phục vụ cho những người trực tiếp đọc code (dev, QA).
  - **Chú thích @DisplayName (JUnit 5)**: Cung cấp một câu mô tả đầy đủ, tự nhiên, bao gồm cả các chi tiết kỹ thuật nếu cần (phương thức HTTP, endpoint, mã trạng thái)..

- **Ví dụ thực tế**:
  ```java
  @Test
  @DisplayName("GET /users/{id}: Should return user data and status 200 when ID is valid")
  public void testGetUserByIdValidSucceeds() {
      given()
          .pathParam("id", 1)
          .when()
          .get("/users/{id}")
          .then()
          .statusCode(200)
          .body("name", equalTo("John Doe"));
  }

  @Test
  @DisplayName("GET /users/{id}: Should return an error and status 404 when ID does not exist")
  public void testGetUserByIdInvalidFailsWithNotFound() {
      given()
          .pathParam("id", 999)
          .when()
          .get("/users/{id}")
          .then()
          .statusCode(404);
  }
  ```


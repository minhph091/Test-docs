# Sử dụng Rest Assured để Kiểm thử API RESTful

## 1. Giới thiệu
Rest Assured là một thư viện Java để kiểm thử API RESTful. Nó cung cấp cú pháp đơn giản, dễ đọc để gửi các HTTP request và xác minh response,cho phép tester viết các kịch bản kiểm thử tự động để xác minh phản hồi của API, như trạng thái HTTP, nội dung JSON/XML, hay header. Tài liệu này hướng dẫn cách sử dụng Rest Assured để kiểm thử API, bao gồm cách thiết lập, tạo body request JSON, gửi request và kiểm tra kết quả.

## 2. Tạo Body Request JSON với Text Blocks
Text Blocks (tính năng từ Java 15) giúp viết chuỗi JSON dễ đọc và gọn gàng hơn so với chuỗi thông thường. Đây là cách tạo body request JSON bằng Text Blocks trong Rest Assured.

### 2.1. Cú pháp cơ bản
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

### 2.2. Sử dụng trong Rest Assured
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

### 2.3. Lợi ích của Text Blocks
- **Dễ đọc**: JSON được định dạng tự nhiên, không cần thêm ký tự thoát (`\`) hoặc nối chuỗi.
- **Bảo trì dễ dàng**: Dễ chỉnh sửa cấu trúc JSON mà không lo lỗi cú pháp.
- **Tích hợp biến**: Có thể chèn biến vào Text Block bằng cách sử dụng `String.format` hoặc concatenation.

### 2.4. Chèn biến vào Text Block
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

### 2.5. Lưu ý
- **Yêu cầu Java phiên bản**: Text Blocks chỉ hỗ trợ từ Java 15 trở lên. Nếu dùng phiên bản thấp hơn, bạn cần sử dụng chuỗi thông thường hoặc các thư viện như Jackson/Gson để tạo JSON.
- **Định dạng**: Đảm bảo JSON trong Text Block đúng cú pháp để tránh lỗi khi gửi request.

## 3. Thiết lập dự án
### 3.1. Yêu cầu
- Java JDK 8 trở lên (JDK 15+ để sử dụng Text Blocks).
- Maven hoặc Gradle để quản lý thư viện.
- IDE (IntelliJ, Eclipse, v.v.).

### 3.2. Thêm dependency
Thêm Rest Assured vào file `pom.xml` (nếu dùng Maven):

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.5.0</version>
    <scope>test</scope>
</dependency>
```

### 3.3. Cấu hình cơ bản
Import các package cần thiết trong class test:

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import static io.restassured.RestAssured.given;
```

## 4. Cấu trúc cơ bản của một test case
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

## 5. Các phương thức HTTP phổ biến
### 5.1. GET: Lấy dữ liệu
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

### 5.2. POST: Tạo tài nguyên
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

### 5.3. PUT: Cập nhật tài nguyên
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

### 5.4. DELETE: Xóa tài nguyên
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

## 6. Kiểm tra Response
### 6.1. Kiểm tra Status Code
```java
.then().statusCode(200); // Kiểm tra mã trạng thái 200 OK
```

### 6.2. Kiểm tra Response Body
Sử dụng `jsonPath` để truy xuất dữ liệu trong JSON response:

```java
.then()
    .body("data.id", equalTo(1))
    .body("data.name", equalTo("John Doe"));
```

### 6.3. Kiểm tra Header
```java
.then()
    .header("Content-Type", equalTo("application/json; charset=utf-8"));
```

### 6.4. Kiểm tra Schema
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

## 7. Xác thực (Authentication)
### 7.1. Basic Authentication
```java
given()
    .auth().basic("username", "password")
    .when()
    .get("/secured-endpoint")
    .then()
    .statusCode(200);
```

### 7.2. OAuth 2.0
```java
given()
    .auth().oauth2("access_token")
    .when()
    .get("/secured-endpoint")
    .then()
    .statusCode(200);
```

### 7.3. JWT Authentication
JSON Web Token (JWT) thường được sử dụng để xác thực API. JWT được gửi trong header `Authorization` với định dạng `Bearer <token>`.

#### 7.3.1. Gửi request với JWT
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

#### 7.3.2. Lấy JWT từ API đăng nhập
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

#### 7.3.3. Lưu ý khi sử dụng JWT
- **Lưu trữ JWT an toàn**: Trong môi trường kiểm thử, đảm bảo JWT không bị lộ trong log hoặc mã nguồn.
- **Xử lý token hết hạn**: Thêm logic để làm mới token nếu API trả về lỗi 401 (Unauthorized).
- **Kiểm tra token hợp lệ**: Xác minh cấu trúc và chữ ký của JWT nếu cần (sử dụng thư viện như `jjwt`).

## 8. Tùy chỉnh Request
### 8.1. Thêm Query Parameters
```java
given()
    .queryParam("key", "value")
    .when()
    .get("/endpoint");
```

### 8.2. Thêm Headers
```java
given()
    .header("Authorization", "Bearer token")
    .header("Custom-Header", "value")
    .when()
    .get("/endpoint");
```

### 8.3. Sử dụng Request Specification
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

## 9. Xử lý Response
Rest Assured cung cấp nhiều cách để trích xuất và xử lý response từ API. Dưới đây là các kỹ thuật chi tiết để làm việc với response.

### 9.1. Trích xuất Response
Lưu response để xử lý sau:

```java
@Test
public void testExtractResponse() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    // Trích xuất các giá trị cụ thể
    String name = response.jsonPath().getString("name");
    int id = response.jsonPath().getInt("id");
    assertEquals("John Doe", name);
    assertEquals(1, id);
}
```

### 9.2. Sử dụng JsonPath để xử lý JSON Response
`JsonPath` cho phép truy xuất dữ liệu trong JSON response một cách dễ dàng.

#### 9.2.1. Trích xuất giá trị đơn giản
```java
@Test
public void testJsonPathSimple() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    // Trích xuất các trường cụ thể
    String email = response.jsonPath().getString("email");
    assertEquals("john@example.com", email);
}
```

#### 9.2.2. Trích xuất danh sách
Xử lý response chứa mảng JSON:

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
    assertTrue(names.contains("John Doe"));
}
```

#### 9.2.3. Truy xuất JSON lồng nhau
Xử lý JSON có cấu trúc phức tạp:

```java
@Test
public void testJsonPathNested() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    // Truy xuất trường trong đối tượng lồng nhau
    String city = response.jsonPath().getString("address.city");
    assertEquals("New York", city);
}
```

### 9.3. Sử dụng XmlPath để xử lý XML Response
Nếu API trả về XML, sử dụng `xmlPath`:

```java
@Test
public void testXmlPath() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1/xml")
        .then()
        .statusCode(200)
        .extract().response();

    // Trích xuất giá trị từ XML
    String name = response.xmlPath().getString("user.name");
    assertEquals("John Doe", name);
}
```

### 9.4. Deserialize Response thành Object
Chuyển đổi response thành đối tượng Java (POJO) để dễ xử lý:

```java
public class User {
    private int id;
    private String name;
    private String email;

    // Getters và Setters
}

@Test
public void testDeserializeResponse() {
    User user = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().as(User.class);

    assertEquals("John Doe", user.getName());
    assertEquals("john@example.com", user.getEmail());
}
```

### 9.5. Xử lý Response phức tạp
#### 9.5.1. Trích xuất một phần Response
Lấy một phần của response dưới dạng chuỗi hoặc đối tượng:

```java
@Test
public void testExtractPartialResponse() {
    Response response = given()
        .baseUri("https://api.example.com")
        .when()
        .get("/users/1")
        .then()
        .statusCode(200)
        .extract().response();

    // Trích xuất một phần JSON
    String addressJson = response.jsonPath().getString("address");
    assertTrue(addressJson.contains("New York"));
}
```

#### 9.5.2. Kiểm tra Response lớn
Khi response là một mảng lớn, sử dụng `find` hoặc `findAll` trong JsonPath:

```java
@Test
public void testLargeResponse() {
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
    assertNotNull(user);
}
```

### 9.6. Xử lý Response Header
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
    assertEquals("application/json; charset=utf-8", contentType);
}
```

### 9.7. Xử lý Response Time
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
    assertTrue(responseTime < 1000, "Response time should be less than 1 second");
}
```

### 9.8. Lưu Response vào File
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
        fail("Failed to save response to file: " + e.getMessage());
    }
}
```

### 9.9. Lưu ý khi xử lý Response
- **Kiểm tra Content-Type**: Đảm bảo response đúng định dạng (JSON, XML, v.v.) trước khi xử lý.
- **Xử lý lỗi**: Kiểm tra status code và nội dung lỗi nếu API trả về lỗi (400, 500, v.v.).
- **Logging**: Sử dụng `.log().all()` để ghi log response khi debug.
- **Hiệu suất**: Khi xử lý response lớn, tối ưu hóa bằng cách chỉ trích xuất dữ liệu cần thiết.

## 10. Best Practices
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

## 11. Kết luận
Rest Assured là một công cụ mạnh mẽ, dễ sử dụng để kiểm thử API RESTful. Với cú pháp đơn giản, hỗ trợ Text Blocks để tạo JSON body, các tính năng xác thực như JWT, và khả năng xử lý response linh hoạt, nó giúp tự động hóa kiểm thử một cách hiệu quả. Hãy áp dụng các ví dụ trên và tùy chỉnh theo nhu cầu dự án của bạn.

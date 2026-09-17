# Lộ trình: Từ C# cơ bản đến viết được một REST API

> Mục tiêu: sau khi đọc hết file này, bạn hiểu được cú pháp C# thường dùng trong code API,
> hiểu ASP.NET Core xử lý một request như thế nào, dùng EF Core để đọc/ghi database,
> và tự tay build được một CRUD API hoàn chỉnh.

**Môi trường:** .NET 8 (LTS) + C# 12. Cài SDK tại https://dotnet.microsoft.com/download.
IDE: Visual Studio 2022, Rider, hoặc VS Code + extension C# Dev Kit.
Kiểm tra: `dotnet --version`.

> .NET 10 hiện đã là bản LTS mới nhất. Toàn bộ kiến thức dưới đây vẫn đúng trên .NET 10;
> chỉ cần đổi `net8.0` thành `net10.0` trong file `.csproj`.

---

## Mục lục

- [C# 12](#c-12)
  - [Basic syntax](#basic-syntax)
  - [Extension method](#extension-method)
  - [Null-conditional operators](#null-conditional-operators)
  - [Type-testing operators and cast expressions](#type-testing-operators-and-cast-expressions)
  - [Lambda expressions and anonymous functions](#lambda-expressions-and-anonymous-functions)
  - [Pattern matching](#pattern-matching)
  - [Records](#records)
  - [Partial classes](#partial-classes)
- [ASP.NET Core](#aspnet-core)
  - [Web API là gì?](#web-api-là-gì)
  - [Tạo project](#tạo-project)
  - [Program.cs — trái tim của ứng dụng](#programcs--trái-tim-của-ứng-dụng)
  - [Dependency Injection & service lifetime](#dependency-injection--service-lifetime)
  - [Controller](#controller)
  - [Minimal API (cách viết ngắn, không cần controller)](#minimal-api-cách-viết-ngắn-không-cần-controller)
  - [Validation](#validation)
  - [Xử lý lỗi tập trung](#xử-lý-lỗi-tập-trung)
  - [Configuration & Secrets](#configuration--secrets)
  - [Logging](#logging)
  - [Authentication với JWT (khái quát)](#authentication-với-jwt-khái-quát)
  - [Test API](#test-api)
- [EF Core](#ef-core)
  - [Cài package](#cài-package)
  - [Entity — class map với bảng](#entity--class-map-với-bảng)
  - [DbContext — cầu nối tới database](#dbcontext--cầu-nối-tới-database)
  - [Migration — quản lý thay đổi schema](#migration--quản-lý-thay-đổi-schema)
  - [CRUD](#crud)
  - [Những điều cần nhớ về EF Core](#những-điều-cần-nhớ-về-ef-core)
- [Database](#database)
  - [Khái niệm cần biết](#khái-niệm-cần-biết)
  - [Quan hệ](#quan-hệ)
  - [SQL cơ bản (nên biết dù dùng EF Core)](#sql-cơ-bản-nên-biết-dù-dùng-ef-core)
  - [Chọn database cho project học](#chọn-database-cho-project-học)
  - [Quy ước tốt](#quy-ước-tốt)
- [Thực hành: build API "Product" từ 0](#thực-hành-build-api-product-từ-0)
- [Checklist tự đánh giá](#checklist-tự-đánh-giá)
- [Tài liệu tham khảo](#tài-liệu-tham-khảo)

---

## C# 12

### Basic syntax

Một file C# tối giản (.NET 8 dùng *top-level statements*, không cần viết `class Program`):

```csharp
// Program.cs
Console.WriteLine("Hello World");
```

**Biến và kiểu dữ liệu**

```csharp
int soNguyen = 10;
long soLon = 10_000_000_000;
decimal tienTe = 199.99m;      // dùng decimal cho tiền, KHÔNG dùng double
double soThuc = 3.14;
bool dungSai = true;
string chuoi = "xin chào";
DateTime thoiGian = DateTime.UtcNow;   // API nên lưu UTC
Guid id = Guid.NewGuid();

var tuSuyLuan = 42;            // var: compiler tự suy ra kiểu (ở đây là int)
const int MAX = 100;           // hằng số, không đổi được
```

**String interpolation** — nối chuỗi bằng `$`:

```csharp
string ten = "An";
int tuoi = 25;
Console.WriteLine($"Tôi là {ten}, {tuoi} tuổi");         // Tôi là An, 25 tuổi
Console.WriteLine($"Giá: {tienTe:N0} đ");                 // format số
```

**Điều kiện & vòng lặp**

```csharp
if (tuoi >= 18) Console.WriteLine("Đủ tuổi");
else if (tuoi >= 16) Console.WriteLine("Gần đủ");
else Console.WriteLine("Chưa đủ");

// switch expression (C# 8+) — ngắn hơn switch-case cũ
string loai = tuoi switch
{
    < 13 => "Trẻ em",
    < 18 => "Thiếu niên",
    _    => "Người lớn"          // _ là "mọi trường hợp còn lại"
};

for (int i = 0; i < 5; i++) Console.WriteLine(i);

foreach (var item in new[] { "a", "b", "c" }) Console.WriteLine(item);

while (tuoi < 30) tuoi++;
```

**Collection** — 3 loại dùng nhiều nhất trong API:

```csharp
// Array: cố định độ dài
int[] mangSo = [1, 2, 3];                    // C# 12: collection expression

// List<T>: thêm/xoá được — dùng nhiều nhất
List<string> danhSach = ["An", "Bình"];
danhSach.Add("Chi");
danhSach.Remove("An");
Console.WriteLine(danhSach.Count);

// Dictionary<TKey, TValue>: tra cứu theo key
Dictionary<int, string> tenTheoId = new()
{
    [1] = "An",
    [2] = "Bình"
};
if (tenTheoId.TryGetValue(1, out var found)) Console.WriteLine(found);
```

**Class, property, method** — nền tảng của mọi thứ trong API:

```csharp
public class Product
{
    // Property: dữ liệu của object
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public bool IsActive { get; private set; } = true;   // chỉ sửa được từ bên trong class

    // Constructor: chạy khi tạo object
    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }

    // Method: hành vi
    public decimal PriceWithVat(decimal vatRate = 0.1m) => Price * (1 + vatRate);

    public void Deactivate() => IsActive = false;
}

// Dùng:
var p = new Product("Bàn phím", 500_000);
Console.WriteLine(p.PriceWithVat());   // 550000
```

**Interface & Dependency Injection mindset** — cực kỳ quan trọng cho ASP.NET Core:

```csharp
// Interface = bản hợp đồng: "ai implement tôi thì phải có các method này"
public interface IProductService
{
    Task<Product?> GetByIdAsync(int id);
    Task<List<Product>> GetAllAsync();
}

// Class implement interface
public class ProductService : IProductService
{
    public Task<Product?> GetByIdAsync(int id) => /* ... */ Task.FromResult<Product?>(null);
    public Task<List<Product>> GetAllAsync()   => Task.FromResult(new List<Product>());
}
```

Vì sao cần interface? Để controller phụ thuộc vào `IProductService` (bản hợp đồng) chứ không
phụ thuộc vào class cụ thể → thay đổi implementation hoặc viết unit test mà không sửa controller.

**async / await** — bắt buộc phải hiểu, vì mọi thao tác I/O trong API đều async:

```csharp
// Task<T>  = "một giá trị T sẽ có trong tương lai"
// async    = "hàm này có await bên trong"
// await    = "chờ ở đây nhưng KHÔNG block thread, thread đi phục vụ request khác"
public async Task<string> LayDuLieuAsync()
{
    using var http = new HttpClient();
    string json = await http.GetStringAsync("https://api.example.com/data");
    return json;
}
```

Quy tắc: hàm async trả `Task`/`Task<T>`, tên kết thúc bằng `Async`, và **không** dùng
`.Result` hay `.Wait()` (gây deadlock) — luôn `await`.

**Exception handling**

```csharp
try
{
    var result = 10 / soChia;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Lỗi chia 0: {ex.Message}");
}
catch (Exception ex)         // bắt mọi lỗi còn lại
{
    Console.WriteLine($"Lỗi: {ex.Message}");
    throw;                   // ném lại để tầng trên xử lý
}
finally
{
    Console.WriteLine("Luôn chạy");
}
```

### Extension method

Thêm method cho một class **có sẵn** mà không cần sửa class đó (kể cả class của .NET).
Điều kiện: đặt trong `static class`, method là `static`, tham số đầu có từ khoá `this`.

```csharp
public static class StringExtensions
{
    public static bool IsNullOrEmptyOrWhiteSpace(this string? value)
        => string.IsNullOrWhiteSpace(value);

    public static string ToSlug(this string value)
        => value.Trim().ToLowerInvariant().Replace(' ', '-');
}

// Gọi như thể nó là method của string:
"Bàn Phím Cơ".ToSlug();          // "bàn-phím-cơ"
```

Trong ASP.NET Core bạn sẽ thấy pattern này khắp nơi — `services.AddControllers()`,
`app.UseAuthentication()` đều là extension method. Cách hay dùng để gom code đăng ký DI:

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.AddScoped<IProductService, ProductService>();
        return services;                      // return để chain được
    }
}

// Program.cs
builder.Services.AddApplicationServices();
```

### Null-conditional operators

`null` là nguyên nhân crash phổ biến nhất. C# có nhóm toán tử xử lý gọn:

```csharp
Product? p = LayProduct();       // dấu ? = có thể null (nullable reference type)

// ?.  gọi member chỉ khi object không null, ngược lại trả null
int? len = p?.Name.Length;       // p null => len null, không crash

// ?[] tương tự cho index
var first = danhSach?[0];

// ??  null-coalescing: lấy giá trị bên phải nếu bên trái null
string ten = p?.Name ?? "Không rõ";

// ??= gán chỉ khi đang null
List<string>? list = null;
list ??= new List<string>();

// !   null-forgiving: "tôi chắc chắn không null" — dùng hạn chế, sai là crash
var chacChan = p!.Name;
```

Kết hợp trong API để tránh `NullReferenceException`:

```csharp
var product = await _db.Products.FindAsync(id);
if (product is null) return NotFound();      // guard clause: thoát sớm
return Ok(product);                          // dưới đây product chắc chắn không null
```

Bật kiểm tra null của compiler trong `.csproj` (mặc định đã bật ở template .NET 8):

```xml
<Nullable>enable</Nullable>
```

### Type-testing operators and cast expressions

```csharp
object obj = "hello";

// is: kiểm tra kiểu — kèm pattern để khai báo biến luôn
if (obj is string s)
    Console.WriteLine(s.Length);          // s đã là string

// is not
if (obj is not null) { /* ... */ }

// as: ép kiểu, thất bại trả null (không throw) — chỉ cho reference type
string? s2 = obj as string;

// (T)x: ép kiểu trực tiếp, thất bại thì throw InvalidCastException
string s3 = (string)obj;

// Ép kiểu số (có thể mất dữ liệu)
double d = 3.99;
int i = (int)d;                            // 3 (cắt phần thập phân)

// Parse chuỗi an toàn — dùng TryParse
if (int.TryParse("123", out int num)) Console.WriteLine(num);
```

Khi nào dùng gì: ưu tiên `is T x` (an toàn + gọn), dùng `(T)x` khi bạn *muốn* nó throw nếu sai.

### Lambda expressions and anonymous functions

Lambda = hàm viết ngắn không cần tên: `(tham số) => biểu thức`.

```csharp
Func<int, int, int> cong = (a, b) => a + b;      // Func: có giá trị trả về
Action<string> log = msg => Console.WriteLine(msg);  // Action: void
Func<Product, bool> reNhat = p => p.Price < 100_000;
```

Lambda là nền tảng của **LINQ** — thứ bạn dùng liên tục khi query dữ liệu:

```csharp
var products = new List<Product> { /* ... */ };

var ketQua = products
    .Where(p => p.IsActive && p.Price < 1_000_000)   // lọc
    .OrderByDescending(p => p.Price)                  // sắp xếp
    .Select(p => new { p.Id, p.Name })                // chọn field (anonymous type)
    .Take(10)                                         // lấy 10 cái đầu
    .ToList();                                        // thực thi

// Các method hay dùng khác
var mot     = products.FirstOrDefault(p => p.Id == 5);   // null nếu không có
var coKhong = products.Any(p => p.Price > 0);            // bool
var dem     = products.Count(p => p.IsActive);
var tong    = products.Sum(p => p.Price);
var nhom    = products.GroupBy(p => p.IsActive);
```

Trong minimal API, cả endpoint cũng là lambda:

```csharp
app.MapGet("/products", async (AppDbContext db) => await db.Products.ToListAsync());
```

### Pattern matching

Kiểm tra "hình dạng" của dữ liệu và tách giá trị ra cùng lúc.

```csharp
// 1. Type pattern
if (obj is int n && n > 0) { /* ... */ }

// 2. switch expression + property pattern
string PhanLoai(Product p) => p switch
{
    { Price: 0 }                        => "Miễn phí",
    { Price: < 100_000, IsActive: true } => "Giá rẻ",
    { IsActive: false }                 => "Ngừng bán",
    _                                   => "Bình thường"
};

// 3. Relational + logical pattern (and / or / not)
string Grade(int diem) => diem switch
{
    >= 9 and <= 10 => "A",
    >= 7 and < 9   => "B",
    >= 5 and < 7   => "C",
    < 5 and >= 0   => "D",
    _              => throw new ArgumentOutOfRangeException(nameof(diem))
};

// 4. Tuple pattern
string Ket(int a, int b) => (a, b) switch
{
    (0, 0) => "Cả hai bằng 0",
    (0, _) => "a bằng 0",
    (_, 0) => "b bằng 0",
    _      => "Khác 0"
};

// 5. List pattern (C# 11+)
int[] arr = [1, 2, 3];
string desc = arr switch
{
    []            => "rỗng",
    [var only]    => $"một phần tử: {only}",
    [var f, .., var l] => $"đầu {f}, cuối {l}",
};
```

Ứng dụng thực tế trong API — map exception sang HTTP status:

```csharp
var statusCode = ex switch
{
    KeyNotFoundException     => StatusCodes.Status404NotFound,
    ArgumentException        => StatusCodes.Status400BadRequest,
    UnauthorizedAccessException => StatusCodes.Status401Unauthorized,
    _                        => StatusCodes.Status500InternalServerError
};
```

### Records

`record` = class/struct được compiler sinh sẵn constructor, so sánh theo **giá trị**,
`ToString()` đẹp, và `with` để copy. Rất phù hợp làm **DTO** (Data Transfer Object) trong API.

```csharp
// Positional record — 1 dòng thay cho ~30 dòng class
public record ProductDto(int Id, string Name, decimal Price);

var a = new ProductDto(1, "Bàn phím", 500_000);
var b = new ProductDto(1, "Bàn phím", 500_000);
Console.WriteLine(a == b);          // True — so sánh giá trị (class sẽ là False)
Console.WriteLine(a);               // ProductDto { Id = 1, Name = Bàn phím, Price = 500000 }

// with: tạo bản copy và đổi vài field (bản gốc không đổi = immutable)
var c = a with { Price = 450_000 };

// Deconstruct
var (id, name, price) = a;

// Record có thể thêm member như class thường
public record CreateProductRequest(string Name, decimal Price)
{
    public bool IsValid => !string.IsNullOrWhiteSpace(Name) && Price >= 0;
}

// record struct: cho value type nhỏ
public readonly record struct Money(decimal Amount, string Currency);
```

**Vì sao API cần DTO tách khỏi Entity?**
- Không lộ field nội bộ (`PasswordHash`, `IsDeleted`) ra ngoài.
- Client gửi lên thiếu/thừa field không làm hỏng entity trong DB (tránh over-posting).
- Đổi cấu trúc DB mà không phá vỡ contract của API.

```csharp
// Entity (map với DB)               →  DTO (trả cho client)
public class User { int Id; string Email; string PasswordHash; }
public record UserDto(int Id, string Email);   // không có PasswordHash
```

### Partial classes

`partial` cho phép chia **một** class ra nhiều file; compiler ghép lại thành một.

```csharp
// Person.cs
public partial class Person
{
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
}

// Person.Helpers.cs
public partial class Person
{
    public string FullName => $"{FirstName} {LastName}";
}
```

Dùng khi nào:
- **Code generator**: file sinh tự động (EF Core reverse engineering, Swagger client, gRPC)
  nằm một file; code bạn viết tay nằm file khác → chạy lại generator không mất code của bạn.
- Class quá lớn, muốn tách theo nhóm chức năng.

Cũng có `partial method` — khai báo ở một file, cài đặt ở file khác. Trong .NET 8 bạn sẽ gặp
`[GeneratedRegex]` dùng partial method:

```csharp
public partial class Validators
{
    [GeneratedRegex(@"^[\w\.-]+@[\w\.-]+\.\w{2,}$")]
    private static partial Regex EmailRegex();     // thân hàm do compiler sinh

    public static bool IsEmail(string input) => EmailRegex().IsMatch(input);
}
```

---

## ASP.NET Core

### Web API là gì?

Client (web/mobile) gửi **HTTP request** → server xử lý → trả **HTTP response** (thường là JSON).

| Thành phần | Ví dụ | Ý nghĩa |
|---|---|---|
| Method | `GET` | Lấy dữ liệu (không đổi dữ liệu) |
| | `POST` | Tạo mới |
| | `PUT` | Cập nhật toàn bộ |
| | `PATCH` | Cập nhật một phần |
| | `DELETE` | Xoá |
| URL | `/api/products/5?page=1` | Tài nguyên + query string |
| Header | `Authorization: Bearer xxx` | Metadata (token, content type) |
| Body | `{"name":"Bàn phím"}` | Dữ liệu gửi lên (POST/PUT/PATCH) |

**Status code** cần nhớ:

| Code | Nghĩa | Khi nào trả |
|---|---|---|
| 200 OK | Thành công | GET/PUT thành công |
| 201 Created | Đã tạo | POST thành công (kèm header `Location`) |
| 204 No Content | Thành công, không có body | DELETE thành công |
| 400 Bad Request | Client gửi sai | Validation thất bại |
| 401 Unauthorized | Chưa đăng nhập | Thiếu/sai token |
| 403 Forbidden | Đã đăng nhập nhưng không có quyền | Sai role |
| 404 Not Found | Không tìm thấy | Id không tồn tại |
| 409 Conflict | Xung đột | Trùng email, trùng key |
| 500 Internal Server Error | Lỗi server | Bug, exception chưa xử lý |

### Tạo project

```bash
dotnet new webapi -n MyApi                     # Minimal API (mặc định từ .NET 8)
dotnet new webapi -n MyApi --use-controllers   # Controller-based — dùng cho phần dưới
cd MyApi
dotnet run
```

Cấu trúc thư mục nên theo:

```
MyApi/
├── Program.cs                  # điểm khởi động + cấu hình
├── appsettings.json            # config (connection string, ...)
├── appsettings.Development.json # config riêng cho môi trường dev
├── Controllers/                # nhận request, trả response
├── Models/  (hoặc Entities/)   # class map với bảng DB
├── DTOs/                       # record dùng để nhận/trả dữ liệu
├── Data/                       # DbContext, Migrations
└── Services/                   # business logic
```

### Program.cs — trái tim của ứng dụng

```csharp
var builder = WebApplication.CreateBuilder(args);

// ===== 1. ĐĂNG KÝ SERVICE (Dependency Injection container) =====
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();               // UI test API

builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddScoped<IProductService, ProductService>();

builder.Services.AddCors(o => o.AddDefaultPolicy(p =>
    p.WithOrigins("http://localhost:3000").AllowAnyHeader().AllowAnyMethod()));

var app = builder.Build();

// ===== 2. MIDDLEWARE PIPELINE (thứ tự RẤT quan trọng) =====
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();          // mở https://localhost:xxxx/swagger
}

app.UseHttpsRedirection();
app.UseCors();
app.UseAuthentication();         // xác định "bạn là ai" — phải TRƯỚC Authorization
app.UseAuthorization();          // xác định "bạn được làm gì"
app.MapControllers();            // route request tới controller

app.Run();
```

**Middleware pipeline** — request đi qua từng lớp theo thứ tự khai báo, response đi ngược lại:

```
Request  →  HttpsRedirection → Cors → Authentication → Authorization → Endpoint
Response ←──────────────────────────────────────────────────────────────┘
```

Sai thứ tự là bug khó tìm (ví dụ để `UseAuthorization()` trước `UseAuthentication()`
thì user luôn bị coi là chưa đăng nhập).

### Dependency Injection & service lifetime

Bạn **không** `new` service trong controller. Bạn khai báo ở constructor, ASP.NET Core tự "tiêm" vào.

```csharp
builder.Services.AddSingleton<ICache, MemoryCache>();   // 1 instance cho cả app
builder.Services.AddScoped<IProductService, ProductService>();  // 1 instance / 1 request
builder.Services.AddTransient<IEmailSender, EmailSender>();     // instance mới mỗi lần inject
```

| Lifetime | Sống bao lâu | Dùng cho |
|---|---|---|
| `Singleton` | Toàn bộ đời app | Cache, config, HttpClient factory |
| `Scoped` | Một HTTP request | **DbContext, service, repository** (mặc định nên chọn) |
| `Transient` | Mỗi lần yêu cầu | Object nhẹ, không giữ state |

⚠️ Không inject `Scoped` vào `Singleton` — sẽ throw lỗi lúc chạy (captive dependency).

### Controller

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]                       // bật auto model validation + binding thông minh
[Route("api/[controller]")]           // [controller] => "products"
public class ProductsController : ControllerBase   // ControllerBase, không phải Controller
{
    private readonly IProductService _service;

    // Constructor injection
    public ProductsController(IProductService service) => _service = service;

    // GET /api/products?page=1&pageSize=20
    [HttpGet]
    public async Task<ActionResult<List<ProductDto>>> GetAll(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        var items = await _service.GetAllAsync(page, pageSize);
        return Ok(items);
    }

    // GET /api/products/5
    [HttpGet("{id:int}")]                        // :int là route constraint
    public async Task<ActionResult<ProductDto>> GetById(int id)
    {
        var product = await _service.GetByIdAsync(id);
        return product is null ? NotFound() : Ok(product);
    }

    // POST /api/products
    [HttpPost]
    public async Task<ActionResult<ProductDto>> Create([FromBody] CreateProductRequest req)
    {
        var created = await _service.CreateAsync(req);
        // 201 + header Location: /api/products/{id}
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    // PUT /api/products/5
    [HttpPut("{id:int}")]
    public async Task<IActionResult> Update(int id, UpdateProductRequest req)
    {
        var ok = await _service.UpdateAsync(id, req);
        return ok ? NoContent() : NotFound();
    }

    // DELETE /api/products/5
    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
    {
        var ok = await _service.DeleteAsync(id);
        return ok ? NoContent() : NotFound();
    }
}
```

**Model binding** — ASP.NET Core lấy dữ liệu từ đâu:

| Attribute | Nguồn | Ví dụ |
|---|---|---|
| (không có) | Route hoặc query, tự suy luận | `GetById(int id)` |
| `[FromRoute]` | Đường dẫn URL | `/products/5` |
| `[FromQuery]` | Query string | `?page=1` |
| `[FromBody]` | JSON body | `{"name":"..."}` |
| `[FromHeader]` | HTTP header | `X-Request-Id` |
| `[FromForm]` | Form data / upload file | `IFormFile` |

**Các helper trả response** của `ControllerBase`: `Ok()`, `Created()`, `CreatedAtAction()`,
`NoContent()`, `BadRequest()`, `NotFound()`, `Unauthorized()`, `Forbid()`, `Conflict()`,
`Problem()`, `StatusCode(int)`.

### Minimal API (cách viết ngắn, không cần controller)

```csharp
var group = app.MapGroup("/api/products").WithTags("Products");

group.MapGet("/", async (AppDbContext db) =>
    await db.Products.Select(p => new ProductDto(p.Id, p.Name, p.Price)).ToListAsync());

group.MapGet("/{id:int}", async (int id, AppDbContext db) =>
    await db.Products.FindAsync(id) is Product p
        ? Results.Ok(new ProductDto(p.Id, p.Name, p.Price))
        : Results.NotFound());

group.MapPost("/", async (CreateProductRequest req, AppDbContext db) =>
{
    var p = new Product { Name = req.Name, Price = req.Price };
    db.Products.Add(p);
    await db.SaveChangesAsync();
    return Results.Created($"/api/products/{p.Id}", new ProductDto(p.Id, p.Name, p.Price));
});
```

Chọn cái nào? Minimal API cho service nhỏ / vài endpoint. Controller cho project lớn,
nhiều endpoint, cần filter/attribute và tổ chức rõ ràng. Kiến thức DI, middleware, binding
là **giống nhau** cho cả hai.

### Validation

Cách 1 — Data Annotations (đủ dùng cho hầu hết trường hợp):

```csharp
using System.ComponentModel.DataAnnotations;

public record CreateProductRequest(
    [Required, StringLength(200, MinimumLength = 2)] string Name,
    [Range(0, 1_000_000_000)] decimal Price,
    [EmailAddress] string? ContactEmail);
```

Với `[ApiController]`, nếu validation fail ASP.NET Core **tự động** trả 400 kèm chi tiết lỗi
(chuẩn `ProblemDetails`) — bạn không cần viết `if (!ModelState.IsValid)`.

Cách 2 — FluentValidation, khi rule phức tạp:

```csharp
// dotnet add package FluentValidation.AspNetCore
public class CreateProductValidator : AbstractValidator<CreateProductRequest>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(200);
        RuleFor(x => x.Price).GreaterThan(0);
    }
}
```

### Xử lý lỗi tập trung

Đừng bọc `try/catch` ở mọi action. Dùng một middleware / exception handler chung:

```csharp
// .NET 8: IExceptionHandler
public class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext ctx, Exception ex, CancellationToken ct)
    {
        logger.LogError(ex, "Unhandled exception");

        var (status, title) = ex switch
        {
            KeyNotFoundException => (StatusCodes.Status404NotFound, "Không tìm thấy"),
            ArgumentException    => (StatusCodes.Status400BadRequest, "Dữ liệu không hợp lệ"),
            _                    => (StatusCodes.Status500InternalServerError, "Lỗi hệ thống")
        };

        ctx.Response.StatusCode = status;
        await ctx.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Status = status, Title = title, Detail = ex.Message
        }, ct);
        return true;
    }
}

// Program.cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
app.UseExceptionHandler();
```

⚠️ Đừng trả `ex.StackTrace` ra production — lộ thông tin nội bộ.

### Configuration & Secrets

`appsettings.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyApiDb;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Jwt": { "Issuer": "MyApi", "ExpiresInMinutes": 60 }
}
```

Đọc config bằng Options pattern:

```csharp
public class JwtOptions { public string Issuer { get; set; } = ""; public int ExpiresInMinutes { get; set; } }

builder.Services.Configure<JwtOptions>(builder.Configuration.GetSection("Jwt"));

// Inject: IOptions<JwtOptions> options  →  options.Value.Issuer
```

⚠️ **Không commit password/secret vào git.** Dùng User Secrets khi dev:

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "Server=...;Password=..."
```

Trên server thật: dùng biến môi trường hoặc Azure Key Vault / AWS Secrets Manager.

### Logging

```csharp
public class ProductService(ILogger<ProductService> logger) : IProductService
{
    public async Task<Product?> GetByIdAsync(int id)
    {
        logger.LogInformation("Lấy product {ProductId}", id);   // structured logging
        // KHÔNG dùng $"Lấy product {id}" — mất khả năng query theo field
        ...
    }
}
```

Mức độ: `Trace` < `Debug` < `Information` < `Warning` < `Error` < `Critical`.

### Authentication với JWT (khái quát)

```csharp
// dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opt =>
    {
        opt.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });
builder.Services.AddAuthorization();
```

Rồi bảo vệ endpoint:

```csharp
[Authorize]                            // bắt buộc đăng nhập
[Authorize(Roles = "Admin")]           // bắt buộc role
[AllowAnonymous]                       // mở public dù controller có [Authorize]
```

Lưu mật khẩu: **luôn** hash (BCrypt / ASP.NET Core Identity `PasswordHasher`), không bao giờ lưu plaintext.

### Test API

- **Swagger UI**: `dotnet run` rồi mở `/swagger` — click thử được ngay.
- **File `.http`** (VS/VS Code hỗ trợ sẵn):

```http
### Lấy danh sách
GET https://localhost:7001/api/products

### Tạo mới
POST https://localhost:7001/api/products
Content-Type: application/json

{ "name": "Bàn phím cơ", "price": 1500000 }
```

- **Postman** hoặc `curl` cho trường hợp phức tạp.

---

## EF Core

**ORM** (Object-Relational Mapper): bạn viết C# (LINQ), EF Core dịch thành SQL và map kết quả
về object — không phải viết SQL tay và map từng column.

### Cài package

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer     # hoặc .Npgsql / .Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-ef                          # CLI để tạo migration
```

### Entity — class map với bảng

```csharp
public class Product
{
    public int Id { get; set; }                  // tên "Id" => tự nhận làm primary key
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;

    // Foreign key + navigation property (quan hệ nhiều-1)
    public int CategoryId { get; set; }
    public Category? Category { get; set; }
}

public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Navigation property (quan hệ 1-nhiều)
    public List<Product> Products { get; set; } = [];
}
```

### DbContext — cầu nối tới database

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();      // mỗi DbSet = 1 bảng
    public DbSet<Category> Categories => Set<Category>();

    // Fluent API: cấu hình chi tiết hơn attribute
    protected override void OnModelCreating(ModelBuilder mb)
    {
        mb.Entity<Product>(e =>
        {
            e.HasKey(p => p.Id);
            e.Property(p => p.Name).IsRequired().HasMaxLength(200);
            e.Property(p => p.Price).HasPrecision(18, 2);   // decimal(18,2)
            e.HasIndex(p => p.Name);

            e.HasOne(p => p.Category)
             .WithMany(c => c.Products)
             .HasForeignKey(p => p.CategoryId)
             .OnDelete(DeleteBehavior.Restrict);            // chặn xoá category còn product
        });

        // Seed data
        mb.Entity<Category>().HasData(new Category { Id = 1, Name = "Điện tử" });
    }
}
```

### Migration — quản lý thay đổi schema

```bash
dotnet ef migrations add InitialCreate     # sinh code mô tả thay đổi schema
dotnet ef database update                  # áp dụng lên database thật
dotnet ef migrations remove                # bỏ migration cuối (chưa apply)
dotnet ef migrations list
dotnet ef database update PreviousMigration # rollback về migration trước
```

Migration được commit vào git — cả team và server production dùng chung.

### CRUD

```csharp
public class ProductService(AppDbContext db) : IProductService
{
    // ===== READ =====
    public async Task<List<ProductDto>> GetAllAsync(int page, int pageSize) =>
        await db.Products
            .AsNoTracking()                         // read-only => nhanh hơn, ít RAM
            .Where(p => p.Stock > 0)
            .OrderByDescending(p => p.CreatedAt)
            .Skip((page - 1) * pageSize).Take(pageSize)   // phân trang
            .Select(p => new ProductDto(p.Id, p.Name, p.Price))  // projection: chỉ SELECT cột cần
            .ToListAsync();

    public async Task<Product?> GetByIdAsync(int id) =>
        await db.Products
            .Include(p => p.Category)               // JOIN để load navigation property
            .FirstOrDefaultAsync(p => p.Id == id);

    // ===== CREATE =====
    public async Task<Product> CreateAsync(CreateProductRequest req)
    {
        var p = new Product { Name = req.Name, Price = req.Price, CategoryId = req.CategoryId };
        db.Products.Add(p);
        await db.SaveChangesAsync();                // đến đây mới thực sự INSERT
        return p;                                   // p.Id đã được DB gán
    }

    // ===== UPDATE =====
    public async Task<bool> UpdateAsync(int id, UpdateProductRequest req)
    {
        var p = await db.Products.FindAsync(id);
        if (p is null) return false;

        p.Name = req.Name;                          // EF theo dõi thay đổi (change tracking)
        p.Price = req.Price;
        await db.SaveChangesAsync();                // sinh UPDATE chỉ cho cột đã đổi
        return true;
    }

    // ===== DELETE =====
    public async Task<bool> DeleteAsync(int id)
    {
        var p = await db.Products.FindAsync(id);
        if (p is null) return false;

        db.Products.Remove(p);
        await db.SaveChangesAsync();
        return true;
    }
}
```

### Những điều cần nhớ về EF Core

**Deferred execution** — query chỉ chạy khi bạn gọi `ToListAsync()`, `FirstOrDefaultAsync()`,
`CountAsync()`, `AnyAsync()`… Trước đó nó chỉ là "công thức", nên bạn có thể build query động:

```csharp
var query = db.Products.AsNoTracking();
if (!string.IsNullOrWhiteSpace(keyword))
    query = query.Where(p => p.Name.Contains(keyword));
if (minPrice.HasValue)
    query = query.Where(p => p.Price >= minPrice);

var total = await query.CountAsync();               // SQL #1
var items = await query.Skip(0).Take(20).ToListAsync();  // SQL #2
```

**Bẫy N+1** — lỗi performance phổ biến nhất:

```csharp
// ❌ SAI: 1 query lấy products + N query lấy từng category
var products = await db.Products.ToListAsync();
foreach (var p in products) Console.WriteLine(p.Category!.Name);   // mỗi vòng 1 query

// ✅ ĐÚNG: 1 query duy nhất
var products = await db.Products.Include(p => p.Category).ToListAsync();
```

**Đừng `ToList()` quá sớm** — sẽ kéo cả bảng về RAM rồi mới lọc:

```csharp
// ❌ tải hết bảng về app rồi lọc trong C#
var rẻ = (await db.Products.ToListAsync()).Where(p => p.Price < 100_000);

// ✅ lọc ở database
var rẻ = await db.Products.Where(p => p.Price < 100_000).ToListAsync();
```

**Raw SQL** khi LINQ không diễn đạt được (luôn dùng tham số để tránh SQL injection):

```csharp
var list = await db.Products
    .FromSqlInterpolated($"SELECT * FROM Products WHERE Price > {min}")
    .ToListAsync();
```

**Transaction** khi cần nhiều thao tác thành công/thất bại cùng nhau:

```csharp
await using var tx = await db.Database.BeginTransactionAsync();
try
{
    db.Orders.Add(order);
    await db.SaveChangesAsync();
    product.Stock -= qty;
    await db.SaveChangesAsync();
    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

*Lưu ý:* một lần `SaveChangesAsync()` đã tự nằm trong transaction; chỉ cần transaction tường minh
khi có **nhiều** lần `SaveChanges` phải nguyên tử với nhau.

---

## Database

### Khái niệm cần biết

| Khái niệm | Giải thích |
|---|---|
| **Table / Row / Column** | Bảng / dòng (một record) / cột (một field) |
| **Primary key (PK)** | Cột định danh duy nhất mỗi dòng — thường `Id` tự tăng |
| **Foreign key (FK)** | Cột trỏ tới PK bảng khác, tạo quan hệ |
| **Index** | "Mục lục" giúp WHERE/JOIN/ORDER BY nhanh hơn (đánh đổi: ghi chậm hơn, tốn disk) |
| **Unique constraint** | Đảm bảo không trùng (ví dụ email user) |
| **Transaction** | Nhóm lệnh: tất cả thành công hoặc tất cả rollback (ACID) |
| **Normalization** | Tách bảng để không lặp dữ liệu |
| **Nullable** | Cột cho phép NULL hay không |

### Quan hệ

| Loại | Ví dụ | Cách làm |
|---|---|---|
| One-to-Many | 1 Category có nhiều Product | FK `CategoryId` ở bảng Product |
| One-to-One | 1 User có 1 Profile | FK + unique constraint |
| Many-to-Many | Product ↔ Tag | Bảng trung gian `ProductTags(ProductId, TagId)` |

Many-to-many trong EF Core 8 — chỉ cần khai báo hai `List<>`, EF tự tạo bảng nối:

```csharp
public class Product { public List<Tag> Tags { get; set; } = []; }
public class Tag     { public List<Product> Products { get; set; } = []; }
```

### SQL cơ bản (nên biết dù dùng EF Core)

```sql
-- Truy vấn
SELECT Id, Name, Price FROM Products
WHERE Price BETWEEN 100000 AND 500000 AND Name LIKE '%phím%'
ORDER BY Price DESC
OFFSET 0 ROWS FETCH NEXT 20 ROWS ONLY;      -- phân trang (SQL Server)

-- JOIN
SELECT p.Name, c.Name AS Category
FROM Products p
INNER JOIN Categories c ON p.CategoryId = c.Id;   -- LEFT JOIN nếu muốn giữ product không có category

-- Nhóm và tổng hợp
SELECT CategoryId, COUNT(*) AS Total, AVG(Price) AS AvgPrice
FROM Products GROUP BY CategoryId HAVING COUNT(*) > 5;

-- Thay đổi dữ liệu
INSERT INTO Products (Name, Price, CategoryId) VALUES ('Chuột', 250000, 1);
UPDATE Products SET Price = 300000 WHERE Id = 1;
DELETE FROM Products WHERE Id = 1;

-- Schema
CREATE TABLE Categories (
    Id   INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL
);
CREATE INDEX IX_Products_Name ON Products(Name);
```

### Chọn database cho project học

| DB | Connection string mẫu | Ghi chú |
|---|---|---|
| **SQLite** | `Data Source=app.db` | Nhẹ nhất, 1 file, không cần cài server → **nên dùng khi mới học** |
| SQL Server LocalDB | `Server=(localdb)\\mssqllocaldb;Database=MyApiDb;Trusted_Connection=True` | Có sẵn với Visual Studio |
| PostgreSQL | `Host=localhost;Database=myapi;Username=postgres;Password=xxx` | Phổ biến nhất cho production |

### Quy ước tốt

- Tên bảng số nhiều (`Products`), tên cột PascalCase (`CreatedAt`).
- Lưu thời gian ở **UTC** (`DateTime.UtcNow` hoặc `DateTimeOffset`), format khi trả về client.
- Tiền dùng `decimal(18,2)`, **không** dùng `float/double`.
- Chuỗi có tiếng Việt: `NVARCHAR` (Unicode), không phải `VARCHAR`.
- Đặt index cho cột hay dùng trong `WHERE` / `JOIN` / `ORDER BY`.
- Cân nhắc *soft delete* (`IsDeleted bit`) thay vì xoá thật, nếu dữ liệu cần audit.

---

## Thực hành: build API "Product" từ 0

Làm theo đúng thứ tự này là bạn có một API chạy được.

**1. Tạo project**

```bash
dotnet new webapi -n ProductApi --use-controllers
cd ProductApi
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
```

**2. Entity** — `Models/Product.cs`

```csharp
namespace ProductApi.Models;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

**3. DbContext** — `Data/AppDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using ProductApi.Models;

namespace ProductApi.Data;

public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder mb)
    {
        mb.Entity<Product>(e =>
        {
            e.Property(p => p.Name).IsRequired().HasMaxLength(200);
            e.Property(p => p.Price).HasPrecision(18, 2);
        });
    }
}
```

**4. DTO** — `DTOs/ProductDtos.cs`

```csharp
using System.ComponentModel.DataAnnotations;

namespace ProductApi.DTOs;

public record ProductDto(int Id, string Name, decimal Price, int Stock);

public record CreateProductRequest(
    [Required, StringLength(200, MinimumLength = 2)] string Name,
    [Range(0.01, 1_000_000_000)] decimal Price,
    [Range(0, int.MaxValue)] int Stock);

public record UpdateProductRequest(
    [Required, StringLength(200, MinimumLength = 2)] string Name,
    [Range(0.01, 1_000_000_000)] decimal Price,
    [Range(0, int.MaxValue)] int Stock);
```

**5. Service** — `Services/ProductService.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using ProductApi.Data;
using ProductApi.DTOs;
using ProductApi.Models;

namespace ProductApi.Services;

public interface IProductService
{
    Task<List<ProductDto>> GetAllAsync(int page, int pageSize);
    Task<ProductDto?> GetByIdAsync(int id);
    Task<ProductDto> CreateAsync(CreateProductRequest req);
    Task<bool> UpdateAsync(int id, UpdateProductRequest req);
    Task<bool> DeleteAsync(int id);
}

public class ProductService(AppDbContext db) : IProductService
{
    private static ProductDto ToDto(Product p) => new(p.Id, p.Name, p.Price, p.Stock);

    public Task<List<ProductDto>> GetAllAsync(int page, int pageSize) =>
        db.Products.AsNoTracking()
          .OrderByDescending(p => p.CreatedAt)
          .Skip((page - 1) * pageSize).Take(pageSize)
          .Select(p => new ProductDto(p.Id, p.Name, p.Price, p.Stock))
          .ToListAsync();

    public async Task<ProductDto?> GetByIdAsync(int id)
    {
        var p = await db.Products.AsNoTracking().FirstOrDefaultAsync(x => x.Id == id);
        return p is null ? null : ToDto(p);
    }

    public async Task<ProductDto> CreateAsync(CreateProductRequest req)
    {
        var p = new Product { Name = req.Name, Price = req.Price, Stock = req.Stock };
        db.Products.Add(p);
        await db.SaveChangesAsync();
        return ToDto(p);
    }

    public async Task<bool> UpdateAsync(int id, UpdateProductRequest req)
    {
        var p = await db.Products.FindAsync(id);
        if (p is null) return false;

        p.Name = req.Name;
        p.Price = req.Price;
        p.Stock = req.Stock;
        await db.SaveChangesAsync();
        return true;
    }

    public async Task<bool> DeleteAsync(int id)
    {
        var p = await db.Products.FindAsync(id);
        if (p is null) return false;

        db.Products.Remove(p);
        await db.SaveChangesAsync();
        return true;
    }
}
```

**6. Controller** — `Controllers/ProductsController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using ProductApi.DTOs;
using ProductApi.Services;

namespace ProductApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ProductsController(IProductService service) : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<List<ProductDto>>> GetAll(int page = 1, int pageSize = 20)
        => Ok(await service.GetAllAsync(page, pageSize));

    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDto>> GetById(int id)
        => await service.GetByIdAsync(id) is { } dto ? Ok(dto) : NotFound();

    [HttpPost]
    public async Task<ActionResult<ProductDto>> Create(CreateProductRequest req)
    {
        var created = await service.CreateAsync(req);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpPut("{id:int}")]
    public async Task<IActionResult> Update(int id, UpdateProductRequest req)
        => await service.UpdateAsync(id, req) ? NoContent() : NotFound();

    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
        => await service.DeleteAsync(id) ? NoContent() : NotFound();
}
```

**7. Program.cs**

```csharp
using Microsoft.EntityFrameworkCore;
using ProductApi.Data;
using ProductApi.Services;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(o => o.UseSqlite("Data Source=app.db"));
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.MapControllers();
app.Run();
```

**8. Tạo database & chạy**

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet run
```

Mở `https://localhost:<port>/swagger` → thử POST tạo product → GET xem danh sách.

**9. Bài tập mở rộng** (làm để thật sự nắm được)

1. Thêm entity `Category` và quan hệ one-to-many với `Product`.
2. Thêm search + filter: `GET /api/products?keyword=phím&minPrice=100000`.
3. Trả về kết quả phân trang có metadata: `{ items, page, pageSize, totalCount, totalPages }`.
4. Thêm global exception handler (mục "Xử lý lỗi tập trung").
5. Thêm JWT auth: `POST /api/auth/login`, và `[Authorize]` cho POST/PUT/DELETE.
6. Viết unit test cho `ProductService` bằng xUnit + EF Core InMemory provider.
7. Viết lại toàn bộ endpoint bằng Minimal API để so sánh hai cách.

---

## Checklist tự đánh giá

Bạn đã nắm được nếu trả lời được:

- [ ] `Task`, `async`, `await` khác nhau ở đâu? Vì sao không dùng `.Result`?
- [ ] `?.` và `??` xử lý null như thế nào?
- [ ] Khi nào dùng `record` thay `class`?
- [ ] Vì sao DTO phải tách khỏi Entity?
- [ ] `Singleton` / `Scoped` / `Transient` khác nhau ra sao? DbContext dùng cái nào?
- [ ] Thứ tự middleware quan trọng ở chỗ nào?
- [ ] GET/POST/PUT/DELETE trả status code nào? 401 khác 403 thế nào?
- [ ] Vấn đề N+1 là gì và fix bằng gì?
- [ ] `AsNoTracking()` dùng khi nào và vì sao nhanh hơn?
- [ ] Migration để làm gì? Lệnh nào tạo, lệnh nào apply?

## Tài liệu tham khảo

- C# docs: https://learn.microsoft.com/dotnet/csharp/
- ASP.NET Core: https://learn.microsoft.com/aspnet/core/
- EF Core: https://learn.microsoft.com/ef/core/
- Tutorial chính thức (Web API + EF Core): https://learn.microsoft.com/aspnet/core/tutorials/first-web-api
- REST API best practices: https://learn.microsoft.com/azure/architecture/best-practices/api-design

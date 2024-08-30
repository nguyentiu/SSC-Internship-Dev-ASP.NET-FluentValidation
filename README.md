# SSC-Internship-Dev-ASP.NET-FluentValidation
# Hướng Dẫn Sử Dụng FluentValidation Trong ASP.NET Core
## Giới Thiệu
FluentValidation là một thư viện phổ biến trong .NET để thực hiện việc xác thực dữ liệu một cách dễ dàng và linh hoạt. Nó cung cấp một cách tiếp cận theo phong cách fluent API, giúp bạn viết các quy tắc xác thực một cách rõ ràng và dễ hiểu hơn so với cách xác thực truyền thống. Trong bài viết này, chúng ta sẽ tìm hiểu cách tích hợp FluentValidation vào ASP.NET Core, cách tạo và áp dụng các quy tắc xác thực, cũng như cách quản lý lỗi xác thực.

## 1. Cài Đặt FluentValidation
Để bắt đầu sử dụng FluentValidation trong dự án ASP.NET Core, bạn cần cài đặt gói NuGet `FluentValidation.AspNetCore`.

Mở terminal hoặc Package Manager Console trong Visual Studio và chạy lệnh sau:

```bash
dotnet add package FluentValidation.AspNetCore
```
Hoặc bạn có thể cài đặt qua NuGet Package Manager trong Visual Studio.

## 2. Tạo Model Và Validator
Giả sử bạn có một model đơn giản cho một sản phẩm (`Product`) như sau:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Description { get; set; }
}
```
Tiếp theo, bạn sẽ tạo một lớp `ProductValidator` kế thừa từ `AbstractValidator<Product>`, và định nghĩa các quy tắc xác thực cho model này:

```csharp
using FluentValidation;

public class ProductValidator : AbstractValidator<Product>
{
    public ProductValidator()
    {
        RuleFor(product => product.Name)
            .NotEmpty().WithMessage("Product name is required.")
            .Length(2, 100).WithMessage("Product name must be between 2 and 100 characters.");

        RuleFor(product => product.Price)
            .GreaterThan(0).WithMessage("Product price must be greater than zero.");

        RuleFor(product => product.Description)
            .MaximumLength(500).WithMessage("Product description cannot exceed 500 characters.");
    }
}
```
Trong ví dụ trên:

- `RuleFor(product => product.Name)`: Xác định rằng thuộc tính `Name` của `Product` phải thỏa mãn các quy tắc đã chỉ định, như không được rỗng và phải có độ dài từ 2 đến 100 ký tự.
- `RuleFor(product => product.Price)`: Giá của sản phẩm phải lớn hơn 0.
- `RuleFor(product => product.Description)`: Mô tả sản phẩm không được vượt quá 500 ký tự.
## 3. Tích Hợp FluentValidation Vào ASP.NET Core
Để tích hợp FluentValidation vào ASP.NET Core, bạn cần đăng ký các validators trong  `Startup` hoặc `Program.cs`.

Trong ASP.NET Core 6 trở lên, bạn có thể cấu hình trong `Program.cs` như sau:

```csharp
using FluentValidation;
using FluentValidation.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Đăng ký FluentValidation
builder.Services.AddValidatorsFromAssemblyContaining<ProductValidator>();
builder.Services.AddFluentValidationAutoValidation();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```
Trong đoạn mã trên:

- `AddValidatorsFromAssemblyContaining<ProductValidator>()`: Đăng ký tất cả các validators trong assembly chứa `ProductValidator`.
- `AddFluentValidationAutoValidation()`: Tự động xác thực các model dựa trên các validators được đăng ký.
## 4. Áp Dụng Xác Thực Trong Controller
Sau khi bạn đã cấu hình FluentValidation, các quy tắc xác thực sẽ tự động được áp dụng khi bạn gửi dữ liệu tới các endpoints trong controllers.

Giả sử bạn có một `ProductsController` như sau:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult CreateProduct([FromBody] Product product)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        // Logic tạo sản phẩm
        return Ok(product);
    }
}
```
Khi bạn gửi một yêu cầu POST với dữ liệu không hợp lệ, FluentValidation sẽ tự động xác thực dữ liệu và trả về lỗi nếu không đạt yêu cầu.

Ví dụ: Nếu bạn gửi một sản phẩm với tên rỗng hoặc giá trị giá bằng 0, yêu cầu sẽ bị từ chối và bạn sẽ nhận được phản hồi chứa các thông báo lỗi xác thực.

## 5. Xử Lý Lỗi Xác Thực Tuỳ Chỉnh
Bạn có thể tùy chỉnh cách hiển thị các lỗi xác thực bằng cách cấu hình trong `Startup` hoặc `Program.cs`.

```csharp
builder.Services.AddFluentValidationAutoValidation(config =>
{
    config.DisableDataAnnotationsValidation = true; // Tắt kiểm tra bằng DataAnnotations
});
```
Bạn cũng có thể tạo một middleware hoặc filter để xử lý lỗi xác thực một cách tùy chỉnh hơn, chẳng hạn như trả về một định dạng lỗi cụ thể mà bạn mong muốn.

## 6. Kết Hợp FluentValidation Với Dependency Injection
FluentValidation hoạt động tốt với Dependency Injection (DI) trong ASP.NET Core. Bạn có thể đăng ký các validators của mình như các dịch vụ scoped hoặc singleton, và sử dụng chúng trong các services khác.

```csharp
public class CustomService
{
    private readonly IValidator<Product> _productValidator;

    public CustomService(IValidator<Product> productValidator)
    {
        _productValidator = productValidator;
    }

    public void ValidateProduct(Product product)
    {
        var result = _productValidator.Validate(product);
        if (!result.IsValid)
        {
            // Xử lý lỗi
            throw new ValidationException(result.Errors);
        }
    }
}
```
## 7. Tổng kết
FluentValidation là một công cụ mạnh mẽ và linh hoạt cho việc xác thực dữ liệu trong ASP.NET Core. Nó cho phép bạn tách biệt logic xác thực khỏi các lớp model và controllers, đồng thời cung cấp một cách tiếp cận rõ ràng và dễ bảo trì. Bằng cách sử dụng FluentValidation, bạn có thể viết mã xác thực dễ hiểu và có khả năng tái sử dụng cao.

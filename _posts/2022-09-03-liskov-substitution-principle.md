---
layout: post
title: SOLID - Liskov Substitution Principle
date: 2022-09-03 16:01 +0700
categories: [Design Pattern, SOLID]
tags: [solid, liskov substitution principle, udemy, design patterns in c# and .net]
---

## 1. Đặt vấn đề

Để làm rõ nguyên lý này, chúng ta sẽ đến với ví dụ về bài toán hình học. Giả sử
chúng ta có đối tượng hình chữ nhật như dưới đây

```csharp
public class Rectangle
{
  public int Width { get; set; }
  public int Height { get; set; }

  public Rectangle()
  {

  }

  public Rectangle(int width, int height)
  {
    Width = width;
    Height = height;
  }

  public override string ToString()
  {
    return $"Width: {Width} - Height: {Height}";
  }
}
```

```csharp
public class Demo
{
  static public int Area(Rectangle r) => r.Width * r.Height;

  static void Main([]string args)
  {
    Rectangle rc = new Rectangle(2, 3);
    WriteLine($"{rc} has area {Area(rc)}");
  }
}
```

Sau khi đối tượng `Rectangle` đã hoạt động tốt, chúng ta quyết định
thực hiện điều tương tự với đối tượng `Square` (hình vuông). Và kế thừa
là cách nhanh nhất để làm điều này.

Hình vuông có `width` và `height` giống nhau, do đó ta quyết định sẽ
ghi đè phương thức `setter` cho 2 thuộc tính này.

```csharp
public class Square : Rectangle
{
  public new int Width
  {
    set { base.Width = base.Height = value; }
  }

  public new int Height
  {
    set { base.Height = base.Width = value; }
  }
}
```

Việc thay đổi này hoàn toàn hoạt động giống như cách chúng ta mong muốn.

```csharp
public class Demo
{
  static public int Area(Rectangle r) => r.Width * r.Height;

  static void Main([]string args)
  {
    Rectangle rc = new Rectangle(2, 3);
    WriteLine($"{rc} has area {Area(rc)}");

    Square sq = new Square();
    sq.Width = 4;
    WriteLine($"{sq} has area {Area(sq)}");
  }
}
```

## 2. Vi phạm nguyên lý

Ta có thể thấy rõ rằng, `Square` kế thừa từ `Rectangle`, như vậy các đối tượng
thuộc lớp `Square` đồng thời cũng thuộc lớp `Rectangle`.

Vậy nếu trong ví dụ trên, ta thay đổi `Square` thành `Rectangle` thì liệu
rằng chương trình còn hoạt động như chúng ta mong muốn hay không?

```csharp
public class Demo
{
  static public int Area(Rectangle r) => r.Width * r.Height;

  static void Main([]string args)
  {
    Rectangle rc = new Rectangle(2, 3);
    WriteLine($"{rc} has area {Area(rc)}");

    Rectangle sq = new Square();
    sq.Width = 4;
    WriteLine($"{sq} has area {Area(sq)}");
  }
}
```

Lúc này, diện tích của hình vuông bên dưới không còn là `16` nữa mà là `0`. Lý do
bởi vì lúc này `sq` đang thuộc kiểu `Rectangle`, khi đặt giá trị cho thuộc tính
`Width`, chỉ có duy nhất `Width` thay đổi. Thuộc tính `Height` mặc định nang
giá trị `0`.

Điều này vi phạm nguyên lí `Liskov Substitution`, nguyên lí này chỉ ra rằng chúng ta luôn
có thể chuyển đổi đối tượng từ lớp kế thừa về lớp gốc mà các thao tác chức năng vẫn hoạt
động như thông thường.

Áp dụng vào ví dụ trên, khi đối tượng thuộc lớp `Square` được ép về lớp `Rectangle`, thao
tác tính diện tích vẫn phải hoạt động bình thường.

## 3. Cách giải quyết

Cách thức giải quyết của ví dụ bên trên khá đơn giản. Thay vì dùng từ khoá `new` để làm
mới phương thức `setter`, ta có thể kết hợp `virtual` và `override`.

```csharp
public class Rectangle
{
  public virtual int Width { get; set; }
  public virtual int Height { get; set; }

  public Rectangle()
  {

  }

  public Rectangle(int width, int height)
  {
    Width = width;
    Height = height;
  }

  public override string ToString()
  {
    return $"Width: {Width} - Height: {Height}";
  }
}
```

```csharp
public class Square : Rectangle
{
  public override int Width
  {
    set { base.Width = base.Height = value; }
  }

  public override int Height
  {
    set { base.Height = base.Width = value; }
  }
}
```

Khi chúng ta đặt `Width` cho hình vuông `sq` (lúc này đang được ép kiểu về `Rectangle`),
hệ thống sẽ nhận thấy thuộc `Width` trên `Rectangle` là `virtual`, do đó nó sẽ xem
xét trên `Square` có `override` lại thuộc tính này hay không. Và do chúng ta đã `override`
nên hệ thống sẽ `set` giá trị dựa trên thuộc tính `Width` được khai báo ở `Square` (
  đồng thời đặt giá trị cho `Height`
)

```csharp
public class Demo
{
  static public int Area(Rectangle r) => r.Width * r.Height;

  static void Main([]string args)
  {
    Rectangle rc = new Rectangle(2, 3);
    WriteLine($"{rc} has area {Area(rc)}");

    Rectangle sq = new Square();
    sq.Width = 4;
    WriteLine($"{sq} has area {Area(sq)}");
  }
}
```

## Tham khảo

- [Design Patterns in C# and .NET](https://www.udemy.com/course/design-patterns-csharp-dotnet/)

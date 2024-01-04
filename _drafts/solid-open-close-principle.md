---
title: SOLID - Open Close Principle
categories:
- Design Pattern
- SOLID
tags:
- solid
- open close principle
- udemy
- design patterns in c# and .net
---
Đây là nguyên lí thứ 2 trong bộ 5 nguyên lý SOLID. Ý tưởng của nguyên lý này chỉ rằng:
1 đối tượng của phần mềm nên được mở rộng để thêm mới nhưng đóng lại khi thay đổi. Tức là,
những thứ như `class`, `function`,... nên được tạo theo 1 cách thức mà những `logic` của
nó có thể được mở rộng ra mà không cần phải thay đổi những đoạn mã nguồn ban đầu.

<!--more-->

## 1. Đặt vấn đề

Giả sử chúng ta có 1 trang buôn bán sản phẩm. Các sản phẩm có các thuộc tính như

- Màu sắc
- Kích thước

Người dùng có thể dùng tính năng `filter` để lọc danh sách sản phẩm, từ đó tìm
ra được các sản phẩm ưng ý để thêm vào giỏ hàng.

Trước hết, chúng ta sẽ xây dựng cấu trúc `Product` như dưới đây

```csharp
public enum Color
{
  Red, Green, Blue
}

public enum Size
{
  Small, Medium, Large, Yuge
}

public class Product
{
  public string Name;
  public Color Color;
  public Size Size;

  public Product(string name, Color color, Size size)
  {
    Name = name;
    Color = color;
    Size = size;
  }
}
```

Để cung cấp tính năng lọc sản phẩm, ta tiến hành tạo `class` `ProductFilter`
và viết các phương thức lọc:

- `FilterBySize`
- `FilterByColor`

```csharp
public class ProductFilter
{
  public IEnumerable<Product> FilterBySize(IEnumerable<Product> products, Size size)
  {
    foreach (var p in products)
      if (p.Size == size)
        yield return p;
  }

  public IEnumerable<Product> FilterByColor(IEnumerable<Product> products, Color color)
  {
    foreach (var p in products)
      if (p.Color == color)
        yield return p;
  }
}
```

Khi thực thi chương trình dưới đây, chúng ta sẽ lấy được danh sách các sản phẩm
có màu xanh trong hệ thống.

```csharp
public class Demo
{
  static void Main(string[] args)
  {
    var apple = new Product("Apple", Color.Green, Size.Small);
    var tree = new Product("Tree", Color.Green, Size.Large);
    var house = new Product("House", Color.Blue, Size.Large);

    Product[] products = {apple, tree, house};

    var pf = new ProductFilter();
    WriteLine("Green products (old):");
    foreach(v p in pf.FilterByColor(products, Color.Green))
    {
      WriteLine($" - {p.Name} is green");
    }
  }
}
```

## 2. Vi phạm nguyên lý

Lúc này chúng ta đã có hệ thống lọc sản phẩm theo 2 phương pháp:

- Lọc theo màu
- Lọc theo kích thước

2 phương thức lọc này đều do `class` `ProductFilter` cung cấp. Tuy nhiên, giả sử lúc này khác hàng
yêu cầu hệ thống phải cung cấp chức năng lọc theo cả màu sắc lẫn kích thước. Chúng ta sẽ phải quay lại
`class` `ProductFilter` để chỉnh sửa, thêm vào 1 phương thức mới. Và cứ tiếp diễn như vậy mỗi lần
khách hàng yêu cầu chức năng lọc mới.

```csharp
public class ProductFilter
{
  public IEnumerable<Product> FilterBySizeAndColor(IEnumerable<Product> products, Size size, Color color)
  {
    foreach (var p in products)
      if (p.Size == size && p.Color == color)
        yield return p;
  }
}
```

Và điều này đã vi phạm nguyên lý `Open Close`, ý tưởng của nguyên lý này chỉ rằng 1 `class` nên cung cấp
khả năng để mở rộng, nhưng không nên chỉnh sửa từ trong chính bản thân `class`.

Khi áp dụng vào `class` `ProductFilter`, ta có thể diễn giải như sau. `class` `ProductFilter` nên
cung cấp khả năng để thêm vào các bộ lọc mới. Đồng thời, không nên chỉnh sửa trực tiếp bên trong `class`
này, tức là không một ai quay lại `class` `ProductFilter` để chỉnh sửa `class` này và thêm vào các
phương thức lọc mới (như cách chúng ta làm với phương thức `FilterBySizeAndColor`). Lý do đưa ra của việc
làm này đó là sản phẩm đã được cung cấp cho khách hàng, chúng ta không thể truy cập vào để sửa chữa
trực tiếp được.

## 3. Cách giải quyết

Cách giải quyết đơn giản nhất cho vấn đề này đó là áp dụng tính chất kế thừa. Bên cạnh đó, chúng
ta có thể sử dụng `interface` và 1 `Enterprise Pattern` tên là `Specification Pattern`. Sử dụng `pattern`
này có thể giúp chúng ta tránh được việc vi phạm nguyên lý `Open Close`.

Để thực hiện `pattern` này, ta tiến hành định nghĩa 1 số `interface`

```csharp
public interface ISpecification<T>
{
  bool IsSatisfied(T t);
}

public interface IFilter<T>
{
  IEnumerable<T> Filter(IEnumerable<T> items, ISpecification<T> spec);
}
```

Lúc này, để cung cấp tính năng lọc, ta định nghĩa từng `Specification` tương ứng

```csharp
public class ColorSpecification : ISpecification<Product>
{
  private Color color;

  public ColorSpecification(Color color)
  {
    this.color = color;
  }

  public bool IsSatisfied(Product t)
  {
    return t.color == color;
  }
}

public class SizeSpecification : ISpecification<Product>
{
  private Size Size;

  public SizeSpecification(Size Size)
  {
    this.Size = Size;
  }

  public bool IsSatisfied(Product t)
  {
    return t.Size == Size;
  }
}
```

Tiếp đó, định nghĩa `class` `BetterFilter` thực thi `IFilter` đảm nhận chức năng lọc.

```csharp
public class BetterFilter : IFilter<Product>
{
  public IEnumerable<Product> Filter(IEnumerable<Product> products, ISpecification<Product> spec)
  {
    foreach (var p in products)
      if (spec.IsSatisfied(p))
        yield return p;
  }
}
```

Khi đó, ta có thể dùng chương trình dưới đây để lọc ra các sản phẩm có màu xanh

```csharp
public class Demo
{
  static void Main(string[] args)
  {
    var apple = new Product("Apple", Color.Green, Size.Small);
    var tree = new Product("Tree", Color.Green, Size.Large);
    var house = new Product("House", Color.Blue, Size.Large);

    Product[] products = {apple, tree, house};

    var pf = new ProductFilter();
    WriteLine("Green products (old):");
    foreach(var p in pf.FilterByColor(products, Color.Green))
    {
      WriteLine($" - {p.Name} is green");
    }

    var bf = new BetterFilter();
    var colorSpec = new ColorSpecification(Color.Green);
    WriteLine("Green products (new):");
    foreach(var p in bf.Filter(products, colorSpec))
    {
      WriteLine($" - {p.Name} is green");
    }
  }
}
```

Khi cần lọc sản phẩm theo nhiều điều kiện (cả màu sắc và kích thước), ta có thể mở rộng
`filter` thông qua `combinator`, cách thức thực hiện như dưới đây

```csharp
public class AndSpecification<T> : ISpecification<T>
{
  private ISpecification<T> first, second;

  public AndSpecification(ISpecification<T> first, ISpecification<T> second)
  {
    this.first = first;
    this.second = second;
  }

  public bool IsSatisfied(T t)
  {
    return first.IsSatisfied(t) && second.IsSatisfied(t);
  }
}
```

```csharp
public class Demo
{
  static void Main(string[] args)
  {
    var apple = new Product("Apple", Color.Green, Size.Small);
    var tree = new Product("Tree", Color.Green, Size.Large);
    var house = new Product("House", Color.Blue, Size.Large);

    Product[] products = {apple, tree, house};

    var pf = new ProductFilter();
    WriteLine("Green products (old):");
    foreach(var p in pf.FilterByColor(products, Color.Green))
    {
      WriteLine($" - {p.Name} is green");
    }

    var bf = new BetterFilter();
    var colorSpec = new ColorSpecification(Color.Green);
    WriteLine("Green products (new):");
    foreach(var p in bf.Filter(products, colorSpec))
    {
      WriteLine($" - {p.Name} is green");
    }

    WriteLine("Large blue items:");
    var sizeLargeSpec = new SizeSpecification(Size.Large);
    var colorBlueSpec = new ColorSpecification(Color.Blue);
    var andSpec = new AndSpecification<Product>(sizeLargeSpec, colorBlueSpec);

    foreach (var p in bf.Filter(products, andSpec))
    {
      WriteLine($" - {p.Name} is large and blue");
    }
  }
}
```

Thông qua ví dụ trên, chúng ta có thể thấy rõ việc thêm vào các bộ lọc mới cực kì dễ dàng
thông qua việc tạo thêm các `Specification`. Đồng thời, `class` `BetterFilter` đảm nhận
chức năng lọc sản phẩm chỉ cần viết 1 lần và hiếm khi cần phải thay đổi.

## Tham khảo

- [Design Patterns in C# and .NET](https://www.udemy.com/course/design-patterns-csharp-dotnet/)

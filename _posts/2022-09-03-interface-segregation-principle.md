---
layout: post
title: SOLID - Interface Segregation Principle
date: 2022-09-03 16:58 +0700
categories: [Design Pattern, SOLID]
tags: [solid, interface segregation principle, udemy, design patterns in c# and .net, decorator]
---

Đây là nguyên lý thứ 4 trong bộ 5 nguyên lí SOLID. Nguyên lý này chỉ ra rằng `interface` của
1 chương trình nên được phân tách nhỏ sao cho những người lập trình chỉ cần thực thi những
`interface` mà họ cần. Tránh việc phải thực thi 1 `interface` quá lớn, bao gồm quá nhiều
những phương thức thừa thãi.

## 1. Đặt vấn đề

Giả sử chúng ta làm việc với đối tượng `Document`, xoay quanh đối tượng này, chúng ta
có các thao tác liên quan như in ấn, scan, chuyển fax,... Để nhanh chóng, chúng ta
định nghĩa hết tất cả đối tượng này vào trong 1 `interface` duy nhất.

Tiếp đến, ta có đối tượng `MultiFunctionPrinter` thực thi `interface` này và định nghĩa
toàn bộ các phương thức tương ứng. Đến đây, chương trình hoạt động hoàn toàn bình thường.

```csharp
public class Document
{

}

public interface IMachine
{
  void Print(Document d);
  void Scan(Document d);
  void Fax(Document d);
}

public class MultiFunctionPrinter : IMachine
{
  public void Print(Document d)
  {
    //
  }

  public void Scan(Document d)
  {
    //
  }

  public void Fax(Document d)
  {
    //
  }
}
```

## 2. Vi phạm nguyên lí

Sau đó, chúng ta mở rộng hệ thống, hỗ trợ các thao tác quản lý tài liệu thông qua 1 máy in
cũ hơn, đặt tên `OldFashionPrinter`, chiếc máy này chỉ có duy nhất 1 chức năng `Print`, không
thể `Scan` hay `Fax`.

Lúc này, để đảm bảo hệ thống hoạt động được, ta phải thực thi `interface` `IMachine` cho lớp
này. Tuy nhiên, ta phải viết hướng dẫn rõ ràng bởi vì lớp này không thể `Scan` hay `Fax`.
Điều này gây nhầm lẫn rất lớn, bởi vì `class` `IMachine` thực thi `interface` `IMachine` nhưng
nó không thể thực hiện hết chức năng của `interface` này.

Điều này đã vi phạm nguyên lí `Interface Segregation`, nguyên lí này đảm bảo rằng:

> People don't pay for thing they don't need

Tức là, chúng ta không cần phải thực hiện những tính năng mà chúng ta không cần đến.

```csharp
public class OldFashionPrinter : IMachine
{
  public void Print(Document d)
  {
    //
  }

  public void Scan(Document d)
  {
    throw new NotImplementedException();
  }

  public void Fax(Document d)
  {
    throw new NotImplementedException();
  }
}
```

## 3. Cách giải quyết

Cách thức giải quyết khá đơn giản, thay vì khai báo 1 `interface` chịu trách nhiệm cho
rất nhiều công việc cùng lúc. Chúng ta nên chia nhỏ `interface` này, mỗi `interface`
nhỏ chịu trách nhiệm cho 1 công việc cụ thể. (`Seperation of Concern`)

```csharp
public interface IPrinter
{
  void Print(Document d);
}

public interface IScanner
{
  void Scan(Document d);
}

public interface IFaxer
{
  void Fax(Document d);
}
```

Lúc này, giả sử 1 đối tượng `Photocopier` có khả năng `Print` và `Scan`, chúng ta chỉ cần
thực hiện như dưới đây:

```csharp
public class Photocopier : IPrinter, IScanner
{
  void Print(Document d)
  {

  }

  void Scan(Document d)
  {

  }
}
```

Hoặc nếu muốn 1 `interface` mang tính trừu tượng hơn, ta hoàn toàn có thể kết hợp các
`interface` nhỏ thành 1 `interface` lớn hơn thông qua tính kế thừa của `interface`.

```csharp
public interface IMultiFunctionDevice : IScanner, IPrinter, IFaxer
{

}
```

Lúc này, ta có thể định nghĩa lớp `MultiFunctionMachine` thực thi `interface` `IMultiFunctionDevice`
và định nghĩa tất cả các phương thức:

```csharp
public class MultiFunctionMachine : IMultiFunctionDevice
{
  public void Print(Document d) {}
  public void Scan(Document d) {}
  public void Fax(Document d) {}
}
```

Hoặc, nếu đã có các lớp thực thi những `interface` con, ta có thể dùng luôn các lớp con này.
Cách thức thực hiện dưới đây còn được biết đến với tên gọi `decorator pattern`.

```csharp
public class MultiFunctionMachineComposition : IMultiFunctionDevice
{
  private IPrinter printer;
  private IScanner scanner;
  private IFaxer faxer;

  public MultiFunctionMachineComposition(IPrinter printer, IScanner scanner, IFaxer faxer)
  {
    this.printer = printer;
    this.scanner = scanner;
    this.faxer = faxer;
  }

  public void Print(Document d)
  {
    printer.Print(d);
  }

  public void Scan(Document d)
  {
    scanner.Scan(d);
  }

  public void Fax(Document d)
  {
    faxer.Fax(d);
  }
}
```

## Tham khảo

- [Design Patterns in C# and .NET](https://www.udemy.com/course/design-patterns-csharp-dotnet/)

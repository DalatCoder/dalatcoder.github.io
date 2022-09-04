---
layout: post
title: SOLID - Dependency Inversion Principle
date: 2022-09-03 18:08 +0700
categories: [Design Pattern, SOLID]
tags: [solid, dependency inversion principle, udemy, design patterns in c# and .net]
---

Đây là nguyên lý cuối cùng trong bộ 5 nguyên lý SOLID. Nguyên lý này khuyên rằng:
các thành phần cấp cao của hệ thống không nên phụ thuộc trực tiếp vào các thành
phần cấp thấp hơn. Thay vào đó, cả 2 nên cùng phụ thuộc vào các thành phần trừu
tượng (`interface`, `abstract class`,...).

Thêm vào đó, các thành phần trừu tượng không nên phụ thuộc vào các lớp thực thi (
  `concrete implementations`
).
Thay vào đó, các lớp thực thi phải phụ thuộc vào các thành phần trừu tượng (`abstraction`).

<!--more-->

## 1. Đặt vấn đề

```csharp
public enum Relationship
{
  Parent,
  Child,
  Sibling
}

public class Person
{
  public string Name;
}

public class Research
{
  // high-level module depends on low-level module
  public Research(Relationships relationships)
  {

  }

  static void Main(string[] args)
  {
    var parent = new Person { Name = "John" };
    var child1 = new Person { Name = "Chris" };
    var child2 = new Person { Name = "Mary" };

    var relationships = new Relationships();
    relationships.AddParentAndChild(parent, child1);
    relationships.AddParentAndChild(parent, child2);

    new Research(relationships);
  }
}
```

Tạo `low-level` `module` đặt tên `Relationships`, module này minh hoạ việc lưu giữ mối
quan hệ giữa những đối tượng `Person`.

```csharp
// low-level part
public class Relationships
{
  private List<(Person, Relationship, Person)> relations = new List<(Person, Relationship, Person)>();

  public void AddParentAndChild(Person parent, Person child)
  {
    relations.Add((parent, Relationship.Parent, child));
    relations.Add((child, Relationship.Child, parent));
  }

  public List<(Person, Relationship, Person)> Relations => relations;
}
```

Tiếp đến, để tiến hành xử lý dữ liệu trên danh sách mối quan hệ, chúng ta tạo 1 `module`
cấp cao hơn, đặt tên `Research`.

```csharp
public class Research
{
  public Research(Relationships relationships)
  {
    // accessing low-level interface for researching
    var relations = relationships.Relations;

    foreach (var r in relations.Where(
      x => x.Item1.Name == "John" &&
           x.Item2 == Relationship.Parent
    ))
    {
      WriteLine($"John has a child called {r.Item3.Name}");
    }
  }

  static void Main(string[] args)
  {
    var parent = new Person { Name = "John" };
    var child1 = new Person { Name = "Chris" };
    var child2 = new Person { Name = "Mary" };

    var relationships = new Relationships();
    relationships.AddParentAndChild(parent, child1);
    relationships.AddParentAndChild(parent, child2);

    new Research(relationships);
  }
}
```

Lúc này, khi chạy chương trình, ta sẽ lấy được danh các con của đối tượng `John`.

## 2. Vi phạm nguyên lý

Mặc dù chương trình chạy được, tuy nhiên, tại `class` `Research` (`high-level`), chúng
ta đang truy cập đến danh sách `relations` thông qua 1 `low-level` `API` từ
`class` `Relationships`. Nếu như `class` `Relationships` quyết định thay đổi cấu
trúc lưu trữ bên trong, việc thay đổi này sẽ ảnh hưởng đến `Research`.

## 3. Cách giải quyết

Thay vì để `class` `Research` phụ thuộc vào `low-level` `API`, chúng ta nên để
cả 2 phụ thuộc vào `abstraction`.

Tiến hành tạo 1 `interface` đặt tên `IRelationshipBrowser`, `interface` này
định nghĩa các phương thức cần thiết để xử lý dữ liệu về `relations`.

```csharp
public interface IRelationshipBrowser
{
  IEnumerable<Person> FindAllChildrenOf(string name);
}
```

Tiếp đến, tiến hành thực thi `interface` này cho lớp `Relationships`

```csharp
// low-level part
public class Relationships : IRelationshipBrowser
{
  private List<(Person, Relationship, Person)> relations = new List<(Person, Relationship, Person)>();

  public void AddParentAndChild(Person parent, Person child)
  {
    relations.Add((parent, Relationship.Parent, child));
    relations.Add((child, Relationship.Child, parent));
  }

  public List<(Person, Relationship, Person)> Relations => relations;

  public IEnumerable<Person> FindAllChildrenOf(string name)
  {
    foreach (var r in relations.Where(
      x => x.Item1.Name == "John" &&
           x.Item2 == Relationship.Parent
    ))
    {
      yield return r.Item3;
    }
  }
}
```

Lúc này, tại `high-level` `class` `Research`, thay vì phụ thuộc trực tiếp
vào `class` `Relationships` để lấy dữ liệu về `relations`, ta sửa lại
để `class` `Research` phụ thuộc vào `interface` `IRelationshipBrowser`. (
  Bởi vì trên `interface` này đã có các phương thức xử lý dữ liệu `relations`
)

```csharp
public class Research
{
  public Research(IRelationshipBrowser browser)
  {
    foreach (var p in browser.FindAllChildrenOf("John"))
      WriteLine($"John has a child called {p.Name}");
  }

  static void Main(string[] args)
  {
    var parent = new Person { Name = "John" };
    var child1 = new Person { Name = "Chris" };
    var child2 = new Person { Name = "Mary" };

    var relationships = new Relationships();
    relationships.AddParentAndChild(parent, child1);
    relationships.AddParentAndChild(parent, child2);

    new Research(relationships);
  }
}
```

Lúc này, `class` `Relationships` có thể thoải mái thay đổi cấu trúc lưu
trữ bên trong. Thay vì dùng `tuple`, có thể dùng `dictionary`,...

Mặc dù cấu trúc thay đổi, tuy nhiên cả 2 `class` đều tuân theo `interface`
`IRelationshipBrowser`, do đó chương trình vẫn sẽ hoạt động bình thường.
Không có điều gì ảnh hưởng đến `class` `Research` cả.

## Tổng kết

- Single Responsibility Principle

  - A class should noly have one reason to change
  - `Seperation of concerns` - different class handling different, independent
  tasks/problems

- Open - Closed principle

  - Classes should be open for extension but closed for modification

- Liskov substition principle

  - You should be able to substitute a base type for a subtype

- Interface Segregation Principle

  - Don't put too much into an interface
  - Split into seperate interfaces
  - `YAGNI` - You Ain't Going to Need It

- Dependency Inversion Principle

  - High-level modules should not depend upon low-level ones
  - Use abstractions

## Tham khảo

- [Design Patterns in C# and .NET](https://www.udemy.com/course/design-patterns-csharp-dotnet/)

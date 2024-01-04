---
layout: post
title: SOLID - Single Responsibility Principle
categories:
- Design Pattern
- SOLID
tags:
- solid
- single responsibility
- udemy
- design patterns in c# and .net
---
`Single Responsibility` hay còn gọi là nguyên lý đơn trách nhiệm. Đây là lời khuyên
đầu tiên trong bộ 5 nguyên lý `SOLID`. Ý tưởng đằng sau nguyên lý này đó là: mỗi
`class`, `module` hoặc `function` trong 1 chương trình chỉ nên chịu trách nhiệm
cho 1 thứ duy nhất và chỉ có duy nhất 1 lý do để thay đổi. Chúng ta hãy cùng
tìm hiểu nguyên lý này thông qua minh hoạ dưới đây.

<!--more-->

## 1. Đặt vấn đề

Giả sử ban đầu chúng ta tạo 1 `class` để lưu trữ nhật ký, đặt tên `Journal`

```csharp
public class Journal
{
  private readonly List<string> entries = new List<string>();
  private static int count = 0;

  public int AddEntry(string text)
  {
    entries.Add($"{++count}: {text}");
    return count; // memento pattern
  }

  public void RemoveEntry(int index)
  {
    entries.RemoveAt(index);
  }

  public override string ToString()
  {
    return string.Join(Environment.NewLine, entries);
  }
}

public class Demo
{
  static void Main(string[] args)
  {
    var j = new Journal();
    j.AddEntry("First Entry");
    j.AddEntry("Second Entry");

    WriteLine(j);
  }
}
```

Hiện tại, 2 phương thức trong `class` `Journal` đều liên quan tới đối tượng
nhật ký, bao gồm:

- `AddEntry`: thêm nhật ký mới
- `RemoveEntry`: xoá bỏ nhật ký theo vị trí

## 2. Vi phạm nguyên lý

Bây giờ, hãy hình dung chúng ta muốn lưu trữ những dòng nhật ký này. Có thể lưu
trữ vào tập tin hoặc cơ sở dữ liệu,... Chúng ta có thể nhanh chóng tạo phương
thức `Save` và `Load` trực tiếp vào `class` `Journal` như dưới đây

```csharp
class Journal
{
  public void Save(string filename)
  {
    File.WriteAllText(filename, ToString());
  }

  public static Journal Load(string filename)
  {

  }

  public static Journal Load(Uri uri)
  {

  }
}
```

Tuy nhiên, khi thực hiện như trên, chúng ta đã vi phạm nguyên lý `Single Responsibility`. Lúc này `class` `Journal` phải đảm nhiệm 2 trách nhiệm:

- Thực hiện các thao tác liên quan đến nhật ký
- Lưu trữ và load dữ liệu

Do đó, `class` `Journal` cũng có 2 lý do để thay đổi. Từ đó, vi phạm nguyên
lý `Signle Responsibility`.

## 3. Cách giải quyết

Để giải quyết vấn đề vi phạm nguyên lý, chúng ta có thể refactor những phương
thức liên quan đến việc lưu trữ vào 1 `class` riêng, đặt tên `Persistence`.

```csharp
class Persistence
{
  public void SaveToFile(Journal j, string filename, bool overwrite = false)
  {
    if (override || !File.Exists(filename))
      File.WriteAllText(filename, j.ToString());
  }
}
```

Thông qua việc phân tách `class` chúng ta cũng áp dụng nguyên lý `Seperation of Concern`. Tức là, mỗi `class` chỉ nên quan tâm đến 1 thứ.

- `Journal`: quan tâm đến việc thực hiện các thao tác trên nhật ký
- `Persistence`: quan tâm đến việc lưu trữ dữ liệu

Chương trình cuối cùng sẽ như dưới đây:

```csharp
public class Demo
{
  static void Main(string[] args)
  {
    var j = new Journal();
    j.AddEntry("First Entry");
    j.AddEntry("Second Entry");

    var p = new Persistence();
    var filename = "data.txt";
    p.SaveToFile(j, filename, true);
  }
}
```

## Tham khảo

- [Design Patterns in C# and .NET](https://www.udemy.com/course/design-patterns-csharp-dotnet/)

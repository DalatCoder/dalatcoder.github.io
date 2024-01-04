---
layout: post
title: Bắt đầu với Jekyll
categories:
- Newbie
- Jekyll
tags:
- newbie
- jekyll
---
Làm quen với Jekyll để tạo nhanh trang blog cá nhân.

<!-- more -->

## 1. Jekyll và Chirpy

[Tài liệu về chirpy](https://chirpy.cotes.page/)

## 2. Cài đặt

[Getting Started](https://chirpy.cotes.page/posts/getting-started/)

## 3. Tạo post mới

[Writing a new post](https://chirpy.cotes.page/posts/write-a-new-post/)

Để tạo nhanh 1 bài viết mới, có thể sử dụng công cụ [Jekyll Compose](https://github.com/jekyll/jekyll-compose)

- Chạy lệnh: `bundle exec jekyll compose "[post-name]"`, ví dụ: `bundle exec jekyll compose "get-started-with-jekyll"`
- Chỉnh sửa `post metadata`, trong đó:
  - `title`: Tiêu đề bài viết
  - `date`: Ngày tạo bài viết
  - `categories`: Phân loại bài viết. Ở ví dụ dưới đây, bài viết này thuộc phân loại con `Jekyll` trong phân loại `Newbie`
  - `tags`: Đánh dấu các thẻ liên quan

```md
---
layout: post
title: Bắt đầu với Jekyll
date: 2023-01-07 17:02 +0700
categories: [Newbie, Jekyll]
tags: [newbie, jekyll]
---
```

- Tạo nội dung tóm tắt cho bài viết: phần nội dung tóm tắt được viết phía dưới phần `metadata` và được ngăn cách với
  phần nội dung chính bởi `<!-- more -->`

```md
---
layout: post
title: Bắt đầu với Jekyll
date: 2023-01-07 17:02 +0700
categories: [Newbie, Jekyll]
tags: [newbie, jekyll]
---

Làm quen với Jekyll để tạo nhanh trang blog cá nhân.

<!-- more -->
```

- Hoàn thiện phần nội dung còn lại của bài viết

## 4. Chạy local server

Để xem bài viết nháp, có thể khởi chạy local server thông qua lệnh: `bundle exec jekyll s`. Sau đó, truy cập vào đường dẫn `http://localhost:4000/` để xem danh sách các bài viết.

## 5. Xuất bản

Xuất bản thông qua `Github Action`, chỉ cần `commit` và `push` lên `Github`.

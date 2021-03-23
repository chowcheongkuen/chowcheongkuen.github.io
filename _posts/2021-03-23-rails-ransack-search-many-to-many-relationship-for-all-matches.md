---
layout: post
title: Rails + Ranscak: 多对多关系的全匹配搜索
date: 2021-01-31 14:03
---

## 要解决的问题

一个商品会被打上多个标签，同一个标签也会被关联到多个不同的商品上。现在需要开发一个商品搜索接口，支持根据选择的多个标签过滤商品。

例如有四个商品，分别为 Product 1、Product 2、Product 3、Product 4。四个标签 A、B、C、D，ID 分别都为 1、2、3、4。商品与标签的关系如下：

- Product 1: A, B, C, D
- Product 2: A, C, D
- Product 3: B, C, D
- Product 4: D

如果搜索接口传入 A、C 标签的 ID，应该返回 Product 1 和 2。

这个问题类似 [Rails, Ransack: How to search HABTM relationship for “all” matches instead of “any”](https://stackoverflow.com/questions/13409627/rails-ransack-how-to-search-habtm-relationship-for-all-matches-instead-of-a)。

## Model 示例

执行生成 model 的命令：

```shell
$ rails g model product title
$ rails g model tag name
$ rails g model tagging position:integer product:references tag:references
```

Product model:

```ruby
class Product < ApplicationRecord
  has_many :taggings, dependent: :destroy
  has_many :tags, through: :taggings
end
```

Tag model:

```ruby
class Tag < ApplicationRecord
  has_many :taggings, dependent: :destroy
  has_many :product, through: :taggings
end
```

Tagging model:

```ruby
class Tagging < ApplicationRecord
  belongs_to :product
  belongs_to :tag
end
```

然后在 rails console 里按照 Product 1 ~ 4 和 Tag A 到 D 插入数据。

## 首先想到的解决方法

首先想到的解决方法是用 ransack 提供的 `tags_id_in`，但这样会把 Product 1、2、3 都搜索出来，因为 ransack 自带的 `in` 是 or 的搜索，不是 and。

```
irb(main): q = Product.ransack tags_id_in: [1,3]
irb(main): r = q.result(distinct: true)
irb(main): r.pluck :title
=> ["Product 1", "Product 2", "Product 3"]
```

## 解决方案

从 StackOverOverflow 这个 2012 年的[答案](https://stackoverflow.com/a/13445889/888089)，结合 ransack 的文档，经过自己的测试后，使用以下的代码可以解决本篇博客遇到的问题。


```
class Product < ApplicationRecord
  # association macros
  has_many :taggings, dependent: :destroy
  has_many :tags, through: :taggings

  # scopes
  scope :tag_ids_match_all, -> (*tag_ids) { where(matches_all_tags_arel(tag_ids)) }

  # class methods
  def self.matches_all_tags_arel(tag_ids)
    products = Product.arel_table
    tags = Tag.arel_table
    taggings = Tagging.arel_table

    products[:id].in(
      products.project(products[:id])
             .join(taggings).on(products[:id].eq(taggings[:product_id]))
             .join(tags).on(taggings[:tag_id].eq(tags[:id]))
             .where(tags[:id].in(tag_ids))
             .group(products[:id])
             .having(tags[:id].count.eq(tag_ids.length))
    )
  end

  def self.ransackable_scopes(auth_object = nil)
    super + [:tag_ids_match_all]
  end
end
```

搜索语句：

```
irb(main): q = Product.ransack tag_ids_match_all: [1,3]
irb(main): r = q.result(distinct: true)
irb(main): r.pluck :title
=> ["Product 1"]
```

`matches_all_tags_arel` 方法用到了 Arel，自己平时用得也不多，把搜集到的一些 Arel 资料也列一下。

- [The definitive guide to Arel, the SQL manager for Ruby](https://jpospisil.com/2014/06/16/the-definitive-guide-to-arel-the-sql-manager-for-ruby.html)
- [ActiveRecord & ARel](https://www.slideshare.net/flah00/activerecord-arel)
- [arel/select_manager_test.rb](https://github.com/rails/rails/blob/main/activerecord/test/cases/arel/select_manager_test.rb)
- [brynary / arel](https://github.com/brynary/arel)

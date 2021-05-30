---
layout: post
title: 一些 Rails console 小 tips
date: 2021-04-18 20:28
---

## 不重启 rails console 重新加载代码

当需要在 rails console 里测试一些代码改动时，可以使用 `reload!` 命令来重新加载最新代码。需要注意的一个地方时，如果此时你已经初始化了一些对象，这些对象是不会被更新的。`reload!` 只是重新加载代码。

```
irb(main):025:0> reload!
Reloading...
=> true
```

## 执行 rake task

可以按照以下方式在 rails console 里执行 rake task：

```
irb(main):042:0> Rails.application.load_tasks
irb(main):043:0> Rake::Task["db:seed"].invoke
irb(main):044:0> Rake::Task["another_task"].invoke(arg1, arg2)
```

## 获取上一次输出到 rails console 里的值

有时候我们会在执行了代码后，才想起需要用一个变量保存返回值。此时不需要再执行一次代码，可以用下划线 `_` 来获取上一次的输出。示例：

```
irb(main):042:0> Rake::Task["db:seed"].invoke
=> nil
irb(main):043:0> _
=> nil
irb(main):044:0> a = b
Traceback (most recent call last):
        1: from (irb):44
NameError (undefined local variable or method `b' for main:Object)
irb(main):045:0> _
=> #<NameError: undefined local variable or method `b' for main:Object>
irb(main):046:0> a = 34
=> 34
irb(main):047:0> _
=> 34
```

## 切换 rails console 对象上下文

当需要在一个指定的对象上调用多次方法时，可以用 `irb` 切换到该对象的上下文环境，以减少重复输入。

```
irb(main):011:0> VeryLongNameOfObject.name
=> "some name"
irb(main):012:0> VeryLongNameOfObject.attribute_name
=> "other value"
irb(main):013:0> irb VeryLongNameOfObject
irb#1(VeryLongNameOfObject):001:0> name
=> "some name"
irb#1(VeryLongNameOfObject):002:0> attribute_name
=> "other value"
irb#1(VeryLongNameOfObject):003:0> exit
=> #<IRB::Irb: @context=#<IRB::Context:0x00007f919b1d9140>, @signal_status=:IN_EVAL, @scanner=#<RubyLex:0x00007f91ad997758>>
```

## 隐藏输出

有些方法会在 rails console 里打印超长的内容，影响 console 的查看。可以在调用方法后加上 `; nil` 来隐藏输出。

```
irb(main):011:0> SomeClass.update_all
=> Very long output
irb(main):012:0> SomeClass.update_all ; nil
=> nil
```

## 在 console 里发起请求

可以在 rails console 里直接发起网络请求，以便检查响应内容、session 等信息。

```
irb(main):004:0> app.get "http://localhost:3000"
Started GET "/" for 127.0.0.1 at 2021-04-18 23:52:04 +0800

=> 200
irb(main):006:0> app.response
irb(main):007:0> app.session
```

## 使用 sandbox 模式回滚数据改动

`rails console --sandbox` 命令启动沙盒模式，在沙盒模式下对数据库的所有数据变更，都会在退出 console 后回滚。**不建议在生产环境使用。**

```
$ rails console --sandbox
irb(main):003:0> Product.all.size
   (0.1ms)  SELECT COUNT(*) FROM "products"
=> 4
irb(main):005:0> Product.all.destroy_all
irb(main):006:0> Product.all.size
   (0.3ms)  SELECT COUNT(*) FROM "products"
=> 0
irb(main):007:0> exit

$ rails console
irb(main):001:0> Product.all.size
   (0.5ms)  SELECT sqlite_version(*)
   (0.3ms)  SELECT COUNT(*) FROM "products"
=> 4
```

## 查找方法定义的位置

Rails 经常会有些很 magic 的方法不知道定义在哪里，可以通过以下方式查找：

```
irb(main):006:0> ApplicationRecord.method(:has_one).source_location
=> ["/path/to/gems/activerecord-6.1.3/lib/active_record/associations.rb", 1607]
irb(main):007:0> Product.instance_method(:full_title).source_location
```

Ref: [Rails console awesomeness](https://longliveruby.s3-us-west-1.amazonaws.com/rails_console_awesomeness.pdf)

**-EOF-**

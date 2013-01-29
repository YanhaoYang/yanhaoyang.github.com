---
layout: post
title: "A pitfall of updating foreign key with `update_attribute`"
date: 2013-01-29 17:23
comments: true
categories:
---

Suppose there are following models and associations:
``` ruby

  class Foo < ActiveRecord::Base
    belongs_to :bar
  end

  class Bar < ActiveRecord::Base
    has_one :foo
  end
```

Instead of changing the associated object `:bar`, you can assign a Bar object to it:

``` ruby
  foo.bar = Bar.find(1)
```

you may want to update the foreign key directly with `update_attribute` sometimes:

``` ruby
  foo.update_attribute :bar_id, 1
```

However, this doesn't always work.

For example, in following code `update_attribute` does nothing:

``` ruby
  foo.bar = Bar.find(1)
  foo.update_attribute :bar_id, 2
```

After `update_attribute` has been called, `foo#bar_id` remains 1. What's wrong with it?

This is a result of the autosave mechanism of ActiveRecord. When `Foo#bar=` is called, a callback for saving associations will be created, which will set `foo.bar_id` to `bar.id`.

In above code, the actual process is:

* foo.bar_id is set to 2
* the callback for autosave is called, which set foo.bar_id to 1
* foo.save saves nothing because the attribute is not changed.

I am not sure the problem is a bug or not.

There are two ways to update the value correctly. First, reload the object before calling `update_attribute`:

``` ruby
  foo.bar = Bar.find(1)
  foo.reload
  foo.update_attribute :bar_id, 2
```

Second, use a bar object as parameter for `update_attribute`:

``` ruby
  foo.bar = Bar.find(1)
  foo.update_attribute :bar_id, Bar.find(2)
```

These are two workarounds I think of.

---
layout: post
title: "Two ways to embed JSON data in views of Rails"
date: 2011-12-16 21:03
comments: true
categories:
---

Several days ago, I wrote a piece of code to embed some JSON data in a web page, which will be used by Javascipt later. It is not likely to change during a session, so it is more convenient to embed it in the web page than requsting the data by Ajax.

The code in the web page is like:

``` html
  <script type="text/javascript">
      var listData = '<%= ActiveSupport::JSON.encode(@data) %>'.evalJSON();
  </script>
```

I am using Prototype framework, FYI.

This implementation worked for a while, before the error rose. It turned out that quotation marks in the data could break the Javascript. For example, if the @data is like this:

``` ruby
  @data = { :name => 'some "name"' }
```

After some research, I found two ways to solve the problem.

First, dump the JSON string with String#dump. The ruby code:

``` html
  <script type="text/javascript">
      var listData = <%= ActiveSupport::JSON.encode(@data).dump %>.evalJSON();
  </script>
```

Please note, the quotation marks around <%= ... %> have been removed.

Second, let the SCRIPT tag only hold the JSON data, then retrieve the data in Javascript with innerHTML:

``` html
  <script id="json_data" type="application/json">
      <%= ActiveSupport::JSON.encode(@data) %>
  </script>

  <script type="text/javascript">
      var listData = $('json_data').innerHTML.evalJSON();
  </script>
```

Those are some small fixes, but they solved big troubles.

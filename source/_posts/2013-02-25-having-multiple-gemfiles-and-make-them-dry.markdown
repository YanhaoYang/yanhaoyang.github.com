---
layout: post
title: "Having Multiple Gemfiles and Make them DRY"
date: 2013-02-25 21:13
comments: true
categories:
---

## Problem and solutions

Suppose you are developing a great gem for your great application. For convenience, you want to use local path in your Gemfile locally, but you want to let it install the gem from Github on deployment. Due to the limitation of Bundler, you cannot define a same gem from different sources in your Gemfile. One workaround is to switch the definitions in Gemfile by checking an environment varialbe.

``` ruby Gemfile
gem 'rails', '3.2.12'
gem ...
gem ...

if ENV['MY_GEM_SOURCE'] == 'local'
  gem "my_great_gem", :path => "path/to/my_great_gem"
 else
  gem "my_great_gem", :git => "git@github.com:path/to/my_great_gem.git"
end
```

This works well during development. But when you want to deloy it to server and run `bundle install --deployment ...`, bundler complaints that there's no Gemfile.lock. What if you just run `bundle install` locally, create the Gemfile.lock and check it in? You have to regenerate the lock file each time you switch between development and deployment.

Another solution is to create two Gemfiles, one for development, another for deployment.

``` ruby Gemfile-for-development
gem 'rails', '3.2.12'
gem ...
gem ...

gem "my_great_gem", :path => "path/to/my_great_gem"
```

``` ruby Gemfile-for-deployment
gem 'rails', '3.2.12'
gem ...
gem ...

gem "my_great_gem", :git => "git@github.com:path/to/my_great_gem.git"
```
You just tell bundler which Gemfile to use by setting environment variable `BUNDLE_GEMFILE`, `bundle` and `rails` commands will work as usual.

However, when you change something in the Gemfiles, you have to update both files. That's not DRY.

## Make them DRY

We can take some logic out of models by moving the code to concerns. We can also extract some some code from a view and move it to a partial. Can we do the same to a Gemfile? Although Gemfile is just a ruby file, we cannot use `require` to load a nother file. After checking source code fo bundler, I found the secrect. In following code, bundler loads Gemfile:

``` ruby lib/bundler/dsl.rb
def eval_gemfile(gemfile)
  instance_eval(Bundler.read_file(gemfile.to_s), gemfile.to_s, 1)
rescue SyntaxError => e
  bt = e.message.split("\n")[1..-1]
  raise GemfileError, ["Gemfile syntax error:", *bt].join("\n")
end
```

Similarly, we can load other file this way in Gemfiles. For example:

``` ruby Gemfile-base
gem 'rails', '3.2.12'
gem ...
gem ...
```

``` ruby Gemfile-for-development
gemfile = 'Gemfile-base'
instance_eval(Bundler.read_file(gemfile.to_s), gemfile.to_s, 1)

gem "my_great_gem", :path => "path/to/my_great_gem"
```

``` ruby Gemfile-for-deployment
gemfile = 'Gemfile-base'
instance_eval(Bundler.read_file(gemfile.to_s), gemfile.to_s, 1)

gem "my_great_gem", :git => "git@github.com:path/to/my_great_gem.git"
```

Now we have made them DRY.

## Work with multiple Gemfiles

The simplest way is to set the environment variable manually, for example:
    export BUNDLE_GEMFILE=Gemfile-for-development

If you use rbenv and the plugin rbenv-vars, I created another simple rbenv plugin [rbenv-set-vars](https://github.com/YanhaoYang/rbenv-set-vars) to manage multiple environment variable sets.

To deploy the application with Capistrano, you need to set which Gemfile to use with following setting:
    set :bundle_gemfile,  "Gemfile-for-deployment"

You also need to pass `BUNDLE_GEMFILE` to rails server so that it can load dependencies correctly.


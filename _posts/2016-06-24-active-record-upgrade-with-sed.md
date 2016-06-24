---
layout: post
title: Upgrade from ActiveRecord 3 to ActiveRecord 4 with a little help from sed
---

I've been working on a project that is fairly large, around 300k lines of ruby code.  We were asked to upgrade to Rails 4 early on and that work has yielded many subtle syntax changes, especially for ActiveRecord queries.

Some of these issues I was able to solve with a gem [ ArelConverter ](https://github.com/pallan/arel_converter) which has been pretty good at locating 60-70% of deprecated syntax.

One issue we learned when we had to convert scopes to respond to `#call` was that some of the developers used `lambda` and others use the `->` rocket syntax.  Well, we needed to standardize that and I was hoping to strengthen my sed skills at the same time.

If you're not familiar with sed, `$ man sed` right now, but it's basically a command line search and replace tool that I find to be predictable and a pleasure to work with.  Combined with `ag` and `xargs` it can be really powerful.

Again, I'm looking where we use lambda's across the repository.  `Ag` is really good for this:

```bash
$ ag 'lambda'
app/models/user.rb
163:  scope :with_program, lambda { |program| where("user_programs.program_id IN (?)", program).joins(:user_programs).group("users.id") }
164:  scope :with_company, lambda { |company| where("users.company_id in (?)", company)}
```

So for example, that first line needs to actually become:

```ruby
scope :with_program, ->(program) { where("user_programs.program_id IN (?)", program).joins(:user_programs).group("users.id") }
```

Sed is not destructive by default, meaning you are able to run your substitutions almost 'interactively' and confirm what you're trying to do.

Let's give that a shot and start small by just replacing the word `lambda` with the rocket `->`

```bash
$ sed -n 's/lambda/->/p' app/models/user.rb

  scope :with_program, -> { |program| where("user_programs.program_id IN (?)", program).joins(:user_programs).group("users.id") }
```

If you did `git status`, you'll see we didn't actually edit that file, instead sed is just printing to STDOUT the changes it made.  This is nice if you want to pipe it somewhere else. The `-n` flag limits the output to only the lines that contained a substitution (I wish I learned about that flag sooner!).

Ok, you probably didn't come here to learn sed so let me show a way to do this across all your ruby files and replace it so `->` is used, with and without arguments.

```bash
$ ag -l 'lambda' -G '.rb' | xargs sed -n -E 's:lambda(.*)\|(.*)\|:->(\2)\1:p'
```

Breaking that down:

```
ag
 -l # only give back file names
 'lambda' # search for any lines with the word lambda
 -G # limit file names to *.rb

xargs # send each file name to sed

sed
 -n # limit output to only lines touched
 -E # use perl regex
```

Take a look at that output, did everything change correctly? If it did, great let's _actually_ edit those files.

```bash
$ ag -l 'lambda' -G '.rb' | xargs sed -i .bak -n -E 's:lambda(.*)\|(.*)\|:->(\2)\1:p'
```

If you run `git status` you'll likely see a lot of changes.  Go ahead and confirm everything looks good.  The `.bak` files you see are copies of the original files so if you royally messed up, you can use those files or I usually just `git reset`.

One of my 2016 goals has been to get better at the command line with tools like sed, xargs, ag, grep, awk and more.  I hope this helps some others who are trying to do something similar!

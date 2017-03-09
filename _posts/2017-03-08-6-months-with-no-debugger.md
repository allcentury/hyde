---
layout: post
title: 6 months without a debugger
---

Six months ago I started a new job at [commercetools](https://www.commercetools.com) and while I had written a few small node.js apps, I did not have anywhere close to the dev setup I had with ruby.  With ruby, I had become ~~an expert~~, no a ~~master~~, rather I was *reliant* on [pry](https://github.com/pry/pry) for my day to day.

Pry was so ingrained in my normal workflow that I [set up](https://github.com/allcentury/dotfiles/blob/master/.vimrc#L125-L126) vim shortcuts to put a `binding.pry` under my cursor whenever needed, via:

```
" add a require 'pry' and binding.pry at current cursor location
map <leader>bp :s/\(^.*\n\)/require 'pry'\rbinding.pry\r\1/g<cr>:noh<cr>3k==2.2j:w<cr>
```

Later to get rid of them, I made [_another_](https://github.com/allcentury/dotfiles/blob/master/.vimrc#L136) vim shortcut to grep through all the text to delete any trailing `binding.pry`'s in the code.  So, you can imagine my workflow:

1. Run test!
2. Failure
3. Darn it
4. move to file:line in stacktrace
5. add a binding.pry
6. re-run test
7. Oh.
8. Fix.
10. Repeat

That worked really well and I got fancy with the pry prompts.  For instance, `show-doc`, `show-source`, `ls` and my favorite `whereami`.  I'd do the same when debugging gems, if my stacktrace pointed to a gem I'd do:

1. bundle open some_gem
2. move to file:line in stacktrace
3. add a binding.pry

I've found [bugs in the mongo ruby driver](https://jira.mongodb.org/browse/RUBY-1201) using this technique.

Now though, in new territory, using node.js I decided to stop being so reliant on a debugger.  This wasn't a "I'm too good for a debugger", it was just me reevaluating whether a tool I was using had become a crutch.  I can remember times where I was in that 10 step loop (see above) endlessly and I thought to myself, this is a really slow way to develop software.

Of course, the internet has no shortage of opinions on the matter.  I know what [Linus has said](http://lwn.net/2000/0914/a/lt-debugger.php3), what [Robert Martin](http://www.artima.com/weblogs/viewpost.jsp?thread=23476) said and that Kernighan once wrote that the most effective debugging tool is still careful thought, coupled with judiciously placed print statements.

All of those opinions are completely valid, but in my time developing software I can say with confidence there is no absolute truth to finding and fixing issues in code.  I've done it 100 ways, some which took me seconds and some which took me weeks.

So I decided to not look for a node.js equivalent to pry and that meant I fell back on the ol' C way of debugging that I grew up on, print statements.  Or rather in node.js case, console.log statements.  It was not an easy transition.  There were two main pain points early on, 1. I was still learning node.js and learning a language with print statements is *slow*.  Very, very slow.  2. Console.log did not always give me the output I was expecting.  In fact many objects in javascript don't really respond to `toString()` in a meaningful way.

I had to start creating some tooling again to help me because either I'd get buried in documentation or I'd go digging through source code just to find out what methods an object had.  Here comes more vim keys...

```vim
map <leader>jsm :s/\(^.*\n\)/console.log(`Available methods: ${Object.getOwnPropertyNames(someObj)}`);\r\1/g<cr>:noh<cr>3k==2.2j:w<cr>
```

I'd hit `<leader>jsm` and it adds this line:

```js
console.log(`Available methods: ${Object.getOwnPropertyNames(someObj)}`);
```

I'd then replace `sombObj` with the variable in question and rerun program.  I'd then get a print out of methods to the screen.  How about inspecting an object?  Well if it was my own object I was forcing myself to always write a `toString()` method on any class.  That's fine for my own objects but for objects I didn't want to monkey patch, I'd have another vim command that did this:

```js
console.log(JSON.stringify(someObj));
```

You've all done that before I'm sure.

I'd gotten to the point where I couldn't run the program from the commandline anymore because my thumbs were going to fall off.  That led me to making sure I could run a node file at any time via `<leader>e`.  That command would then know how to execute the file based on the extension, it looks something like this:

```vim
" execute file if we know how
function! ExecuteFile(filename)
  :w
  :silent !clear
  if match(a:filename, '\.rb$') != -1
    exec ":!bundle exec ruby " . a:filename
  elseif match(a:filename, '\.js$') != -1
    exec ":!node " . a:filename
  elseif match(a:filename, '\.sh$') != -1
    exec ":!bash " . a:filename
  elseif match(a:filename, '\.c$') != -1
    exec ":!gcc " . a:filename . " ; ./a.out"
  elseif match(a:filename, '\.go$') != -1
    exec ":!go run " . a:filename
  else
    exec ":!echo \"Don't know how to execute: \"" . a:filename
  end
endfunction
```

I also set up a similar command to run tests via `<leader>t`.  It would look at the file extension and run the test file.

So what was my 10 step process with pry is now a little different with node.js.

1. Run test!
2. Failure
3. Darn it
4. move to file:line in stacktrace
5. add a console.log statement
6. re-run test
7. See output.
8. Fix.
9. Remove console.log statement
10. Repeat

So still 10 steps.  However, I did notice a conscious change in behavior.  Before with pry, I wouldn't start to debug my code until I was at the familiar pry prompt.  Yes, you read that right.  I would instinctively add the `binding.pry` and just rerun and wait for the prompt before starting to check local variables, method definitions, etc.  I imagine that just manifested out of habit and speed - there are many times in the day where my fingers are faster than my brain.  With my node.js workflow, I started to notice myself combing through code line by line with much greater detail because it was so slow to debug with print statements.  I didn't want to keep adding print statements for every variable I forgot and I was tired of adding `toString()` to every single object when I was primarily just enforcing that API because of debugging.

Now 6 months later, I decided to see if using the [node debugger](https://nodejs.org/api/debugger.html) would be a welcome change to my workflow.  I've found that I don't care to use it when it's my own code.  I'm now in the habit of really spending time with the error or test failure, going back through the code in a meaningful way with a bit more information than I had before.  It's slowed me down in terms of key commands but that metric is useless in programming.  Instead, I'm focusing on the problem and it's leading me to better results.  If you find yourself rifling through code or tests but anchored to a debugger to get the job done, take a step back for a few days and see what comes of it.  I find I'm writing a lot less 'quick code'.

I've set a new goal now - I'm going to try to not use the debugger on code that I have written.  For code that I didn't write, such as the zillion npm packages that are out there - I'm giving myself a bit more wiggle room.  These packages, most of which are still in ES5, are not easy for me to navigate.  I'd say I'm proficient in javascript now but for large libraries, there is no way for me to debug someone else's code quickly.  Instead, I'm allowing myself to use the debugger when the stacktrace is pointing to something in my node_modules folder.  I haven't automated this just yet but I will and when I do, I'll update this blog post.  Thanks for reading!


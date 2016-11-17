---
layout: post
title: Search and replace for ES6
---

I'm here to share the simplest of tips for vim users that are transitioning to ES6 fat arrows.

It's really common to still see examples on npm packages like this:

```js
sftp.open(config.ftpPath + "/" + m_fileName, "r", function(err, fd) {
  sftp.fstat(fd, function(err, stats) {
    var totalBytesRead = 0;
    function callbackFunc(err, bytesRead, buf, pos) {
      if(err) {
        writeToErrorLog("downloadFile(): Error retrieving the file.");
        errorOccured = true;
        sftp.close(fd);
      }
    }
  })
});
```

I prefer not to have a linter fix my code for me but there are somethings I'm starting to automate.  The first is the use of [fat arrows](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) over `function(params)` syntax.

I made a quick vim alias to search and replace the use of `function(*args)` to `(*args) =>`.  Prepare for an anticlimactic code snippet...

```vim
:%s/(\(.*\)function\(.*)\)/(\1\2 =>/g<cr>
```
Try that with the interactive prompt and see if it looks ok.  The breakdown of the regex looks complicated but it's not.  It could likely be improved but so far it's working for me.

```regex
(\(.*\)function - look for the word function within a set of parens.  That way function myFunc(*args) does not get replaced but somethin.method('arg', function(*args) { does.  Create a match group for the args before the word function
\(.*)\) - create a match group for the arguments within function(*args) so we don't lose the arguments
/(\1\2 =>/ - replace with the first group and second group followed by our fat arrow
g<cr> - do this globally and leave the vim command prompt
```

Assuming that worked for you, let's add it to our `~/.vimrc`:

```
" replace function(*args) with fat arrows =>
map <leader>fa :%s/(\(.*\)function\(.*)\)/(\1\2 =>/g<cr>
```

Resource your vimrc in vim (either restart vim or `:source ~/.vimrc`).  Now on your old ES5 code hit your leader key and then `fa`.  My leader key is the comma so that means: `,fa`.

The code above should turn into:

```js
sftp.open(config.ftpPath + "/" + m_fileName, "r", (err, fd) => {
  sftp.fstat(fd, (err, stats) => {
    var totalBytesRead = 0;
    function callbackFunc(err, bytesRead, buf, pos) {
      if(err) {
        writeToErrorLog("downloadFile(): Error retrieving the file.");
        errorOccured = true;
        sftp.close(fd);
      }
    }
  })
});
```

If that didn't work for you let me know and I'll try to improve the regex. Enjoy!

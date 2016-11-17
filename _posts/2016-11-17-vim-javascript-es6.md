---
layout: post
title: Search and replace for ES6
---

I'm here to share the simplest of tips for vim users that are transitioning to ES6 fat arrows.

It's really common to still see examples on npm packages like this:

```js
sftp.open(config.ftpPath + "/" + m_fileName, "r", function(err, fd) {
sftp.fstat(fd, function(err, stats) {
    var bufferSize = stats.size,
        chunkSize = 16384,
        buffer = new Buffer(bufferSize),
        bytesRead = 0,
        errorOccured = false;

    while (bytesRead < bufferSize && !errorOccured) {
        if ((bytesRead + chunkSize) > bufferSize) {
            chunkSize = (bufferSize - bytesRead);
        }
        sftp.read(fd, buffer, bytesRead, chunkSize, bytesRead, callbackFunc);
        bytesRead += chunkSize;
    }

    var totalBytesRead = 0;
    function callbackFunc(err, bytesRead, buf, pos) {
        if(err) {
            writeToErrorLog("downloadFile(): Error retrieving the file.");
            errorOccured = true;
            sftp.close(fd);
        }
        totalBytesRead += bytesRead;
        data.push(buf);
        if(totalBytesRead === bufferSize) {
            m_fileBuffer = Buffer.concat(data);
            writeToLog("downloadFile(): File saved to buffer.");
            sftp.close(fd);
            m_eventEmitter.emit('downloadFile_Complete');
        }
    }
});
```

I prefer not to have a linter fix my code for me but there are somethings I'm starting to automate.  The first is the use of [fat arrows](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) over `function(params)` syntax.

I made a quick vim alias to search and replace the use of `function(*args)` to `(*args) =>`.  Prepare for an anticlimactic code snippet...

```vim
:%s/function\(.*)\)/\1 =>/gc
```
Try that with the interactive prompt and see if it looks ok.

Assuming it does, let's add it to our `~/.vimrc`:

```
" replace function(*args) with fat arrows =>
map <leader>fa :%s/function\(.*)\)/\1 =>/g<cr>
```

Resource your vimrc in vim (either restart vim or `:source ~/.vimrc).  Now on your old ES5 code hit your leader key and then `fa`.  My leader key is the comma so that means: `,fa`.

Enjoy!


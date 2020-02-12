# gitsync

Efficiently sync git-controlled source with a remote server.

Uploads both local git commits and the uncommitted data (in files known to
git) to the specified remote host via SSH.  After running the script, the head
of the current branch and the state of the working tree on the remote side
will match local ones.

Intended audience are developers who want to **develop** locally because
that's where they have a nicely configured IDE, but must **run** their
software remotely because that's where the datasets or other runtime
environment lies.

## Usage

To use, just invoke `gitsync <hostname>` in a git-controlled directory.  Prior
to first use you need to ensure that the remote host contains a checkout of
the repository in the same relative location under your `$HOME`.  For example,
if you are working from `~/work/repo` a `work/repo` must exist when you ssh to
the server.

`gitsync` may also be invoked from IDEs or editors like Emacs and Vim.
Typical usage looks like this:

```
$ gitsync jump+megalodon-int-dev.node
Sending 71e4cae0..9be29a91
Resetting jump+megalodon-int-dev.node:work/ae to 9be29a91
Applying uncommitted:
 src/Cargo.toml |    2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

In this example the commit range `71e4cae0..9be29a91` was missing on the
remote server, so those commits were packed up and transferred.  After that,
the remote branch was reset to `9be29a91`.  Finally, a diff of uncommitted
changes was transferred and applied to the remote working tree.  If any of
these things are unnecessary (e.g. there are no missing commits because you've
locally done `git reset --hard HEAD^`), `gitsync` will detect it and skip that
part.

## Emacs

To use `gitsync` from Emacs, you can invoke it as follows:

```lisp
(defvar gitsync-remote-host "jump+megalodon-int-dev.node")

(defun gitsync-run (hostname)
  (save-some-buffers)
  (message "Syncing %s..."
           (string-trim
            (shell-command-to-string
             "git rev-parse --show-toplevel")))
  (shell-command (format "gitsync %s" hostname)))

(global-set-key "\C-\M-u"
                (lambda ()
                  (interactive)
                  (gitsync-run gitsync-remote-host)))
```

## License

`gitsync` is distributed under the terms of both the [MIT
license](https://opensource.org/licenses/MIT) and the [Apache License (Version
2.0)](http://www.apache.org/licenses/LICENSE-2.0).  Contributing changes is
assumed to signal agreement with these licensing terms.

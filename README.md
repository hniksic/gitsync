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

The output tells the user precisely what actions were performed.  In this
example the local `HEAD` was `9be29a91` and there were some uncommitted
changes to `src/Cargo.toml`.  `gitsync` worked out that the commit range
`71e4cae0..9be29a91` was missing from the remote server, so it packed up those
commits and transferred them to the remote.  It then hard-reset the remote
branch to `9be29a91`, updating its `HEAD` and clearing leftover modifications
to the remote working tree, if any.  Finally, a diff of local uncommitted
changes, was transferred and applied to the remote tree.

In general, if any of the described steps are unnecessary (e.g. there are no
new commits because you've locally done `git reset --hard HEAD^`), `gitsync`
will detect that and skip the corresponding action.

## Emacs

The best way to use `gitsync` from Emacs is to bind it to a key.  You can copy
this code to your init file and modify it to suit your needs.

```lisp
(defvar gitsync-default-remote "jump+megalodon-int-dev.node")

;; example key binding - use a key combination you like
(global-set-key "\C-\M-u"
  (lambda () (interactive) (gitsync-run gitsync-default-remote)))

(defun gitsync-run (hostname)
  "Run gitsync on the provided host name."
  (save-some-buffers)
  (message "Syncing %s..."
           (string-trim
            (shell-command-to-string
             "git rev-parse --show-toplevel")))
  (shell-command (format "gitsync %s" hostname)))
```

## License

`gitsync` is distributed under the terms of both the [MIT
license](https://opensource.org/licenses/MIT) and the [Apache License (Version
2.0)](http://www.apache.org/licenses/LICENSE-2.0).  Contributing changes is
assumed to signal agreement with these licensing terms.

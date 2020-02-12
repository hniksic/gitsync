# gitsync

Efficiently sync git-controlled source with a remote server.

Uploads both local git commits and the uncommitted data (in files known to
git) to the specified remote host via SSH.  After running the script, the head
of the current branch and the state of the working tree on the remote side
will match local ones.

To use, just invoke `gitsync <hostname>` in a git-controlled directory.
Before first use you must ensure that the remote host contains a checkout of
the repository in the same relative location under your `$HOME`.  For example,
if you are working from `~/work/repo` a `work/repo` must exist when you ssh to
the server.

## Usage

Typical usage is to run `gitsync hostname` from the shell or from your IDE.
For example:

```
$ gitsync jump+megalodon-int-dev.node
Sending 71e4cae0..9be29a91
Resetting jump+megalodon-int-dev.node:work/ae to 9be29a91
Applying uncommitted:
 src/Cargo.toml |    2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

In this example the commit range `71e4cae0..9be29a91` was found missing on the
remote server, so those commits were packed up and sent.  After that, the
remote branch was reset to `9be29a91`.  Finally, uncommitted changes were
applied to the remote working tree.  If it is unnecessary to send commits
(e.g. because you've locally done `git reset --hard HEAD^`), `gitsync` will
detect it and omit that step.  Likewise, if there are no uncommitted local
changes, `gitsync` won't be "applying uncommitted".

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

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

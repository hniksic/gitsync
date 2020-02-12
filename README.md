# gitsync

Efficiently sync git-controlled source with a remote server.

Uploads both committed and uncommitted data (in files known to git) to the
remote host.  After running the script, the head of the current branch and
the state of the working tree on the remote side will match local ones.

The remote host must contain a checkout of the repository in the same
location under your `$HOME`.  For example, if you are working from
`~/work/repo` a `work/repo` must exist when you ssh to the server.

Usage from Emacs:

```lisp
(defun sync-git (hostname)
  (save-some-buffers)
  (message "Syncing %s..."
           (string-trim
            (shell-command-to-string
             "git rev-parse --show-toplevel")))
  (shell-command (format "gitsync %s" hostname)))

(global-set-key "\C-\M-u"
                (lambda ()
                  (interactive)
                  (sync-git "jump+megalodon-int-dev.node")))
```

## License

`gitsync` is distributed under the terms of both the [MIT
license](https://opensource.org/licenses/MIT) and the [Apache License (Version
2.0)](http://www.apache.org/licenses/LICENSE-2.0).  Contributing changes is
assumed to signal agreement with these licensing terms.

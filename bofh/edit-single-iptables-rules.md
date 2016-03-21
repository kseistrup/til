## Edit single iptables rules

```sh
$ sudo iptables-save > iptables.txt     # dump the current rules
$ ${EDITOR} iptables.txt                # edit/remove relevant line(s)
$ sudo iptables-restore < iptables.txt  # apply the updated rules list
```

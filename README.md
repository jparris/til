# TIL - Today I Learned

Notes about C, Bash, Unix, and ViM.

---
## Meta
This is a page is a selection of notes and tips to make life easier when working in a POSIX environment. Some tips are elementary, and some are fairly specific, sophisticated, or obscure. Most of these notes will work in a vannila enviroment.

## Bash
### Go to the previous directory
`cd -` will take you to the previous directory. The previous directory is stored in the `$OLD_PWD` env var.

### Redirect only stderr.
The trick is to redirect stderr to stdout with `2>&1` and then redirect stdout to the bit bucket with `>/dev/null`. E.g.,
```
<command> 2>&1 >/dev/null | grep "or somthing" 
```

### Searching Man Pages
Use the apropos command to search through man pages.
```
$ apropos /proc
slabinfo (5)         - kernel slab allocator statistics
nfsiostat (8)        - Emulate iostat for NFS mount points using /proc/self/mountstats
xqmstats (8)         - Display XFS quota manager statistics from /proc
```
## 80's Nostalgia 
![80's Nostalgia](https://raw.githubusercontent.com/jparris/til/master/imgs/the_more_you_know.jpg)

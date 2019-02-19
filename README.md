# Linux
Linux reference

# Free disk space
```console
dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | xargs sudo apt-get -y purge
```

# curl 
```console
curl -o Python-3.7.2.tgz https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tgz
```

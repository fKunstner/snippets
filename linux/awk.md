Select arguments (here `ls` for files with `>0` size)
```
ls -l | awk '{if ($5 != 0) print $9}'
```

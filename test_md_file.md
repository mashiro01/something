---
tags: []
created: 2020-06-09T05:42:17.807Z
modified: 2020-06-09T05:53:20.665Z
---

> 学到的小技巧的试验场

```ditaa {cmd=true args=["-E"] hide=true}

+---------------+
|     c1FF      |
|     STACK     |
+---------------+
```

```flow
st=>start: Start:>http://www.google.com[blank]
e=>end:>http://www.google.com
op1=>operation: My Operation
sub1=>subroutine: My Subroutine
cond=>condition: Yes
or No?:>http://www.google.com
io=>inputoutput: catch something...
para=>parallel: parallel tasks

st->op1->cond
cond(yes)->io->e
cond(no)->para
para(path1, bottom)->sub1(right)->op1
para(path2, top)->op1
```

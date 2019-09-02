rank() 排序相同时会重复，总数不会变  

dense_rank()排序相同时会重复，总数会减少

row_number() 会根据顺序计算



```mysql
--案例数据
1   a   10
2   a   12
3   b   13
4   b   12
5   a   14
6   a   15
7   a   13
8   b   11
9   a   16
10  b   17
11  a   14
```



```mysql
select id,
name,
rank()over(partition by name order by sal desc ) rp,
dense_rank() over(partition by name order by sal desc ) drp,
row_number() over(partition by name order by sal desc) rmp
from f_test
```



```mysql
1    1    1
2    2    2
3    3    3
4    4    4
1    1    1
2    2    2
3    3    3
3    3    4
5    4    5
6    5    6
7    6    7
```


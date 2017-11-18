# sed和awk学习

## awk

#### 删除重复行
```
awk '!($0 in array) { array[$0]; print}' file
```

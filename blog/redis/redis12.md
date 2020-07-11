#### bitmap位图

位存储，只要只用两个状态的都可以用

统计疫情感染人数 0未感染 1感染  010001000111101一共14亿个

```
setbit sign 0 1 #第0位为1（只有0和1两个值）
setbit sign 2 0 #第2位为0

getbit sign 2   #查看第2位的值

bitcount sign   #统计为1的个数
```

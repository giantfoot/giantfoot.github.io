### geospatial地理位置

GEOADD key longitude latitude member [longitude latitude member ...]

将指定的地理空间位置（纬度、经度、名称）添加到指定的key中。这些数据将会存储到sorted set这样的目的是为了方便使用GEORADIUS或者GEORADIUSBYMEMBER命令对数据进行半径查询等操作。

该命令以采用标准格式的参数x,y,所以经度必须在纬度之前。这些坐标的限制是可以被编入索引的，区域面积可以很接近极点但是不能索引。具体的限制，由EPSG:900913 / EPSG:3785 / OSGEO:41001 规定如下：

有效的经度从-180度到180度。
有效的纬度从-85.05112878度到85.05112878度。
当坐标位置超出上述指定范围时，该命令将会返回一个错误。

> 它是如何工作的？

sorted set使用一种称为Geohash的技术进行填充。经度和纬度的位是交错的，以形成一个独特的52位整数. 我们知道，一个sorted set 的double score可以代表一个52位的整数，而不会失去精度。

这种格式允许半径查询检查的1 + 8个领域需要覆盖整个半径，并丢弃元素以外的半径。通过计算该区域的范围，通过计算所涵盖的范围，从不太重要的部分的排序集的得分，并计算得分范围为每个区域的sorted set中的查询。

> 使用什么样的地球模型（Earth model）？

这只是假设地球是一个球体，因为使用的距离公式是Haversine公式。这个公式仅适用于地球，而不是一个完美的球体。当在社交网站和其他大多数需要查询半径的应用中使用时，这些偏差都不算问题。但是，在最坏的情况下的偏差可能是0.5%，所以一些地理位置很关键的应用还是需要谨慎考虑。

返回值integer-reply, 具体的:

添加到sorted set元素的数目，但不包括已更新score的元素。
```
设置地点的经纬度
geoadd china:city 116.40 39.90 beijing 106.50 29.53 chongqing
获取经纬度
geopos china:city beijing
获取两个位置的距离
geodist china:city beijing chongqing  km
查找查找附近的人
georadius 以给定的位置为中心，找出某一半径的元素
georadius china:city 110 30 1000km [withdist显示距离][withcoord显示经纬度][count n 显示n个元素]
```

GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]

这个命令和 GEORADIUS 命令一样， 都可以找出位于指定范围内的元素， 但是 GEORADIUSBYMEMBER 的中心点是由给定的位置元素决定的， 而不是像 GEORADIUS 那样， 使用输入的经度和纬度来决定中心点

指定成员的位置被用作查询的中心。

```
redis> GEOADD Sicily 13.583333 37.316667 "Agrigento"
(integer) 1
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEORADIUSBYMEMBER Sicily Agrigento 100 km
1) "Agrigento"
2) "Palermo"
redis>
```

GEOHASH 返回一个或多个位置元素的Geohash表示，返回的事11个字符的geohash字符串
```
将二维的经纬度转换为一维的字符串，如果两个字符串越接近，那么距离越近
geohash china:city bejing chongqing
```

底层使用zset实现，比如
```
zrange china:city 0 -1 查看全部元素
```

GEOADD
GEODIST
GEOHASH
GEOPOS
GEORADIUS
GEORADIUSBYMEMBER

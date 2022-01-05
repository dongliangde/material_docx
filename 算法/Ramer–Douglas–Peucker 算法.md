# Ramer–Douglas–Peucker 算法

## 要素简化

 

对于 LineString 和 Polygon 要素，一个很重要的优化手段就是根据当前的缩放等级进行简化，尽可能减少渲染数据量。简化的依据有二： 

1. 太短的线段 和 太小的多边形 都可以直接过滤掉
2. 一根折线中对整体形状影响较小的顶点可以过滤掉

 线段顶点简化的基础算法是 [Ramer–Douglas–Peucker 算法](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Ramer%25E2%2580%2593Douglas%25E2%2580%2593Peucker_algorithm) ，思路如下： 

1. 首先保留 Polyline 的首尾两个顶点并连线
2. 找到剩余顶点中距离这条线段最远的顶点，保留该距离
3. 如果该距离小于阈值，丢弃
4. 如果该距离大于阈值，保留。并和首尾两个端点分割成两个子线段
5. 分治法处理两个子线段，回到 1

下图（来自 wiki）演示了这个过程：红点被丢弃，绿点被保留。 

![v2-19dbcafe69fd8b560aa51ca576f2587c_b](F:\md资料文件\img\v2-19dbcafe69fd8b560aa51ca576f2587c_b.gif)

​					Ramer–Douglas–Peucker 简化算法 

显然阈值（ε）选取的越大，最后简化后留下的顶点数（n）就越少，Polyline 也就越不平滑：

![微信截图_20210827130457](F:\md资料文件\img\微信截图_20210827130457.png)

我们以 geojson-vt 的实现为例：  

```
export default function simplify(coords, first, last, sqTolerance) {
    let maxSqDist = sqTolerance; // 最大距离
    const mid = (last - first) >> 1; // 找到中点 index
    let minPosToMid = last - first;
    let index; // 最大距离顶点 index
	
  	// 两端点
    const ax = coords[first];
    const ay = coords[first + 1];
    const bx = coords[last];
    const by = coords[last + 1];
		
  	// 找到和线段距离最大的点
    for (let i = first + 3; i < last; i += 3) {
      	// 计算顶点到线段的距离平方
        const d = getSqSegDist(coords[i], coords[i + 1], ax, ay, bx, by);
        if (d > maxSqDist) {
            index = i;
            maxSqDist = d;
        }
    }
		
	// 将距离（平方）与阈值进行比较
    if (maxSqDist > sqTolerance) {
      	// 递归处理前一半线段
        if (index - first > 3) simplify(coords, first, index, sqTolerance);
      	// 保存最大距离，表示这个顶点需要被保留
        coords[index + 2] = maxSqDist;
	      // 递归处理后一半线段
        if (last - index > 3) simplify(coords, index, last, sqTolerance);
    }
}
```


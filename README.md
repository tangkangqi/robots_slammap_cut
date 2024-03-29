# robots_slammap_cut
 This repos shows how could we divide a geography region into some num of sub-region by only using its coordinate information(binary-value map).

## 背景
在规划清扫路线时，假设我们能按区域进行清扫， 可以更容易避免重复清扫。而近两年，的确有的厂家开发了地图划分的功能，分区域清扫，可以便于定位,更便捷。

- 下图是一个扫地机器人对多个房间的原始SLAM地图：


![image](bw.jpg)

实际上，我们可以人为划分房间区域， 通过相邻位置、形状、连通性等，这些只需要利用到房间区域像素点位置信息。


## 预处理

由于维度简单，下面我们将通过数据挖掘中常用的聚类算法对图像进行分割， 尽管聚类常用于数据分类而非图像领域，且是无监督学习不需要标注训练集的， 但我们将发现它们在基于空间坐标的区域分割有很好的效果。

我们的输入是图片像素的X、Y坐标向量。在此之前我们对图像做了二值化预处理，对房间区域（图片中的白色区域），我们的输入变量是实际的x, y坐标， 对非房间区域（图片黑色区域），x, y 坐标 用 -100 替代。 这样能确保他们不干扰到房间区域的分类。


## K-means
目前最常用的聚类算法是K-means，基本思想是通过变量点与K个簇中心点的距离划分簇点群，并不断迭代簇点群与簇中心点直至收敛。通过K-means区域划分效果如下：
![image](kmeans_20200614-16:58:03.png)



我们发现采用K-means，在房间边缘，还会有一些干扰。这是因为K-means 本质是簇间距离均分， 并没有利用到点的连通信息。而实际上我们肉眼划分区域利用到的信息包括了房间的形状、相邻位置、连通关系。

 
## DBSCAN
DBSCAN 是比较常见的基于连通关系（密度可达）的聚类算法，但缺点是不能决定簇的个数，只要两个区域稍有连通就会标记为同一簇：
![image](dbscan_20200614-17:28:36.png)




## 谱聚类
谱聚类Spectural Clustering 则包括了DBSCAN 和Kmeans的优点：可以预设簇的个数， 并且其聚类过程利用了相邻和联通关系。谱聚类可以简单看成对变量的拉普拉斯矩阵的前k个特征向量构成的矩阵的转置再分成k组，而拉普拉斯矩阵包括了邻接矩阵， 这相当于给分类簇间的连通关系设了相似度权重。



如下是谱聚类采用最临近法求邻接矩阵时，按分类簇数从3到14的区域划分情况：
![image](spc_20200614-19:03:07.png)
基本上各簇数下的划分都有按房间区域隔离，簇数等于6～9时最符合我们人眼的感知。


## 结果对比
此外我们也选取了层次聚类、高斯混合模型GMM聚类划分的结果与Kmeans、谱聚类进行对比：

![image](slam5_20200614-18:05:47.png)

可见谱聚类分类最符合我们肉眼划分结果： 边缘无干扰、区域划分稳定。理论上谱聚类利用到连通信息划分也非常适合地理二维坐标点的分类。

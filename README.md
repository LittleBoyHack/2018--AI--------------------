# 2018南京AI应用大赛-智能城市-干线物流供需预测题目


## 1.背景和目标
公路干线物流是这样一种场景：托运方（货主）在运满满平台发布货源信息，通常需要4.2-17米的货车，货源包括但不限于：钢件，化工原料，建材，农产品，快递等。承运方（司机）寻找可以承接的运输任务，然后进行单程的公路运输行为。当前的货运供需市场是不透明的，发货量和司机数量并不匹配，去程和回程也不匹配。供需双方受到很多因素影响。这造成了巨大的资源浪费。比如空载率高，货运价格波动严重等。<br>
本方案的目标是根据提供的历史数据，预测某个地区未来七天的司机量和货物量。是一个典型的时间序列的预测问题。

## 2数据描述
赛题中提供的数据有如下的8类：司机电话日志、货量表、司机量表、货主画像、司机画像、城市公路距离记录、城市天气记录、发货记录。

## 3.解题思路
### （1）利用其周期性求解
司机量和货量数据均呈现出强烈的周期性，并且通过对历史数据的观察发现其循环周期为一个周。本方案可以通过对历史数据建模，求出一个星期内每一天对目标数量的影响权值。进而对6.1-6.7号各地区的司机量和货量进行预测。具体可以用公式描述如下：
y_i=u_i*base,   i=1,2,3,4,5,67
公式中y_i表示未来某地区未来第i天的货物量或者司机量，而u_i表示该地区未来第i天周期影响权重。base表示未来七天的一个周期内的变化的基准值。
### （2）利用天气因素进行校准
司机量和货物量的变化除了和周期相关外和天气因素也有关系。同理，可以通过对历史数据进行建模，求出天气因素对最终司机量和货物量的影响因子。那么，可以用公式描述为：<br>
y_i=(u_i+w_i)*base,   i=1,2,3,4,5,67<br>
w_i 表示未来第i天天气对该地区司机量和货物量的影响值。
### （3）利用邻近周期数据求解base值
由公式可知，u_i和w_i可以通过历史数据求得。但是假想的base值是不知道的，这个值该如何得到呢？思考后得出结论：和待预测的时间段越靠近的时间段其基准base值是最接近的，可以直接通过最邻近周期的数值求解base值，即通过5.24-5.30日的历史数据求解base值。
### （4）base值求解的延拓思考
第（三）步确定了base值提取的数据来源，但是如何求解这个base值呢？最简单的一种思路是求解邻近周期数据的平均值作为其base值。继续深入思考，显然可以找到一个比均值更优秀的base值，这个base值可以是满足在最邻近时间段上的让评价函数（RMSE）损失值最小的值。具体如公式所示：<br>
![image](https://github.com/LittleBoyHack/2018--AI--------------------/blob/master/src/equals.png)<br>
选取上述公式的值作为base值，在实际的验证中让模型性能有了飞跃的提升。再进一步思考，base值不一定是一个单一的数值，最邻近周期内其他的一些统计特征也可以作为base的组成成分。这些因素也可能对需要预测的时间段有影响。因此，在实验中进一步对base值内容进行扩充，引入了邻近时间段内一些统计特征如均值、方差、最大值、最小值、差分、二次差分等特性。实践证明，这些值的引入对预测效果有提升。继续深挖，发现很多提供的数据信息到目前为止都没有用上，这造成了信息的浪费。回头思考发现发货记录表格里有关于近邻周期内发货需求的详细描述，发货记录的具体字段含义如下：
<br>
id：货物id<br>
shipper_id：货主id<br>
start_city_id：始发地城市id<br>
end_city_id：目的地城市id<br>
truck_length：车长<br>
truck_weight：货物重量	<br>
cargo_capacipy：货物体积<br>
cargo_type：货物类型<br>
truck_type：车型<br>
handling_type：装卸方式<br>
expect_freight：运费<br>
day：日期<br>

  该数据记录了各地发货的详细信息，同时有日期标注。考虑到邻近时间段内发货货物的构成成份也可能影响待预测时间段的司机量和货物量。在这里对base的内容进行了进一步的扩充，加上了邻近周期类发货货物的构成成分，如装卸方式、车型、货物重量和货物体积等特征。

  通过上述的四步思考，逐步完善了解题最终的解题思路。进而引入了一种BCKs-RM（Basis-Cycle-Keys Regression Model）回归预测方法。
  <br>
  ![image](https://github.com/LittleBoyHack/2018--AI--------------------/blob/master/src/BCKs-RM.png)
  <br>
该方案通过时间划窗法将将基准(Basis)特征、周期（Cycle）特征和天气特征（Keys）构造融合，并通过xgboost回归算法建立其和未来7天的司机量和货物量的映射关系。


## 4.说明

goods_predict.py:是货物量预测的源码，司机量预测过程思路和过程和这个基本一致，所以没有整理出来。<br>
PPT文件夹：是答辩删减版的PPT,可供参考。<br>
代码是临时整理的，直接运行可能有问题，仅供参考。<br>
思路参考：
[博客链接](https://www.jianshu.com/p/31e20f00c26f?spm=5176.9876270.0.0.3a54e44a5rYfWp)




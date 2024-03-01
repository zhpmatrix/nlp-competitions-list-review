#### 比赛介绍

https://sites.google.com/view/wsdm24-docqa

#### 冠军方案介绍

[wsdm 2024，基于大模型进行多文档问答](https://mp.weixin.qq.com/s/wKjpVYx21SDthwk7ZJ5r1Q)

#### 数据样例介绍

相比其他场景下的数据，增加了**history**的数据。

```
{
"uuid": "xxxxx",
"history": [
 {"question": xxx, "history": xxx},
 {"question": xxx, "history": xxx},
 ...
],
"documents": 
[
"Jun 17th through Fri the 21st, 2024 at the Seattle Convention Center, Vancouver Convention Center.", "Workshops within a “track” will take place in the same room (or be co-located), and workshop organizers will be asked to work closely with others in their track ...", 
...
],
"question": "Where will CVPR 2024 happen?",
"answer": "CVPR 2024 will happen at the Seattle Convention Center, Vancouver.",
"keywords": # Will not be given.
[
"Vancouver", "CVPR 2024", "Seattle Convention Center"
] 
}
```

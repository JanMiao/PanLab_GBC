# PanLab_GBC
该代码仓库是：PanLab用于猪品种鉴别及血统成分分析的参考文档


**下述分析均需要填充好的SNP数据，不允许有缺失位点存在**

## 1. 猪品种鉴别
### a. 如果原始SNP数据不大，可以使用网页进行上传。
推荐使用网页版品种鉴定工具 [iDIGs](http://alphaindex.zju.edu.cn/iDIGs_en/)，步骤如下：
![image](https://github.com/JanMiao/PanLab_GBC/blob/main/images/iDIGs_instruction.PNG)


**注意**
1. 默认情况下，iDIGs会将每一个待预测个体，分类为我们参考库（Repository）里124个品种中的一个。**预计准确性超过98%**。
2. 如果在上图第3步中选择合适的参考品种，可提高预测准确性。**预计几乎完全准确**。
3. 如您上传的个体可能有部分不在这124个品种中，可设置`Detect Anomaly = YES`。对真实品种不在参考库中的个体，iDIGs会将其品种预测为NA。注意在这种情况下，请不要再选择参考品种。该模式下预测准确性会有较大下降，**预计准确性90%左右**。


### b. 如果原始数据较大，不便通过网络上传。
可以使用命令行版本的iDIGs进行分析：
```{R}
# 如果不指定参考品种
Rscript iDIGs_cmd.R -p toy -r v11
# 如果需要指定参考品种
Rscript iDIGs_cmd.R -p toy -r v11 -b DU,LW,LR
```
结果文件为：`report.html` 和 `report.txt`


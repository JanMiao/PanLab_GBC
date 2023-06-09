# PanLab_GBC
该代码仓库是：PanLab用于猪品种鉴别及血统成分分析的指导文档。欢迎大家尝试、提交意见和bug。


下述所有分析均需要填充好的SNP数据，**不允许有缺失位点存在** !!!  
如果不做填充，可以使用如下代码删除所有含有缺失的位点。  
`/disk191/miaoj/software/plink --bfile [inFile] --geno 0 --make-bed --out [outFile]`

## 1. 品种鉴别
### 1.1 如果原始SNP数据不大，可以使用网页进行上传。
推荐使用网页版品种鉴定工具 [iDIGs](http://alphaindex.zju.edu.cn/iDIGs_en/)，步骤如下：
![image](https://github.com/JanMiao/PanLab_GBC/blob/main/images/iDIGs_instruction.PNG)


**注意**
1. 默认情况下，iDIGs会将每一个待预测个体，分类为我们参考库（Repository）里124个品种中的一个。**预计准确性超过98%**。
2. 如果在上图第3步中选择合适的参考品种（每一个待预测个体会被分类为参考品种中的一个品种），可提高预测准确性。**预计几乎完全准确**。
3. 如您上传的个体可能有部分不在这124个品种中，可设置`Detect Anomaly = YES`。对真实品种不在参考库中的个体，iDIGs会将其品种预测为NA。注意在这种情况下，请不要再选择参考品种。该模式下预测准确性会有较大下降，**预计准确性90%左右**。


### 1.2 如果原始数据较大，不便通过网络上传。
可以使用命令行版本的iDIGs进行分析：
```{R}
# 如果不指定参考品种
Rscript /disk195/zz/shinyApp/iPIGs_en/iDIGcmd/iDIGs_cmd.R -p toy -r v11
# 如果需要指定参考品种
Rscript /disk195/zz/shinyApp/iPIGs_en/iDIGcmd/iDIGs_cmd.R -p toy -r v11 -b DU,LW,LR
# 参数说明
p: plink二进制文件前缀
r: 参考基因组版本（v11 或者 v10）
b: 参考品种，以逗号进行分割。默认使用所有品种。（品种缩写请参考iDIGs官网：Repository）
a: 是否对未知品种进行检测 （默认为FALSE）
```
结果文件为：`report.html` 和 `report.txt`


### 1.3 对于WGS数据(SNP数目太多)，使用如下代码先提取相同位点。
```
# Normalize your markerID in bim file and copy files
/disk191/miaoj/software/MakeSNPid file.bim file.tmp.bim
cp file.bed file.tmp.bed
cp file.fam file.tmp.fam
# extract overlapped SNPs
plink --bfile file.tmp --extract /disk195/zz/shinyApp/iPIGs_en/data/MarkerID_11.txt --make-bed --out upload
# then run iDIGs with file.tmp.bed(bim,fam)
```

## 2. 血统成分分析（基于R包 [GBC](https://github.com/JanMiao/GBC)）
### 2.1 使用iDIGs参考数据集
使用类似如下代码。
```
library(GBC)
library(data.table)
library(ggplot2)
# 选择血统来源
breeds=c("LW", "LWH") # 品种缩写请参考iDIGs官网（Repository）
plink_dir="/disk191/miaoj/software/"
gbc = GBCpred(RDS="/disk195/zz/shinyApp/iPIGs_en/data/REF_data11_freq.rds", test_prefix="LL_LW_merge.phase", breedused=breeds , nmarkers=NULL, testMode = FALSE, method="lm", plink_dir=plink_dir)
#gbc2 = GBCpred(RDS="/disk195/zz/shinyApp/iPIGs_en/data/REF_data11_freq.rds", test_prefix="LL_LW_merge", breedused=breeds , nmarkers=NULL, testMode = FALSE, method="lasso", plink_dir=plink_dir)
# 画图
p = GBCplot(GBCres=gbc, FontSize=10)
ggsave("gbc.pdf", p)
```

### 2.2 自建参考数据集
参考[GBC](https://github.com/JanMiao/GBC)说明文档。

### 2.3 天邦GBC数据集
为天邦的安徽池州场和广西贵港场分别构建了参考集，数据文件位于`/disk192/miaoj/GBC/TB_GBC`  
**由于天邦数据的位点较多，使用下述新代码进行GBC分析。**


```
cd /disk192/miaoj/GBC/TB_GBC/test
library(GBC)
library(Rcpp)
library(data.table)
library(ggplot2)
source("/disk192/miaoj/GBC/TB_GBC/data_check.R")
source("/disk192/miaoj/GBC/TB_GBC/TB_GBC.R")
### 1. set parameters
plink_dir = "/disk191/miaoj/software/"
test_prefix="GXGGtest"
breedused = c("TB5", "Duroc", "Yorkshire", "Pietrain", "Landrace") # GXGG
# breedused = c("Duroc", "Yorkshire", "Pietrain", "Landrace") # AHCZ

### 2. 检查test数据集中的染色体数目及是否存在重复位点
check_data(test_prefix = test_prefix)
# 如果有非常规染色体需要删除，如仅仅保留1-18号染色体
chrs = seq(1,18)
rm_chr(plink_dir=plink_dir, test_prefix=test_prefix, keepChr=chrs, out_prefix="ab")
# 删除重复位点
rmDupSNP(plink_dir=plink_dir, test_prefix=test_prefix,out_prefix = "noDup")

### 3. GBC analysis
gbc = GBCpreds(RDS="/disk192/miaoj/GBC/TB_GBC/GXGG_frequency.rds", test_prefix=test_prefix, breedused=breedused, nmarkers=NULL, testMode = TRUE, method="lm", plink_dir=plink_dir)
#gbc = GBCpreds(RDS="/disk192/miaoj/GBC/TB_GBC/AHCZ_frequency.rds", test_prefix=test_prefix, breedused=breedused, nmarkers=NULL, testMode = TRUE, method="lm", plink_dir=plink_dir)
# admixture plot
p = GBCplot(GBCres=gbc, FontSize=5)
ggsave("gbc.pdf", p)
# label outliers in PCA plot
source("/disk192/miaoj/GBC/TB_GBC/PCAplot1.R")
# The 'breed_file' parameter needs to specify a file that is tab-separated, with the individual ID in the first column and the corresponding breed information in the second column.
PCAplots(plink_prefix=test_prefix, sampleID = NULL, breed_file="breed.txt", PDFprefix="test", sampleLabel = c("A","B"), plink_dir=plink_dir)
ggsave("PCA_outliers.pdf", PCAplot)
```


## 3. 其他问题
### 3.1 如何debug其他用户运行失败的iDIGs网页任务？
**品种鉴定分析**
```
# 拷贝出现bug的job的目录
cd /disk195/zz/shinyApp/iPIGs_en/temp/
cp -r 1679447939/ debug/
cd debug

# 加载 & 查看用户所用参数
load("par.RData")
res
# 一行行运行R脚本，查找问题
/disk195/zz/shinyApp/iPIGs_en/debug/debug.R
```
**Panel 设计**
```
# 拷贝出现bug的job的目录
cd /disk195/zz/shinyApp/iPIGs_en/temp/
cp -r 1679447939/ debug/
cd debug

# 加载 & 查看用户所用参数
load("par.RData")
res
# 一行行运行R脚本，查找问题
/disk195/zz/shinyApp/iPIGs_en/analysisScript/Panel.R
```

### 3.2 如何调整GBC结果的图片？
```
# 基于 #2 中获得的gbc
GBCres=gbc
nbreed <- ncol(GBCres)
nsample <- nrow(GBCres)
Sample <- rep(rownames(GBCres), each = nbreed)
Breeds <- rep(colnames(GBCres), nsample)
propotion <- as.vector(t(GBCres))
res <- data.frame(Sample, Breeds, propotion)

# 修改下面ggplot代码
library(ggplot2)
p <- ggplot2::ggplot(res, aes(x = Sample, y = propotion, fill = Breeds)) +
  geom_bar(position = "stack", stat = "identity") +
  labs(x = "Unknown Animals", y = "Proportion") +
  theme(
    axis.text.x = element_text(face = "bold", size = FontSize, color = "darkblue", angle = 270, vjust=0.35),
    axis.title = element_text(face = "bold", size = 12, color = "brown")
  )
ggsave("gbc.pdf", p)

```
## 4. 常见问题（FAQ）
### 4.1 出现如下报错：`Error in AFMatrix[breedused, ] : subscript out of bounds`
这是因为使用的品种缩写名有误。请参考[iDIGs](http://alphaindex.zju.edu.cn/iDIGs_en/)官网下**Repository**子页面获取正确的品种缩写名。

### 4.2 出现如下报错：`cannot open compressed file '{path}/{prefix}GBCdata.rds'， probable reason 'No such file or directory'`
这是因为plink前缀参数中只能包含plink文件前缀，不能包含文件路径。我们建议将原始数据软连接到当前目录后运行。

### 4.3 如果存在一些函数不存在的bug，可重新安装GBC的R包。
```
### 卸载包
remove.packages("GBC")
# 退出R & 重新启动
在Rstudio中使用： .rs.restartR()
# 重新安装GBC
install.packages("/disk192/miaoj/GBC/GBC_0.5.tar.gz", repos = NULL)
library(GBC)
```


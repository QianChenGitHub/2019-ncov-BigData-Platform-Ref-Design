# 一种可用于2019武汉冠状病毒疫情控制的大数据处理平台参考设计
项目目的
    本项目旨在提议一个可用于抗击2019 武汉冠状病毒的大数据处理平台，用于处理全民疫情大数据，记录全民的疫情相关数据，并处理这些数据：
1. 实时挖掘可能的疫情传染警报信息通知相关人员。
2. 分析感染人员的历史数据，生成疫情接触感染树，用于指导发现疑似病例及时采取隔离措施以降低疫情进一步传染扩散。

    本项目处在早期构思阶段，目前仅有一些前期类似项目所用的源代码可参考或重用于本平台的开发。
数据模型所涉及数据所有单位较多，数据获取比较困难，本设计只是提出一个能够处理这些数据的系统架构，具体的数据获取途径与权限不在本项目的设计范畴。
欢迎感兴趣的朋友一起参与开发，也希望本项目能对正在进行类似项目开发的相关研发单位的开发人员有一定的参考作用。

    大数据处理平台参考设计系统架构
![Demo Image](https://github.com/charlesDataCenterFPGA/2019-ncov-BigData-Platform-Ref-Design/blob/master/architecture.png)

系统工作流程：
数据导人与实时流处理阶段：
实时动态获取的原始信息数据源通过网关（Gateway）上传到数据中心的Kafka 服务器收集并统一管理与分发，原始数据可以采用avro格式编码。

    新导入数据首先在流处理引擎（这里为spark streaming）里被用于实时分析，如果有危险情况立即发送信息给相关人员： 
个体快速实时分类（初步分析）
1. 如果自己是病源给当前在同一个范围内的人发送信息已警示危险，同时抄送公安机关。
2. 如果自己所在区域有其他病例，则给本人发送信息告知危险，同时抄送公安机关。

    Kafka服务器上的数据除了用于实时分析还被通过streamSets转换成Parquet格式并导入到大数据分布式存储系统HDFS，用于后续历史数据批处理(这里使用Impala用于数据表格查询)并生成感染树： 
1. 深入挖掘数据信息，以及关联信息。
2. 生长病例关联树：接触记录，扩散范围 =》 生成数状图，人员关系视图，地图视图

    除了通过kafka服务器收集实时数据，本系统平台还支持将已有数据库管理平台的数据ETL 导入到本平台中，并统一交由批处理服务器（这里为Impala）进型历史数据处理。

    最后生成的人员关系图以及地图视图的可视化工作在Solr服务器中完成。

数据模型schema 总览
![Demo Image](https://github.com/charlesDataCenterFPGA/2019-ncov-BigData-Platform-Ref-Design/blob/master/%E6%80%BB%E8%A7%88.PNG)

流数据schema 用于判断实时数据的状态，并在必要情况下生成警报信息
![Demo Image](https://github.com/charlesDataCenterFPGA/2019-ncov-BigData-Platform-Ref-Design/blob/master/%E6%B5%81%E6%95%B0%E6%8D%AE.PNG)

分析诊疗状态记录
1. 如果不为空而且!=0 则发出警报信息给相关人员包含：身份证，当前地点（查询位置表格），诊疗状态 
2. 如果诊疗状态为空，则判断体温，如果体温>37.5摄氏度则发出警报信息给相关人员：身份证，当前地点（查询位置表格），体温值

历史数据
![Demo Image](https://github.com/charlesDataCenterFPGA/2019-ncov-BigData-Platform-Ref-Design/blob/master/%E5%8E%86%E5%8F%B2%E6%95%B0%E6%8D%AE.PNG)

流行病史分析算法（初步想法，待完善，需要医疗相关专业人士帮助）:
    如果手机漫游到过武汉，或者有GPS至武汉的记录，或者现场交易电子支付记录，检查行踪范围是否为被污染区，或者是否有跟病例交易记录则判定为疑似病例。
进一步查询该疑似病例的诊疗状态表格，如果状态为已经被确诊，则分析该样本的所有出行轨迹，包含“个人出行记录数据”，“高危地点出现记录数据”，“电商购物交易记录数据”，“电子支付交易记录数据”，“蜂窝漫游数据”，将所有这些记录中出现的相关人员(例如:同一时段乘坐了同一公共交通工具)生成一级树形，被接触人为树枝，当前人为生长这些树枝的树结点。
    批处理服务可以在服务器非高峰运行时段（例如夜间）循环往复上述分析，并生长树状图。
    工作人员可以随时发起历史查询指令，来对系统最新的分析结果生成可视化的报表。

##### 无法使用虚拟环境，虚拟环境会缺系统包 报错ImportError: No module named 'encodings'

1. 打包整个原始安装包

2. 打包环境
zip -r py_env.zip pyspark_env/

```shell


# 压缩文件名 py_env.zip
# 环境文件夹 python3
# cluster提交
spark-submit --master yarn --deploy-mode cluster    --archives hdfs://iteach-cdh-01/user/py_env.zip#PY2  --queue root.spark     --name beike_used      --jars /xdfdata/LogSpark/thridLib/mysql-connector-java-8.0.19.jar     --files /xdfdata/PySpark/hive-site.xml   --conf spark.yarn.appMasterEnv.PYSPARK_PYTHON=./PY2/python3/bin/python3 --conf spark.yarn.appMasterEnv.PYSPARK_DRIVER_PYTHON=./PY2/python3/bin/python3  --conf "spark.executor.extraClassPath=/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/hive/lib/*"     --num-executors 10     --driver-memory 4G     --executor-cores 8     --executor-memory 6G     --py-files /xdfdata/iteach-bigdata-prism/dblib.zip /xdfdata/iteach-bigdata-prism/beike_used.py
```



### 主要指定 driver和executor Python版本

```shell
--archives hdfs:///user/root/py_env.zip#PY2 
--conf spark.yarn.appMasterEnv.PYSPARK_PYTHON=./PY2/py_env/bin/python
--conf spark.yarn.appMasterEnv.PYSPARK_DRIVER_PYTHON=./PY2/py_env/bin/python 
```


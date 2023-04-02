
Learn How to Interrelate Apache Hudi with Redshift Spectrum Hands on Labs
![Capture](https://user-images.githubusercontent.com/39345855/229358201-787c168b-760d-4234-8342-a4251c1e9585.JPG)

# Video Link

### Glue Job 
```
try:
    import sys
    import os
    from pyspark.context import SparkContext
    from pyspark.sql.session import SparkSession
    from awsglue.context import GlueContext
    from awsglue.job import Job
    from awsglue.dynamicframe import DynamicFrame
    from pyspark.sql.functions import col, to_timestamp, monotonically_increasing_id, to_date, when
    from pyspark.sql.functions import *
    from awsglue.utils import getResolvedOptions
    from pyspark.sql.types import *
    from datetime import datetime, date
    import boto3
    from functools import reduce
    from pyspark.sql import Row

    import uuid
    from faker import Faker
except Exception as e:
    print("Modules are missing : {} ".format(e))


args = getResolvedOptions(sys.argv, ["JOB_NAME"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

#
# spark = (SparkSession.builder.config('spark.serializer', 'org.apache.spark.serializer.KryoSerializer') \
#          .config('spark.sql.hive.convertMetastoreParquet', 'false') \
#          .config('spark.sql.catalog.spark_catalog', 'org.apache.spark.sql.hudi.catalog.HoodieCatalog') \
#          .config('spark.sql.extensions', 'org.apache.spark.sql.hudi.HoodieSparkSessionExtension') \
#          .config('spark.sql.legacy.pathOptionBehavior.enabled', 'true').getOrCreate())
#
# sc = spark.sparkContext
# glueContext = GlueContext(sc)
# job = Job(glueContext)
# logger = glueContext.get_logger()

# =================================INSERTING DATA =====================================
global faker
faker = Faker()


class DataGenerator(object):

    @staticmethod
    def get_data():
        return [
            (
                uuid.uuid4().__str__(),
                faker.name(),
                faker.random_element(elements=('IT', 'HR', 'Sales', 'Marketing')),
                faker.random_element(elements=('CA', 'NY', 'TX', 'FL', 'IL', 'RJ')),
                str(faker.random_int(min=10000, max=150000)),
                str(faker.random_int(min=18, max=60)),
                str(faker.random_int(min=0, max=100000)),
                str(faker.unix_time()),
                faker.email(),
                faker.credit_card_number(card_type='amex'),

            ) for x in range(100)
        ]




# ============================== Settings =======================================
db_name = "hudidb_raw"
table_name = "employees"
recordkey = 'emp_id'
precombine = "ts"
PARTITION_FIELD = 'state'
path = "s3://sql-server-dms-demo/tmp1/"
method = 'upsert'
table_type = "COPY_ON_WRITE"
# ====================================================================================

hudi_part_write_config = {
    'className': 'org.apache.hudi',

    'hoodie.table.name': table_name,
    'hoodie.datasource.write.table.type': table_type,
    'hoodie.datasource.write.operation': method,
    'hoodie.datasource.write.recordkey.field': recordkey,
    'hoodie.datasource.write.precombine.field': precombine,

    'hoodie.datasource.hive_sync.mode': 'hms',
    'hoodie.datasource.hive_sync.enable': 'true',
    'hoodie.datasource.hive_sync.use_jdbc': 'false',
    'hoodie.datasource.hive_sync.support_timestamp': 'false',
    'hoodie.datasource.hive_sync.database': db_name,
    'hoodie.datasource.hive_sync.table': table_name,

}

data = DataGenerator.get_data()
columns = ["emp_id", "employee_name", "department", "state", "salary", "age", "bonus", "ts", "email", "credit_card"]
spark_df = spark.createDataFrame(data=data, schema=columns)
print("**")
print(spark_df.show())
print("**")
spark_df.write.format("hudi").options(**hudi_part_write_config).mode("append").save(path)

"""
--additional-python-modules faker==11.3.0
--conf spark.serializer=org.apache.spark.serializer.KryoSerializer --conf spark.sql.hive.convertMetastoreParquet=false
--datalake-formats hudi

"""
```

### Redshift Spectrum 
```
create external schema landing
from data catalog
database 'hudidb_raw'
iam_role 'aXXXXXXXXXXXXXXXX'
region 'us-east-1';
create external database if not exists;
```

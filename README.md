# Home_sales

# Define Spark version
spark_version = 'spark-3.5.4'
os.environ['SPARK_VERSION'] = spark_version

# Install Java and Spark
!apt-get update
!apt-get install openjdk-11-jdk-headless -qq > /dev/null
!wget -q https://downloads.apache.org/spark/$SPARK_VERSION/$SPARK_VERSION-bin-hadoop3.tgz
!tar xf $SPARK_VERSION-bin-hadoop3.tgz
!pip install -q findspark

# Set Environment Variables
os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-11-openjdk-amd64"
os.environ["SPARK_HOME"] = f"/content/{spark_version}-bin-hadoop3"

# Initialize PySpark
import findspark
findspark.init()

 Create a Spark Session

from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("HomeSalesColab").getOrCreate()

Data Ingestion

The dataset is stored in an AWS S3 bucket and can be loaded directly into a PySpark DataFrame:

from pyspark import SparkFiles
url = "https://2u-data-curriculum-team.s3.amazonaws.com/dataviz-classroom/v1.2/22-big-data/home_sales_revised.csv"
spark.sparkContext.addFile(url)
sales_df = spark.read.csv(SparkFiles.get("home_sales_revised.csv"), sep=",", header=True)
sales_df.show()

Data Analysis Workflow

The notebook performs the following operations:

1. Create a Temporary View

sales_df.createOrReplaceTempView("home_sales")

2. Compute Average Price for 4-Bedroom Homes Sold Per Year

sql = """
    SELECT YEAR(date) AS year, ROUND(AVG(price), 2) AS avg_price
    FROM home_sales
    WHERE bedrooms = 4
    GROUP BY year
    ORDER BY year DESC
"""
spark.sql(sql).show()

3. Compute Average Price of 3-Bedroom, 3-Bathroom Homes

sql = """
    SELECT date_built as year_built, ROUND(AVG(price), 2) AS avg_price
    FROM home_sales
    WHERE bedrooms = 3 AND bathrooms = 3
    AND sqft_living >= 2000
    AND floors = 2
    GROUP BY year_built
    ORDER BY year_built DESC
"""
spark.sql(sql).show()

4. Compute Average Home Price Per "View" Rating

import time
start_time = time.time()
sql = """
    SELECT view, ROUND(AVG(price), 2) AS avg_price
    FROM home_sales
    GROUP BY view
    HAVING avg_price >= 350000
    ORDER BY view DESC
"""
spark.sql(sql).show()

print("--- %s seconds ---" % (time.time() - start_time))

5. Cache the Table and Optimize Query Performance

spark.sql("CACHE TABLE home_sales")
print("Is Cached?", spark.catalog.isCached('home_sales'))

6. Re-run Cached Query & Compare Runtime

start_time = time.time()
spark.sql(sql).show()
print("--- %s seconds ---" % (time.time() - start_time))

7. Partition Data by "date_built" and Save in Parquet Format

sales_df.write.partitionBy("date_built").parquet('p_home_sales', mode='overwrite')

8. Read & Query Parquet Data

df_parquet = spark.read.parquet('p_home_sales')
df_parquet.createOrReplaceTempView('p_home_sales')
spark.sql(sql).show()

9. Uncache the Table

spark.sql("UNCACHE TABLE home_sales")
print("Is Cached?", spark.catalog.isCached("home_sales"))

Running the Notebook in Google Colab

Open Google Colab

Upload or open the Home_Sales.ipynb file

Run all cells sequentially

Analyze the output and query performance

Performance Insights

Caching the table improves query execution speed.

Partitioning by date_built optimizes query performance.

Parquet format provides better read performance compared to CSV

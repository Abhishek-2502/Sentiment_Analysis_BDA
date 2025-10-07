# How can sentiment analysis of publicly available product reviews help in predicting market trends and consumer behavior?
 - **Tools:** Apache Spark (ML for sentiment analysis), MongoDB (storage), Hive (querying datasets), HDFS (storage).
 - **Expected Outcome:** Predict market trends and consumer behavior based on sentiment analysis of product reviews.

## Pipeline Flow

1. **CSV Input → HDFS**

   * Upload all product review CSVs to HDFS (`/SentimentAnalysis/Input`).

2. **Hive Table**

   * Create `sentimentdb.reviews` as an external table pointing to HDFS CSVs.
   * Query and explore raw review data.

3. **Apache Spark ML**

   * Load reviews from Hive.
   * Preprocess text: tokenize → remove stopwords → TF features.
   * Train logistic regression to predict sentiment (positive/negative).
   * Store predictions to MongoDB and aggregate trends per brand/category.

4. **MongoDB Storage**

   * Stores detailed review sentiment predictions (`results` collection).
   * Can be queried directly or used for dashboards.

5. **Trend Analysis in Hive**

   * Aggregate metrics: `positive_percentage`, `recommend_percentage`, `total_reviews`.
   * Identify best-performing categories and brands.

6. **Visualization**

   * Connect MongoDB or Hive to Power BI / Tableau for dashboards.
   * Track market trends, sentiment, and customer recommendation patterns.

✅ **Outcome:**

* Each product review gets a sentiment prediction.
* Aggregated trends per brand/category are stored in Hive (`trend_summary`) and MongoDB (`results`).
* Dashboards can visualize market trends and customer behavior patterns.

        +---------------------+
        | Product Review CSVs |
        +---------------------+
                    |
                    v
        +---------------------+
        |     HDFS Storage    |
        |  (/Sentiment/Input) |
        +---------------------+
                    |
                    v
        +---------------------+
        |        Hive         |
        |  (reviews table)    |
        +---------------------+
                    |
                    v
        +---------------------+
        |   Apache Spark ML   |
        | (Tokenize + TF-IDF  |
        | + Logistic Model)   |
        +---------------------+
                    |
                    v
        +---------------------+
        |     MongoDB Atlas   |
        | (Predictions Store) |
        +---------------------+
                    |
                    v
        +---------------------+
        | Power BI / Tableau  |
        | (Market Trend Viz)  |
        +---------------------+

## Dataset Link
```
https://www.kaggle.com/datasets/datafiniti/consumer-reviews-of-amazon-products?resource=download
```

## Directory Structure
```plaintext
SentimentAnalysis/
├── amazon_product_reviews.csv
├── SentimentAnalysis.scala
├── SentimentAnalysis.jar     # Auto Generated in further steps
├── classes/                  
└── README.md
```

## HDFS (CSV) 
### Upload product review CSVs to HDFS.
```
hdfs dfs -mkdir /SentimentAnalysis
hdfs dfs -mkdir /SentimentAnalysis/Input
```

```
hdfs dfs -put /home/talentum/Desktop/SentimentAnalysis/amazon_product_reviews.csv /SentimentAnalysis/Input
```



## HIVE (SentimentDB.Reviews)
```
hive
```

### Create sentimentdb.reviews as an external table pointing to HDFS CSVs.
```
CREATE DATABASE sentimentdb;
USE sentimentdb;
```

```
DROP TABLE IF EXISTS sentimentdb.reviews;
```

```
CREATE EXTERNAL TABLE sentimentdb.reviews (
  id STRING,
  dateAdded STRING,
  dateUpdated STRING,
  name STRING,
  asins STRING,
  brand STRING,
  categories STRING,
  primaryCategories STRING,
  imageURLs STRING,
  keys STRING,
  manufacturer STRING,
  manufacturerNumber STRING,
  reviews_date STRING,
  reviews_dateSeen STRING,
  reviews_didPurchase STRING,
  reviews_doRecommend STRING,
  reviews_id STRING,
  reviews_numHelpful STRING,
  reviews_rating STRING,
  reviews_sourceURLs STRING,
  reviews_text STRING,
  reviews_title STRING,
  reviews_username STRING,
  sourceURLs STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
  "separatorChar" = ",",
  "quoteChar" = "\"",
  "escapeChar" = "\\"
)
STORED AS TEXTFILE
LOCATION '/SentimentAnalysis/Input'
TBLPROPERTIES("skip.header.line.count"="1");
```

### Query and explore raw review data.
```
SELECT name, brand, reviews_rating, reviews_text 
FROM sentimentdb.reviews 
LIMIT 10;
```



## DOCKER
### Install Docker
```
sudo apt install docker.io -y
```

### Give Permission
```
sudo usermod -aG docker $USER && newgrp docker
```
### Pull and Run MongoDB
```
docker pull mongo:4.4
```
```
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v ~/mongodb/data:/data/db \
  mongo:4.4
```
 


## SPARK (Sentiment + Trend Analysis)
### Install Scala
```
sudo apt install scala
```
### Check Spark Path
```
echo $SPARK_HOME
```

### Create SentimentAnalysis.scala and paste code from SentimentAnalysis.scala

### Make classes directory
```
mkdir -p classes
```

### Compile Spark Job

```
scalac -classpath "$(hadoop classpath):/home/talentum/spark/jars/*" -d classes SentimentAnalysis.scala
```

```
jar -cvf SentimentAnalysis.jar -C classes/ .
```

### Run Spark Job
```
spark-submit \
  --class SentimentAnalysis \
  --master local[*] \
  --packages org.mongodb.spark:mongo-spark-connector_2.11:2.4.3 \
  SentimentAnalysis.jar
```




## Analyze Market Trends (Hive)

### View Sentiment Summary
 - Aggregate metrics: positive_percentage, recommend_percentage, total_reviews. 
```
SELECT * FROM sentimentdb.trend_summary LIMIT 10;
```

### Identify Best Categories
```
SELECT category, AVG(positive_percentage) AS avg_positive
FROM sentimentdb.trend_summary
GROUP BY category
ORDER BY avg_positive DESC;
```

### Predict Consumer Behavior
```
SELECT brand, recommend_percentage, positive_percentage
FROM sentimentdb.trend_summary
ORDER BY recommend_percentage DESC;
```

## Explore in MongoDB (Storage for dashboard/trend visualization)
### Check predictions stored for dashboards:
```
docker exec -it container_id bash
```
```
mongo
use sentimentdb
```

```
db.results.find().limit(5).pretty()
```


## Visualization

 - Connect MongoDB or Hive to Power BI / Tableau.

 - Track market trends, sentiment, and customer recommendation patterns.

## Author

 - Abhishek Rajput
 - Arnav Jain
 - Aayusha Bhatia
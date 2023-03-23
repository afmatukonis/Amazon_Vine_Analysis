# Amazon_Vine_Analysis
Module-17

The purpose of this challenge is to analyze Amazon reviews written by members of the paid Amazon Vine program using Amazon Web Services (AWS) and Cloud Based Data Extraction (ETL). We were given over 50 datasets to choose from and then we used PySpark to perfrom the ETL provess to extract the data, transform the data, connect to the AWS RDS instance and then load the transformed data into pgAdmin. Then we used Pandas to determine if there was any bias towards favorable reviews from the Vine members. 

**Analysis**
**Deliverable 1: Perform ETL on Amazon Product Reviews**

I selected the shoes dataset to analyze and the analysis was performed as follows in Google Colabs.

![Screenshot 2023-03-23 at 1 24 19 AM](https://user-images.githubusercontent.com/118235205/227144808-3cf8a948-7d6f-4105-ae25-1dd536b903d5.png)

First I imported the different drivers and downloaded the Postgres driver. 
!wget https://jdbc.postgresql.org/download/postgresql-42.2.16.jar

Then a spark session was crreated and the Postgres driver was loaded into Spark. 
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("M17-Amazon-Challenge").config("spark.driver.extraClassPath","/content/postgresql-42.2.16.jar").getOrCreate()

The shoe dataset was loaded into the Spark DataFrame. 
from pyspark import SparkFiles
url = "https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Shoes_v1_00.tsv.gz"

spark.sparkContext.addFile(url)

df = spark.read.option("encoding", "UTF-8").csv(SparkFiles.get("amazon_reviews_us_Shoes_v1_00.tsv.gz"), sep="\t", header=True, inferSchema=True)

df.show()

In pgAdmin I created four tables using the schema provided. 

![Screenshot 2023-03-23 at 1 29 25 AM](https://user-images.githubusercontent.com/118235205/227146055-e773c10e-e209-4177-9f85-bc4a9a628a1d.png)

Back in Google Colabs I took the dataset that we imported and created four new DataFrames to match the tables. 

- customers_table 

customers_df = df.groupby("customer_id").agg({"customer_id": 'count'}).withColumnRenamed("count(customer_id)", "customer_count")
customers_df.show()

![Screenshot 2023-03-23 at 1 07 12 AM](https://user-images.githubusercontent.com/118235205/227146483-38e508a5-45c8-4921-a98e-9ded791aec74.png)

- products_table

products_df = df.select(["product_id", "product_title"]).drop_duplicates()
products_df.show()

![Screenshot 2023-03-23 at 1 07 20 AM](https://user-images.githubusercontent.com/118235205/227154433-745e2dcd-9781-49bb-80db-ede41efb110d.png)

- review_id_table

review_id_df = df.select(['review_id', 'customer_id', 'product_id', 'product_parent', to_date("review_date", 'yyyy-MM-dd').alias("review_date")])
review_id_df.show()

![Screenshot 2023-03-23 at 1 07 27 AM](https://user-images.githubusercontent.com/118235205/227154330-2640dd0e-5fb7-4bfc-8a3d-8b5a3d372714.png)

- vine_table

vine_df = df.select("review_id", "star_rating", "helpful_votes", "total_votes", "vine", "verified_purchase")
vine_df.show()

![Screenshot 2023-03-23 at 1 07 33 AM](https://user-images.githubusercontent.com/118235205/227154383-883b27d2-9b78-4848-a2ad-e629bfbf9caa.png)

Then we used AWS RDS to connect to the pgAdmin Postgres database. 

![Screenshot 2023-03-23 at 2 05 21 AM](https://user-images.githubusercontent.com/118235205/227154681-8a7f96e5-9d4c-42f8-ba9d-00a34bb9672d.png)

The four dataframes were then written to the tables we created using AWS. 

They appear as these:

customers_table:
![Screenshot 2023-03-23 at 1 09 15 AM](https://user-images.githubusercontent.com/118235205/227154952-fd879842-1bc8-4b1a-b236-1a7bb964c62e.png)


product_table:
![Screenshot 2023-03-23 at 1 09 49 AM](https://user-images.githubusercontent.com/118235205/227154990-8555bc76-c554-45eb-b9a2-7f7cdb75a20c.png)

review_id_table:
![Screenshot 2023-03-23 at 1 10 22 AM](https://user-images.githubusercontent.com/118235205/227155144-e50649db-d756-4825-8041-7376f70b0293.png)

vine_table:
![Screenshot 2023-03-23 at 1 11 00 AM](https://user-images.githubusercontent.com/118235205/227155217-7b82c693-3d3a-4ede-9849-538c762dee63.png)

Finally in pgAdmin we exported the vine_table as vine_table.csv for different Pandas calculations.

**Deliverable 2: Determine Bias of Vine Reviews**

Using the vine_table.csv file we imported that into jupyter notebook and used Pandas to perform a analysis on the reviews. 

First we filtered the data for all the rows where the total votes count was greater than 20. Then we filtered that to retrieve all the rows where the number of helpful votes/total votes was greater than 50%. Then we did two different filters on the previous filtered dataset, one to retrieve all the rows with reviews throught the Vine program (vine == Y) and one to all the others (vine == N). 

We then got the overall total number of reviews as 27,009. From this we found that there were 22 paid reivews for the shoes on Amazon, and 13 had five star reviews meaning that 59% of paid reviews had a 5 star rating. We found that there were 26,987 unpaid reviews with 14,475 five star reviews, meaning 54% of unpaid reviews had a five star rating. 

![Screenshot 2023-03-23 at 2 16 40 AM](https://user-images.githubusercontent.com/118235205/227157419-445a9f1b-0315-4d2c-860c-c8a659b96b81.png)

![Screenshot 2023-03-23 at 2 16 50 AM](https://user-images.githubusercontent.com/118235205/227157456-8e9b03d6-2b3a-45b1-8dff-f45b5805e05a.png)

**Summary**

From the analyis we found that 59% of paid reviews had a 5-star rating and 54% of unpaid reviews had a 5-star rating. When Looking at the total number of reviews we can see that based on percentage there seems to be a small bias towards Vine Member reviews since paid reviews were at 59% and unpaid were at 54%.

**Suggested Additional Analysis**

To further see if there is a bias towards Vine members and five-star reviews I would perform the same analysis on different datasets. Just becasue there is not a bais towards shoes and paid reivews does not mean that there is not a bias on other itmes. I would also check four star reviews as well to see if there is a larger number of four star reivews for paid members. 

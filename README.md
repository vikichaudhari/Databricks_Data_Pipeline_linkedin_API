Data extraction from any LinkedIn page for roles.

Below Python script that performs a Google Custom Search to find LinkedIn profiles related to the given keyword, location, and experience criteria. It then collects and prints the LinkedIn profile links.
Note- we can extract max 100 profile links at single run.
Make sure to replace 'YOUR_GOOGLE_CUSTOM_SEARCH_API_KEY' and 'YOUR_SEARCH_ENGINE_ID' with your actual Google Custom Search API key and search engine ID.

import requests
import time  # Import the time module to use the sleep function

# Define your Google Custom Search API key and search engine ID
api_key = 'Your API KEY'
search_engine_id = 'Your Engine Id'

keyword = "AWS DevOps"
place="India"
experience = "5+ years Experience"
links = []

results_per_page = 10
total_results = 10
num_pages = (total_results + results_per_page - 1) // results_per_page

for page in range(num_pages):
    start_index = page * results_per_page

    # Define the URL for the Google Custom Search API
    url = f'https://www.googleapis.com/customsearch/v1?key={api_key}&cx={search_engine_id}&q={keyword}+{place}+{experience}+-intitle:"profiles"+-inurl:"dir/"+site:in.linkedin.com/in/+OR+site:in.linkedin.com/pub/&as_oq=bachelor+degree+licence&start={start_index}'
    response = requests.get(url)

    if response.status_code == 200:
        data = response.json()

        for item in data.get('items', []):
            link = item.get('link')
            if link and '//in.linkedin.com/in/' in link:
                links.append(link)
        
        # Introduce a delay of 1 second between requests to avoid exceeding the rate limit
        time.sleep(1)
    elif response.status_code == 429:
        # If you get a 429 status code, you're making requests too quickly, so introduce a longer delay
        time.sleep(10)  # Sleep for 10 seconds and then retry
    else:
        print(f"Failed to retrieve page {page + 1}. Status code: {response.status_code}")

print("Collected LinkedIn profile links:")
print(links)



















RapidAPI service to fetch LinkedIn profile data for the collected LinkedIn profile links.
In Below code 
the get_profile() function sends a request to the RapidAPI endpoint with the LinkedIn profile URL and retrieves the profile data. The retrieved data is then appended to the json_responses list. The script handles exceptions in case there are issues with fetching data for a particular profile URL.

import requests
def get_profile(link):
    url = "https://fresh-linkedin-profile-data.p.rapidapi.com/get-linkedin-profile"

    querystring = {"linkedin_url": link+"/"}

    headers = {
        "X-RapidAPI-Key": "Your Rapid API Key",
        "X-RapidAPI-Host": "fresh-linkedin-profile-data.p.rapidapi.com" 
    }
    response = requests.get(url, headers=headers, params=querystring)
    return response.json()

json_responses=[]
for link in links:
    json_responses.append(get_profile(link)['data'])
print(json_responses)













Below PySpark  code to create a DataFrame with a defined schema for above Json response data. 

from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, ArrayType, MapType

# Sample JSON response data (replace this with your actual data)
# Initialize Spark session
spark = SparkSession.builder \
    .appName("LinkedInDataProcessing") \
    .getOrCreate()

# Define the schema for the DataFrame
schema = StructType([
    StructField("about", StringType(), True),
    StructField("city", StringType(), True),
    StructField("company", StringType(), True),
    StructField("company_domain", StringType(), True),
    StructField("company_employee_range", StringType(), True),
    StructField("company_industry", StringType(), True),
    StructField("company_linkedin_url", StringType(), True),
    StructField("company_logo_url", StringType(), True),
    StructField("company_website", StringType(), True),
    StructField("company_year_founded", StringType(), True),
    StructField("connections_count", IntegerType(), True),
    StructField("country", StringType(), True),
    StructField("current_company_join_month", IntegerType(), True),
    StructField("current_company_join_year", IntegerType(), True),
    StructField("educations", ArrayType(
        StructType([
            StructField("date_range", StringType(), True),
            StructField("school_linkedin_url", StringType(), True),
            StructField("start_year", StringType(), True),
            StructField("degree", StringType(), True),
            StructField("description", StringType(), True),
            StructField("field_of_study", StringType(), True),
            StructField("eduId", StringType(), True),
            StructField("school_id", StringType(), True),
            StructField("school", StringType(), True),
            StructField("activities", StringType(), True),
            StructField("grade", StringType(), True),
            StructField("start_month", StringType(), True),
            StructField("end_month", StringType(), True),
            StructField("end_year", StringType(), True)
        ]),
        True
    )),
    StructField("email", StringType(), True),
    StructField("experiences", ArrayType(
        StructType([
            StructField("date_range", StringType(), True),
            StructField("company_logo_url", StringType(), True),
            StructField("current_company_join_year", StringType(), True),
            StructField("current_company_join_month", StringType(), True),
            StructField("company_id", StringType(), True),
            StructField("start_year", StringType(), True),
            StructField("description", StringType(), True),
            StructField("company_linkedin_url", StringType(), True),
            StructField("title", StringType(), True),
            StructField("duration", StringType(), True),
            StructField("start_month", StringType(), True),
            StructField("location", StringType(), True),
            StructField("company", StringType(), True),
            StructField("is_current", StringType(), True),
            StructField("end_month", StringType(), True),
            StructField("end_year", StringType(), True)
        ]),
        True
    )),
    StructField("first_name", StringType(), True),
    StructField("followers_count", IntegerType(), True),
    StructField("full_name", StringType(), True),
    StructField("headline", StringType(), True),
    StructField("hq_city", StringType(), True),
    StructField("hq_country", StringType(), True),
    StructField("hq_region", StringType(), True),
    StructField("job_title", StringType(), True),
    StructField("languages", StringType(), True),
    StructField("last_name", StringType(), True),
    StructField("linkedin_url", StringType(), True),
    StructField("location", StringType(), True),
    StructField("phone", StringType(), True),
    StructField("profile_id", StringType(), True),
    StructField("profile_image_url", StringType(), True),
    StructField("public_id", StringType(), True),
    StructField("school", StringType(), True),
    StructField("skills", StringType(), True),
    StructField("state", StringType(), True),
    StructField("urn", StringType(), True)
])

# Create DataFrame from JSON data with the specified schema
df = spark.createDataFrame(json_responses, schema=schema)

# Show the DataFrame schema and data
df.printSchema()


Below PySpark to write the DataFrame to a Parquet file format and save it to Databricks storage.

df.repartition(1).write.format("parquet").save("dbfs:/FileStore/tables/linkedindata.parquet",header=True,mode="overwrite")
print("Paequet file has been saved successfully in Databricks storage.")


Below code you provided reads the Parquet file named 'linkedindata.parquet' located in the specified Databricks storage path.

from pyspark.sql.functions import collect_list
file=spark.read.format("parquet").load('dbfs:/FileStore/tables/linkedindata.parquet')
display(file)












We have two columns which contains nested or map values so we have to select important kay values and create separate columns for those keys and merge the columns with our main data dataframe.

Below code attempts to process data from the loaded parquet file DataFrame, filter it based on the full_name column, explode the experiences array column, select specific fields, and create a new DataFrame for each full_name. These individual DataFrames are then combined into a single DataFrame using unionAll and displayed. If there are issues with the data or schema, it catches and handles AnalysisException by printing an error message.

from pyspark.sql.functions import col, concat_ws, lit, explode
from pyspark.sql.utils import AnalysisException

try:
    # Assuming you have a list of names
    # Select the "full_name" column from the DataFrame
    names_df = file.select("full_name")
    names_list = [row.full_name for row in names_df.collect()]

    # Create an empty list to store the resulting DataFrames
    result_dataframes = []

    for name in names_list:
        # Filter the DataFrame to get the row for the desired full_name
        filtered_df = file.filter(file["full_name"] == name)

        # Explode the "experiences" column and alias it as "col"
        json_exploded = filtered_df.select(explode(filtered_df.experiences).alias("col"))

        # Select the desired fields from the exploded DataFrame
        json_col_df = json_exploded.select(
            "col.date_range",
            "col.company_logo_url",
            "col.current_company_join_year",
            "col.current_company_join_month",
            "col.company_id",
            "col.start_year",
            "col.description",
            "col.company_linkedin_url",
            "col.title",
            "col.duration",
            "col.start_month",
            "col.location",
            "col.company",
            "col.is_current",
            "col.end_month",
            "col.end_year"
        )

        # Add the full_name column to the DataFrame
        json_col_df = json_col_df.withColumn("full_name", lit(name))

        json_col_df = json_col_df.select(
            col("full_name"),
            col("date_range").alias("experience_date_range"),
            col("duration").alias("experience_duration"),
            col("description").alias("experience_description"),
            col("company").alias("experience_company"),
            col("title").alias("experience_title")
        )

        # Append the result DataFrame to the list
        result_dataframes.append(json_col_df)

    df1 = result_dataframes[0]
    # Iterate through the list of DataFrames and union them
    for df in result_dataframes[1:]:
        df1 = df1.unionAll(df)

    # Show the resulting combined DataFrame
    display(df1)

except AnalysisException as e:
    print("Data is not available for experiences")
    print(e)



Below code snippet creates a SparkSession, drops a column named "experiences" from a DataFrame named file, and then performs a right join with another DataFrame named df1 using the "full_name" column as the key. If there are no errors, the resulting DataFrame is assigned to final_selected.

from pyspark.sql import SparkSession
from pyspark.sql.functions import collect_list, col

try:
    # Assuming you have a SparkSession named 'spark'
    spark = SparkSession.builder.appName("example").getOrCreate()

    # Drop the original "experiences" column from final_selected
    final_selected = file.drop("experiences")

    # Join the aggregated DataFrame with final_selected using "full_name" as the key
    combined_df = df1.join(final_selected, on=["full_name"], how="right")

    final_selected = combined_df

except Exception as e:
    print("Data is not available for experiences")
    print(e)


performing an aggregation on df1 DataFrame, grouping it by the "full_name" column, and collecting a list of "experience_duration" values for each group. Then you are doing a right join with the final_selected DataFrame on the "full_name" column and assigning the result to combined_df. Finally, you are displaying the combined_df DataFrame.

  df_selected_aggregated = df1.groupBy("full_name").agg(
        collect_list(col("experience_duration")).alias("agg_experience_duration"),
        )
combined_df = df_selected_aggregated.join(final_selected, on=["full_name"], how="right")
final_selected = combined_df
display(combined_df)

 
Below code to calculate the total experience years from the agg_experience_duration column in the final_selected DataFrame using a User-Defined Function (UDF). Then, it adds a new column called total_experience with the calculated total experience years rounded to one decimal place.
from pyspark.sql.functions import udf, round
from pyspark.sql.types import FloatType

# Define a function to calculate total experience years from duration strings
def calculate_total_experience_years(duration_list):
    total_years = 0
    for exp in duration_list:
        # Split the duration string into parts and process years and months
        parts = exp.split(' ')
        for idx, part in enumerate(parts):
            if 'yr' in part:
                total_years += int(parts[idx - 1])
            elif 'mos' in part:
                total_years += int(parts[idx - 1]) / 12
    return total_years

# Register the function as a UDF
calculate_experience_years_udf = udf(calculate_total_experience_years, FloatType())

# Calculate total experience years and add it as a new column
final_selected  = final_selected .withColumn("total_experience",
                                    calculate_experience_years_udf(final_selected .agg_experience_duration))

# Round the total experience years to 2 decimal places
final_selected  = final_selected .withColumn("total_experience", round(final_selected .total_experience, 1))

# Show the resulting DataFrame with the added column
display(final_selected )

 
code attempts to process a list of names and their corresponding education data from the file DataFrame. It filters and explodes the educations column, selects specific fields, and creates a list of DataFrames for each name. Then, it unions these DataFrames into a single DataFrame df1 and displays the result. If there is an error during the process, it catches the AnalysisException and prints an error message.
from pyspark.sql.functions import col, concat_ws, lit, explode
from pyspark.sql.utils import AnalysisException

try:
    # Assuming you have a list of names
    # Select the "full_name" column from the DataFrame
    names_df = file.select("full_name")
    names_list = [row.full_name for row in names_df.collect()]

    # Create an empty list to store the resulting DataFrames
    result_dataframes = []

    for name in names_list:
        # Filter the DataFrame to get the row for the desired full_name
        filtered_df = file.filter(file["full_name"] == name)

        # Explode the "experiences" column and alias it as "col"
        json_exploded = filtered_df.select(explode(filtered_df.educations).alias("col"))

        # Select the desired fields from the exploded DataFrame
        json_col_df = json_exploded.select(
        "col.date_range",
        "col.school_linkedin_url",
        "col.start_year",
        "col.degree",
        "col.field_of_study",
        "col.eduId",
        "col.school_id",
        "col.school",
        "col.activities",
        "col.grade",
        "col.start_month",
        "col.end_month",
        "col.end_year"
        )


        # Add the full_name column to the DataFrame
        json_col_df = json_col_df.withColumn("full_name", lit(name))

        json_col_df = json_col_df.select(
            col("full_name"),
            col("date_range").alias("educations_date_range"),
            col("degree").alias("educations_degree"),
            col("field_of_study").alias("educations_field_of_study"),
            col("school").alias("educations_school"),
        )

        # Append the result DataFrame to the list
        result_dataframes.append(json_col_df)

    df1 = result_dataframes[0]
    # Iterate through the list of DataFrames and union them
    for df in result_dataframes[1:]:
        df1 = df1.unionAll(df)

    # Show the resulting combined DataFrame
    display(df1)

except AnalysisException as e:
    print("Data is not available for experiences")
    print(e)
Aggregate columns from a DataFrame df1, drop a column named "educations" from final_selected, perform a right join between df1 and final_selected on the "full_name" column, and display the resulting combined DataFrame using the display function.
from pyspark.sql import SparkSession
from pyspark.sql.functions import collect_list, col

try:
    # Assuming you have a SparkSession named 'spark'
    spark = SparkSession.builder.appName("example").getOrCreate()

    # Aggregate the columns from df1

    # Drop the original "experiences" column from final_selected
    final_selected = final_selected.drop("educations")

    # Join the aggregated DataFrame with final_selected using "full_name" as the key
    combined_df = df1.join(final_selected, on=["full_name"], how="right")

    final_selected = combined_df

    # Show the resulting combined DataFrame
    display(combined_df)

except Exception as e:
    print("Data is not available for experiences")
    print(e)


 



The given code snippet renames specific columns in the DataFrame final_selected and selects a subset of columns with a specified order. It also removes duplicate rows based on the selected columns, resulting in a clean, distinct dataset with the specified column names and order for further analysis or processing.
from pyspark.sql.functions import col

# Assuming 'final_selected' is your DataFrame
final = final_selected \
    .withColumnRenamed("full_name", "Full Name") \
    .withColumnRenamed("linkedin_url", "LinkedIn Url") \
    .withColumnRenamed("job_title", "Job Title") \
    .withColumnRenamed("location", "Location") \
    .withColumnRenamed("total_experience", "Total Experience") \
    .withColumnRenamed("skills", "Skills") \
    .withColumnRenamed("company", "Current Company") \
    .withColumnRenamed("current_company_join_year", "Current Company Join Year") \
    .withColumnRenamed("experience_title", "Experience Title") \
    .withColumnRenamed("experience_company", "Experience Company") \
    .withColumnRenamed("experience_duration", "Experience Duration") \
    .withColumnRenamed("experience_date_range", "Experience DateRange") \
    .withColumnRenamed("experience_description", "Experience Description") \
    .withColumnRenamed("educations_date_range", "Educations DateRange") \
    .withColumnRenamed("educations_degree", "Educations Degree") \
    .withColumnRenamed("educations_field_of_study", "Educations FieldOfStudy") \
    .withColumnRenamed("educations_school", "Educations School") \
    .withColumnRenamed("about", "About") \
    .withColumnRenamed("headline", "Headline")

# Select the specified columns with the specified order
final = final.select(['Headline', 'Job Title', 'Full Name', 'Location', 'Skills', 'Current Company',
                     'Current Company Join Year', 'Experience Title', 'Experience Company', 'Total Experience',
                     'Experience DateRange', 'Educations School', 'Experience Description', 'Educations DateRange',
                     'Educations FieldOfStudy', 'Educations Degree', 'About', 'LinkedIn Url'])

final=final.distinct()

 

The given code snippet performs data cleaning operations on the DataFrame `final`. It removes special characters and bullet points from the "Experience Description" column, trims leading and trailing spaces for all string columns, converts all columns to StringType, replaces null values with empty strings in string columns, and replaces null values in the "Current Company Join Year" column with empty strings. Finally, it orders the DataFrame by the "Full Name" column and displays the cleaned and processed data.
from pyspark.sql.functions import regexp_replace,trim,col,when

# Assuming 'final' is your DataFrame
final = final.withColumn("Experience Description", 
                        regexp_replace(col("Experience Description"), '[^a-zA-Z0-9\s]', ''))  # Remove special characters
final = final.withColumn("Experience Description", regexp_replace(col("Experience Description"), "•", ""))
# Trim leading and trailing spaces
final = final.withColumn("Experience Description", trim(col("Experience Description")))

# Iterate over columns and trim leading and trailing spaces for StringType columns
for col_name in final.columns:
    if isinstance(final.schema[col_name].dataType, StringType):
        final = final.withColumn(col_name, trim(col(col_name)))

for column in final.columns:
    if final.schema[column].dataType == StringType():
        final = final.withColumn(column, col(column).cast(StringType())) 
        final = final.fillna('', subset=[column])

final = final.withColumn("Current Company Join Year", when(col("Current Company Join Year").isNull(), "").otherwise(col("Current Company Join Year")))

final = final.orderBy("Full Name")

display(final)


 Final Data Link
Linkedn Usecase Final Data.xlsx

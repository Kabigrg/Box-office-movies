# Library init

import requests
from azure.storage.blob import BlobServiceClient, BlobClient, ContentSettings,ContainerClient
import os
import gzip
import requests
import pandas as pd
from io import BytesIO
import shutil

# Url Initialization

name_url = 'https://datasets.imdbws.com/name.basics.tsv.gz'
title_url = 'https://datasets.imdbws.com/title.basics.tsv.gz'
principal_url = 'https://datasets.imdbws.com/title.principals.tsv.gz'
ratings_url = 'https://datasets.imdbws.com/title.ratings.tsv.gz'

## code that will get the files and upload them to Azure Blob

AZURE_STORAGE_CONNECTION_STRING ='DefaultEndpointsProtocol=https;AccountName=stboxoffice;AccountKey=8PVYmm5+zCALb3D68QpEAa7Ceuy/79j8pQInN0Go7fYgOHsIVebBnbUuItZvwJyf8TjH5WIRa9ss+AStmE8kew==;EndpointSuffix=core.windows.net'
container_name = 'stboxoffice'
blob_service_client = BlobServiceClient.from_connection_string(AZURE_STORAGE_CONNECTION_STRING)
container_client = blob_service_client.get_container_client(container_name)


gz_links = [name_url, title_url, principal_url, ratings_url]

for gz_link in gz_links:
    # Get the gz file
    response = requests.get(gz_link, stream=True)
    
    # Create a temporary file to store the gz file
    with open('temp.gz', 'wb') as out_file:
        shutil.copyfileobj(response.raw, out_file)
    del response

    # Create a unique blob name
    blob_name = 'imdb_datasets/' + os.path.basename(gz_link)

    # Create a blob client using the blob name from the container client
    blob_client = container_client.get_blob_client(blob_name)

    # Upload the gz file to Azure blob storage
    with open('temp.gz', 'rb') as data:
        blob_client.upload_blob(data)

    # Delete the temporary file
    os.remove('temp.gz')

gz_file_path = 'title.gz'
# Create a temporary .gz file
with open(gz_file_path, 'wb') as out_file:
    shutil.copyfileobj(response.raw, out_file)

df = pd.read_csv(gz_file_path, compression='gzip', header=0, sep='\t')
df.head()

# setup azure blob connection
AZURE_STORAGE_CONNECTION_STRING ='DefaultEndpointsProtocol=https;AccountName=stboxoffice;AccountKey=8PVYmm5+zCALb3D68QpEAa7Ceuy/79j8pQInN0Go7fYgOHsIVebBnbUuItZvwJyf8TjH5WIRa9ss+AStmE8kew==;EndpointSuffix=core.windows.net'
container_name = 'stboxoffice'
blob_service_client = BlobServiceClient.from_connection_string(AZURE_STORAGE_CONNECTION_STRING)
container_client = blob_service_client.get_container_client(container_name)


# select unique value of a column name title
unique_titles = df['originalTitle'].unique().tolist()

unique_tconst = df['tconst'].unique().tolist()

print(len(unique_tconst))

# initialize an empty list to hold API responses
omdb_responses = []
count = 0
# loop through that list call api
for tconst in unique_tconst:
    # assuming you are calling an API that takes title as a parameter
    # replace with actual API endpoint
    # http://www.omdbapi.com/?i=tt3896198&apikey=478896bc
    response = requests.get(f"http://www.omdbapi.com/?i={tconst}&apikey=478896bc") 
    # append the response to the list
    count = count + 1 
    print(count)
    omdb_responses.append(response.json())
    if count > 20000:
        break
    
# create a dataframe from the API responses
omdb_df = pd.DataFrame(omdb_responses)
omdb_df.head()

# Data Frame conversion and blob definition with help from Professor Bien-Aime

# convert the dataframe to a CSV string
csv_data = omdb_df.to_csv(index=False)
omdb_df.to_csv("api_responses.csv")

# define the blob name
blob_name = 'processed/api_responses.csv'

# get a blob client
blob_client = blob_service_client.get_blob_client(container_name, blob_name)

# upload the CSV data to Azure Blob Storage
blob_client.upload_blob(csv_data)

CONNECTION_STRING = 'DefaultEndpointsProtocol=https;AccountName=stboxoffice;AccountKey=8PVYmm5+zCALb3D68QpEAa7Ceuy/79j8pQInN0Go7fYgOHsIVebBnbUuItZvwJyf8TjH5WIRa9ss+AStmE8kew==;EndpointSuffix=core.windows.net'
container_name = 'stboxoffice'
def upload_blob(byte_stream,blob_name) :
    blob_service_client = BlobServiceClient.from_connection_string(CONNECTION_STRING)
    container_client = blob_service_client.get_container_client(container_name)
    blob_client = container_client.get_blob_client(blob_name)
    content_settings = ContentSettings(content_type='text/plain')
    blob_client.upload_blob(byte_stream, blob_type="BlockBlob", content_settings=content_settings, overwrite=True)

name_url = 'https://datasets.imdbws.com/name.basics.tsv.gz'
title_url = 'https://datasets.imdbws.com/title.basics.tsv.gz'
principal_url = 'https://datasets.imdbws.com/title.principals.tsv.gz'
ratings_url = 'https://datasets.imdbws.com/title.ratings.tsv.gz'


response_name = requests.get(name_url)
response_name = requests.get(title_url)
response_name = requests.get(principal_url)
response_name = requests.get(ratings_url)


upload_blob(response_name.content, "name.gz")
upload_blob(response_name.content, "title.gz")
upload_blob(response_name.content, "principal.gz")
upload_blob(response_name.content, "ratings.gz")
## response_name.content

## upload_blob(response_name.content, "")
    
import json
import snowflake.connector

# Setting up Snowflake connection 
conn_location = '/work/'
connect = json.loads(open(str(conn_location+'/SF_cred.json')).read())
username    = connect['secrets']['user']
password    = connect['secrets']['password']
account     = connect['secrets']['account']
role        = connect['secrets']['role']
# Connect to Snowflake
conn = snowflake.connector.connect(
    user        = username,
    password    = password,
    account     = account,
    role        = role
    )

import pandas as pd
    
dfF1 = pd.read_csv("/work/api_responses.csv")

omdb_df = pd.DataFrame(dfF1)
omdb_df.head()


cursor = conn.cursor()

table = "IMDB"

# Create the table if it doesn't exist
create_table_sql = f"CREATE TABLE IF NOT EXISTS {table} (Column1 INT, Column2 VARCHAR, Column3 BOOLEAN)"
cursor.execute(create_table_sql)

# Generate the SQL INSERT statement
insert_sql = f"INSERT INTO {table} (Column1, Column2, Column3) VALUES (?, ?, ?)"

# Convert DataFrame to a list of tuples
values = [tuple(row) for row in omdb_df.values]

# Insert data into the table
cursor.executemany(insert_sql, values)

# Commit the changes
conn.commit()

# Close the cursor and connection
cursor.close()
conn.close()


# Iterating trough the columns
for col in dfF1.columns:
    column_name = col.upper()
    
    if (dfF1[col].dtype.name == "int" or dfF1[col].dtype.name == "int64"):
        create_tbl_sql = create_tbl_sql + column_name + " int"
    elif dfF1[col].dtype.name == "object":
        create_tbl_sql = create_tbl_sql + column_name + " varchar(16777216)"
    elif dfF1[col].dtype.name == "datetime64[ns]":
        create_tbl_sql = create_tbl_sql + column_name + " datetime"
    elif dfF1[col].dtype.name == "float64":
        create_tbl_sql = create_tbl_sql + column_name + " float8"
    elif dfF1[col].dtype.name == "bool":
        create_tbl_sql = create_tbl_sql + column_name + " boolean"
    else:
        create_tbl_sql = create_tbl_sql + column_name + " varchar(16777216)"
    
    # Deciding next steps. Either column is not the last column (add comma) else end create_tbl_statement
    if dfF1[col].name != dfF1.columns[-1]:
        create_tbl_sql = create_tbl_sql + ",\n"
    else:
        create_tbl_sql = create_tbl_sql + ")"

conn.cursor().execute(create_tbl_sql)

# Write the data from the DataFrame to the f1_pre_table.
write_pandas(
    conn=conn,
    df=dfF1,
    table_name=f1_pre_table,
    database=demo_db,
    schema=f1_pre_schema
)
        

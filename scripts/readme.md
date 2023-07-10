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


    ## Code that will download zip file to query the apis  from Professor Bien-Aime

# Send a HTTP request to the URL
response = requests.get(title_url, stream=True)

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

# create a dataframe from the API responses
omdb_df = pd.DataFrame(omdb_responses)
omdb_df.head()



## Code that will download zip file to query the apis

# Send a HTTP request to the URL
response = requests.get(title_url, stream=True)

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

# create a dataframe from the API responses
omdb_df = pd.DataFrame(omdb_responses)
omdb_df.head()

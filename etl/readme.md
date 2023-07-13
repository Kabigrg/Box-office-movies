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

 # Write the data from the DataFrame to the f1_pre_table.
write_pandas(
    conn=conn,
    df=dfF1,
    table_name=f1_pre_table,
    database=demo_db,
    schema=f1_pre_schema
)

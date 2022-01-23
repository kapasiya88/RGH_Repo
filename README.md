******Project flow:******

Source file  EmployeeData<YYYYMMDDHHMISS>.csv will arrive to a particular path on Unix server with timestamp which will make the file unique everytime.
Same file will be reflected on S3 and then will get loaded snowflake external stage.
From external stage data will be loaded to snowflake transient table via snowpipe with few datatype conversions.
Once data is loaded to transient table then newly inserted data will be captured into Append_only stream and all the CDC will be captured into CDC stream.
Both the streams will be read by different tasks to load the data into snowflake table finally.

******Execution Steps:******

****1) Create an AWS IAM user to connect to Unix server using the AWS CLI.****

****2) Create a bitbucket on s3 where all the intended files need to be uploaded.****
           https://github.com/kapasiya88/RGH_Repo/blob/9136a33e4e9f072c2088cdaa745adfc27ae84e67/src/Commands%20to%20upload%20files%20from%20unix%20server%20to%20s3
  
****3) Create an integration object to establish connection between Snowflake and AWS.****
              https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/integration%20object
  
  ***4) Create a file format to describe the properties of inbound file.***
              https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/file%20format
  
  ***5) Create an external stage to point to S3 location.***   
          https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/external%20stage
  
  ***5) Create transient table where data will be populated from external stage via snowpipe.***  
        https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/transient%20table
  
  ***6) Create a snowpipe which will populate the data after few data transformation to a transient table.Datatype conversions are included in snowpipe.
          Eg:to_number,to_decimal,to_date***  
          https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/Snowpipe
  
  ***7) create streams to capture the CDC.***                                                                                                                            
    a) Stream to capture the newly inserted data.                                                                                                              
          https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/Insert%20Stream
          
          
  b) Stream to capture the CDC data.
         https://github.com/kapasiya88/RGH_Repo/blob/998fe86a11ad64007f9524ba2e1f58be5c6d47f0/src/CDC%20stream
  
  ***8) Snowflake table where data will be loaded.***
           https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/Permanent%20table
  
  ***9) Create task to load data newly inserted data from stream to snowflake table .***
           https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/Task%20to%20load%20new%20data%20to%20main%20table
  
  ***10) Create task to load CDC data from stream to snowflake table .***
         https://github.com/kapasiya88/RGH_Repo/blob/a7da27d4963736a702ceb43eb5ce01c1d774d1c0/src/Task%20to%20load%20CDC%20data%20to%20main%20table


-------------------------------------------------------------------------------------------------------



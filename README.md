Steps:
----------------------------------------------------------------------------------------------------
***1) Create an AWS IAM user to connect to Unix server using the AWS CLI.***

***2) Create a bitbucket on s3 where all the intended files need to be uploaded.***

    Commands to upload files from unix server to s3:
      Oms-MacBook-Pro:~ ompayla$ aws configure
      AWS Access Key ID [****************KC4A]: <access key id>
      AWS Secret Access Key [****************tEAK]: <secret access key>
      Default region name [ap-south-1]: ap-south-1 
      Default output format [json]: json        
      Oms-MacBook-Pro:~ ompayla$ aws s3 cp /Users/ompayla/Desktop/Kajal/study/Assignment/Employee.csv s3://rgh-assignment/

    NOTE: All the process of uploading the file/files on AWS can be automated by creating a unix shell script which can be called from a scheduler.
  
  ***3) Create an integration object to establish connection between Snowflake and AWS.***
  
      create or replace storage integration s3_int                                   
       type = external_stage                                                                                                                            
       storage_provider = s3                                                                                                                       
       enabled = true                                                                                                                              
       storage_aws_role_arn = 'arn:aws:iam::211678918935:role/aws_snowflake_integrate/                                                                  
       storage_allowed_locations = ('s3://rgh-assignment/');
  
  ***4) Create a file format to describe the properties of inbound file.***
  
        create or replace file format rgh_assignment.employee_data.my_csv_format                                                                                    
        type = csv field_delimiter = ',' skip_header = 1 null_if = ('NULL', 'null') empty_field_as_null = true,                             
         error_on_column_count_mismatch=false;
  
  ***5) Create an external stage to point to S3 location.***   
  
         create or replace stage rgh_assignment.employee_data.my_s3_stage                                                                              
         storage_integration = s3_int                                                                                                                             
         url = 's3://rgh-assignment/'                                                                                                                        
         file_format = rgh_assignment.employee_data.my_csv_format;
  
  ***5) Create transient table where data will be populated from external stage via snowpipe.***  
  
      create or replace transient table rgh_assignment.employee_data.employee_data_trans (EmpId number,Prefix string,FirstName string,LastName string,
      Gender string,Email string,DateOfBirth date,DateOfJoining date,Salary float,LastPercentHike string,SSN string,PhoneNo string,PlaceName string,Country 
      string,City string,State string,ZIP string,Region string,UserName string,Pasword string);
  
  ***6) Create a snowpipe which will populate the data after few data transformation to a transient table.Datatype conversions are included in snowpipe.
          Eg:to_number,to_decimal,to_date***  
  
            create or replace pipe rgh_assignment.employee_data.snowpipe_emp                                                                             
             auto_ingest=true as                                                                                                                                  
             copy into rgh_assignment.employee_data.EMPLOYEE_DATA_TRANS                                                                                          
              from (select to_number(t.$1) as emp_id,t.$2 as prefix,t.$3 as first_name,t.$4 last_name,                                                           
              t.$5 gender, t.$6 email_id,to_date(t.$7,'MM/DD/YYYY') date_of_birth,to_date(t.$8,'MM/DD/YYYY') date_of_joining,                            
              to_decimal(t.$9) salary,t.$10 prcnthike,t.$11 SSN,t.$12 phone_number,t.$13 place,t.$14 country,t.$15 city,t.$16 state,                   
              to_number(t.$17) pincode,t.$18 region,t.$19 username,t.$20 pasword                                                                                 
              from @rgh_assignment.employee_data.my_s3_stage/ t)                                                                                  
            ON_ERROR='CONTINUE'                                                                                                                             
            file_format = (format_name='rgh_assignment.employee_data.my_csv_format');
  
  ***7) create streams to capture the CDC.***                                                                                                                            
    a) Stream to capture the newly inserted data.                                                                                                              
          create or replace stream rgh_assignment.employee_data.employee_new on table rgh_assignment.employee_data.employee_data_trans append_only=true;
 
  
    b) Stream to capture the CDC data.                                                                                                                           
           create or replace  stream rgh_assignment.employee_data.employee_CDC on 
             table rgh_assignment.employee_data.employee_data_trans; 
  
  ***8) Snowflake table where data will be loaded.***
 
         create or replace table rgh_assignment.employee_data.employee_data (EmpId number,Prefix string,FirstName string,LastName string,Gender string,
           Email string,DateOfBirth date,DateOfJoining date,Salary float,LastPercentHike string,SSN string,PhoneNo string,PlaceName string,Country string,
           City string,State string,ZIP string,Region string,UserName string,Pasword string);
  
  ***9) Create task to load data newly inserted data from stream to snowflake table .***
  
           CREATE OR REPLACE TASK rgh_assignment.employee_data.employee_insert                                                                               
             WAREHOUSE = compute_wh                                                                                                                            
             SCHEDULE = '1 minute'                                                                                                                                
             WHEN SYSTEM$STREAM_HAS_DATA('rgh_assignment.employee_data.employee_new')                                                                            
             as                                                                                                                                                
             insert into rgh_assignment.employee_data.employee_data select * from rgh_assignment.employee_data.employee_new;
  
  ***10) Create task to load CDC data from stream to snowflake table .***
 
              CREATE OR REPLACE TASK rgh_assignment.employee_data.employee_merge                                                                            
               WAREHOUSE = compute_wh                                                                                                                        
               SCHEDULE = '1 minute'                                                                                                                            
               WHEN SYSTEM$STREAM_HAS_DATA('rgh_assignment.employee_data.employee_CDC')                                                 
                as                                                                                                                                             
               merge into rgh_assignment.employee_data.employee_data  employee_data using  employee_CDC                                                           
                  on employee_data.Emp_id=employee_CDC.Emp_id                                                                                                    
                    when matched then update                                                                                                                     
                 set employee_data.Email=nvl(employee_CDC.Email,employee_data.Email),
                       employee_data.Salary=nvl(employee_CDC.Salary,employee_data.Salary),
                        employee_data.LastPercentHike=nvl(employee_CDC.LastPercentHike,employee_data.LastPercentHike),
                        employee_data.PhoneNo=nvl(employee_CDC.PhoneNo,employee_data.PhoneNo),
                        employee_data.PlaceName=nvl(employee_CDC.PlaceName,employee_data.PlaceName),
                        employee_data.Country=nvl(employee_CDC.Country,employee_data.Country),
                        employee_data.City=nvl(employee_CDC.City,employee_data.City),
                        employee_data.State=nvl(employee_CDC.State,employee_data.State),
                        employee_data.ZIP=nvl(employee_CDC.ZIP,employee_data.ZIP),
                        employee_data.Region=nvl(employee_CDC.Region,employee_data.Region),
                        employee_data.UserName=nvl(employee_CDC.UserName,employee_data.UserName),
                        employee_data.Pasword=nvl(employee_CDC.Pasword,employee_data.Pasword);







-------------------------------------------------------------------------------------------------------



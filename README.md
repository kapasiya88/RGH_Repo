Create an AWS IAM user to connect to Unix server using the AWS CLI.
----------------------------------------------------------------------------------------------------
Create a bitbucket on s3 where all the intended files need to be uploaded.
Commands to upload files from unix server to s3:
Oms-MacBook-Pro:~ ompayla$ aws configure
AWS Access Key ID [****************KC4A]: AKIATCSIL7ELYK6WNKD3
AWS Secret Access Key [****************tEAK]: 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
Default region name [ap-south-1]: ap-south-1 
Default output format [json]: json           
Oms-MacBook-Pro:~ ompayla$ aws s3 cp /Users/ompayla/Desktop/Kajal/study/Assignment/Employee.csv s3://rgh-assignment/

NOTE: All the process of uploading the file/files on AWS can be automated by creating a unix shell script which can be called from a scheduler.

-------------------------------------------------------------------------------------------------------


# Workshop

In this workshop we are going to use several AWS tools and services, among them, S3, Glue ETL, Glue Data Catalog, Quicksight Deeque to make an ETL and Data Quality process.

Create a new AWS account or connect to an existing one.

We are going to need to create a policy and role using IAM, for this:

1. Create role to use in Glue

    * Go to service**IAM**
    * Go to section**Roles**
   * **Create role**
    * Select**Glue**as the service that the role will use ->**Next**
   * **Create Policy**
    * Select JSON
    * Copy JSON
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "VisualEditor0",
                "Effect": "Allow",
                "Action": [
                    "logs: CreateLogStream",
                    "s3: *",
                    "logs: CreateLogGroup",
                    "logs: PutLogEvents",
                    "glue: *"
                ],
                "Resource": "*"
            }
        ]
    }
    ```
   * **Review Policy**
    * Give name *"my-glue-policy"*
   * **Create policy**
    * Return to the role tab in the browser and click the **refresh** icon
    * Select policy created
   * **Next: Tags**->**Next: Review**
    * Give name *"my-glue-role"*
   * **Create role**

2. Launch crawler on external S3 source (Glue Data Catalog)

    * Go to service**AWS Glue**
    * Go to section**Crawlers**
    * Click on**Add crawler**
    * Crawler name *"my-crawler-source"* ->**Next**
    * Data stores -> Next
    * Choose**S3**and select *Specified path in another account*, enter *"s3: // amazon-reviews-pds / parquet / product_category = Electronics"*
   * **Add another store "No"**->**Next**
   * **Choose an existing IAM role**, select the created role ->**Next**
    * Frequency**Run on Demand**->**Next**
    * Add database ->**Database name***"db-source"* ->**Create**->**Next**
   * **Finish**
    * Click on**Run it now**

    When launching the crawler, what it does is analyze the data of the selected * Data store * and deduce information regarding the existing columns, data types and some extra data regarding the table, these tables can be viewed from * Glue Data Catalog * or from any service that has integration to this, such as the * Athena * service.
    
    An interesting feature of crawlers is that they have table versioning, that is, if the crawler is triggered more than once and the crawler source is modified, it will register a new version in the table, leaving a record of the version. previous.

3. Create bucket in S3

    * Go to the service**S3**
   * **Create bucket**
   * **Bucket name***"bucket-glue-target-'last-name-name'"* (must be unique for the region) 
   * **Create**
    * Repeat for bucket *"bucket-glue-libraries-'last-name-name'"* (must be unique for the region)

4. Configure Glue Job

    * Go to AWS Glue ->**Jobs**->**Add job**
    * Give name *"my-job-glue"* and choose created role
    * Jump to the * This job runs * section, select * A new script to be authored by you * ->**Next**
   * **Save job and edit script**
    * Copy the content of the file *"glue-job.py"*
    * Change * line 22 * ​​to the path of the Glue job created *"s3: // bucket-glue-target-'last-name-surname' / target / products"*
    * Click**Save**and then**Run job**

5. Launch crawler on target S3

    * Same steps as in *"Step 2"* with the name *"my-crawler-target"*
   * **Data store S3***"s3: // bucket-glue-target-'last-name' / target / products"*
   * **Add database**, name *"db-target"* ->**Next**->**Finish**
    * Click on**Run it now**

6. Integrate Deequ in Glue Job for data analysis (Spark Scala)

    * Go to S3 select bucket *"bucket-glue-libraries-'name-surname'"*,**Create folder**name *"jar"*, enter the created folder and click**Upload**, select the JAR *"deequ-1.0.1.jar"* and click**Upload**
    * Go to AWS Glue ->**Jobs**->**Add job**
    * Give name *"my-job-analyze-deequ"* and choose created role
    * Change**Glue version**to * Spark 2.4, Scala (Glue version 1.0)
    * Change**This job runs**to * A new script to be authored by you *
    * In**Scala class name**write *"GlueApp"*
   * **Script file name***"glue-deequ-integration-analyze"*
    * In**Security configuration, script libraries, and job parameters (optional)**,**Dependent jars path***"s3: //bucket-glue-libraries-'name-lastname'/jar/deequ-1.0. 1.jar "* ->**Next**
   * **Save job and edit script**
    * Copy the code from the file *"glue-analyze-deequ.scala"*
    * Change on * line 19 * to path *"s3: // bucket-glue-target-'last-name-name' / target / deequ / products_analysis"*
    * Change * line 21 * to the path of the Glue job created *"s3: // bucket-glue-target-'last-name' / target / products"*
   * **Save**
   * **Run job**

7. Edit crawler target

    In order to view the results in Athena, they must have been subsequently added to the Glue Catalog.

    * Go to the**target**crawler ->**Edit**
   * **Next**->**Next**->**Next**
    * In * Add another data store * choose**Yes**->**Next**
    * In * Include path * write *"s3: // bucket-glue-target-'surname-name' / target / deequ / products_analysis"* ->**Next**until finishing the process
    * Run the crawler again so that the new table is added to the Glue Catalog


8. Check results with Athena

    * Go to the service**Athena**
    * Select base * db-target *
    * To see the records of a table, do an SQL query on it

9. Integrate Deequ in Glue Job for data testing (Spark Scala)

    * Go to AWS Glue ->**Jobs**->**Add job**
    * Give name *"MY-job-validate-deequ"* and choose created role
    * Change**Glue version**to * Spark 2.4, Scala (Glue version 1.0)
    * Change**This job runs**to * A new script to be authored by you *
    * In**Scala class name**write *"GlueApp"*
    * **Script file name***"glue-deequ-integration-validate"*
    * In**Security configuration, script libraries, and job parameters (optional)**,**Dependent jars path***"s3: //bucket-glue-libraries-'name-lastname'/jar/deequ-1.0. 1.jar "* ->**Next**
    * **Save job and edit script**
    * Copy the code from the file *"script-glue-deequ.scala"*
    * Change in * line 19 * to the path *"s3: // bucket-glue-target-'name-surname' / target / deequ / products_results"*
    * Change * line 21 * to the path of the Glue job created *"s3: // bucket-glue-target-'last-name' / target / products"*
    * **Save**
    * **Run job**

10. Edit crawler target

    In order to view the results in Athena, they must have been subsequently added to the Glue Catalog.

    * Go to the**target**crawler ->**Edit**
   * **Next**->**Next**->**Next**
    * In * Add another data store * choose**Yes**->**Next**
    * In * Include path * write *"s3: // bucket-glue-target-'surname-name' / target / deequ / products_results"* ->**Next**until finishing the process
    * Run the crawler again so that the new table is added to the Glue Catalog

11. Check results with Athena

    * Go to the service**Athena**
    * Select base * db-target *
    * To see the records of a table, do an SQL query on it

12. Automating the flow

    To optimize times for business agility, Glue offers automation features, such as * Triggers * and * Workflows *

    To add a trigger it is necessary to follow these steps:

    * Go to the Glue console to the section * Triggers * -> * Add trigger *
    * Give a name to the trigger
    * Set * Trigger type * to * Job Events *
    * Select the job *"my-job-glue"* ->**Next**
    * In * Choose jobs to trigger * select the job *"my-job-analyze-deequ"* ->**Next**->**Finish**

    Now every time the job *"my-job-glue"* is executed, at the end of its execution and if this is successful the job *"my-job-analyze-deequ"* will start

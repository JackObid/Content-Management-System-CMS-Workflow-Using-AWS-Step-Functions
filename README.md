# Content-Management-System-CMS-Workflow-Using-AWS-Step-Functions
Update or import content data from CSV files stored in an S3 bucket into a content management system (CMS)

**Objective:** Update or import content data from CSV files stored in an S3 bucket into a content management system (CMS).

#### Prerequisites
1. **AWS Account**: Please ensure you have an AWS account with the necessary permissions.
2. **AWS CLI**: Installed and configured.
3. **Git**: Installed.
4. **Terraform**: Installed.
5. **Amazon S3 Bucket**: Store the CSV files.
6. **DynamoDB Table**: To store CMS metadata or other relevant information.

#### Workflow Steps

1. **Setup Infrastructure**
    - **Clone Repository**:
      ```bash
      git clone https://github.com/aws-samples/step-functions-workflows-collection
      cd step-functions-workflows-collection/distributed-map-csv-iterator-tf
      ```
    - **Initialize Terraform**:
      ```bash
      terraform init
      ```
    - **Apply Terraform Configuration**:
      ```bash
      terraform apply
      ```
      - Confirm with `yes`.

2. **Upload CSV File to S3**
    - Upload your CSV file to the designated S3 bucket.

3. **Modify State Machine**
    - Update the state machine definition to process each content item and store metadata in DynamoDB.
    - Example State Machine Definition:
      ```json
      {
        "Comment": "A description of my state machine",
        "StartAt": "ProcessCSV",
        "States": {
          "ProcessCSV": {
            "Type": "Map",
            "ItemProcessor": {
              "ProcessorConfig": {
                "Mode": "DISTRIBUTED",
                "ExecutionType": "CHILD"
              },
              "StartAt": "UpdateCMS",
              "States": {
                "UpdateCMS": {
                  "Type": "Task",
                  "Resource": "arn:aws:states:::dynamodb:putItem",
                  "Parameters": {
                    "TableName": "YOUR-DYNAMODB-TABLE",
                    "Item": {
                      "ContentID": {
                        "S.$": "States.ArrayGetItem($.Item, 0)"
                      },
                      "VideoTitle": {
                        "S.$": "States.ArrayGetItem($.Item, 1)"
                      },
                      "VideoPublishTime": {
                        "S.$": "States.ArrayGetItem($.Item, 2)"
                      },
                      "Views": {
                        "N.$": "States.ArrayGetItem($.Item, 3)"
                      }
                    }
                  },
                  "End": true
                }
              }
            },
            "End": true
          }
        }
      }
      ```

4. **Trigger State Machine**
    - **Start Execution**:
      - Use the following input:
        ```json
        {
          "BucketName": "YOUR-BUCKET-NAME",
          "FileKey": "PREFIX/metrics.csv"
        }
        ```

5. **Verify Data in DynamoDB**
    - Go to the DynamoDB console and explore the table to verify the content metadata has been updated.

6. **Cleanup Resources**
    - **Destroy Resources**:
      ```bash
      terraform destroy
      ```
      - Confirm with `yes`.

These detailed workflows provide a structured approach to implementing the Batch Data Import and Content Management Systems using AWS Step Functions and associated AWS services. Adjust the parameters and resource names according to your specific environment and requirements.

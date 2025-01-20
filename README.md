# AWS-CODEPIPELINE
Exercise about deploying your project with S3, CodePipeline, and SAM's project.

1. Create a zip file of the project

```powershell
zip -r repo-pipeline-jon.zip repo-pipeline/ -x ".git"
```

2. Create a bucket in S3
  
>[!important]
>The version control should be enabled

![image](https://github.com/user-attachments/assets/9a295b0f-01cd-432e-8537-f7bc860749ec)


3. Upload the zip file of the project to the bucket created

```powershell
aws s3 cp repo-pipeline-jon.zip s3://prueba-pipeline-bucket-jon --profile COMPUTOS-REINGENIERIA
```

4. Create the roles

Use these policies or create custom policies.

Rol: code-build-jon-role

![image](https://github.com/user-attachments/assets/40c95d46-d00c-4aba-a1a2-94be108f6d49)

>[!important]
>For better security, it's recommended not to use the wildcard * in AWS IAM policies because it grants access to all resources. Instead, you should specify more granular permissions and resource ARNs (Amazon Resource Names).

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "s3-object-lambda:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "CloudWatchLogsFullAccess",
            "Effect": "Allow",
            "Action": [
                "logs:*",
                "cloudwatch:GenerateQuery"
            ],
            "Resource": "*"
        }
    ]
}
```

Rol: code-deploy-cloudFormation-role

This role needs a lot of permissions.

![image](https://github.com/user-attachments/assets/680783b6-9d36-41ac-98b1-45a8978add4d)

For this example, a policy was created that contains permissions for ApiGateway.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "apigateway:POST",
                "apigateway:GET",
                "apigateway:PUT",
                "apigateway:DELETE",
                "apigateway:PATCH"
            ],
            "Resource": "*"
        }
    ]
}
```

5. Create a compilation project. -> Go to CodePipeline -> Go to projects -> Go to Compilation Project

  It is necessary to select the origin: choose Amazon S3, select the bucket created in S3, and enter the name of the project's ZIP file.

  ![image](https://github.com/user-attachments/assets/0fb060fb-693a-4a94-952f-8f460ce8c7e7)

6. Enter the script of compilation and create project 

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
        python: 3.13
  build:
    commands:
      - aws s3 cp s3://prueba-pipeline-bucket-jon/repo-pipeline-jon.zip .
      - unzip -o repo-pipeline-jon.zip
      - cd repo-pipeline
      - sam build -t sam-app/template.yaml
      - sam package --template-file sam-app/template.yaml --s3-bucket prueba-pipeline-bucket-jon --output-template-file packaged-template.yaml
artifacts:
  files:
    - repo-pipeline/packaged-template.yaml
```

![image](https://github.com/user-attachments/assets/993c864f-f009-44ab-958d-47418172af9f)


7. Now search for 'Canalización (Pipeline)' and create a pipeline.

Enter the name of the pipeline and choose the replacement option.

In the service role, you can create a role or use the one provided by the service.

![image](https://github.com/user-attachments/assets/ddb69637-8396-4ef2-99fb-c9625ae6c2d1)

8. Select the origin again during the pipeline creation.

   Choose AWS CodePipeline.

![image](https://github.com/user-attachments/assets/877343c4-d599-4bf6-8244-71b597d08182)


9. Choose another build provider, select AWS CodeBuild, and select the project created in the compilation project section.
   
![image](https://github.com/user-attachments/assets/42e990ad-e0cf-4a7e-9b5f-7605db896e5b)

10. Add deployment stage.

    Select AWS CloudFormation as the deployment provider.
    Select 'Create or update a stack' in the action mode and enter the name of the stack.
    Select 'BuildArtifact' in the artifact's name.
    In the file's name, enter the name you wrote in the YAML file, such as repo-pipeline/packaged-template.yaml.
    And finally, select the options 'CAPABILITY_IAM' and 'CAPABILITY_AUTO_EXPAND'.    

![image](https://github.com/user-attachments/assets/789222ea-8bf1-42e5-bb01-7e6e795cd9ba)

![image](https://github.com/user-attachments/assets/eeadbebd-33e1-4cdb-a38a-f1433b1a8b58)


11. In CloudFormation, under the 'Stacks' section, select the stack, and in the stack details, find the endpoint to use for the project.

![image](https://github.com/user-attachments/assets/eec07b29-0eef-4fc9-8b97-4c746d71ea07)


12. Example:

https://rq1t0dt84d.execute-api.us-east-1.amazonaws.com/Prod/hello/?id=test

We can create a register with the ID 'test' and execute the Lambda function using the endpoint. In the URL endpoint, send the ID, in this case 'test'

![image](https://github.com/user-attachments/assets/5d82808f-1dee-43e7-8bc1-05c96a66baa9)


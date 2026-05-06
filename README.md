# **Lab 13.1: Automating Code Deployments with a CI/CD Pipeline**

## **Lab overview and objectives**

In this lab, you will create an AWS CodeCommit repository (also known as a repo) and an AWS CodePipeline pipeline. You will configure the pipeline to automatically apply updates to the café website as changes are saved to the repository.

After completing this lab, you should be able to:

- Create a new CodeCommit repository
- Clone and update a CodeCommit repository
- Create a pipeline by using CodePipeline

## **Duration**

This lab will require approximately **60 minutes** to complete.

## **Scenario**

Now that the café website is in production, Frank wants a reliable process in place to track code changes and update the site when changes are made. He asked Sofía to find a way to centralize the website code and add version control. He has also asked if it's possible to automatically update the website instead of manually running scripts and uploading files when changes are made.

Mateo, a café regular and AWS consultant who specializes in automating repeatable processes, was chatting with Sofía about his work. He mentioned using CodeCommit to collaborate on projects with other developers. Sofía shared that she was researching CodeCommit to centralize the café website's code and asked Mateo for suggestions about automating updates to the site. Mateo suggested using CodePipeline because it easily integrates with both CodeCommit and Amazon Simple Storage Service (Amazon S3).

When you *start* the lab, many services are deployed for you. In this lab, you will focus on the services that are represented in the following diagram. As in past labs, the VS Code IDE is used as the development environment. The developer runs commands from VS Code IDE to update the code in the S3 bucket that hosts the website. The second bucket in the diagram will be used to set up a continuous integration and continuous delivery (CI/CD) pipeline.

![image.png](attachment:49d61f45-7525-44e1-867e-1b31843a9cef:image.png)

By the *end* of this lab, you will have created the architecture in the following diagram. You will have created a CodeCommit repository to store the website code. You will have also created a CodePipeline pipeline to automate updates to the website when changes are pushed to the CodeCommit repository. Artifacts that CodePipeline uses to deploy and update your website application will be hosted on an S3 bucket that is separate from the bucket that hosts the café website.

![image.png](attachment:5613c28f-d750-4913-9426-6df7ea0dfab4:image.png)

## **Task 1: Preparing the development environment**

Before you can start working on this lab, you must import some files and install some packages in the Visual Studio Code Integrated Development Environment (VS Code IDE).

- Connect to the VS Code IDE.
    - At the top of these instructions, choose Details followed by  **AWS: Show**
    - Copy values from the table for the following and paste it into an editor of your choice for use later.
        - **LabIDEURL**
        - **LabIDEPassword**
        - In a new browser tab, paste the value for **LabIDEURL** to open the VS Code IDE.
        - On the prompt window **Welcome to code-server**, enter the value for **LabIDEPassword** you copied to the editor earlier, choose **Submit** to open the VS Code IDE.
        
        ![image.png](attachment:a5348417-72bd-4f0a-862b-32202ddc5c23:image.png)
        
1. Download and extract the files that you need for this lab.
    - In the VS Code IDE bash terminal (at the bottom of the IDE), run the following command:
        
        ```bash
        wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-91558/13-lab-ci-cd/code.zip -P /home/ec2-user/environment
        ```
        
        ![image.png](attachment:2d8185ed-10c3-49cb-8854-6aaf13e42bbe:image.png)
        
    - Notice that a **code.zip** file was downloaded to the VS Code IDE. The file is listed in the Environment window.
    - Extract the file:
        
        ```
        unzip code.zip
        ```
        
2. Run a script that upgrades the version of the AWS CLI installed on the VS Code IDE.
    - Set permissions on the script so that you can run it, and then run it:
        
        ```bash
        chmod +x ./resources/setup.sh && ./resources/setup.sh
        ```
        
    - When prompted for an IP address, enter the IPv4 address that the internet uses to contact your computer. You can find your IPv4 address at [https://whatismyipaddress.com](https://whatismyipaddress.com/).
        
        **Note**: The IPv4 address that you set is the one that will be used in the bucket policy. Only requests that originate from this IPv4 address will be allowed to load the website pages. Do not set it to 0.0.0.0 because the S3 bucket's block public access settings will prevent access.
        
    - When prompted for an email address, enter one that you have access to as you complete the lab.
3. Verify the version of AWS CLI installed.
    - In the VS Code IDE Bash terminal, run the following command:
        
        ```bash
        aws --version
        ```
        
        The output should indicate that version 2 is installed.
        
4. Verify that the SDK for Python is installed.
    - Run the following command:
        
        ```
        pip3 show boto3
        ```
        
        **Note**: If you see a message about not using the latest version of pip, ignore the message.
        

Keep the VS Code IDE open in your browser. You will use it later in this lab.

## **Task 2: Creating a CodeCommit repository**

When a developer manages code in their local working environment, it is difficult to track and manage changes. This approach also limits the ability for multiple developers to collaborate on the same codebase.

AWS CodeCommit is a secure, highly scalable, managed source control service that hosts private Git repositories. CodeCommit makes it easy for teams to securely collaborate on code with contributions encrypted in transit and at rest.

In this task, you will create a CodeCommit repository to host the café website code.

1. Create a CodeCommit repository to host the codebase.
    - In the search bar at the top, search and select `CodeCommit` to open the AWS CodeCommit service console.
    - From the navigation pane, choose **Create repository**.
    - Configure the following settings:
        - **Repository name:** Enter `front_end_website`
        - **Description:** Enter `Repository for the cafe website front end`
        - Keep the rest of the default settings.
    - Choose **Create**.
2. Create a test file.
    - In the **front_end_website** section, choose **Create file**.
    - Copy and paste the following code into the **front_end_website** text box:
        
        ```html
        <!DOCTYPE html>
        <html>
        	<head>
        	<title>Test page</title>
        	</head>
        	<body>
                 <h1>
        		    This is a sample HTML page.
                 </h1>
        	</body>
        </html>
        
        ```
        
    - In the **Commit changes to main** section, configure the following settings:
        - **File name:** Enter `test.html`
        - **Author name:** Enter your name.
        - **Email address:** Enter the same email address that you used in Task 1.
        - **Commit message:** Enter `This is my first commit.`
        - Choose **Commit changes**.
        
        ![image.png](attachment:7e3605bb-6c09-46af-b992-d26722300c13:image.png)
        
3. Review your commit.
    - In the left navigation pane, under **Repositories**, choose **Commits**.
    - Choose the link for the commit ID. Only one should be listed.
    - Review the information about this commit.
        
        **Note:** On this page, you see the author of the commit, the commit message, the date of the commit, and the name and contents of the file that was added.
        

Now that you have created a CodeCommit repository to host and manage your code changes, you can use it as a source to automate publishing updates to the café website.

## **Task 3: Creating a pipeline to automate website updates**

So far, you have been running commands from VS Code IDE to update the code in the S3 bucket for the website. In this task, you will configure a CI/CD pipeline by using CodePipeline to automate the website updates. The pipeline will use the code in your CodeCommit repository to deploy changes to the website's S3 bucket.

1. Return to the VS Code IDE.
2. In the Explorer section, expand *Environment*, which is located in the upper-left corner.
3. Expand the **resources** folder, and open the file named **cafe_website_front_end_pipeline.json**.
    
    This file defines configuration that will be used to deploy your new pipeline. Review the following code snippets to understand how the pipeline is configured.
    
    The following lines declare the AWS Identity and Access Management (IAM) role that will be associated with the pipeline.
    
    ```json
    "pipeline": {
      "roleArn": "arn:aws:iam::<FMI_1>:role/RoleForCodepipeline",
    ```
    
    The following code snippet defines the *Source* that your pipeline will use to create and update your application. In this case, the source is your CodeCommit repository, *front_end_website*. Note that the pipeline will be configured to use the *main* branch.
    
    ```json
    {
      "name": "Source",
       "actions": [
        {
            "inputArtifacts": [],
             "name": "Source",
             "actionTypeId": {
             "category": "Source",
             "owner": "AWS",
             "version": "1",
             "provider": "CodeCommit"
             },
             "outputArtifacts": [
               {
                 "name": "MyApp"
               } 
             ],
             "configuration": {
                "RepositoryName": "front_end_website",
                 "BranchName": "main"
              },
              "runOrder": 1
             }
           ]
        }
    ```
    
    The following section of code defines the *Deploy* stage. The deployment will update code in Amazon S3. The *configuration* settings define details about the deployment target. In this case, this section defines the S3 bucket name, configures files to be extracted from a .zip file, and sets a caching policy.
    
    ```json
    {
        "name": "Deploy",
        
        "actions": [
            {
                "inputArtifacts": [
                    {
                        "name": "MyApp"
                    }
                ],
                "name": "CafeWebsite",
                "actionTypeId": {
                    "category": "Deploy",
                    "owner": "AWS",
                    "version": "1",
                    "provider": "S3"
            },
                "outputArtifacts": [],
                "configuration": {
                    "BucketName": "<FMI_2>",
                    "Extract": "true",
                    "CacheControl": "max-age=14"
                },
                "runOrder": 1
            }
        ]
    }
    ```
    
    The final section of the code defines the *artifactStore*. This is the S3 bucket where CodePipeline artifacts will be stored.
    
    ```json
        "artifactStore": {
            "type": "S3",
            "location": "codepipeline-us-east-1-<FMI_1>-website"
        },
        "name": "cafe_website_front_end_pipeline",
        "version": 1
    }
     
    ```
    
4. Update the cafe_website_front_end_pipeline.json file:
    - Replace the two *<FMI_1>* placeholders with the your AWS account ID.
        
        **Note:** To find your account ID, run the following command: `aws sts get-caller-identity`
        
    - Replace the *<FMI_2>* placeholder with the name of the bucket that has *s3bucket* in the name.
        
        **Note:** To retrieve a list of the S3 buckets in your account, run the following command: `aws s3 ls`
        
    - From the navigation pane, choose  menu, then choose **File > Save**.
5. To create the pipeline, run the following commands.
    
    ```bash
    cd ~/environment/resources
    aws codepipeline create-pipeline --cli-input-json file://cafe_website_front_end_pipeline.json
    ```
    
    The command returns output similar to the following example:
    
    ```json
    {
        "pipeline": {
        "name": "cafe_website_front_end_pipeline",
            "roleArn": "arn:aws:iam::111122223333:role/RoleForCodepipeline",
            "artifactStore": {
                "type": "S3",
                "location": "codepipeline-us-east-1-111122223333-website"
            },
            "stages": [
                {
                    "name": "Source",
                    "actions": [
                        {
                            "name": "Source",
                            "actionTypeId": {
                                "category": "Source",
                                "owner": "AWS",
                                "provider": "CodeCommit",
                                "version": "1"
    ```
    
    You may need to press the Q key to return to the command prompt.
    
    If you receive an error, double check your account ID and the bucket name that you used to update the placeholders. Correct any typos in the cafe_website_front_end_pipeline.json file and run the command again.
    
6. Navigate to the CodePipeline console.
    
    The **Pipelines** section lists the **cafe_website_front_end_pipeline** Pipeline.
    
7. Choose the **cafe_website_front_end_pipeline** hyperlink and review the pipeline status, as shown in the following image.
    
    ![image.png](attachment:e0d84456-f0db-4ba8-a697-15d8556825dc:image.png)
    
    The pipeline should deploy successfully. Code was deployed using CodeCommit as the source and the café website S3 bucket as the target. This means the bucket should have been updated with the *test.html* file.
    
8. Verify the automated deployment.
    - Return to the VS Code IDE bash terminal.
    - To find your Amazon CloudFront distribution domain name, run the following command:
        
        ```bash
        aws cloudfront list-distributions --query DistributionList.Items[0].DomainName --output text
        ```
        
    - Update the following URL by replacing *<cloudfront_domain>* with the value that was returned by the previous command: `https://<cloudfront_domain>/test.html`
        
        The updated URL is similar to [*https://aaabbb111222.cloudfront.net/test.html*](https://aaabbb111222.cloudfront.net/test.html).
        
    - Open a new browser tab, and enter the URL that you just created.
        
        You reach a sample webpage similar to the following:
        
        ![image.png](attachment:7f5ca3f3-9213-4ab1-90c2-b354ea002a6d:image.png)
        
    - Keep this browser tab open. You will return to it later.

Well done! Now, when you update the repository, the pipeline will automatically update your website.

Next, you will clone the repository to your VS Code IDE. This will provide the ability to edit the files locally and synchronize with your centralized CodeCommit repository.

## **Task 4: Cloning a repository in VS Code IDE**

It's more efficient to edit code in an IDE than it is to use the CodeCommit console. In this task, you will clone a local copy of the repository in your VS Code IDE work environment.

1. Retrieve the SSH clone URL for your repository.
    - Navigate to the CodeCommit console.
    - In the navigation pane, choose **Repositories**, then choose the repository created *front_end_website*.
    - In the **Clone URL** column, choose **Clone HTTPS(GRC)** to copy the URL to your clipboard.
    
    Note: URL should look similar to *codecommit::us-east-1://front_end_website*
    
2. Clone your repository using the SSH URL.
    - Return to the VS Code IDE terminal.
    - Run the following command to clone the repository to your VS code IDE, replace the *<<Clone URL>*> with the value you copied.
        
        ```bash
        cd ..
        git clone <<Clone URL>>
        ```
        
    - Note the output similar to below
        
        ```basic
        Cloning into 'front_end_website'... remote: Counting objects: 3, done.Unpacking objects: 100% (3/3), 290 bytes | 290.00 KiB/s, done
        ```
        
    - Note that the file is also visible in the explorer
    
    ![image.png](attachment:b8189e3b-7507-4cef-b885-a104432309e3:image.png)
    

You now have a local clone of your repository that can be edited using your IDE. Next, you will lean how to manage your local copy of the repository and synchronize changes with CodeCommit.

## **Task 5: Exploring the Git integration with the VS Code IDE**

You can interact with the repository through the command line, but VS Code provides Git integration in the IDE to make it easier to manage your repository. In this task, you will perform simple repository management operations by using this integration.

1. Explore repository branch management.
    - Locate the branch icon, which is located in the lower-left corner of the IDE.
        - Source Control option is opened.
    - From the top, choose three dots  to the right of *SOURCE CONTROL*.
    - From the menu displayed, choose **Source Control Repositories**
    - Your repository *front_end_website* is displayed.
        
        The IDE is currently set up to communicate with the **main** branch.
        
2. Explore and edit the repository files.
    
    Remember that the repository was cloned to the local folder environment/front_end_website. You can open repository files in your IDE to view and edit them.
    
    - From the explorer menu, expand the **front_end_website** folder to reveal the **test.html** file, which is shown in the following image.
    
    ![image.png](attachment:f71c7042-4266-4fab-ac22-98816512b4cf:image.png)
    
    - Open the **test.html** file and edit the page title on line 4. Replace the current title with the following text:
        
        ```
        Best test page ever.
        ```
        
    - Save your changes.
    - Number **1** now appears next to **branch** icon in the left pane. This indicates that changes have been made to the code that need to be committed to the branch.
    - Hover over  and you will notice a message **Source Control 1 pending change**
        
        **Note:** This is because you made a change to the file but it is not yet committed to thr repository main branch.
        
3. Commit your changes to the **main** branch.
    - Choose the  icon to open the **Source Control**.
        
        **Note:** You don't have to use the IDE integration. You could also use Git from the command line. For example, if you enter the following command, you should find the *test.html* file listed in the output.
        

```bash
cd ~/environment/front_end_website
git status
```

- In the **Message** text box, enter `Updated the title` as shown in the following image.
    
    **Note:** Notice that one file, *test.html*, is listed under **Changes**. This is the same file that was returned by the *git status* command.
    
- On the **Commit** button, choose  *more actions* which is located to the right.
- Choose *Commit & Push*, then choose **Yes**.
- This will push the code and trigger the code pipeline which deploys new version of the file to website.
1. Review the changes in CodeCommit.
    - Navigate to the CodeCommit console.
    - Choose the **front_end_website** repository link.
    - In the navigation pane, under **Repositories**, choose **Commits**.
    - Choose the link for the commit ID with the most recent commit date.
    - Go to the **test.html** section, and notice the highlighted lines, which are shown in the following image.
    
    ![image.png](attachment:fd39baa9-13cc-408b-a660-311b2f49aff6:image.png)
    

The first highlighted line is preceded by a minus sign (-) and highlighted in red. This shows the contents of the line before the change was made. The second highlighted line is preceded by a plus sign (+) and highlighted in green. This line shows the current content of the line.

Great work! You have confirmed that VS Code IDE is able to push changes to your CodeCommit repository.

1. Now, return to the tab that contains the café website and refresh the page.
    
    Notice that the browser tab title has changed, as shown in the following image. This proves that the pipeline deployed the changes that you committed from your local repository.
    
    ![image.png](attachment:f56a0b49-b2dd-474e-9725-19a26a04f6c0:image.png)
    

**Note:** You might need to refresh the page a few times before you see the new tab title.

Now that you have learned to manage your local repository and synchronize it with CodeCommit, it's time to update the repository with the actual café website code.

## **Task 6: Pushing the café website code to CodeCommit**

In this task, you will first remove the test.html file. Then, you will update the local repository with the café website code. Finally, you will verify that your pipeline built the café website on S3. You will also verify that *max-age* is set to 14 to confirm that the latest changes are being applied and the caching was updated.

1. Return to the VS Code IDE tab.
2. On the explorer, Under *Environment* folder, expand the **front_end_website** folder and delete the **test.html** file, as shown in the following image.

![image.png](attachment:eca96696-3ff9-4f07-b667-afce6f8c4994:image.png)

1. In the VS Code IDE bash terminal, run the following commands.
    
    The second command will copy all of the contents under the *website* folder to the *front_end_website* folder.
    

```bash
cd ~/environment
cp -r ./resources/website/* front_end_website
```

1. To delete the *website* folder, so that there is one source of truth, run the following command.

```bash
rm -r ./resources/website
```

Now, your local repository has the folder structure shown in the following image:

![image.png](attachment:a7edfccc-abc4-468f-95e5-53b6a3982df0:image.png)

1. Commit your changes.
    - Choose the *Branch* icon (now it is showing number of new files to be pushed).
    - Enter the following commit message: `Providing the website`
    - On the **Commit** button, choose  *more actions* which is located to the right.
    - Choose *Commit & Push*, then choose **Yes**.
    - To the right of **front_end_website**, choose the options icon and choose **Commit**.
    - The **Source control Graph** in the lower pane shows history of changes done to the repository.
2. Return to the browser tab that shows the *test.html* page.
3. Update the URL by removing **test.html** from the address, and then submit the updated URL.
    
    The updated address is similar to the following example:
    `https://aaabbb111222.cloudfront.net`
    
    The browser now displays the café website. Well done!
    
4. Verify that your pipeline applied the cache-control setting, which was configured in Task 3.
    - Remain on the café website page, and open your browser's developer tools.
        
        **Note:** To open the developer tools, open your browser's context menu (right-click) and choose **Inspect**.
        
    - Choose the **Network** tab, and then refresh the webpage.
    - Choose **pastries.js**.
    - Choose the **Headers** tab, and locate the **Response Headers** section, as shown in the following image.
        
        ![image.png](attachment:799217c5-f8e1-409d-be70-40f13feb7030:image.png)
        
        Notice that the **cache-control** value is set to *max-age=14*, which indicates that pipeline updated the cache settings. This means that the website is being built from the  most recent repository update.
        
        **Note:** If **cache-control** is set to *max-age=0*, the pipeline might still be applying the update to the S3 bucket. Wait a few seconds, refresh the page, and choose **pastries.js** again.
        

Congratulations! You have moved the codebase to a secure and scalable managed service. You have also ensured that the website will stay up to date with the latest improvements.

## **Update from the café**

![image.png](attachment:56e78cc9-4ae7-4302-bc7d-964ec6779885:image.png)

This is a big win for the café! Now that the codebase is centralized, the café can bring in more developers to collaborate and enhance the site as the business grows. Sofía doesn't have to remember to update the website, because the CI/CD pipeline will automatically deploy changes to the website. She can track versioning and implement a code approval process. If any issues arise from updates, she can use CodeCommit to review commit logs to trace changes and resolve code bugs.

# Build and Deploy React App to AWS S3 using CircleCI



## Step 1: Create your app
```
$ npx create-react-app ci-cd-s3-example
  cd example-app
  npm start
```

## Step 2: Push to Github
Create repo and push your project to the Github repo
```
$ git init
  git add .
  git commit -m "first commit"
  git branch -M main
  git remote add origin https://github.com/account-name/project-name.git
  git push -u origin main
```

## Step 3: Setup AWS S3 Bucket

1. The first step is to create an IAM user with programatic access and save Access and Secret keys.

2. The next step is to assign Administrative Access to the user.

3. Next step on AWS is to Create Bucket withh all public access.

4. On the Permissions page, under Bucket policy, paste the following code. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::bucket-name/*"
        }
    ]
}
```

5. Click on Properties, go down to Static website hosting. Enable static website.

6. Click edit, type index.html inside index document and save.

## Step 4: Setup Circleci

1. Go to https://circleci.com/signup/ and signup with your existing Github account.
2. Click Setup Up Project and choose the repo.
    1. If you have already .circleci folder with config.yml then choose **fastest** and type branch name.
    2. if you want to add new branch with .circleci folder and default config.yml then choose **faster**.
    3. If you want to add config.yml with edition then choose **faster** and select Node option.

    Edit config.yml with the following:

    ```
    version: 2.1
        jobs:
        build:
            working_directory: ~/repo
            docker:
            - image: circleci/node:12-browsers
            steps:
            - checkout
            - restore_cache:
                keys:
                - v1-dependencies-{{ checksum "package-lock.json" }}
                - v1-dependencies-
            - run:
                name: Install dependencies
                command: npm install
            - save_cache:
                key: v1-dependencies-{{ checksum "package-lock.json" }}
                paths:
                    - node_modules
            - run:
                name: "What branch am I on now?"
                command: echo $CIRCLE_BRANCH
            - run:
                name: Build
                command: |
                    if [ $CIRCLE_BRANCH = 'main' ]; then
                    npm run build
                    fi
            - persist_to_workspace:
                root: .
                paths:
                    - .
        deploy:
            working_directory: ~/repo
            docker:
            - image: innovatorjapan/awscli:latest
            steps:
            - attach_workspace:
                at: .
            - run:
                name: "What branch am I on now?"
                command: echo $CIRCLE_BRANCH
            - run:
                name: Deploy
                command: |
                    if [ $CIRCLE_BRANCH = 'main' ]; then
                    aws s3 sync build s3://ci-cd-s3-example --delete --exact-timestamps;
                    fi
        workflows:
        build_and_deploy:
            jobs:
            - build:
                filters:
                    branches:
                    only:
                        - main
            - deploy:
                requires:
                    - build
                filters:
                    branches:
                    only:
                        - main
    ```

    4. Click Commit and Run

    5. Job will fail. It is OK.

    6. Go to Enviromental Variables and add the following:

    ```
    AWS_ACCESS_KEY_ID = ****
    AWS_DEFAULT_REGION = ****
    AWS_SECRET_ACCESS_KEY = ****
    ```
    7. Go back to your project and rerun failed worklow. Then the workflow should run successfully 

## Step 5: Browse wesite

1. After the successful build, head over to your AWS S3 and reload your bucket and see build files reflected.
2. Now, to visit your hosted site, click on Properties and scroll down to Static website hosting
3 Click on the full url to be redirected to your site:
 - http://ci-cd-s3-example.s3-website.ap-northeast-2.amazonaws.com/



pipeline{
    agent any 
      environment {        
        ACCOUNT_NO          = '913211044704'
        AWS_DEFAULT_REGION  = 'us-east-1'
        ECR_REPO            = 'elastic'
        S3_Bucket           = 'elascc'
        JOB_NAME            = 'beanstalk'
        APPLICATION_NAME    = 'Dhaval'	
        EB_ENVIRONMENT_NAME = 'Dhaval-env-1' 
        AWS_PROFILE         = 'default'  
   	
    }

    stages{
         stage('Building docker image'){
            steps {
                script 
                {
                    sh '''
                    docker build -t elastic -f Dockerfile .
                    '''
                }
           }
        }
         stage('Pushing image to ECR'){
            steps {
                script 
                {
                    sh '''
                    eval $(aws ecr get-login --region $AWS_DEFAULT_REGION --profile $AWS_PROFILE | sed "s/-e none //")                    
                    docker tag elastic:latest $ACCOUNT_NO.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$ECR_REPO:$BUILD_ID                  
                    docker push $ACCOUNT_NO.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$ECR_REPO:$BUILD_ID   
                    docker rmi elastic:latest        
                    '''
                }
           }
        }
        stage('Archive docker run file.'){
            steps {
                script 
                {
                    sh '''
                    sed -i 's+ecr_image+'$ACCOUNT_NO'.dkr.ecr.'$AWS_DEFAULT_REGION'.amazonaws.com/'$ECR_REPO':'$BUILD_ID'+g' Dockerrun.aws.json
                    zip -r app.zip  Dockerrun.aws.json
                    aws s3 cp app.zip s3://$S3_Bucket/$EB_ENVIRONMENT_NAME-$BUILD_ID/ --profile $AWS_PROFILE        
                    '''
                }
           }
        }
         stage('Deploy app.'){
            steps {
                script 
                {
                    sh '''
                    aws elasticbeanstalk delete-application-version --application-name $APPLICATION_NAME --version-label $EB_ENVIRONMENT_NAME-$BUILD_ID --delete-source-bundle --region $AWS_DEFAULT_REGION --profile $AWS_PROFILE
                    aws elasticbeanstalk create-application-version --application-name $APPLICATION_NAME --version-label $EB_ENVIRONMENT_NAME-$BUILD_ID --source-bundle S3Bucket=$S3_Bucket,S3Key=$EB_ENVIRONMENT_NAME-$BUILD_ID/app.zip --region $AWS_DEFAULT_REGION --profile $AWS_PROFILE
                    aws elasticbeanstalk update-environment --environment-name $EB_ENVIRONMENT_NAME --version-label $EB_ENVIRONMENT_NAME-$BUILD_ID --region $AWS_DEFAULT_REGION --profile $AWS_PROFILE                                                                                                                                                   
                    '''
                }
           }
        }                          
    }
}

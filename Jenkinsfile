pipeline {
  agent any

  environment {
    STACK_NAME     = 'acit-ec2-trio'
    TEMPLATE_FILE  = 'ec2-trio.yaml'
    REGION         = 'us-east-2'

    // Replace these with real values or pass via Jenkins parameters
    VPC_ID         = ' vpc-0a4adde1cad005409'
    SUBNET_ID      = 'subnet-0cb91aa4bd49ae30f'
    KEY_NAME       = 'ec2-user'         // must exist in us-east-2
    AMI_ID         = 'ami-0c803b171269e2d72'      // Amazon Linux 2 in us-east-2 (verify)
    INSTANCE_TYPE  = 't2.micro'
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout Source') {
      steps {
        echo 'Checking out Git repo'
        checkout scm
      }
    }

    stage('Deploy EC2 Trio Stack') {
      steps {
        echo 'Launching 3 EC2 instances'
        sh '''
          aws cloudformation deploy \
            --stack-name "$STACK_NAME" \
            --template-file "$TEMPLATE_FILE" \
            --region "$REGION" \
            --capabilities CAPABILITY_NAMED_IAM \
            --parameter-overrides \
              VpcId="$VPC_ID" \
              SubnetId="$SUBNET_ID" \
              KeyName="$KEY_NAME" \
              AmiId="$AMI_ID" \
              InstanceType="$INSTANCE_TYPE"
        '''
      }
    }

    stage('Describe Stack Outputs') {
      steps {
        echo 'EC2 Instance IDs:'
        sh '''
          aws cloudformation describe-stacks \
            --stack-name "$STACK_NAME" \
            --region "$REGION" \
            --query "Stacks[0].Outputs[*].[OutputKey,OutputValue]" \
            --output table
        '''
      }
    }
  }

  post {
    success {
      echo 'EC2 stack deployed successfully!'
    }
    failure {
      echo 'Deployment failed. Check above for errors.'
    }
  }
}


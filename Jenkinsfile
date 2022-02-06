pipeline{
  agent {label 'slave'}
  parameters{
    string (name: 'keyname', defaultValue: 'sshkey1', description: ' ')
    string (name: 'SecurityGroup', defaultValue: 'sg-017c097bb1674f881', description: ' ')
    string (name: 'Subnet', defaultValue: 'subnet-0ad08acb398ada09d', description: ' ')
            }
  stages{
       stage('hosting application'){
        steps{
          sh "ls"
          sh "aws rds create-db-instance --db-instance-identifier test-mysql-instance1 --db-name cpms --db-instance-class db.t2.micro --vpc-security-group-ids "+SecurityGroup+" --engine mysql --engine-version 5.7 --db-parameter-group-name default.mysql5.7 --publicly-accessible  --master-username admin --master-user-password ramana4u2021 --allocated-storage 10 --region us-east-2"
          sleep(450)
          script{
              def cmd = "aws rds describe-db-instances --db-instance-identifier test-mysql-instance1 --region us-east-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem = readJSON text: output
              println(jsonitem)
              myJson = jsonitem['DBInstances']['Endpoint']['Address']
              println(myJson)
           }
           sh "sudo sed -i.bak 's/endpoint/${myJson}/g' userdata.txt"
          script{
              def cmd = "aws elbv2 create-load-balancer --name my-load-balancer --subnets "+Subnet+" subnet-0a22ca2d020ca46c1 --security-groups "+SecurityGroup+" --region us-east-2 "
              def output = sh(script: cmd,returnStdout: true)
              jsonitem1 = readJSON text: output
              println(jsonitem1)
              sleep(100)
            }
          script{
              def cmd = "aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 --target-type instance --vpc-id vpc-048331c397b1a9bc3 --region us-east-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem2 = readJSON text: output
              println(jsonitem2)
              sleep(180)
               }
           sh "aws elbv2 create-listener --load-balancer-arn ${jsonitem1['LoadBalancers'][0]['LoadBalancerArn']} --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --region us-east-2"
           sh "aws autoscaling create-launch-configuration --launch-configuration-name my-lc3-cli --image-id ami-0fb653ca2d3203ac1 --instance-type t2.micro --security-groups " + SecurityGroup + " --key-name " + keyname + " --iam-instance-profile demo-Role --user-data file://userdata.txt --region us-east-2"
           sh "aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg3-cli --launch-configuration-name my-lc3-cli --max-size 2 --min-size 1 --desired-capacity 1 --target-group-arns ${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --availability-zones us-east-2c --region us-east-2"
        }
       }
  }
}

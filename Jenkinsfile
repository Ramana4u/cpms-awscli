pipeline{
  agent {label 'main'}
  parameters{
    string (name: 'keyname', defaultValue: 'sshkey1', description: ' ')
    string (name: 'SecurityGroup', defaultValue: 'sg-017c097bb1674f881', description: ' ')
    string (name: 'Subnet', defaultValue: 'subnet-0ad08acb398ada09d', description: ' ')
            }
  stages{
       stage('LB'){
        steps{
            sh "aws elb create-load-balancer --load-balancer-name my-load-balancer --listeners "protocol HTTP, LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80,default-actions {Type=forward,TargetGroupArn=${jsonitem['TargetGroups'][0]['TargetGroupArn']}}" --subnets "+Subnet+"  --security-groups " + SecurityGroup + " --region us-east-2"
            script{
              def cmd = "aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 --target-type ip --vpc-id vpc-048331c397b1a9bc3 --region us-east-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem = readJSON text: output
              println(jsonitem)
              sleep(180)
            }
        }
    }
    stage('Create Launch config'){
      steps{
         sh "aws autoscaling create-launch-configuration --launch-configuration-name my-lc3-cli --image-id ami-0fb653ca2d3203ac1 --instance-type t2.micro --security-groups " + SecurityGroup + " --key-name " + keyname + " --iam-instance-profile demo-Role --user-data file://userdata.txt --region us-east-2"
          } 
       }
    stage('ASG'){
      steps{
         sh "aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg3-cli --launch-configuration-name my-lc3-cli --max-size 2 --min-size 1 --desired-capacity 1 --target-group-arns ${jsonitem['TargetGroups'][0]['TargetGroupArn']} --availability-zones us-east-2c --region us-east-2"
          } 
        }
  }
}

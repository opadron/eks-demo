
#### Initial notes

- EKS cluster setup materials based adapted from the AWS EKS User Guide.
  - [link](https://docs.aws.amazon.com/eks/latest/userguide)
  - Demo will follow the management console instructions.

- Environment variables:

  ```
  export AWS_PROFILE=iarpa                 # my AWS profile name
  export AWS_REGION=us-west-2              # AWS region
  export KUBECONFIG=~/.kube/configs/iarpa  # location of my kubeconfig
  ```

- Prerequisits:
  - [aws-cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
  - [aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)
  - [kubectl client](https://kubernetes.io/docs/tasks/tools/)
  - [helm](https://helm.sh/docs/intro/install)


#### Create EKS cluster, use existing private subnets.

- Create an IAM Role for EKS control plane.
  - [link](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html)

- Create an IAM Role for handling cluster authentication.
  - Edit trust relationships; list authorized user ARNs.
  
    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": [
              "arn:aws:iam::123456789012:user/JohnDoe",
              "arn:aws:iam::123456789012:user/JaneDoe"
            ]
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```

#### Set up OIDC provider for cluster (needed for later)

- [link](https://docs.aws.amazon.com/eks/latest/userguide/authenticate-oidc-identity-provider.html)


#### Bootstrap IAM authentication for cluster

- [link](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)
- Follow the section titled "To add an IAM user or role to an Amazon EKS
  cluster".
- Update your local kubeconfig to assume the new role when authenticating.


#### Set up fargate

- [link](https://docs.aws.amazon.com/eks/latest/userguide/fargate-getting-started.html)
- create IAM role for fargate pod execution
- create fargate profile
  - use the above role
  - only private subnets
  - namespaces:
    - default
    - kube-system
    - cert-manager
    - flux
    - ingress-nginx
- patch the coredns deployment
  - remove the `compute-type : ec2` annotation


#### Set up application/network load balancing

- tag public and private subnects for ALB and NLB
  - [link](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)

- install AWS load balancer controller
  - [link](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)


#### Install kubernetes metrics server

- [link](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html)


#### Install vertical pod autoscaler

- [link](https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.html)


#### (Optional) read up on horizontal pod autoscaler

- [link](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html)


#### Demo 1: Sample application

- php-apache
  - without vpa or hpa
  - add vpa + recreate pod
  - add hpa + stress test with loader job


#### Demo 2: IARPA Smart task port

- add image pull secrets
- demo job template


#### Bonus Demo: job cleaner

- show how to work around the lack of a job cleaner with a custom cron job.

#### Wind Down

1. Delete fargate profiles
2. Delete cluster
3. Delete IAM role for fargate pod execution
4. Delete IAM policy for load balancer controller
5. Delete IAM role for cluster access
6. Delete IAM role for EKS control plane


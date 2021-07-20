
#### Initial notes

- EKS cluster setup materials based adapted from the AWS EKS User Guide.
  - [link](https://docs.aws.amazon.com/eks/latest/userguide)
  - Demo will follow the management console instructions.

- Environment variables:

  ```console
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
  
    ```json
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

- [link](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)


#### Bootstrap IAM authentication for cluster

- Configure kubectl for cluster access.

  ```console
  aws eks update-kubeconfig --name EKS_DEMO
  ```

  - [link](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#eks-configure-kubectl)

  - Ensure that you can access the cluster.

    ```console
    kubectl get svc
    ```

- Add the cluster access role the aws-auth config map.
  - [link](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)

- Update your local kubeconfig to assume the new role when authenticating.

  ```diff
         - us-west-2
         - eks
         - get-token
  +      - --role
  +      - arn:aws:iam::001122334455:role/ROLE_NAME
         - --cluster-name
         - CLUSTER_NAME
         command: aws
  ```

  - Ensure that you can access the cluster.

    ```
    $ kubectl get svc
    ```


#### Set up fargate

- [link](https://docs.aws.amazon.com/eks/latest/userguide/fargate-getting-started.html)
- create IAM role for fargate pod execution
- create fargate profile
  - use the above role
  - only private subnets
  - namespaces:
    - default
    - kube-system
- patch the coredns deployment
  - remove the `compute-type : ec2` annotation


#### Set up application/network load balancing

- tag public and private subnets for ALB and NLB
  - [link](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)

- install AWS load balancer controller
  - [link](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)
  - IMPORTANT: Since we're only running on Fargate, EC2 metadata is not
    available, so region and vpc id have to be specified explicitly. See
    [this](https://github.com/kubernetes-sigs/aws-load-balancer-controller/issues/1659#issuecomment-728369347)
    issue for details.

    ```console
    helm upgrade -i aws-load-balancer-controller \
        eks/aws-load-balancer-controller         \
            --set clusterName=CLUSTER_NAME       \
            --set serviceAccount.create=false    \
            --set serviceAccount.name=aws-load-balancer-controller \
            --set region="$AWS_REGION"           \
            --set vpcId=vpc-00112233445566778 \
            -n kube-system
    ```

- apply ingress class in this repo

  ```console
  kubectl apply -f ingress-class.yaml
  ```


#### (Optional, for Demo 1) Install kubernetes metrics server

- [link](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html)


#### (Optional, for Demo 1) Install vertical pod autoscaler

- [link](https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.html)


#### (Optional, for Demo 1) read up on horizontal pod autoscaler

- [link](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html)


#### (Optional) Demo 1: Sample application

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
7. Delete OIDC identity provider for cluster (IAM -> identity providers)
8. Remove the tags on the public and private subnets.


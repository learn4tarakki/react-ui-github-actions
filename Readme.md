# React with Vite + Npm + AWS 

# Youtube Video explaining this repo: 
- https://youtu.be/3Czf9vzZ0jI

# LinkedIn Post: 
- https://www.linkedin.com/posts/anup-g-82510415_deploy-react-app-on-aws-with-github-actions-activity-7081943044618358784-jysc?utm_source=share&utm_medium=member_desktop

# AWS assume role used in deploy.yaml file
- `github-assume-role` IAM role (arn:aws:iam::362942319756:role/github-assume-role) 

- # Use Github OIDC Provider to AWS, so that Github Actions can access AWS resources. 
- steps: 
    - first we use aws-actions/configure-aws-credentials github action
    - it ask us to create role in AWS and use this role arn in the action's 'role-to-assume' parameter 
        - To create role (example - github-assume-role in AWS account of Anup), we go to AWS IAM roles and click create role: 
        - Trusted Policy: 
            - choose Web Identity
            - click on create new - to add custom identity
            - use following details to create custom identity: 
                - provider URL: Use https://token.actions.githubusercontent.com
                - audience": Use sts.amazonaws.com
            - Go to newly added Github OIDC in AWS and make sure following thumbprints are present, check this link: https://github.blog/changelog/2023-06-27-github-actions-update-on-oidc-integration-with-aws/
                - The two known intermediary thumbprints at this time are:
                    - 6938fd4d98bab03faadb97b34396831e3780aea1
                    - 1c58a3a8518e8759bf075b76b750d4f2df264fcd    
            - then come back on (select trusted identity step again) and select the custom IdP that just created by you from dropdown and click next.
            - Following is the trust policy, should ultimate be like:
            - ```
                    {
                        "Version": "2012-10-17",
                        "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Federated": "arn:aws:iam::362942319756:oidc-provider/token.actions.githubusercontent.com"
                            },
                            "Action": "sts:AssumeRoleWithWebIdentity",
                            "Condition": {
                                "StringEquals": {
                                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                                },
                                "StringLike": {
                                    "token.actions.githubusercontent.com:sub": "repo:learn4tarakki/*:*"
                                }
                            }
                        }]
                    }
                ```
                - Permission policy attached to `github-assume-role`, to push to AWS S3
                ```
                {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Sid": "VisualEditor0",
                            "Effect": "Allow",
                            "Action": [
                                "s3:PutObject",
                                "s3:GetObject",
                                "s3:CreateBucket",
                                "s3:ListBucket"
                            ],
                            "Resource": [
                                "arn:aws:s3:::react-vite",
                                "arn:aws:s3:::react-vite/*"
                            ]
                        }
                    ]
                }
                ```
                - Permission policy attached to `github-assume-role` IAM role, to invalidate AWS Cloudfront cache
                ```
                {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Sid": "Statement1",
                            "Effect": "Allow",
                            "Action": [
                                "cloudfront:CreateInvalidation",
                                "cloudfront:GetDistribution",
                                "cloudfront:GetStreamingDistribution",
                                "cloudfront:GetDistributionConfig",
                                "cloudfront:GetInvalidation",
                                "cloudfront:ListInvalidations",
                                "cloudfront:ListStreamingDistributions",
                                "cloudfront:ListDistributions"
                            ],
                            "Resource": [
                                "arn:aws:cloudfront::362942319756:distribution/*"
                            ]
                        }
                    ]
                }
                ```
- References: 
    - https://scalesec.com/blog/oidc-for-github-actions-on-aws/
    - https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services
    - https://github.com/aws-actions/amazon-ecr-login
    - https://github.com/aws-actions/configure-aws-credentials
    - https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-push.html

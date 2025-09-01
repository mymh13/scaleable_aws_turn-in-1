#### This project is created by mymh13 (N.Häll) for the CLO24 - Skalbara Molnapplikationer course during the Clouddeveloper Program at Campus Mölndal. (2025)  
  
This project will primarily be in Swedish.  
  
Uppgiftsbeskrivning på svenska följer längre ner.
  
Normally I primarily work using English, but this is just a repo for a school assignment where I chose to use Swedish for once. The assignment is to create a scaleable AWS-runtime environment hosting Wordpress.  
  
Denna uppgift beskrivs i introduktionen till uppgiftens pdf (kommer bifogas in i detta repo senare, det ligger externt för stunden):  
  
"Målet med själva uppgiften är att kunna bygga en driftsmiljö i AWS som kan hosta Wordpress: miljön skall vara skalbar, fungerande och ha en grundläggande robusthet med viss säkerhet.
Avgränsningen för uppgiften nämns specifikt, därför har jag för avsikt att presentera ett ”hur bygger vi vidare på detta?”-avsnitt som kan fungera på tre sätt:  
  
- dels visar den på grundläggande kunskap
- dels visar det att skalbarhetsmålet nås och finns med redan initialt
- dels så vill jag visa på en flexibilitet: jag bygger gärna själv mer avancerade lösningar och testar teknik, men:  
  
Ibland är det absolut nödvändigt att följa en uppgiftsbeskrivning, och där kan dokumentationen fungera som ett bra område att fånga upp potentiella förslag till förbättringar. Jag vill gärna simulera det i denna uppgift."  
  
---
#### Repository Layout
```yaml
/infra
    /templates
        root.yaml # master stack – orchestrates Nested Stacks
        00-network.yaml # VPC, subnets, IGW, NATGW, routes
        20-alb.yaml # ALB, Target Group, Listener, SG
        30-efs.yaml # EFS, Mount Targets, SG
        40-rds-mariadb.yaml # DB Subnet Group, SG, MariaDB instance
        50-asg-wordpress.yaml # LT + ASG for LAMP/WordPress, SG, scaling policy
        60-bastion-deploy.yaml # Deploy/Bastion EC2 in public subnet (SSH), SG
    /parameters
    dev.json # example parameter set for root.yaml
```
  
#### Deploy flow:
  
1. Upload the six child templates (the numbered ones) to an S3 bucket.
2. Update root.yaml TemplateURLs or use local packaging (aws cloudformation package), then create the single root stack.
3. Deleting the root tears everything down.

#### Always validate template yaml-updates:
  
```bash
aws cloudformation validate-template --template-body file://infra/templates/root.yaml
```
  
#### Upload the templates to S3
  
```bash
aws s3 cp infra/templates/00-network.yaml     s3://$TemplateBucket/${TemplatePrefix}00-network.yaml
aws s3 cp infra/templates/20-alb.yaml        s3://$TemplateBucket/${TemplatePrefix}20-alb.yaml
aws s3 cp infra/templates/30-efs.yaml        s3://$TemplateBucket/${TemplatePrefix}30-efs.yaml
aws s3 cp infra/templates/40-db-sg.yaml      s3://$TemplateBucket/${TemplatePrefix}40-db-sg.yaml
aws s3 cp infra/templates/45-db-mariadb-ec2.yaml s3://$TemplateBucket/${TemplatePrefix}45-db-mariadb-ec2.yaml
aws s3 cp infra/templates/50-asg-wordpress.yaml  s3://$TemplateBucket/${TemplatePrefix}50-asg-wordpress.yaml
aws s3 cp infra/templates/root.yaml          s3://$TemplateBucket/${TemplatePrefix}root.yaml
  ```

#### Create the stack (daily bring-up)
  
```bash
aws cloudformation create-stack \
  --region eu-west-1 \
  --stack-name wp-cf-root \
  --template-url https://s3.eu-west-1.amazonaws.com/wordpress-iac-<account-id>-eu-west-1/wordpress-iac/templates/root.yaml \
  --parameters file://infra/parameters/dev.params.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
  ```

#### Update the stack (apply new template changes)
  
```bash
aws cloudformation update-stack \
  --region eu-west-1 \
  --stack-name wp-cf-root \
  --template-url https://s3.eu-west-1.amazonaws.com/wordpress-iac-<account-id>-eu-west-1/wordpress-iac/templates/root.yaml \
  --parameters file://infra/parameters/dev.params.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
  ```

#### Cleaning up the stacks (daily teardown)

```bash
aws cloudformation delete-stack \
  --region eu-west-1 \
  --stack-name wp-cf-root

aws cloudformation wait stack-delete-complete \
  --region eu-west-1 \
  --stack-name wp-cf-root
```

---
  
# IMPORTANT
  
Below is just a bash-template I use to copy-paste code into Word to keep the formatting. Ignore!
```bash
aws cloudformation validate-template --template-body file://infra/templates/20-alb.yaml
```
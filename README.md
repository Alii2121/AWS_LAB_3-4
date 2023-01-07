# AWS_LAB_3-4

## Create a VPC with an App. LoadBalancer connected to 2 Public EC2 in a diff AZs with Nginx Configured as a proxy to pass traffic to an internal facing Network LB connected to 2 Private EC2s with a web app on them.

### 4 EC2s

 ![Screenshot from 2023-01-07 12-12-13](https://user-images.githubusercontent.com/103090890/211149472-d5745f12-1cd9-4dc4-9868-5581fe213ad6.png)

### ALB and NLB

 ![Screenshot from 2023-01-07 12-12-42](https://user-images.githubusercontent.com/103090890/211149494-a45fd1c3-a3ef-4d47-9c4d-75432a1a4068.png)

![Screenshot from 2023-01-07 12-12-55](https://user-images.githubusercontent.com/103090890/211149520-e22bb1c3-a010-4949-aa6d-95e96c512700.png)


![Screenshot from 2023-01-07 12-13-12](https://user-images.githubusercontent.com/103090890/211149530-2f5a8d9a-396c-4bc1-b8c5-4673be166ba6.png)

### Health Status of Target Groups

![Screenshot from 2023-01-07 12-13-24](https://user-images.githubusercontent.com/103090890/211149547-e5f8655b-7119-4981-b417-c077a6ef8d28.png)
![Screenshot from 2023-01-07 12-13-35](https://user-images.githubusercontent.com/103090890/211149550-4c6ebe8e-365d-4e91-8e8a-7230ee65b8a9.png)

### Subnets

![Screenshot from 2023-01-07 12-15-11](https://user-images.githubusercontent.com/103090890/211149570-a3269eb6-9d3c-4933-ba5e-0bffe7fae724.png)
![Screenshot from 2023-01-07 12-15-19](https://user-images.githubusercontent.com/103090890/211149575-de43c539-a8bc-4589-b745-eb0d133c0ad9.png)

![Screenshot from 2023-01-07 12-15-25](https://user-images.githubusercontent.com/103090890/211149583-211c56de-52c1-4463-95c9-8aa9bce63532.png)

### NAT gateway to connect the private subnets to the internet
![Screenshot from 2023-01-07 12-15-36](https://user-images.githubusercontent.com/103090890/211149587-3ab149ad-59c2-4083-9de0-cab7c305ce51.png)

### EC2s acting as reverse proxies

![Screenshot from 2023-01-05 15-30-56](https://user-images.githubusercontent.com/103090890/211149998-b029c9d8-0937-45c4-92f6-d373a43fb920.png)
![Screenshot from 2023-01-05 15-42-47](https://user-images.githubusercontent.com/103090890/211150000-2bed87bc-609a-4839-95d7-4e1ee12c6fac.png)


### User data of Private EC2s


![Screenshot from 2023-01-07 12-49-33](https://user-images.githubusercontent.com/103090890/211149920-da07a1b3-b230-4e9f-8df2-202382aef05a.png)



### Output 

![Screenshot from 2023-01-07 12-11-35](https://user-images.githubusercontent.com/103090890/211150006-cdeafdfa-dd9e-4bd2-ba8f-101e1adb42d0.png)

# LAB 4

## Use the Previous Infrastructure and add Auto Scaling Group to the Private subnets that hosts the app

### ASG Policy


![Screenshot from 2023-01-07 12-50-59](https://user-images.githubusercontent.com/103090890/211150066-34a3b28c-985f-4ea5-860d-8f99fb8ca53e.png)


![Screenshot from 2023-01-07 12-51-53](https://user-images.githubusercontent.com/103090890/211150080-43dd4904-623b-4100-ae60-83803a8e59fb.png)


![Screenshot from 2023-01-07 12-53-46](https://user-images.githubusercontent.com/103090890/211150083-53eaa730-d67c-4fac-a0bb-a94e11036980.png)
### Health Status 

![Screenshot from 2023-01-07 12-59-46](https://user-images.githubusercontent.com/103090890/211150087-9dcf8285-5c34-4004-9946-b223e3bb6398.jpg)



## (Bonus) terraform file to create a vpc and subnets
```HashiCorp
# Create VPC
resource "aws_vpc" "example" {
  cidr_block       = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags = {
    Name = "example-vpc"
  }
}

# Create public subnets
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.example.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = "us-west-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "example-public-${count.index+1}"
  }
}

# Create private subnets
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.example.id
  cidr_block        = "10.0.${count.index+2}.0/24"
  availability_zone = "us-west-2a"
  map_public_ip_on_launch = false

  tags = {
    Name = "example-private-${count.index+1}"
  }
}

# Create security group for public subnets
resource "aws_security_group" "public" {
  name        = "public"
  description = "Allow HTTP and HTTPS traffic"
  vpc_id      = aws_vpc.example.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create security group for private subnets
resource "aws_security_group" "private" {
  name        = "private"
  description = "Allow all traffic"
  vpc_id      = aws_vpc.example.id

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create Application Load Balancer
resource "aws_alb" "example" {
  name            = "example-alb"
  internal        = false
  security_groups = [aws_security_group.alb.id]
  subnets         = [aws_subnet.public.id]

  tags = {
    Name = "example-alb"
  }
}

# Create target group
resource "aws_alb_target_group" "example" {
  name     = "example-target-group"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.example.id
}
resource "aws_alb_target_group_attachment" "example" {
  target_group_arn = aws_alb_target_group.example.arn
  target_id        = aws_instance.example.id
  port             = 80
}

resource "aws_alb_listener" "example" {
  load_balancer_arn = aws_alb.example.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_alb_target_group.example.arn
  }
}
```


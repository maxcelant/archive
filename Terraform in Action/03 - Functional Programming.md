- Variables can be accessed within a given module by using the expression `var.<VAR_NAME>`
- Terraform doesn't let you create your own functions, but it has some that come out of the box.
- You can template out using `${varname}` in a file.
- The `locals` block is used to define variables that can be _reusable_ with your tf config.

```c
locals {
  project_name = "my-awesome-project"
  environment  = "production"
  
  name_prefix  = "${local.project_name}-${local.environment}"
  
  common_tags = {
    Project     = local.project_name
    Environment = local.environment
    ManagedBy   = "Terraform"
  }
}

resource "aws_s3_bucket" "example" {
  bucket = "${local.name_prefix}-data-bucket"
  tags   = local.common_tags
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-instance"
  })
}
```

- In terraform, a **module** is just a directory that hosts `.tf` files. The files in this directory as managed as a group. 
- **Output values** can be passed back to the user or other modules.
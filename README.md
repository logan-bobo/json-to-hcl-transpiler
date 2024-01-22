# json to hcl transpiler
This tool is for altering inline JSON in IAM policies to their HCL native 
resource across an entire terraform project. 

The motivation behind this project was to enable the ability to refactor large 
terraform codebases. Inline JSON overall is not a great idea for defining IAM 
policies for the following reasons:

- No LSP integrations meaning no auto complete when defining the policy
- Policy validation is at runtime (terraform apply) for the inline definition
- Not native HCL

## Example

Given the following policy that is declared as inline JSON... 
```hcl
resource "aws_iam_policy" "my_policy" {
  name        = "foo"
  path        = "/"
  description = "bar"

  policy = <<EOF
{
  "Version" : "2012-10-17",
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : "*",
      "Resource" : "*"
    }
  ]
}
EOF
}
```

We want to move this out to a terraform native (aws provider) resource block to
negate the negative effects mentioned above and improve the developer 
experience...

```hcl
resource "aws_iam_policy" "my_policy" {
  name   = "foo"
  policy = data.aws_iam_policy_document.my_policy.json
}

data "aws_iam_policy_document" "my_policy" {
  statement {
    effect = "Allow"
    actions = [
      "*"
    ]
    resources = [
      "*"
    ]
  }
}
```


---
subcategory: "CodePipeline"
layout: "aws"
page_title: "AWS: aws_codepipeline"
description: |-
  Provides a CodePipeline
---


<!-- Please do not edit this file, it is generated. -->
# Resource: aws_codepipeline

Provides a CodePipeline.

## Example Usage

```python
# DO NOT EDIT. Code generated by 'cdktf convert' - Please report bugs at https://cdk.tf/bug
from constructs import Construct
from cdktf import Token, TerraformStack
#
# Provider bindings are generated by running `cdktf get`.
# See https://cdk.tf/provider-generation for more details.
#
from imports.aws.codepipeline import Codepipeline
from imports.aws.codestarconnections_connection import CodestarconnectionsConnection
from imports.aws.data_aws_iam_policy_document import DataAwsIamPolicyDocument
from imports.aws.data_aws_kms_alias import DataAwsKmsAlias
from imports.aws.iam_role import IamRole
from imports.aws.iam_role_policy import IamRolePolicy
from imports.aws.s3_bucket import S3Bucket
from imports.aws.s3_bucket_public_access_block import S3BucketPublicAccessBlock
class MyConvertedCode(TerraformStack):
    def __init__(self, scope, name):
        super().__init__(scope, name)
        example = CodestarconnectionsConnection(self, "example",
            name="example-connection",
            provider_type="GitHub"
        )
        codepipeline_bucket = S3Bucket(self, "codepipeline_bucket",
            bucket="test-bucket"
        )
        S3BucketPublicAccessBlock(self, "codepipeline_bucket_pab",
            block_public_acls=True,
            block_public_policy=True,
            bucket=codepipeline_bucket.id,
            ignore_public_acls=True,
            restrict_public_buckets=True
        )
        assume_role = DataAwsIamPolicyDocument(self, "assume_role",
            statement=[DataAwsIamPolicyDocumentStatement(
                actions=["sts:AssumeRole"],
                effect="Allow",
                principals=[DataAwsIamPolicyDocumentStatementPrincipals(
                    identifiers=["codepipeline.amazonaws.com"],
                    type="Service"
                )
                ]
            )
            ]
        )
        codepipeline_policy = DataAwsIamPolicyDocument(self, "codepipeline_policy",
            statement=[DataAwsIamPolicyDocumentStatement(
                actions=["s3:GetObject", "s3:GetObjectVersion", "s3:GetBucketVersioning", "s3:PutObjectAcl", "s3:PutObject"
                ],
                effect="Allow",
                resources=[codepipeline_bucket.arn, "${" + codepipeline_bucket.arn + "}/*"
                ]
            ), DataAwsIamPolicyDocumentStatement(
                actions=["codestar-connections:UseConnection"],
                effect="Allow",
                resources=[example.arn]
            ), DataAwsIamPolicyDocumentStatement(
                actions=["codebuild:BatchGetBuilds", "codebuild:StartBuild"],
                effect="Allow",
                resources=["*"]
            )
            ]
        )
        s3_kmskey = DataAwsKmsAlias(self, "s3kmskey",
            name="alias/myKmsKey"
        )
        codepipeline_role = IamRole(self, "codepipeline_role",
            assume_role_policy=Token.as_string(assume_role.json),
            name="test-role"
        )
        aws_iam_role_policy_codepipeline_policy = IamRolePolicy(self, "codepipeline_policy_7",
            name="codepipeline_policy",
            policy=Token.as_string(codepipeline_policy.json),
            role=codepipeline_role.id
        )
        # This allows the Terraform resource name to match the original name. You can remove the call if you don't need them to match.
        aws_iam_role_policy_codepipeline_policy.override_logical_id("codepipeline_policy")
        Codepipeline(self, "codepipeline",
            artifact_store=[CodepipelineArtifactStore(
                encryption_key=CodepipelineArtifactStoreEncryptionKey(
                    id=Token.as_string(s3_kmskey.arn),
                    type="KMS"
                ),
                location=codepipeline_bucket.bucket,
                type="S3"
            )
            ],
            name="tf-test-pipeline",
            role_arn=codepipeline_role.arn,
            stage=[CodepipelineStage(
                action=[CodepipelineStageAction(
                    category="Source",
                    configuration={
                        "BranchName": "main",
                        "ConnectionArn": example.arn,
                        "FullRepositoryId": "my-organization/example"
                    },
                    name="Source",
                    output_artifacts=["source_output"],
                    owner="AWS",
                    provider="CodeStarSourceConnection",
                    version="1"
                )
                ],
                name="Source"
            ), CodepipelineStage(
                action=[CodepipelineStageAction(
                    category="Build",
                    configuration={
                        "ProjectName": "test"
                    },
                    input_artifacts=["source_output"],
                    name="Build",
                    output_artifacts=["build_output"],
                    owner="AWS",
                    provider="CodeBuild",
                    version="1"
                )
                ],
                name="Build"
            ), CodepipelineStage(
                action=[CodepipelineStageAction(
                    category="Deploy",
                    configuration={
                        "ActionMode": "REPLACE_ON_FAILURE",
                        "Capabilities": "CAPABILITY_AUTO_EXPAND,CAPABILITY_IAM",
                        "OutputFileName": "CreateStackOutput.json",
                        "StackName": "MyStack",
                        "TemplatePath": "build_output::sam-templated.yaml"
                    },
                    input_artifacts=["build_output"],
                    name="Deploy",
                    owner="AWS",
                    provider="CloudFormation",
                    version="1"
                )
                ],
                name="Deploy"
            )
            ]
        )
```

## Argument Reference

This resource supports the following arguments:

* `name` - (Required) The name of the pipeline.
* `pipeline_type` - (Optional) Type of the pipeline. Possible values are: `V1` and `V2`. Default value is `V1`.
* `role_arn` - (Required) A service role Amazon Resource Name (ARN) that grants AWS CodePipeline permission to make calls to AWS services on your behalf.
* `artifact_store` (Required) One or more artifact_store blocks. Artifact stores are documented below.
* `stage` (Minimum of at least two `stage` blocks is required) A stage block. Stages are documented below.
* `tags` - (Optional) A map of tags to assign to the resource. If configured with a provider [`default_tags` configuration block](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#default_tags-configuration-block) present, tags with matching keys will overwrite those defined at the provider-level.
* `variable` - (Optional) A pipeline-level variable block. Valid only when `pipeline_type` is `V2`. Variable are documented below.

An `artifact_store` block supports the following arguments:

* `location` - (Required) The location where AWS CodePipeline stores artifacts for a pipeline; currently only `S3` is supported.
* `type` - (Required) The type of the artifact store, such as Amazon S3
* `encryption_key` - (Optional) The encryption key block AWS CodePipeline uses to encrypt the data in the artifact store, such as an AWS Key Management Service (AWS KMS) key. If you don't specify a key, AWS CodePipeline uses the default key for Amazon Simple Storage Service (Amazon S3). An `encryption_key` block is documented below.
* `region` - (Optional) The region where the artifact store is located. Required for a cross-region CodePipeline, do not provide for a single-region CodePipeline.

An `encryption_key` block supports the following arguments:

* `id` - (Required) The KMS key ARN or ID
* `type` - (Required) The type of key; currently only `KMS` is supported

A `stage` block supports the following arguments:

* `name` - (Required) The name of the stage.
* `action` - (Required) The action(s) to include in the stage. Defined as an `action` block below

An `action` block supports the following arguments:

* `category` - (Required) A category defines what kind of action can be taken in the stage, and constrains the provider type for the action. Possible values are `Approval`, `Build`, `Deploy`, `Invoke`, `Source` and `Test`.
* `owner` - (Required) The creator of the action being called. Possible values are `AWS`, `Custom` and `ThirdParty`.
* `name` - (Required) The action declaration's name.
* `provider` - (Required) The provider of the service being called by the action. Valid providers are determined by the action category. Provider names are listed in the [Action Structure Reference](https://docs.aws.amazon.com/codepipeline/latest/userguide/action-reference.html) documentation.
* `version` - (Required) A string that identifies the action type.
* `configuration` - (Optional) A map of the action declaration's configuration. Configurations options for action types and providers can be found in the [Pipeline Structure Reference](http://docs.aws.amazon.com/codepipeline/latest/userguide/reference-pipeline-structure.html#action-requirements) and [Action Structure Reference](https://docs.aws.amazon.com/codepipeline/latest/userguide/action-reference.html) documentation.
* `input_artifacts` - (Optional) A list of artifact names to be worked on.
* `output_artifacts` - (Optional) A list of artifact names to output. Output artifact names must be unique within a pipeline.
* `role_arn` - (Optional) The ARN of the IAM service role that will perform the declared action. This is assumed through the roleArn for the pipeline.
* `run_order` - (Optional) The order in which actions are run.
* `region` - (Optional) The region in which to run the action.
* `namespace` - (Optional) The namespace all output variables will be accessed from.

A `variable` block supports the following arguments:

* `name` - (Required) The name of a pipeline-level variable.
* `default_value` - (Optional) The default value of a pipeline-level variable.
* `description` - (Optional) The description of a pipeline-level variable.

~> **Note:** The input artifact of an action must exactly match the output artifact declared in a preceding action, but the input artifact does not have to be the next action in strict sequence from the action that provided the output artifact. Actions in parallel can declare different output artifacts, which are in turn consumed by different following actions.

## Attribute Reference

This resource exports the following attributes in addition to the arguments above:

* `id` - The codepipeline ID.
* `arn` - The codepipeline ARN.
* `tags_all` - A map of tags assigned to the resource, including those inherited from the provider [`default_tags` configuration block](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#default_tags-configuration-block).

## Import

In Terraform v1.5.0 and later, use an [`import` block](https://developer.hashicorp.com/terraform/language/import) to import CodePipelines using the name. For example:

```python
# DO NOT EDIT. Code generated by 'cdktf convert' - Please report bugs at https://cdk.tf/bug
from constructs import Construct
from cdktf import TerraformStack
#
# Provider bindings are generated by running `cdktf get`.
# See https://cdk.tf/provider-generation for more details.
#
from imports.aws.codepipeline import Codepipeline
class MyConvertedCode(TerraformStack):
    def __init__(self, scope, name):
        super().__init__(scope, name)
        Codepipeline.generate_config_for_import(self, "foo", "example")
```

Using `terraform import`, import CodePipelines using the name. For example:

```console
% terraform import aws_codepipeline.foo example
```

<!-- cache-key: cdktf-0.20.1 input-e1025a278a59d0c266797634d95f819b31425f7facb05b1eeff5d7b24738c176 -->
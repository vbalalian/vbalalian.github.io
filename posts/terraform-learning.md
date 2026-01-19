---
layout: post
---

# What I Actually Learned Adding Terraform to My Data Pipeline

I put off learning Terraform for months. It seemed like one of those tools that would require weeks of study before I could do anything useful. Infrastructure-as-code sounded like a concept that would be hard to fully grasp.

It wasn't. The core of it is embarrassingly simple.

## What I expected vs. what it actually is

I thought Terraform would be some complex new paradigm. It's not. It's a declarative config file that calls the same APIs as the `gcloud` CLI. You describe what you want to exist, and Terraform figures out how to make it happen.

That's it. That's the whole concept.

```hcl
resource "google_storage_bucket" "artifacts" {
  name     = "my-project-pipeline-artifacts"
  location = "us-west1"
}
```

This creates a GCS bucket. If the bucket already exists and matches the config, Terraform does nothing. If it doesn't exist, Terraform creates it. If it exists but differs from the config, Terraform modifies it.

## The thing that made it click

Running `terraform plan` for the first time is when it made sense. You see a diff: here's what exists now, here's what you said you want, here's what Terraform will do to reconcile the two.

It's not executing a script top to bottom. It's comparing desired state to actual state and computing the minimum changes needed. The state file is Terraform's memory of what it created last time.

Once I understood that, everything else was just learning resource syntax.

## The gotcha nobody warned me about

State has to live somewhere.

When you run Terraform locally, state lives in a `terraform.tfstate` file on your machine. Fine for learning. But if you want to run Terraform in CI/CD, or collaborate with others, that file needs to be in shared storage: a GCS bucket, S3, Terraform Cloud, whatever.

Here's the catch: that storage bucket has to exist *before* you can manage anything else with Terraform. You can't use Terraform to create the bucket that holds Terraform's state. Chicken and egg.

For my project, I'm keeping it simple: local state, manual runs. Infrastructure changes are rare, and automating this for a portfolio project would be overkill. But if I were doing this for a team, I'd create the state bucket manually first, then configure a remote backend.

## What I actually built

My [estore-analytics](https://github.com/vbalalian/estore-analytics) project processes 400M+ e-commerce events through a dbt and Dagster pipeline on GCP. The Terraform config provisions:

- GCS bucket for raw event data
- GCS bucket for CI/CD artifacts (dbt manifest for slim CI)
- BigQuery datasets (raw, staging, marts)
- Service account with appropriate IAM roles

Nothing fancy. But I understand every line, and anyone cloning the repo can spin up identical infrastructure in a few commands.

## Why I'm using OpenTofu going forward

HashiCorp changed Terraform's license in 2023 from MPL (open-source friendly) to BSL (restricts commercial use). OpenTofu is a community fork that kept the original open-source license.

Right now they're functionally identical: same syntax, same providers, same workflow. `tofu init` instead of `terraform init`. That's the only difference.

For a portfolio project it doesn't matter much. But I'd rather build habits on the tool without licensing question marks. So going forward, it's OpenTofu.


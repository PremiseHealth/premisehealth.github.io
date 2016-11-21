---
layout:     post
title:      CloudFormation vs Terraform
author:     kwilson
date:       2016-11-21
summary:    Comparing tools for infrastructure management
categories: devops tools
---

# tldr;

We chose Terraform.

# CloudFormation vs Terraform

The Premise BeWell portal was designed from the ground up to take advantage of AWS cloud services. AWS allows us to quickly manage and easily scale our infrastructure to meet client demands. However, in the beginning, there was not a lot of AWS competency on the team. So we did what any team does in the early phases of development. [The simplest thing that could possibly work](http://wiki.c2.com/?DoTheSimplestThingThatCouldPossiblyWork). This means we did not use any external, automated tools to manage our cloud resources.

But headaches abound manually managing cloud infrastructure. It is very difficult to keep track of everything. So, when the business came to us with a requirement to move to the east region, we identified this as an opportunity to implement an infrastructure management tool. To do it 'right this time'.

# CloudFormation

[CloudFormation](https://aws.amazon.com/cloudformation/) is the AWS product that facilitates the automatic management of AWS resources. You define your infrastructure in JSON templates. Templates can be parameterized and allow for outputs as well. Programmers can think of templates as functions. Once you have a template you call the AWS API with that template and any parameters to create a stack. AWS will take your stack and create any resources defined in that stack. You can even include stacks in your templates to nest stacks.

JSON can be OK to read, but it is not a very nice language to have to write infrastructure code in, so we pulled in [cfndsl, a Ruby DSL for CloudFormation](https://github.com/stevenjack/cfndsl) to build out the templates. In addition, using the console is less than ideal for managing your stacks. So we used [cumulus](https://github.com/cotdsa/cumulus) to manage our stacks from the command line.

Our project setup worked pretty well. It allowed for a lot of flexibility to use code to minimize duplication in our templates. In addition, it was easy to manage stacks and get a preview of what changes needed to be made. It also worked well with continuous integration and allowed for testing and code quality tools.

However, the toolchain caused confusion for a lot of the team members. It was difficult to discern where things should be defined. The flexibility provided to us using Ruby to generate templates was nice, but it was easy to become overly abstract with the code. It seems that infrastructure code benefits from a declarative syntax over an imperative paradigm.

The CloudFormation workflow left a little to be desired as well. New AWS parameters and services are spinning up constantly. CloudFormation seems to lag behind the latest API specifications quite a bit. So not only do you wait for new services, you then have to wait for the CloudFormation resources to catch up.

# Terraform

[Terraform](https://www.terraform.io/) is an open source project maintained by [Hashicorp](https://www.hashicorp.com/) for automating infrastructure management. With Terraform you use [Hashicorp Configuration Language (HCL)](https://github.com/hashicorp/hcl) to declare your infrastructure. You can then use the terraform cli command to apply your changes. Terraform provides modules so you can get some limited code reuse. Terraform needs to track the state of your infrastructure so it knows what to change. To do this, it maintains a state file. These are kept in a dotfile directory locally, but can be hosted remotely.

For our implementation, we chose to store our state remotely on S3. This allows the Platform Engineering team and CI agents to work at the same time with minimal conflicts. We utilized terraform modules such that spinning up an environment is as simple as using a module with different parameters.

There are some difficulties dealing with the static, declarative nature of Terraform files. Particularly with developers that are used to the imperative style of programming. There will likely be more copy-paste since you don't have the flexibility to easily add abstractions. However, I have no complaints about the workflow around our solution. Once you get your project setup with remote state and secrets, it's pretty simple.

# Comparing the Tools

It's a little difficult to do an apples to apples comparison between CloudFormation and Terraform since one is a product and one is a service. But we can look at our experience with both to see which is better for our use case.

Note: This analysis occurred before changesets had been added to CloudFormation.

CloudFormation requires some additional tooling to be added to make it feasible. At a minimum, you need a template generator. For us, that was cfndsl. Nobody wants to define their infrastructure in JSON. There seems to be a lot of options here, but none that had really captured the mindshare enough to be fully mature. We were happy with cfndsl, but we did have to contribute back when new features were added to CloudFormation.

Obviously, CloudFormation also locks you into AWS. We preferred a solution that would allow us to provision things outside of AWS. As an example, it would be nice to be able to define our [Artifactory](https://www.jfrog.com/artifactory/) repositories in Terraform. In fact, we are making use of a database provider to define some of our database objects.

In addition, Terraform providers are updated more quickly than CloudFormation resources when new features become available. It is an open source project and a lot of people contribute to making it great. That being said, we ran across some bugs and weirdness in some of the providers we used. For these, we file an issue and try and figure out a workaround. It's safe to say that there is an equal amount of frustration dealing with the lifecycles associated with either technology. This is probably more of an issue with the Infrastructure as Code problem space than the products themselves.

As a developer, I like to have quick feedback from my tools. Terraform wins this hands down. It is very fast and when something fails, it does not take long to make a modification and try again. Debugging issues can be equally frustrating with either one.

In the end, Terraform was a better option for us. It's simple project layout, ease of development, and the frequency at which new features arrived all were very important to us. As well as the ability to provision non-AWS resources.

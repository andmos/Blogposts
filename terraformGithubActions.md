Github Actions and Terraform, revisited
===

This week we finally got to do some Terraform work again on a new project.

The setup is simple, we have some Azure resources we want to create, so there is no way around Infrastructure as Code and, by our choice, the battle-proven Terraform from Hashicorp.

Since the code lives up on Github, we want to automate the steps involved with the Terraform code and work with a standard pull request workflow.
This includes:

* Lint and format the code.
* Run validation.
* Create the plan to be executed.
* Post the changes as a pull request comment and review the changes.
* If all is good and approved, apply the changes on merge.

or visualized:

![](https://content.hashicorp.com/api/assets?product=tutorials&version=main&asset=public%2Fimg%2Fterraform%2Fautomation%2Fpr-master-gh-actions-workflow.png)

Thankfully this is not a new subject to explore. There are tons of examples on how to work with Terraform and Github Actions, including [an excellent post from Hashicorp](https://developer.hashicorp.com/terraform/tutorials/automation/github-actions) on how to write this workflow.

Upon reading this post and writing out the code, we _nearly_ got what we wanted.

The generated pull request post with the plan itself looks like this with the standard example:

![](https://user-images.githubusercontent.com/1283556/210856729-bb4f080d-8a96-4e88-aeba-dafefb82506d.png)

A couple of things we didn't fancy here. First off, the plan itself is hidden away behind the drop-down menu, making it easy to miss if you're not careful - and you should be - Terraform is unforgiving if some resources are deleted by an error.

Second, the plan has no color highlighting, making it a bit hard to read. It appears just as a wall of text where details might be missed.

And third, the post itself should contain _the brief summary of what is actually going to happen_ without reading the actual plan. I'm talking about this part:

```shell
Plan: 0 to add, 1 to change, 1 to destroy.
```

After some more searching we found [another excelent post by Andrew Walker](https://blog.testdouble.com/posts/2021-12-07-elevate-your-terraform-workflow-with-github-actions/) who introduces enhanced formatting on the `terraform plan` output with the use of `diff`.
`diff` is a nice syntax to color highlight changes in files, just as Github's pull request view has. In a nutshell, `diff` has the following coding:

![](https://cdn-blog.testdouble.com/img/elevate-your-terraform-workflow-with-github-actions/diff-colors.98276d25d0b6c7af1821d6304e6f79d296a61f79b59860d63f98eb3a88f6d3d2.png)

Great to highlight what will be _created_, _deleted_ or _changed_ when a `terraform plan` is executed.

After some inspiration from our friend Andrew and some more tinkering, we came out with an output we were quite happy with:

![](https://user-images.githubusercontent.com/1283556/210859359-591133fa-c13c-4794-82ba-50bf01aed519.png)

The overview itself is much cleaner with less stuff. The checks are still there, but with emojis instead of the text. The summary is also on the top of the comment.

The plan itself shows like this:

![](https://user-images.githubusercontent.com/1283556/210953172-b2de1468-b7c1-4b51-961c-ca3814c5dbd2.png)

The observant Terraform author will notice something here. The symbol for "update in place" should be `~`, not `!`. This is a little trick we did to get the orange color for in-place changes, not just additions and destroyed resources.

So far we are quite happy with this layout.

The following gist contains the workflow. Hopefully it will be of inspiration to others down the road.

<script src="https://gist.github.com/andmos/e349f693d04a27e8630dc358b4a36f24.js"></script>

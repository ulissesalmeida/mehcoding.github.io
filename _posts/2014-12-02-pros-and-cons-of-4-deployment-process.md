---
layout: post
title:  The pros and cons of 4 deployment process techniques
date:   2014-12-02
author: Ulisses Almeida
categories: plataformatec-blog
lang: en
excerpt: The way of delivering your product code to your customer is commonly called “deployment”. It is an important matter because it will impact in how fast your product will respond to changes and the quality of each change.
---

__This is a repost from [Plataformatec blog](http://blog.plataformatec.com.br/2014/12/the-pros-and-cons-of-4-deployment-process-techniques/).__

The way of delivering your product code to your customer is commonly called “deployment”. It is an important matter because it will impact in how fast your product will respond to changes and the quality of each change.

Depending on which deployment decision you take, it will impact your team and how you use your version control system.

As a consultancy, we have worked on lots of projects, and together with our customers, we have devised many ways to deliver their product to their customers. We have seen some patterns, advantages, and challenges on each way, and today I would like to discuss some of them:

* The open-source way
* The pipeline way
* The support branch way
* The feature toggle way

## The open-source way

In the open-source world most of the times we should maintain many versions of the same product. For example, Ruby on Rails has many versions released, like 2, 3.2, 4.0, 4.1. Bugs happen, new features are created, so news releases must be delivered, but in a set of supported released versions. Still, on RoR example, the supported releases are 4.1, 4.0 and 3.2 (http://guides.rubyonrails.org/maintenance_policy.html). But how this releasing works?

The most recent version of the product is maintained on the master branch, the previous major releases have their own branches. On RoR we have the master for the new 4.2 version release, and we still have 4-1-stable, 4-0-stable, 3-2-stable branches. By following this organization we can easily apply changes to the desired versions.

For each release, a tag must be created. For example, there’s a tag for RoR 4.0.0, one for 4.1.0 and so on. With tags, it is possible to navigate between the released versions and if the worst happens, like losing the “version-stable” branch, it’s easy to create another one from the last released version.

Usually, a web product has just one version to be maintained, so usually, we don’t need the “version-stable” branches. We can keep the new product releases on the master branch and generate a tag when we want to package and release a new product version.

When we need a “hotfix” or an urgent feature and the master is not ready yet for production, we can easily create a branch from the latest tagged version, apply the desired changes, create a new tag and release a new version. By the way, using this way you can release any branch you want. All that manipulation of applying commits, merging and creating branches and tags, can be simplified with a powerful version control system like “Git”.

### Strong points

* The flexible package creation and release.
* It works for large teams, primarily when there are planned releases.

### The challenges

* It requires the infra to be flexible enough to support it.
* It requires time to control what can be merged on master before the package creation.
* It will require good ability with the version control system.
* Manage the release versions.

### Common phrases with this approach

“Sorry pals, I forgot to apply that hotfix patch on master”. – A developer after releasing a new product version.

## The pipeline way

Using a pipeline in your deployment process means you have well-defined steps and all steps must be accomplished in order to do a deployment.

Usually, the steps are: run the automated tests, release on test/QA environment, create the release tag, release on production. After the steps are defined, you need some software that allows the team to automate some steps and to add the option of requiring a approval for the next step. For example, you only want to release the package to production when your QA team and PO have approved the version on QA.

Having a pipeline means your master branch is always production ready. Any new code inserted on master branch must pass the pipeline, then it is very important that the team and the pipeline be able to quickly respond to changes.

One important precaution is to be sure that only wanted features are on master because all code on the master will always be deployed on the next software release. I have seen some confusion in this aspect because some companies are a bit more bureaucratic and have some strict deployment rules.

Per example, a feature can only go into production when the QA team and PO approves. Placing the QA process on pipeline means you’ll put the feature on the master that is not ready yet for production. It generates a problem I see regularly with this approach, I’ll call it, for now, the “release lock”.

### Release lock

The release lock can be better understood with an example:

1. The developers have released on master the Feature A and B.
2. The QA team finds a BUG on Feature A.
3. The developers release a Feature C on the master
4. The developers fix the BUG.
5. PO approves Feature A and B and wants a deploy.

Can we deploy a release with Feature C untested and unapproved by PO? Most of the time the companies answer is no.

Some approaches we can do here are: revert Feature C commits, or simply lock code changes on master and the entire team focus on finishing the release with Feature C.

Of course, there are other approaches we can incorporate in the pipeline process, and we’ll see a further discussion about it later in this post.

### Strong points

* With the pipeline, it is easier for everyone in the team to understand how the deployment works.
* The pipeline gives accessibility for anyone in the team to launch a release.
* Less time managing versioning.

### The challenges

* You lose the flexibility to deploy any branches.
* In large teams and some companies, rules can produce release locks often.

### Common phrases with this approach

“What? This feature is already in production?” – A member of QA team looking at the version in production.
“Hey, stop merging on master! We need a release today!” – The product manager after receiving pressure from stakeholders.

## The support branch way

You define a branch as QA or test branch. This branch is used to test features which aren’t ready for production. For example, if your deployment process needs QA/PO features approval.

With this approach, you’ll send the features to support branch. When the feature is approved you send them to the master branch and use the normal deployment process flow. It is important to do a regression test of the merged features on the master. When a regression test finds a defect, it is easy to apply them to the master, since the master is clear of unwanted features.

While using this approach you should be aware that now you have two points of integration. Resolving the merging conflicts twice is a problem that can happen often, but the most troublesome issue is when the integration of the support branch breaks the application functionality.

When the support branch integration is broken you need to analyze when and where the patch with the fix will be applied.

If you apply the fix to support branch, you must remember to apply it to master again.

The other option is to find out which changes made the features incompatible together. When you find that, you can apply those changes to the branch that doesn’t have those changes. Be aware that depending on the way you do this, you may end up needing to release both features together.

### Strong points

* You can easily apply the support branch in any deployment process you choose.
* You mitigate the release lock problem.

### The challenges
* Two points of features integration.
* Requires good abilities with version control system.
* The support branch maintenance.

### Common phrases with this approach

“Gosh! I need to fix that merge conflict again.” – Developer merging on master the QA approved feature.

## The feature toggle way

Sometimes you are using the feature toggle without knowing you are doing it. For example: when you enable some features only for beta users, or enable some application routes only for some network IPs, or create A/B tests for the users. In general, you are using a feature toggle when your application is restricting in some way the access to some features.

To solve the release lock problem, some teams apply the feature toggle in every feature that needs approval. In this way, the team can send unapproved features to production. When the feature is approved, the feature can be turned on without a new deploy. Be aware that sending turned off features to production also means that unapproved feature code will be sent too.

Creating toggles for your features means more code and tests to control what your software does with and without the toggles. Each feature toggle you add increases the complexity and the maintenance cost of your codebase.

Thus, it is important to remove them after the feature approval before it starts damaging your software. I know what you’re thinking, yes, it is true, most of the times the QA/PO team want to test the toggle removal and you might face the release lock problem again.

### Strong points

* You can use the pipeline with only one point of integration.
* You reduce the release lock problem.

### The Challenges

* You may increase the cost of development because of maintenance and removal of feature toggles.
* The feature toggle management.

### Common phrases with this approach
“What this method does?” – A developer asking for his pal.
“It depends, which toggles are on?” – The answer to the first question.

## Conclusion

Most of the challenges of each deployment process require team engagement and organization. It’s hard to decide which one is better because it fully depends on how your team adapts to the process.

Each person perceives problems in different ways. A problem can be a huge matter for one, for another is just a small itch. But if you still want answers for “What is the best option?”, I would say it is the same answer for “Which challenge your team will endure more?”.

Given that you prefer one deployment process over the others, I still think that no one should be attached totally to one process forever. Your problems can change, your team can change, your company rules can change, your application can change. Therefore, your deployment process should change together to deal with the new challenges. You can change your deployment in many ways, for example mixing ideas from each the of processes we discussed.

I’m curious to know how your team delivers features. If you use one of these options, if you mix them or if you do something different. If you want to share this knowledge, please leave a comment below.

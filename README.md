# GitHub actions CI/CD pipeline for MuleSoft

- [Blog post + video] [Part 1: How to set up a CI/CD pipeline to deploy your MuleSoft apps to CloudHub using GitHub Actions](https://www.prostdev.com/post/how-to-set-up-a-ci-cd-pipeline-to-deploy-your-mulesoft-apps-to-cloudhub-using-github-actions)
- [Blog post + video] [Part 2: CI/CD pipeline with MuleSoft and GitHub Actions - secured/encrypted properties](https://www.prostdev.com/post/part-2-ci-cd-pipeline-with-mulesoft-and-github-actions-secured-encrypted-properties)
- [Blog post + video] [Part 3: CI/CD pipeline with MuleSoft and GitHub Actions - MUnit testing](https://www.prostdev.com/post/part-3-ci-cd-pipeline-with-mulesoft-and-github-actions-munit-testing)
- [Blog post + video] [Part 4: CI/CD pipeline with MuleSoft and GitHub Actions - MUnit minimum coverage percentage](https://www.prostdev.com/post/part-4-ci-cd-pipeline-with-mulesoft-and-github-actions-munit-minimum-coverage-percentage)
- [Blog post + video] [Part 5: CI/CD pipeline with MuleSoft and GitHub Actions - Enabling MFA through a Connected App](https://www.prostdev.com/post/part-5-ci-cd-pipeline-with-mulesoft-and-github-actions-enabling-mfa-through-a-connected-app)

> **Note**
> 
> This is a simple Mule application to test. Please update the `app.name` and `env` properties from the `pom.xml` to your own.

## Branches

Different examples are being demonstrated per branch. Here's the summary of each.

||[`main`](https://github.com/alexandramartinez/github-actions/tree/main)|[`connected-app`](https://github.com/alexandramartinez/github-actions/tree/connected-app)|[`cloudhub2`](https://github.com/alexandramartinez/github-actions/tree/cloudhub2)
|-|-|-|-
|Deployment|CH1|CH1|CH2
|Passing secured properties|✅|✅|✅
|MUnit testing in pipeline|✅|✅|❌
|Running MUnit coverage|✅|✅|❌
|Nexus credentials|✅|✅|❌
|Auth|username/password in `pom.xml`|connected app in `pom.xml`|server in `settings.xml` (using connected app)
|Maven version|`3.8.0`|`3.8.0`|`4.1.1`
|Runtime|`4.4.0` through the `muleVersion` property|`4.4.0` through the `muleVersion` property|`4.4.0` through the `releaseChannel` property (`NONE`)

## Other resources

The initial versions of the pipeline are based on the following repository created by Archana Patel: [arch-jn/github-actions-mule-cicd-demo](https://github.com/arch-jn/github-actions-mule-cicd-demo).

- [Docs] [Deploy Applications to CloudHub Using the Mule Maven Plugin](https://docs.mulesoft.com/mule-runtime/latest/deploy-to-cloudhub)
- [Docs] [Deploy Applications to CloudHub 2.0 Using the Mule Maven Plugin](https://docs.mulesoft.com/mule-runtime/latest/deploy-to-cloudhub-2)
- [Docs] [CloudHub 2.0 Architecture - Regions and DNS Records](https://docs.mulesoft.com/cloudhub-2/ch2-architecture#regions-and-dns-records)

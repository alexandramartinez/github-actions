# GitHub actions CI/CD pipeline for MuleSoft

Check out the video tutorial: https://youtu.be/vTRppGhfbPI

Check out the original repo: https://github.com/arch-jn/github-actions-mule-cicd-demo

Thanks [Archana](https://github.com/arch-jn) for creating these resources!!

> **Note**
> 
> This is a simple Mule application to test. Please update the `app.name` and `env` properties from the `pom.xml` to your own.

## Instructions

For a full tutorial + video, see [How to set up a CI/CD pipeline to deploy your MuleSoft apps to CloudHub using GitHub Actions](https://www.prostdev.com/post/how-to-set-up-a-ci-cd-pipeline-to-deploy-your-mulesoft-apps-to-cloudhub-using-github-actions).

### Deploy a Mule app to CloudHub

Set up the following Actions Secrets in `Settings`:

```
ANYPOINT_PLATFORM_USERNAME
```

```
ANYPOINT_PLATFORM_PASSWORD
```

Set up the following `build.yml` at `.github/workflows`

```yaml
name: Build and Deploy to Sandbox

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
    
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout this repo
      uses: actions/checkout@v3
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.m2/repository
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: |
          ${{ runner.os }}-maven-
    - name: Set up JDK 1.8
      uses: actions/setup-java@v3
      with:
        distribution: 'zulu'
        java-version: 8
    - name: Build with Maven
      run: mvn -B package --file pom.xml
    - name: Stamp artifact file name with commit hash
      run: |
        artifactName1=$(ls target/*.jar | head -1)
        commitHash=$(git rev-parse --short "$GITHUB_SHA")
        artifactName2=$(ls target/*.jar | head -1 | sed "s/.jar/-$commitHash.jar/g")
        mv $artifactName1 $artifactName2
    - name: Upload artifact 
      uses: actions/upload-artifact@v3
      with:
          name: artifacts
          path: target/*.jar
        
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:    
    - name: Checkout this repo
      uses: actions/checkout@v3
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.m2/repository
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: |
          ${{ runner.os }}-maven-
    - uses: actions/download-artifact@v3
      with:
        name: artifacts
    - name: Deploy to Sandbox
      env:
        USERNAME: ${{ secrets.anypoint_platform_username }}
        PASSWORD: ${{ secrets.anypoint_platform_password }}
      run: |
        artifactName=$(ls *.jar | head -1)
        mvn deploy -DmuleDeploy \
         -Dmule.artifact=$artifactName \
         -Danypoint.username="$USERNAME" \
         -Danypoint.password="$PASSWORD"
```

## Twitch streams

Part 1:

https://www.twitch.tv/videos/1531787507

Part 2:

https://www.twitch.tv/videos/1538443350

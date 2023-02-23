# GitHub actions CI/CD pipeline for MuleSoft

Check out the video tutorial: https://youtu.be/vTRppGhfbPI

Check out the original repo: https://github.com/arch-jn/github-actions-mule-cicd-demo

Thanks [Archana](https://github.com/arch-jn) for creating these resources!!

> **Note**
> 
> This is a simple Mule application to test. Please update the `app.name` and `env` properties from the `pom.xml` to your own.

## Instructions

### Deploy a Mule app to CloudHub (base)

> For a full tutorial + video, see [How to set up a CI/CD pipeline to deploy your MuleSoft apps to CloudHub using GitHub Actions](https://www.prostdev.com/post/how-to-set-up-a-ci-cd-pipeline-to-deploy-your-mulesoft-apps-to-cloudhub-using-github-actions).

Set up the following Actions Secrets in `Settings`:

The username you use to log in to Anypoint Platform:

```
ANYPOINT_PLATFORM_USERNAME
```

The password you use to log in to Anypoint Platform:

```
ANYPOINT_PLATFORM_PASSWORD
```

Set up the following `build.yml` at `.github/workflows`:

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

### Deploy a Mule app with secured properties

Set up the following Actions Secrets in `Settings`:

The username you use to log in to Anypoint Platform:

```
ANYPOINT_PLATFORM_USERNAME
```

The password you use to log in to Anypoint Platform:

```
ANYPOINT_PLATFORM_PASSWORD
```

The decryption key you use to encrypt/decrypt the properties:

```
DECRYPTION_KEY
```

Add the following snippet into your application's `pom.xml`, under the `cloudHubDeployment` configuration:

```xml
<properties>
  <secure.key>${decryption.key}</secure.key>
</properties>
```

> **Warning**
> 
> Make sure you replace the name of the key to match your application's property. In this example, our key is called `secure.key`.

It should look like this:

```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-clean-plugin</artifactId>
        <version>3.0.0</version>
      </plugin>
      <plugin>
        <groupId>org.mule.tools.maven</groupId>
        <artifactId>mule-maven-plugin</artifactId>
        <version>${mule.maven.plugin.version}</version>
        <extensions>true</extensions>
        <configuration>
          <cloudHubDeployment>
            <uri>https://anypoint.mulesoft.com</uri>
            <muleVersion>${app.runtime}</muleVersion>
            <username>${anypoint.username}</username>
            <password>${anypoint.password}</password>
            <applicationName>${app.name}</applicationName>
            <environment>${env}</environment>
            <workerType>MICRO</workerType>
            <region>us-east-2</region>
            <workers>1</workers>
            <objectStoreV2>true</objectStoreV2>
            <!-- Start: SECURED PROPERTIES CI/CD -->
            <properties>
              <secure.key>${decryption.key}</secure.key>
            </properties>
            <!-- End: SECURED PROPERTIES CI/CD -->
          </cloudHubDeployment>
          <classifier>mule-application</classifier>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

Set up the following `build.yml` at `.github/workflows`:

> **Note**
> 
> There's no need to modify the name of the property here. Only in the `pom.xml` file.

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
        KEY: ${{ secrets.decryption_key }}
      run: |
        artifactName=$(ls *.jar | head -1)
        mvn deploy -DmuleDeploy \
         -Dmule.artifact=$artifactName \
         -Danypoint.username="$USERNAME" \
         -Danypoint.password="$PASSWORD" \
         -Ddecryption.key="$KEY"

```

## Twitch streams

Part 1:

https://www.twitch.tv/videos/1531787507

Part 2:

https://www.twitch.tv/videos/1538443350

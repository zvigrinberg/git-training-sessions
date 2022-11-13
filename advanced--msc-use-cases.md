# Miscellaneous/advanced Use cases + Commands

 - merge --strategy=octopus ( when there are no conflicts) - merge 2 branches or more into current branch. suitable for cases where all branches working on different portion of repo, especially different directories(in this merge strategy , collisions are not allowed).
```shell
mkdir git-octopus-merge-demo
cd git-octopus-merge-demo
git init
## Creating initial work on main branch.
 echo "initial work" >> initial.txt ; git add initial.txt ; git commit -m "initial commit"
 
##Creating 3 features branches, each one with its own directory and file.
for i in {1..3}; do git checkout main ; git checkout -b feature$i; mkdir dir$i; cd dir$i; echo  "file number $i" >> file$i.out; git add file$i.out; git commit -m "feature$i file" ; cd ..; done

##Merging feature1,feature2,feature3 into main
git checkout main
git merge --strategy=octopus  feature1 feature2 feature3

```
 - git fetch from other remote repo(not origin) + checkout from it into worktree/index.
```shell
  git clone git@github.com:RHEcosystemAppEng-Temenos/infinity-helm-chart.git
  cd infinity-helm-chart
  git fetch git@github.com:RHEcosystemAppEng-Temenos/temenos-ocp-deploys-automation.git   main
  git checkout FETCH_HEAD infinity-ms/injections
  ## removing checked out directory from index and move it to be untracked, that way we can work in the directory with the new fetched directory but after we disconnected it from the git repo by tell it not to track it.
  git reset --mixed.
  ## update back FETCH_HEAD to be origin instead of foreign remote repo.
  git fetch.
 ``` 
 - Checkout only part of repo
```shell 
    git clone --sparse --filter=blob:none --depth=1 --branch=master https://github.com/eugenp/tutorials.git
    git sparse-checkout set performance-tests/
    git sparse-checkout add javaxval/
```    
  - Use git reset --soft to "cancel" commits but save your current work on index and worktree 
    , __*Example:*__
  ```shell
  ##for example, you have index and worktree with new files that you want to retain , and you want to remove last commit.  
  ## build repo and initial commit
   mkdir reset-soft-demo
   cd reset-soft-demo/
   git init
   echo "bla bla" > initialwork.out
   git add initialwork.out
   git commit -m "initial commit"
   git status
   git commit --help
   git log
   echo "bla bla 2" >> initialwork.out
   git commit -am "second commit"
   echo "bla bla new" >> newwork.out
   echo "bla bla infra" >> infrawork.out
   ll
   git status
   ## New files that we want to keep work on it after discarding latest commit.
   git add infrawork.out
   git add newwork.out --intent-to-add
   git status
   git log
   ## eliminate last commit(with commit message "second commit") by rewinding HEAD 1 
   ## commit back, but keep worktree and index of new files.
   git reset --soft HEAD@{1}
   git log
   git status
   git diff --cached
   
   ## Please pay attention that we repositioned HEAD to be on previous commit, this reconstruct the state before commiting commit "second commit", and 
   ## bring initialwork.out from this commit into index, so in order to discard it from index and then from worktree, just type.
   git restore --staged --worktree initialwork.out
  ```
  - use reset --hard in order to make mass updates of files with commits each iteration,  and resetting to checkpoints if there are mistakes and replay      show use-case with sed utility, and use commit --amend to add subsequent changes to commit pointed by HEAD.
    Example of add a new defined template to all of the  resources of a big helm chart.
    ```shell
      git clone git@github.com:RHEcosystemAppEng-Temenos/temenos-ocp-deploys-automation.git
      cd temenos-ocp-deploys-automation
      git checkout mainDemo -b annotateWithNewTemplate
      cd infinity-ms
      
      ## Create new defined template named "myNewDefinedTemplateWithAnnotations" inside 
      ## library chart
      vi ../infinity-common-lib/templates/_helpers.tpl
    
      git status
      
      ## Add using sed utility new defined template to under annotations of all needed 
      ## resources.
      find . | grep .yaml | grep -v dbinit | grep -v -E 'configmap|-config|crds|values' | xargs -i sed -i -E '/^[[:space:]]{2,4}annotations:/ a \ \ \ \ {{-  include "infinity-common-lib-myNewDefinedTemplateWithAnnotations" . | nindent 4 }}' {}
      ## See how many files were changed.
      git status
      git diff
    
      ##import new defined templated from library chart to application chart, and run helm template
      export HELM_EXPERIMENTAL_OCI=1
      ## Don't forget to use git blame Chart.yaml + git log commit-id of infinity-common-lib repo change + git checkout commit-id@{1} -p Chart.yaml to demonstrate how to get from history a needed chunk from a file.
      helm dependency update
      helm template infinity-ms .
      ##We got errors because we had a typo in the name of the defined templated inclusion ## in resources, we need to specify "infinity-common-lib.myNewDefinedTemplateWithAnnotations" instead of "infinity-common-lib-myNewDefinedTemplateWithAnnotations", update a dozens of files to do this  change? way no! , this is time to use git reset --hard to discard all this incorrect modifications

      git reset --hard

      ##Now we got clean worktree and index, and we'll re-run sed command, this time with correct defined template name 

      find . | grep .yaml | grep -v dbinit | grep -v -E 'configmap|-config|crds|values' | xargs -i sed -i -E '/^[[:space:]]{2,4}annotations:/ a \ \ \ \ {{- include "infinity-common-lib.myNewDefinedTemplateWithAnnotations" . | nindent 4 }}' {}
    
      ## See that rendering of templates are now passed succesfully and verify on vi editor that the new annotationKey: annotationValue added to all resources
      helm template infinity-ms . | vi -
      ## Add all changes to index
      git add -u . 
      git commit -m "annotate all resources with a group of annotations - myNewDefinedTemplateWithAnnotations"
      
      ## Show new added commit with all of its files
      git show HEAD --name-only
      
      ## we forgot to annotate configmaps resources with this new defined template, so let's do it:
         
      find . | grep configmap | xargs -i sed -i -E  '/^[[:space:]]{2,4}annotations:/ a \ \ \ \ {{- include "infinity-common-lib.myNewDefinedTemplateWithAnnotations" . | nindent 4 }}' {}
    
      git commit -am "annotate configmaps resources with new defined template - myNewDefinedTemplateWithAnnotations"
      git log
      ## or amend the current commit to have modified configmaps also(together with other ## resources)
      git commit --amend
      git log
      git show HEAD --name-only
     
    ```
 - Use commit --amend to delete files from last commit. 
 
 - git add -p for staging selected hunks from file with edit hunk(e options in interactive menu)
 - then use git diff(implicit HEAD) --cached to see that only chunks selected are going to be added on next commit, this diff gives the difference  
   between ref and index.
```shell
   vi functions.java
   ## Add some dummy functions definition in java
   git add functions.java
   git commit -m "add functions.java"
   git status
   vi functions.java
   git add -p functions.java
   git status
   vi functions.java
   git restore --staged functions.java
   git status
   vi functions.java
   git add -p functions.java
   git status
   git diff --cached
   git diff
   git commit -m "only mature function commited"
   git status
   git diff
   ll
   git stash save not mature function saving
   git stash list
   ll
   vi file1.out
   vi functions.java
   git stash list
   git stash pop

```
 - git diff (implicit HEAD) - only changes relative to index.
 
 - git diff two refs, git diff - compare between two refs
 
 - git diff REF path/to/file, compare file in worktree or index relative to same file in REF
 
 - git checkout -p, checkout chunks from file(optional)
 - delete files temporarily, make operations without the files, and restore them from HEAD.
   An example git rm of external values file that shouldn't be part of the packed chart for packing helm chart and then reseting the working tree to 
   HEAD after finishing packing helm charts, Example of deleting external encrypted values.yaml from chart in order to pack chart, and after packing, 
   return it to worktree and undo deleting
   ```shell
    git clone git@github.com:RHEcosystemAppEng-Temenos/infinity-helm-chart.git
    cd infinity-helm-chart/infinity
    ## Verify that you can see external-values.yaml in the worktree
    ll
    git rm external-values.yaml
    git status
    ## Verify that external-values.yaml no present in worktree(directory)
    ll 
    ## Pack the chart without external-values.yaml   
    helm package .
    
    ## by removing deletion of external-values from index, and from worktree, you're 
    ## "reviving" the file back to worktree(it never changed in repo though)
    git reset --hard / git restore --staged --worktree external-values.yaml

   ```
   - auto resolve a lot of conflicts in one shot(all conflicts will be resolved to our xor theirs version)git merge origin/main --strategy=recursive -X theirs - sometimes you forget about a clone for a long time, and you did some local standalone  work in the same repo without pushing it to remote repo, and you want after long time to get latest changes from remote - without getting conflicts, and without deleting the local repo and clone it again from url
   ```shell
     cd ~/git/quarkus-quickstarts/
     ## First you're doing a regular git pull
     git pull
     
   Auto-merging websockets-quickstart/pom.xml
   CONFLICT (content): Merge conflict in websockets-quickstart/pom.xml
   Removing websockets-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging vertx-quickstart/pom.xml
   CONFLICT (content): Merge conflict in vertx-quickstart/pom.xml
   Removing vertx-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging validation-quickstart/pom.xml
   CONFLICT (content): Merge conflict in validation-quickstart/pom.xml
   Removing validation-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging tika-quickstart/pom.xml
   CONFLICT (content): Merge conflict in tika-quickstart/pom.xml
   Removing tika-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging tests-with-coverage-quickstart/pom.xml
   CONFLICT (content): Merge conflict in tests-with-coverage-quickstart/pom.xml
   Removing tests-with-coverage-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging stork-quickstart/pom.xml
   CONFLICT (content): Merge conflict in stork-quickstart/pom.xml
   Removing stork-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging spring-web-quickstart/pom.xml
   CONFLICT (content): Merge conflict in spring-web-quickstart/pom.xml
   Removing spring-web-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging spring-security-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in spring-security-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging spring-security-quickstart/pom.xml
   CONFLICT (content): Merge conflict in spring-security-quickstart/pom.xml
   Removing spring-security-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging spring-scheduled-quickstart/pom.xml
   CONFLICT (content): Merge conflict in spring-scheduled-quickstart/pom.xml
   Removing spring-scheduled-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging spring-di-quickstart/pom.xml
   CONFLICT (content): Merge conflict in spring-di-quickstart/pom.xml
   Removing spring-di-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging spring-data-rest-quickstart/pom.xml
   CONFLICT (content): Merge conflict in spring-data-rest-quickstart/pom.xml
   Removing spring-data-rest-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging spring-data-jpa-quickstart/pom.xml
   CONFLICT (content): Merge conflict in spring-data-jpa-quickstart/pom.xml
   Removing spring-data-jpa-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging spring-boot-properties-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in spring-boot-properties-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging spring-boot-properties-quickstart/pom.xml
   CONFLICT (content): Merge conflict in spring-boot-properties-quickstart/pom.xml
   Removing spring-boot-properties-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging software-transactional-memory-quickstart/pom.xml
   CONFLICT (content): Merge conflict in software-transactional-memory-quickstart/pom.xml
   Removing software-transactional-memory-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-webauthn-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-webauthn-quickstart/pom.xml
   Removing security-webauthn-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-openid-connect-web-authentication-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-openid-connect-web-authentication-quickstart/pom.xml
   Removing security-openid-connect-web-authentication-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-openid-connect-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-openid-connect-quickstart/pom.xml
   Removing security-openid-connect-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-openid-connect-multi-tenancy-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-openid-connect-multi-tenancy-quickstart/pom.xml
   Removing security-openid-connect-multi-tenancy-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-openid-connect-client-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-openid-connect-client-quickstart/pom.xml
   Removing security-openid-connect-client-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-oauth2-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in security-oauth2-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging security-oauth2-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-oauth2-quickstart/pom.xml
   Removing security-oauth2-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-keycloak-authorization-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-keycloak-authorization-quickstart/pom.xml
   Removing security-keycloak-authorization-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-jwt-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in security-jwt-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging security-jwt-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-jwt-quickstart/pom.xml
   Removing security-jwt-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-jpa-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in security-jpa-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging security-jpa-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-jpa-quickstart/pom.xml
   Removing security-jpa-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging security-jdbc-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in security-jdbc-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging security-jdbc-quickstart/pom.xml
   CONFLICT (content): Merge conflict in security-jdbc-quickstart/pom.xml
   Removing security-jdbc-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging scheduler-quickstart/pom.xml
   CONFLICT (content): Merge conflict in scheduler-quickstart/pom.xml
   Removing scheduler-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging rest-json-quickstart/pom.xml
   CONFLICT (content): Merge conflict in rest-json-quickstart/pom.xml
   Removing rest-json-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging rest-client-reactive-quickstart/pom.xml
   CONFLICT (content): Merge conflict in rest-client-reactive-quickstart/pom.xml
   Removing rest-client-reactive-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging rest-client-quickstart/pom.xml
   CONFLICT (content): Merge conflict in rest-client-quickstart/pom.xml
   Removing rest-client-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging rest-client-multipart-quickstart/pom.xml
   CONFLICT (content): Merge conflict in rest-client-multipart-quickstart/pom.xml
   Removing rest-client-multipart-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging redis-quickstart/pom.xml
   CONFLICT (content): Merge conflict in redis-quickstart/pom.xml
   Removing redis-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging reactive-routes-quickstart/pom.xml
   CONFLICT (content): Merge conflict in reactive-routes-quickstart/pom.xml
   Removing reactive-routes-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging reactive-messaging-websockets-quickstart/pom.xml
   CONFLICT (content): Merge conflict in reactive-messaging-websockets-quickstart/pom.xml
   Removing reactive-messaging-websockets-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging reactive-messaging-http-quickstart/pom.xml
   CONFLICT (content): Merge conflict in reactive-messaging-http-quickstart/pom.xml
   Removing reactive-messaging-http-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging rabbitmq-quickstart/rabbitmq-quickstart-producer/pom.xml
   CONFLICT (content): Merge conflict in rabbitmq-quickstart/rabbitmq-quickstart-producer/pom.xml
   Removing rabbitmq-quickstart/rabbitmq-quickstart-producer/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging rabbitmq-quickstart/rabbitmq-quickstart-processor/pom.xml
   CONFLICT (content): Merge conflict in rabbitmq-quickstart/rabbitmq-quickstart-processor/pom.xml
   Removing rabbitmq-quickstart/rabbitmq-quickstart-processor/.mvn/wrapper/MavenWrapperDownloader.java
   Removing rabbitmq-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging qute-quickstart/pom.xml
   CONFLICT (content): Merge conflict in qute-quickstart/pom.xml
   Removing qute-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging quartz-quickstart/pom.xml
   CONFLICT (content): Merge conflict in quartz-quickstart/pom.xml
   Removing quartz-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging optaplanner-quickstart/pom.xml
   CONFLICT (content): Merge conflict in optaplanner-quickstart/pom.xml
   Removing optaplanner-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging opentracing-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in opentracing-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging opentracing-quickstart/pom.xml
   CONFLICT (content): Merge conflict in opentracing-quickstart/pom.xml
   Removing opentracing-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging opentelemetry-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in opentelemetry-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging opentelemetry-quickstart/pom.xml
   CONFLICT (content): Merge conflict in opentelemetry-quickstart/pom.xml
   Removing opentelemetry-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging openapi-swaggerui-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in openapi-swaggerui-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging openapi-swaggerui-quickstart/pom.xml
   CONFLICT (content): Merge conflict in openapi-swaggerui-quickstart/pom.xml
   Removing openapi-swaggerui-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging neo4j-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in neo4j-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging neo4j-quickstart/pom.xml
   CONFLICT (content): Merge conflict in neo4j-quickstart/pom.xml
   Removing neo4j-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging mqtt-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in mqtt-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging mqtt-quickstart/pom.xml
   CONFLICT (content): Merge conflict in mqtt-quickstart/pom.xml
   Removing mqtt-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging mongodb-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in mongodb-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging mongodb-quickstart/pom.xml
   CONFLICT (content): Merge conflict in mongodb-quickstart/pom.xml
   Removing mongodb-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging mongodb-panache-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in mongodb-panache-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging mongodb-panache-quickstart/pom.xml
   CONFLICT (content): Merge conflict in mongodb-panache-quickstart/pom.xml
   Removing mongodb-panache-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging microprofile-metrics-quickstart/pom.xml
   CONFLICT (content): Merge conflict in microprofile-metrics-quickstart/pom.xml
   Removing microprofile-metrics-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging microprofile-health-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in microprofile-health-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging microprofile-health-quickstart/pom.xml
   CONFLICT (content): Merge conflict in microprofile-health-quickstart/pom.xml
   Removing microprofile-health-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging microprofile-graphql-quickstart/pom.xml
   CONFLICT (content): Merge conflict in microprofile-graphql-quickstart/pom.xml
   Removing microprofile-graphql-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging microprofile-graphql-client-quickstart/pom.xml
   CONFLICT (content): Merge conflict in microprofile-graphql-client-quickstart/pom.xml
   Removing microprofile-graphql-client-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging microprofile-fault-tolerance-quickstart/pom.xml
   CONFLICT (content): Merge conflict in microprofile-fault-tolerance-quickstart/pom.xml
   Removing microprofile-fault-tolerance-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging micrometer-quickstart/pom.xml
   CONFLICT (content): Merge conflict in micrometer-quickstart/pom.xml
   Removing micrometer-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging mailer-quickstart/pom.xml
   CONFLICT (content): Merge conflict in mailer-quickstart/pom.xml
   Removing mailer-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging liquibase-quickstart/pom.xml
   CONFLICT (content): Merge conflict in liquibase-quickstart/pom.xml
   Removing liquibase-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging liquibase-mongodb-quickstart/pom.xml
   CONFLICT (content): Merge conflict in liquibase-mongodb-quickstart/pom.xml
   Removing liquibase-mongodb-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging lifecycle-quickstart/pom.xml
   CONFLICT (content): Merge conflict in lifecycle-quickstart/pom.xml
   Removing lifecycle-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kogito-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kogito-quickstart/pom.xml
   Removing kogito-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kogito-pmml-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kogito-pmml-quickstart/pom.xml
   Removing kogito-pmml-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kogito-drl-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kogito-drl-quickstart/pom.xml
   Removing kogito-drl-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kogito-dmn-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kogito-dmn-quickstart/pom.xml
   Removing kogito-dmn-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kafka-streams-quickstart/producer/pom.xml
   CONFLICT (content): Merge conflict in kafka-streams-quickstart/producer/pom.xml
   Auto-merging kafka-streams-quickstart/aggregator/pom.xml
   CONFLICT (content): Merge conflict in kafka-streams-quickstart/aggregator/pom.xml
   Removing kafka-streams-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kafka-quickstart/producer/pom.xml
   CONFLICT (content): Merge conflict in kafka-quickstart/producer/pom.xml
   Auto-merging kafka-quickstart/processor/pom.xml
   CONFLICT (content): Merge conflict in kafka-quickstart/processor/pom.xml
   Removing kafka-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kafka-panache-reactive-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in kafka-panache-reactive-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging kafka-panache-reactive-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kafka-panache-reactive-quickstart/pom.xml
   Removing kafka-panache-reactive-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kafka-panache-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in kafka-panache-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging kafka-panache-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kafka-panache-quickstart/pom.xml
   Removing kafka-panache-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kafka-bare-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kafka-bare-quickstart/pom.xml
   Removing kafka-bare-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging kafka-avro-schema-quickstart/pom.xml
   CONFLICT (content): Merge conflict in kafka-avro-schema-quickstart/pom.xml
   Removing kafka-avro-schema-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging jta-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in jta-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging jta-quickstart/pom.xml
   CONFLICT (content): Merge conflict in jta-quickstart/pom.xml
   Removing jta-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging jms-quickstart/pom.xml
   CONFLICT (content): Merge conflict in jms-quickstart/pom.xml
   Removing jms-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging infinispan-client-quickstart/pom.xml
   CONFLICT (content): Merge conflict in infinispan-client-quickstart/pom.xml
   Removing infinispan-client-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-search-orm-elasticsearch-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-search-orm-elasticsearch-quickstart/pom.xml
   Removing hibernate-search-orm-elasticsearch-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-reactive-routes-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-reactive-routes-quickstart/pom.xml
   Removing hibernate-reactive-routes-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-reactive-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-reactive-quickstart/pom.xml
   Removing hibernate-reactive-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-reactive-panache-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-reactive-panache-quickstart/pom.xml
   Removing hibernate-reactive-panache-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-orm-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-orm-quickstart/pom.xml
   Removing hibernate-orm-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-orm-panache-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-orm-panache-quickstart/pom.xml
   Removing hibernate-orm-panache-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-orm-panache-kotlin-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-orm-panache-kotlin-quickstart/pom.xml
   Removing hibernate-orm-panache-kotlin-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging hibernate-orm-multi-tenancy-quickstart/pom.xml
   CONFLICT (content): Merge conflict in hibernate-orm-multi-tenancy-quickstart/pom.xml
   Removing hibernate-orm-multi-tenancy-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging grpc-tls-quickstart/pom.xml
   CONFLICT (content): Merge conflict in grpc-tls-quickstart/pom.xml
   Removing grpc-tls-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging grpc-plain-text-quickstart/pom.xml
   CONFLICT (content): Merge conflict in grpc-plain-text-quickstart/pom.xml
   Removing grpc-plain-text-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging google-cloud-functions-quickstart/pom.xml
   CONFLICT (content): Merge conflict in google-cloud-functions-quickstart/pom.xml
   Removing google-cloud-functions-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging google-cloud-functions-http-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in google-cloud-functions-http-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging google-cloud-functions-http-quickstart/pom.xml
   CONFLICT (content): Merge conflict in google-cloud-functions-http-quickstart/pom.xml
   Removing google-cloud-functions-http-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging getting-started/pom.xml
   CONFLICT (content): Merge conflict in getting-started/pom.xml
   Removing getting-started/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging getting-started-testing/pom.xml
   CONFLICT (content): Merge conflict in getting-started-testing/pom.xml
   Removing getting-started-testing/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging getting-started-reactive/pom.xml
   CONFLICT (content): Merge conflict in getting-started-reactive/pom.xml
   Removing getting-started-reactive/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging getting-started-reactive-crud/pom.xml
   CONFLICT (content): Merge conflict in getting-started-reactive-crud/pom.xml
   Removing getting-started-reactive-crud/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging getting-started-knative/pom.xml
   CONFLICT (content): Merge conflict in getting-started-knative/pom.xml
   Removing getting-started-knative/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging getting-started-command-mode/pom.xml
   CONFLICT (content): Merge conflict in getting-started-command-mode/pom.xml
   Removing getting-started-command-mode/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging getting-started-async/pom.xml
   CONFLICT (content): Merge conflict in getting-started-async/pom.xml
   Removing getting-started-async/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging funqy-quickstarts/funqy-knative-events-quickstart/pom.xml
   CONFLICT (content): Merge conflict in funqy-quickstarts/funqy-knative-events-quickstart/pom.xml
   Removing funqy-quickstarts/funqy-knative-events-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging funqy-quickstarts/funqy-http-quickstart/pom.xml
   CONFLICT (content): Merge conflict in funqy-quickstarts/funqy-http-quickstart/pom.xml
   Removing funqy-quickstarts/funqy-http-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging funqy-quickstarts/funqy-google-cloud-functions-quickstart/pom.xml
   CONFLICT (content): Merge conflict in funqy-quickstarts/funqy-google-cloud-functions-quickstart/pom.xml
   Removing funqy-quickstarts/funqy-google-cloud-functions-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging funqy-quickstarts/funqy-google-cloud-functions-http-quickstart/pom.xml
   CONFLICT (content): Merge conflict in funqy-quickstarts/funqy-google-cloud-functions-http-quickstart/pom.xml
   Removing funqy-quickstarts/funqy-google-cloud-functions-http-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Removing funqy-quickstarts/funqy-azure-functions-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging funqy-quickstarts/funqy-amazon-lambda-quickstart/pom.xml
   CONFLICT (content): Merge conflict in funqy-quickstarts/funqy-amazon-lambda-quickstart/pom.xml
   Removing funqy-quickstarts/funqy-amazon-lambda-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging funqy-quickstarts/funqy-amazon-lambda-http-quickstart/pom.xml
   CONFLICT (content): Merge conflict in funqy-quickstarts/funqy-amazon-lambda-http-quickstart/pom.xml
   Removing funqy-quickstarts/funqy-amazon-lambda-http-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging context-propagation-quickstart/pom.xml
   CONFLICT (content): Merge conflict in context-propagation-quickstart/pom.xml
   Removing context-propagation-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging config-quickstart/pom.xml
   CONFLICT (content): Merge conflict in config-quickstart/pom.xml
   Removing config-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging cache-quickstart/src/main/resources/META-INF/resources/index.html
   CONFLICT (content): Merge conflict in cache-quickstart/src/main/resources/META-INF/resources/index.html
   Auto-merging cache-quickstart/pom.xml
   CONFLICT (content): Merge conflict in cache-quickstart/pom.xml
   Removing cache-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging awt-graphics-rest-quickstart/pom.xml
   CONFLICT (content): Merge conflict in awt-graphics-rest-quickstart/pom.xml
   Removing awt-graphics-rest-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amqp-quickstart/amqp-quickstart-producer/pom.xml
   CONFLICT (content): Merge conflict in amqp-quickstart/amqp-quickstart-producer/pom.xml
   Removing amqp-quickstart/amqp-quickstart-producer/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amqp-quickstart/amqp-quickstart-processor/pom.xml
   CONFLICT (content): Merge conflict in amqp-quickstart/amqp-quickstart-processor/pom.xml
   Removing amqp-quickstart/amqp-quickstart-processor/.mvn/wrapper/MavenWrapperDownloader.java
   Removing amqp-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amazon-ssm-quickstart/pom.xml
   CONFLICT (content): Merge conflict in amazon-ssm-quickstart/pom.xml
   Removing amazon-ssm-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amazon-sqs-quickstart/pom.xml
   CONFLICT (content): Merge conflict in amazon-sqs-quickstart/pom.xml
   Removing amazon-sqs-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amazon-sns-quickstart/pom.xml
   CONFLICT (content): Merge conflict in amazon-sns-quickstart/pom.xml
   Removing amazon-sns-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amazon-ses-quickstart/pom.xml
   CONFLICT (content): Merge conflict in amazon-ses-quickstart/pom.xml
   Removing amazon-ses-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amazon-s3-quickstart/pom.xml
   CONFLICT (content): Merge conflict in amazon-s3-quickstart/pom.xml
   Removing amazon-s3-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amazon-kms-quickstart/pom.xml
   CONFLICT (content): Merge conflict in amazon-kms-quickstart/pom.xml
   Removing amazon-kms-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Auto-merging amazon-dynamodb-quickstart/pom.xml
   CONFLICT (content): Merge conflict in amazon-dynamodb-quickstart/pom.xml
   Removing amazon-dynamodb-quickstart/.mvn/wrapper/MavenWrapperDownloader.java
   Removing .mvn/wrapper/MavenWrapperDownloader.java
   Automatic merge failed; fix conflicts and then commit the result.
   [zgrinber@zgrinber quarkus-quickstarts]$ git status
   On branch main
   Your branch and 'origin/main' have diverged,
   and have 3 and 72 different commits each, respectively.
     (use "git pull" to merge the remote branch into yours)

   You have unmerged paths.
     (fix conflicts and run "git commit")
     (use "git merge --abort" to abort the merge)

        Unmerged paths:
     (use "git add <file>..." to mark resolution)
    both modified:   amazon-dynamodb-quickstart/pom.xml
    both modified:   amazon-kms-quickstart/pom.xml
    both modified:   amazon-s3-quickstart/pom.xml
    both modified:   amazon-ses-quickstart/pom.xml
    both modified:   amazon-sns-quickstart/pom.xml
    both modified:   amazon-sqs-quickstart/pom.xml
    both modified:   amazon-ssm-quickstart/pom.xml
    both modified:   amqp-quickstart/amqp-quickstart-processor/pom.xml
    both modified:   amqp-quickstart/amqp-quickstart-producer/pom.xml
    both modified:   awt-graphics-rest-quickstart/pom.xml
    both modified:   cache-quickstart/pom.xml
    both modified:   cache-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   config-quickstart/pom.xml
    both modified:   context-propagation-quickstart/pom.xml
    both modified:   funqy-quickstarts/funqy-amazon-lambda-http-quickstart/pom.xml
    both modified:   funqy-quickstarts/funqy-amazon-lambda-quickstart/pom.xml
    both modified:   funqy-quickstarts/funqy-google-cloud-functions-http-quickstart/pom.xml
    both modified:   funqy-quickstarts/funqy-google-cloud-functions-quickstart/pom.xml
    both modified:   funqy-quickstarts/funqy-http-quickstart/pom.xml
    both modified:   funqy-quickstarts/funqy-knative-events-quickstart/pom.xml
    both modified:   getting-started-async/pom.xml
    both modified:   getting-started-command-mode/pom.xml
    both modified:   getting-started-knative/pom.xml
    both modified:   getting-started-reactive-crud/pom.xml
    both modified:   getting-started-reactive/pom.xml
    both modified:   getting-started-testing/pom.xml
    both modified:   getting-started/pom.xml
    both modified:   google-cloud-functions-http-quickstart/pom.xml
    both modified:   google-cloud-functions-http-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   google-cloud-functions-quickstart/pom.xml
    both modified:   grpc-plain-text-quickstart/pom.xml
    both modified:   grpc-tls-quickstart/pom.xml
    both modified:   hibernate-orm-multi-tenancy-quickstart/pom.xml
    both modified:   hibernate-orm-panache-kotlin-quickstart/pom.xml
    both modified:   hibernate-orm-panache-quickstart/pom.xml
    both modified:   hibernate-orm-quickstart/pom.xml
    both modified:   hibernate-reactive-panache-quickstart/pom.xml
    both modified:   hibernate-reactive-quickstart/pom.xml
    both modified:   hibernate-reactive-routes-quickstart/pom.xml
    both modified:   hibernate-search-orm-elasticsearch-quickstart/pom.xml
    both modified:   infinispan-client-quickstart/pom.xml
    both modified:   jms-quickstart/pom.xml
    both modified:   jta-quickstart/pom.xml
    both modified:   jta-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   kafka-avro-schema-quickstart/pom.xml
    both modified:   kafka-bare-quickstart/pom.xml
    both modified:   kafka-panache-quickstart/pom.xml
    both modified:   kafka-panache-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   kafka-panache-reactive-quickstart/pom.xml
    both modified:   kafka-panache-reactive-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   kafka-quickstart/processor/pom.xml
    both modified:   kafka-quickstart/producer/pom.xml
    both modified:   kafka-streams-quickstart/aggregator/pom.xml
    both modified:   kafka-streams-quickstart/producer/pom.xml
    both modified:   kogito-dmn-quickstart/pom.xml
    both modified:   kogito-drl-quickstart/pom.xml
    both modified:   kogito-pmml-quickstart/pom.xml
    both modified:   kogito-quickstart/pom.xml
    both modified:   lifecycle-quickstart/pom.xml
    both modified:   liquibase-mongodb-quickstart/pom.xml
    both modified:   liquibase-quickstart/pom.xml
    both modified:   mailer-quickstart/pom.xml
    both modified:   micrometer-quickstart/pom.xml
    both modified:   microprofile-fault-tolerance-quickstart/pom.xml
    both modified:   microprofile-graphql-client-quickstart/pom.xml
    both modified:   microprofile-graphql-quickstart/pom.xml
    both modified:   microprofile-health-quickstart/pom.xml
    both modified:   microprofile-health-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   microprofile-metrics-quickstart/pom.xml
    both modified:   mongodb-panache-quickstart/pom.xml
    both modified:   mongodb-panache-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   mongodb-quickstart/pom.xml
    both modified:   mongodb-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   mqtt-quickstart/pom.xml
    both modified:   mqtt-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   neo4j-quickstart/pom.xml
    both modified:   neo4j-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   openapi-swaggerui-quickstart/pom.xml
    both modified:   openapi-swaggerui-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   opentelemetry-quickstart/pom.xml
    both modified:   opentelemetry-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   opentracing-quickstart/pom.xml
    both modified:   opentracing-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   optaplanner-quickstart/pom.xml
    both modified:   quartz-quickstart/pom.xml
    both modified:   qute-quickstart/pom.xml
    both modified:   rabbitmq-quickstart/rabbitmq-quickstart-processor/pom.xml
    both modified:   rabbitmq-quickstart/rabbitmq-quickstart-producer/pom.xml
    both modified:   reactive-messaging-http-quickstart/pom.xml
    both modified:   reactive-messaging-websockets-quickstart/pom.xml
    both modified:   reactive-routes-quickstart/pom.xml
    both modified:   redis-quickstart/pom.xml
    both modified:   rest-client-multipart-quickstart/pom.xml
    both modified:   rest-client-quickstart/pom.xml
    both modified:   rest-client-reactive-quickstart/pom.xml
    both modified:   rest-json-quickstart/pom.xml
    both modified:   scheduler-quickstart/pom.xml
    both modified:   security-jdbc-quickstart/pom.xml
    both modified:   security-jdbc-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   security-jpa-quickstart/pom.xml
    both modified:   security-jpa-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   security-jwt-quickstart/pom.xml
    both modified:   security-jwt-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   security-keycloak-authorization-quickstart/pom.xml
    both modified:   security-oauth2-quickstart/pom.xml
    both modified:   security-oauth2-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   security-openid-connect-client-quickstart/pom.xml
    both modified:   security-openid-connect-multi-tenancy-quickstart/pom.xml
    both modified:   security-openid-connect-quickstart/pom.xml
    both modified:   security-openid-connect-web-authentication-quickstart/pom.xml
    both modified:   security-webauthn-quickstart/pom.xml
    both modified:   software-transactional-memory-quickstart/pom.xml
    both modified:   spring-boot-properties-quickstart/pom.xml
    both modified:   spring-boot-properties-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   spring-data-jpa-quickstart/pom.xml
    both modified:   spring-data-rest-quickstart/pom.xml
    both modified:   spring-di-quickstart/pom.xml
    both modified:   spring-scheduled-quickstart/pom.xml
    both modified:   spring-security-quickstart/pom.xml
    both modified:   spring-security-quickstart/src/main/resources/META-INF/resources/index.html
    both modified:   spring-web-quickstart/pom.xml
    both modified:   stork-quickstart/pom.xml
    both modified:   tests-with-coverage-quickstart/pom.xml
    both modified:   tika-quickstart/pom.xml
    both modified:   validation-quickstart/pom.xml
    both modified:   vertx-quickstart/pom.xml
    both modified:   websockets-quickstart/pom.xml

   ##Off course we're not going to resolve everything one by one, then we'll remember that git pull = git fetch + git merge , and we'll do exactly this but with flags that matching that case:
      git reset --hard
      git fetch
      git merge origin/main --strategy=recursive -X theirs
   ## Approve the merge commit, and you'll notice that everything was merged without conflicts, by favoring everything from upstream rather than local in order to settle all the conflicts automatically.
   
   ```
   
 - In case you did commited local work, and didn't push it yet, and in the meantime there are new commits pushed from elsewhere to remote repo, show How to undo unecessary local merge from remote server,  and use instead git pull --rebase.
 ## git bisect demo
   Link to repo that show how to use git bisect with example [can be found here](https://github.com/zvigrinberg/git-bisect-demo)
    

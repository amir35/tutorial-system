**CONTINUOS INTEGRATION WITH TEKTON**

Step 1 — Create the folder structure /tekton/tasks

Step 2 - Create our first Task "git-clone-task.yaml"

Step 3 - Apply the Task to your cluster
oc apply -f tekton/tasks/git-clone-task.yaml

Expected: task.tekton.dev/git-clone-task created

oc get tasks
NAME             AGE
git-clone-task   18s

Step 4 - Run the Task
A Task is only a definition. It doesn't execute anything. To run a Task, we need to create a TaskRun. 
A TaskRun is an instance of a Task that executes the steps defined in the Task.

Create taskrun "git-clone-taskrun.yaml"

oc apply -f git-clone-taskrun.yaml

Expected : taskrun.tekton.dev/git-clone-taskrun created

oc get taskrun
NAME                SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
git-clone-taskrun   True        Succeeded   22s         15s

Now, Pod will be created

oc get pods
NAME                                READY   STATUS      RESTARTS   AGE
git-clone-taskrun-pod               0/1     Completed   0          57s

oc logs -l tekton.dev/taskRun=git-clone-taskrun

Step 5 - Create Maven Build Task "maven-build-task.yaml"

Step 6 - Apply the Task
oc apply -f tekton\tasks\maven-build-task.yaml

Expected: task.tekton.dev/maven-build-task created

oc get Tasks
NAME               AGE
git-clone-task     16m
maven-build-task   24s

Note: Our previous Git Clone TaskRun used:
emptyDir: {}

So the cloned git repository "feature/tekton" was inside that TaskRun's temporary Pod.
That means the new Maven TaskRun cannot access that repository.

So, we need to introduce Pipeline. So that, once git clone task is done, maven task should immediately get the source code.

We need shared workspace between the two Tasks. So, we need to create PVC.

Step 7 - Create PVC "workspace-pvc.yaml"

Apply the PVC
oc apply -f tekton\backend\workspace-pvc.yaml

Expected: persistentvolumeclaim/tekton-workspace-pvc created

oc get pvc
NAME                   STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
tekton-workspace-pvc   Pending                                                                        gp3            <unset>                 23s

Currently PVC STATUS is Pending. Why?
oc describe pvc tekton-workspace-pvc

Check in Events
Type    Reason                Age                From                         Message
  ----    ------                ----               ----                         -------
Normal  WaitForFirstConsumer  8s (x9 over 110s)  persistentvolume-controller  waiting for first consumer to be created before binding

oc get storageclass gp3
NAME            PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp3 (default)   ebs.csi.aws.com   Delete          WaitForFirstConsumer   true                   615d

Step 8 - Create Pipeline "backend-pipeline.yaml"

Apply it
oc apply -f backend\backend-pipeline.yaml

Expected: pipeline.tekton.dev/backend-pipeline created

oc get pipeline.tekton.dev
NAME               AGE
backend-pipeline   25s

The Pipeline is just a blueprint.
We need a PipelineRun to actually execute it.

Step 9 - Create PipelineRun "backend-pipelinerun.yaml"

Apply it: oc create -f backend\backend-pipelinerun.yaml

Expected : pipelinerun.tekton.dev/backend-pipeline-run-sq8lz created

Here, we are using create, not apply. Because we have used generatedName. 
This will create a new PipelineRun every time we run this command.

oc get pipelineruns.tekton.dev
NAME                         SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
backend-pipeline-run-sq8lz   True        Succeeded   69s         10s

oc get pvc
NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
tekton-workspace-pvc   Bound    pvc-c802b342-27e2-4b58-a6a5-5e20adf64396   1Gi        RWO            gp3            <unset>                 14m

oc get taskrun
NAME                                          SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
backend-pipeline-run-sq8lz-clone-repository   True        Succeeded   2m43s       2m30s
backend-pipeline-run-sq8lz-maven-build        True        Succeeded   2m30s       104s

oc get pods
NAME                                              READY   STATUS      RESTARTS   AGE
backend-pipeline-run-sq8lz-clone-repository-pod   0/1     Completed   0          3m12s
backend-pipeline-run-sq8lz-maven-build-pod        0/1     Completed   0          2m59s

oc logs backend-pipeline-run-sq8lz-maven-build-pod

Step 10 - Create BuildConfig "tutorial-service-tekton-buildconfig.yaml"

Here source: type: Binary because Tekton will provide the source.

Apply: oc apply -f backend\tutorial-service-tekton-buildconfig.yaml
Expected : buildconfig.build.openshift.io/tutorial-service-tekton created

oc get buildconfig
NAME                      TYPE     FROM                    LATEST
tutorial-service-tekton   Docker   Binary                  0

Step 11 - Create the Image Build Task "image-build-task.yaml"

Apply: oc apply -f tasks\image-build-task.yaml
Expected: task.tekton.dev/image-build-task created

oc get tasks
NAME               AGE
git-clone-task     105m
image-build-task   22s
maven-build-task   89m

Note: let's test the image-build Task independently, just like we did with Git Clone. We are not adding it in Pipeline now.

Step 12 - Create the Image Build TaskRun "image-build-taskrun.yaml"

Create: oc create -f tasks\image-build-taskrun.yaml
Expected: taskrun.tekton.dev/image-build-taskrun-ttb9v created

oc get taskrun
NAME                                          SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
image-build-taskrun-ttb9v                     Unknown     Running     38s

NAME                                          SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
image-build-taskrun-ttb9v                     True        Succeeded   115s        2s

oc get pods
NAME                                              READY   STATUS      RESTARTS   AGE
image-build-taskrun-ttb9v-pod                     1/1     Running     0          87s

oc logs image-build-taskrun-ttb9v-pod

Note: Our Taskrun to build the image is successful. So lets include it in the Pipeline.

Step 13 - Update backend-pipeline.yaml

Apply the updated Pipeline
oc apply -f backend\backend-pipeline.yaml
Expected: pipeline.tekton.dev/backend-pipeline configured

Run the PipelineRun again
oc create -f backend\backend-pipelinerun.yaml
Expected: pipelinerun.tekton.dev/backend-pipeline-run-k7gt2 created

oc get pipelinerun.tekton.dev
NAME                         SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
backend-pipeline-run-k7gt2   False       Failed      30s         20s

oc get taskrun
NAME                                          SUCCEEDED   REASON       STARTTIME   COMPLETIONTIME
backend-pipeline-run-k7gt2-clone-repository   False       StepFailed   63s         53s

oc get pods
NAME                                              READY   STATUS      RESTARTS   AGE
backend-pipeline-run-k7gt2-clone-repository-pod   0/1     Error       0          78s

oc logs backend-pipeline-run-k7gt2-clone-repository-pod
Defaulted container "step-clone" out of: step-clone, prepare (init), place-scripts (init)
======================================
Starting Git Clone
======================================
Repository:
https://github.com/amir35/tutorial-system.git
Revision:
feature/tekton
Workspace:
/workspace/source
======================================
fatal: destination path '/workspace/source' already exists and is not an empty directory.

Step 14 - Modify backend-pipelinerun.yaml

Create again: oc create -f backend\backend-pipelinerun.yaml
Expected : pipelinerun.tekton.dev/backend-pipeline-run-752kt created

oc get pipelinerun.tekton.dev
NAME                         SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
backend-pipeline-run-752kt   Unknown     Running     34s

NAME                         SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
backend-pipeline-run-752kt   True        Succeeded   3m15s       24s

oc get taskrun
NAME                                          SUCCEEDED   REASON       STARTTIME   COMPLETIONTIME
backend-pipeline-run-752kt-clone-repository   True        Succeeded    3m39s       3m22s
backend-pipeline-run-752kt-image-build        True        Succeeded    2m33s       48s
backend-pipeline-run-752kt-maven-build        True        Succeeded    3m22s       2m33s


oc get pods
NAME                                              READY   STATUS      RESTARTS   AGE
backend-pipeline-run-752kt-clone-repository-pod   0/1     Completed   0          3m59s
backend-pipeline-run-752kt-image-build-pod        0/1     Completed   0          2m52s
backend-pipeline-run-752kt-maven-build-pod        0/1     Completed   0          3m42s
tutorial-service-tekton-2-build                   0/1     Completed   0          3m16s

oc logs backend-pipeline-run-752kt-clone-repository-pod
oc logs backend-pipeline-run-752kt-maven-build-pod
oc logs backend-pipeline-run-752kt-image-build-pod

**CONTINUOS DEPLOYMENT WITH TEKTON**
Step 1 - Create deploy-task.yaml

Apply: oc apply -f tasks\deploy-task.yaml
Expected: task.tekton.dev/deploy-task created

oc get tasks
PS D:\DevOps\tutorial-system\tekton> oc get tasks
NAME               AGE
deploy-task        45s
git-clone-task     154m
image-build-task   49m
maven-build-task   138m

Step 2 - Test the Deploy Task independently

Create deploy-taskrun.yaml

Create: oc create -f tasks\deploy-taskrun.yaml
Expected: taskrun.tekton.dev/deploy-taskrun-gcwt8 created

oc get taskrun
NAME                                          SUCCEEDED   REASON       STARTTIME   COMPLETIONTIME
deploy-taskrun-gcwt8                          True        Succeeded    20s         10s

oc get pods
NAME                                              READY   STATUS      RESTARTS   AGE
deploy-taskrun-gcwt8-pod                          0/1     Completed   0          53s

oc logs deploy-taskrun-gcwt8-pod

Step 3 - Update backend-pipeline.yaml to include deploy-task

Apply: oc apply -f backend\backend-pipeline.yaml
Expected: pipeline.tekton.dev/backend-pipeline configured

oc get pipeline.tekton.dev
NAME               AGE
backend-pipeline   130m

Create: oc create -f backend\backend-pipelinerun.yaml
pipelinerun.tekton.dev/backend-pipeline-run-vff7p created

oc get pipelinerun.tekton.dev
NAME                         SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
backend-pipeline-run-vff7p   Unknown     Running     26s

NAME                         SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
backend-pipeline-run-vff7p   True        Succeeded   3m18s       26s

oc get taskrun
NAME                                          SUCCEEDED   REASON       STARTTIME   COMPLETIONTIME
backend-pipeline-run-vff7p-clone-repository   True        Succeeded    3m44s       3m27s
backend-pipeline-run-vff7p-deploy             True        Succeeded    63s         53s
backend-pipeline-run-vff7p-image-build        True        Succeeded    2m48s       63s
backend-pipeline-run-vff7p-maven-build        True        Succeeded    3m27s       2m48s

oc get pods
NAME                                              READY   STATUS      RESTARTS   AGE
backend-pipeline-run-vff7p-clone-repository-pod   0/1     Completed   0          4m20s
backend-pipeline-run-vff7p-deploy-pod             0/1     Completed   0          98s
backend-pipeline-run-vff7p-image-build-pod        0/1     Completed   0          3m23s
backend-pipeline-run-vff7p-maven-build-pod        0/1     Completed   0          4m2s
tutorial-service-tekton-3-build                   0/1     Completed   0          3m16s

oc logs backend-pipeline-run-vff7p-deploy-pod
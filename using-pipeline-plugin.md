# Using pipeline plugin

We'll use an example of a simple pipeline job.

Using an inline groovy code:

```text
- job:
    name: My pipeline job
    project-type: pipeline
    disabled: false
    dsl: |
      node {
        stage('Write file') {
            dir('mydir'){
               writeFile(file:'hello.txt',text:'hello world!')
               sh """
                 cat hello.txt
               """
            }
        }
      }
```

**dir** is a function that is part of the [Pipeline: Basic Steps plugin]([https://plugins.jenkins.io/workflow-basic-steps/]) \([dir function source code](https://github.com/jenkinsci/workflow-basic-steps-plugin/blob/master/src/main/java/org/jenkinsci/plugins/workflow/steps/PushdStep.java)\)  
**writeFile** is a function to write a file including a it's content

#### Including a file

**The Job definition**

```text
- job:
    name: My second pipeline job
    project-type: pipeline
    disabled: false
    dsl:
      !include-raw:
        - 'mypipeline.inc'
```

**mypipeline.inc**

```text
node {
  stage('Write file') {
      dir('mydir'){
        writeFile(file:'hello.txt',text:'hello world!')
        sh """
           cat hello.txt
        """
      }
  }
}
```


# JJB Basics

We'll cover some basic JJB definitions

### Job

An example for a simple job definition

```text
- job:
    name: job-name
```

### Builders

Builders define actions that the Jenkins job should execute.  
Examples include shell scripts or maven targets. The builders attribute in the Job definition accepts a list of builders to invoke

**A job with a simple shell builder**

```text
- job:
        name: My great job
        project-type: freestyle
        defaults: global
        description: 'Do not edit this job through the web!'
        disabled: false
        display-name: 'My great job'
        concurrent: true
        quiet-period: 5
        block-downstream: false
        block-upstream: false
        retry-count: 3
        logrotate:
          daysToKeep: 3
          numToKeep: 20
          artifactDaysToKeep: -1
          artifactNumToKeep: -1
        builders:
          - shell: "echo hello from $JOB_NAME"
```

The above will create a simple job which will print "hello from My great job"

### Job Template

If you have several jobs which are nearly identical you'll make use of a job template

An example for a job template

```text
- job-template:
    name: '{name}-{suffix}'
    builders:
      - shell: echo "My job name is $JOB_NAME and the suffix is {suffix} and my blessing is {my_blessing}"
```

### Project

The purpose of a project is to collect related jobs together, and provide values for the variables in a Job Template. It looks like this:

**A project with one job**

```text
- job-template:
    name: '{name}-unit-tests'

- project:
    name: mygreatproject
    jobs:
      - '{name}-unit-tests'
```

In the above example we define one job where the {name} variable is the project name so the result job name will be _**mygreatproject-unit-tests**_

**An example of a template and jobs that being realized by a project using this template**

```text
- job-template:
    name: '{name}-{suffix}'
    builders:
      - shell: echo "My job name is $JOB_NAME and the suffix is {suffix}"

- project:
   name: myproject
   suffix:
     - foo
     - bar
   jobs:
    - '{name}-{suffix}'
```

**Another example for using a project and a template**

```text
- job-template:
    name: '{name}-{pyver}'
    builders:
      - shell: 'git co {branch_name}'

- project:
   name: mypoject
   pyver:
    - 26:
       branch_name: old_branch
    - 27:
       branch_name: new_branch
   jobs:
    - '{name}-{pyver}'
```

We have two ****jobs:  
****_**myproject-26**_ where we'll checkout to old\_branch  
_**myproject-27**_ where we'll checkout to new\_branch

### Defaults

Defaults collect job attributes \(including actions\) and will supply those values when the job is created, unless superseded by a value in the ‘Job’\_ definition If a set of Defaults is specified with the name global, that will be used by all Job \(and Job Template\) definitions unless they specify a different Default object with the defaults attribute.

**An example for using a global defaults**

```text
- defaults:
   name: global
   my_blessing: 'Hello World'

- job-template:
    name: '{name}-{suffix}'
    builders:
      - shell: echo "My job name is $JOB_NAME and the suffix is {suffix} and my blessing is {my_blessing}"

- project:
   name: myproject
   suffix:
     - foo:
        my_blessing: 'Good evening'
     - bar
   jobs:
    - '{name}-{suffix}'
```

**Using global and named templates**

```text
- defaults:
    name: mydefaults
    my_blessing: "What's up?"

- defaults:
    name: global
    my_blessing: "How are you?"

- job-template:
    name: '{name}-{suffix}'
    builders:
      - shell: echo "My job name is $JOB_NAME and the suffix is {suffix} and my blessing is {my_blessing}"

- project:
    my_blessing: 'Nice to meet you!'
    name: myproject
    defaults: 'mydefaults'
    suffix:
      - foo
      - bar:
          my_blessing: 'Good evening'
    jobs:
      - '{name}-{suffix}'

- project:
  name: myproject2
  suffix:
    - foo
  jobs:
    - '{name}-{suffix}'
```

**myproject-foo** will use use my\_blessing from 'mydefaults' defaults \(mydefaults is defined in the project\)

**myproject-bar** will use use my\_blessing from the job definition

**myproject2-foo** will use use my\_blessing from global defaults \(It's not defined in other defaults\)

#### Using anchors and aliases in defaults

In YAML we have anchors and aliases

So the following YAML

```text
    - &flag Apple
    - Beachball
    - Cartoon
    - Duckface
    - *flag
```

will be expanded to

```text
- Apple
- Beachball
- Cartoon
- Duckface
- Apple
```

Using the following for JJB defaults:

```text
- wrapper_defaults: &wrapper_defaults
    name: 'wrapper_defaults'
    wrappers:
      - timeout:
          timeout: 180
          fail: true
      - timestamps

- job_defaults: &job_defaults
    name: 'defaults'
    <<: *wrapper_defaults

- job-template:
    name: 'myjob'
    <<: *job_defaults
```

will parsed to:

```text
- wrapper_defaults:
    name: wrapper_defaults
    wrappers:
    - timeout:
        fail: true
        timeout: 180
    - timestamps
- job_defaults:
    name: defaults
    wrappers:
    - timeout:
        fail: true
        timeout: 180
    - timestamps
- job-template:
    name: myjob
    wrappers:
    - timeout:
        fail: true
        timeout: 180
    - timestamps
```

**An example for using anchors and aliases**

```text
- defaults: &mydefaults
        name: mydefaults
        my_blessing: "What's up?"
        day: 'Sunday'

    - defaults:
        name: mydefaults2
        my_blessing: "How are you?"
        <<: *mydefaults

    - job-template:
        name: '{name}-{suffix}'
        builders:
          - shell: echo "My job name is $JOB_NAME and the suffix is {suffix} and my blessing is {my_blessing} and the day is {day}"

    - project:
        name: myproject
        defaults: 'mydefaults'
        suffix:
          - foo
          - bar:
             my_blessing: 'Good evening'
        jobs:
         - '{name}-{suffix}'

    - project:
        name: myproject2
        defaults: 'mydefaults2'
        suffix:
          - foo
        jobs:
          - '{name}-{suffix}'
```

### Including raw code

[https://docs.openstack.org/infra/jenkins-job-builder/definition.html?highlight=defaults\#inclusion-tags](https://docs.openstack.org/infra/jenkins-job-builder/definition.html?highlight=defaults#inclusion-tags)

The tag !include-raw: will treat the given string or list of strings as filenames to be opened as one or more data blob, which should be read into the calling yaml construct without any further parsing. Any data in a file included through this tag, will be treated as string data.

An example including a shell script

```text
- job:
    name: My great job
    project-type: freestyle
    defaults: global
    description: 'Do not edit this job through the web!'
    disabled: false
    display-name: 'My great job'
    builders:
      - shell:
          !include-raw: '001-hello-world.sh'
```


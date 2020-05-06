# Installing Jenkins

### Install Jenkins

For learning purposes it’s good to run a personal Jenkins running in a container

```text
$ docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
```

You can then access Jenkins at [http://localhost:8080](http://localhost:8080) and follow the instructions

Another Option is to Use a pre-made container with plugins installed etc.


Dependencies bug with GitLab
============================

For some reason, the additional dependencies feature of Helmfile is not working with the official GitLab chart.

```
$ helmfile version
helmfile version v0.143.0
$ helm version
version.BuildInfo{Version:"v3.7.0", GitCommit:"eeac83883cb4014fe60267ec6373570374ce770b", GitTreeState:"clean", GoVersion:"go1.16.8"}
```

If you try to template the following Helmfile, it will result in the following error message:

```
repositories:
  - name: aservo
    url: https://aservo.github.io/charts
  - name: gitlab
    url: https://charts.gitlab.io

releases:
  - name: gitlab
    chart: gitlab/gitlab
    version: 5.8.2
    values:
      - certmanager-issuer:
          email: issuer@localhost
    ## Does NOT work:
    dependencies:
      - chart: aservo/util
        version: 0.0.1
        alias: resources
```

Output:

```
$ helmfile template
Adding repo aservo https://aservo.github.io/charts
"aservo" has been added to your repositories

Adding repo gitlab https://charts.gitlab.io
"gitlab" has been added to your repositories

in ./helmfile.yaml: [exit status 1

COMMAND:
  helm dependency up /tmp/chartify2471020974/gitlab/gitlab

OUTPUT:
  Error: gitlab chart not found in repo https://aservo.github.io/charts]
```

However, the same mechanism works with all kind of other charts, e.g. this one works:

```
repositories:
  - name: aservo
    url: https://aservo.github.io/charts
  - name: jenkins
    url: https://charts.jenkins.io

releases:
  - name: jenkins
    chart: jenkins/jenkins
    version: 3.11.5
    ## DOES work:
    dependencies:
      - chart: aservo/util
        version: 0.0.1
        alias: resources
```

I already tried to figure out why, checked for collisions of dependency names in the GitLab chart (`util` / `resources`), checked the chart's API versions, whether they use dependencies in the `Chart.yaml` or `requirements.yaml` etc. but I couldn't find anything without debugging into the code.

The debug output of template looks like this:

```
$ helmfile --debug template
processing file "helmfile.yaml" in directory "."
first-pass rendering starting for "helmfile.yaml.part.0": inherited=&{default map[] map[]}, overrode=<nil>
first-pass uses: &{default map[] map[]}
first-pass rendering output of "helmfile.yaml.part.0":
 0: repositories:
 1:   - name: aservo
 2:     url: https://aservo.github.io/charts
 3:   - name: gitlab
 4:     url: https://charts.gitlab.io
 5: 
 6: releases:
 7:   - name: gitlab
 8:     chart: gitlab/gitlab
 9:     version: 4.12.13
10:     values:
11:       - certmanager-issuer:
12:           email: issuer@localhost
13:     ## Does NOT work:
14:     dependencies:
15:       - chart: aservo/util
16:         version: 0.0.1
17:         alias: resources
18: 

first-pass produced: &{default map[] map[]}
first-pass rendering result of "helmfile.yaml.part.0": {default map[] map[]}
vals:
map[]
defaultVals:[]
second-pass rendering result of "helmfile.yaml.part.0":
 0: repositories:
 1:   - name: aservo
 2:     url: https://aservo.github.io/charts
 3:   - name: gitlab
 4:     url: https://charts.gitlab.io
 5: 
 6: releases:
 7:   - name: gitlab
 8:     chart: gitlab/gitlab
 9:     version: 4.12.13
10:     values:
11:       - certmanager-issuer:
12:           email: issuer@localhost
13:     ## Does NOT work:
14:     dependencies:
15:       - chart: aservo/util
16:         version: 0.0.1
17:         alias: resources
18: 

merged environment: &{default map[] map[]}
helm:GtyWr> v3.7.0+geeac838
Adding repo aservo https://aservo.github.io/charts
exec: helm repo add aservo https://aservo.github.io/charts --force-update
helm:NvkEV> "aservo" has been added to your repositories
"aservo" has been added to your repositories

Adding repo gitlab https://charts.gitlab.io
exec: helm repo add gitlab https://charts.gitlab.io --force-update
helm:iCIlH> "gitlab" has been added to your repositories
"gitlab" has been added to your repositories

running helm fetch gitlab/gitlab --untar -d /tmp/chartify3582487459/gitlab --version 4.12.13
running helm repo list
Removing the dependencies field from the original Chart.yaml.
running helm dependency up /tmp/chartify3582487459/gitlab/gitlab
Error: gitlab chart not found in repo https://aservo.github.io/charts

Removed /tmp/helmfile1078984880/gitlab-values-797946645c
err: [exit status 1

COMMAND:
  helm dependency up /tmp/chartify3582487459/gitlab/gitlab

OUTPUT:
  Error: gitlab chart not found in repo https://aservo.github.io/charts]
in ./helmfile.yaml: [exit status 1

COMMAND:
  helm dependency up /tmp/chartify3582487459/gitlab/gitlab

OUTPUT:
  Error: gitlab chart not found in repo https://aservo.github.io/charts]
```

Unfortunately, I'm not experienced enough with Go to dig into the code myself.

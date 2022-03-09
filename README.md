`.Namespace` bug in `needs`
===========================

For some reason, I cannot use `.Namespace` in a release's `needs` block anymore. This was working with Helmfile 0.141.0 and the same Helm version. Current versions:

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

environments:
  default:

templates:
  defaults: &defaults
    name: "{{`{{ .Environment.Name }}`}}-{{`{{ .Release.Labels.service }}`}}"
    namespace: "{{`{{ .Environment.Name }}`}}"

releases:
  - chart: aservo/util
    version: 0.0.1
    labels:
      service: shared-resources
    <<: *defaults
  - chart: aservo/util
    version: 0.0.1
    labels:
      service: release-resources
    <<: *defaults
    needs:
      - "{{`{{ .Namespace }}`}}-shared-resources"
```

Output:

```
$ helmfile template
Adding repo aservo https://aservo.github.io/charts
"aservo" has been added to your repositories

in ./helmfile.yaml: release(s) "default/default-release-resources" depend(s) on an undefined release "default/{{ .Namespace }}-shared-resources". Perhaps you made a typo in "needs" or forgot defining a release named "{{ .Namespace }}-shared-resources" with appropriate "namespace" and "kubeContext"?
```

With version 0.141.0, the successful output was:

```
Adding repo aservo https://aservo.github.io/charts
"aservo" has been added to your repositories

Templating release=default-shared-resources, chart=aservo/util
Templating release=default-release-resources, chart=aservo/util
```

Unfortunately, I'm not experienced enough with Go to dig into the code myself.

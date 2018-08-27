---
layout: post
title: "Helm Environment Variables with Booleans and Integers"
excerpt: "Use non string values in helm charts without the weirdness"
tags: [dev, helm, docker, k8s, helm, values, chart, env, environment, programming, tech]
comments: true
---
## Background
I have recently been working on a client project hosted in Kubernetes as part of this we made the decision to use [Helm](https://helm.sh/) to package and coordinate the deployment of our kubernetes resources along with environment specific settings (dev, test, prod ect). As a part of this we need to provide settings in a values.yaml file that helm will combine with a user defined template to generate a valid kubernetes yaml file for deployment. 
```yaml
favoriteDrink: coffee
```
- values.yaml, see https://docs.helm.sh/chart_template_guide/#values-files

To provide our applications with the configuration information from the values file we can use the kubernetes env property on container resources
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Release.Name }}
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: FAVORITE_DRINK
      value: {{ .Values.favoriteDrink }}
```
Which will be deployed to kubernetes as 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-release
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: FAVORITE_DRINK
      value: coffee
```
- https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

# Problem
The problem with this is when we start using values that look like they should be a boolean or integer and get cast automatically instead of being handles as strings. Especially problematic when combined with environment variables that must be a string we can end up with unexpected behaviour as below.
```yaml
favoriteDrink: coffee
takesSugar: true
spoonsOfSugar: 2
```
```
failed to create patch: failed to get versionedObject: unable to convert unstructured object to extensions/v1beta1, Kind=Ingress: cannot convert int64 to string

```
Which if we read the [helm tips and tricks](https://github.com/helm/helm/blob/master/docs/charts_tips_and_tricks.md#quote-strings-dont-quote-integers) article first we would expect. But how can we deal with this in a nice way, just adding quotes to our values file as below doesn't help at all.
```yaml
favoriteDrink: coffee
takesSugar: "true"
spoonsOfSugar: "2"
```
- values.yaml
```yaml
- name: FAVORITE_DRINK
    value: {{ .Values.favoriteDrink }}
- name: TAKES_SUGAR
    value: {{ .Values.takesSugar }}
- name: SPOONS_OF_SUGAR
    value: {{ .Values.spoonsOfSugar }}
```
- partial template

# Solutions

## Quote everything
Instead of quoating the values in our values file quote the template substitutions, you could even quote the .Values.favoriteDrink line or add it to your linting or style guide to make sure everything gets converted to a string and can be set by kubernetes. 

```yaml
- name: FAVORITE_DRINK
    value: "{{ .Values.favoriteDrink }}"
- name: TAKES_SUGAR
    value: "{{ .Values.takesSugar }}"
- name: SPOONS_OF_SUGAR
    value: "{{ .Values.spoonsOfSugar }}"
```

## Bang Bang
We can use [!!str](http://yaml.org/type/str.html) to convert the output to a string,  Alternatively we can also use a undefined !! and get the same behaviour giving later developers nice hints of what we intended !!booleanEnv or !!integerEnv will cast the values to string (or even just !!boolean)
```yaml
- name: FAVORITE_DRINK
    value: !!stringEnv {{ .Values.favoriteDrink }}
- name: TAKES_SUGAR
    value: !!booleanEnv {{ .Values.takesSugar }}
- name: SPOONS_OF_SUGAR
    value: !!integerEnv {{ .Values.spoonsOfSugar }}
```

## Quote without the quote marks
Using 
```yaml
- name: FAVORITE_DRINK
    value: {{ quote .Values.favoriteDrink }}
- name: TAKES_SUGAR
    value: {{ quote .Values.takesSugar }}
- name: SPOONS_OF_SUGAR
    value: {{ quote .Values.spoonsOfSugar }}
```

# Thoughts
Each of the three options solve the same problem without having an impact on the deployed application, by making use of tricks of yaml, go templates, and kubernetes we can work around their incompatabilities. While probably the least 'correct' of the three options using the !! syntax of yaml can help to improve the readability of our templates or end up feeling like a lot of boilerplate.
# **Table of Contents**
- [**Table of Contents**](#table-of-contents)
- [**Certification Tip: Imperative Commands**](#certification-tip-imperative-commands)
  - [**Formatting Output with kubectl**](#formatting-output-with-kubectl)
- [**Find options for commands easily in cli**](#find-options-for-commands-easily-in-cli)
- [**Resources**](#resources)
  - [**Practice Questions**](#practice-questions)
  - [**Review and Tips**](#review-and-tips)

# **Certification Tip: Imperative Commands**

While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

- `-dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the `-dry-run=client` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
- `-o yaml`: This will output the resource definition in YAML format on screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

## **Formatting Output with kubectl**

The default output format for all **kubectl** commands is the human-readable plain-text format.

The -o flag allows us to output the details in several different formats.

```bash
kubectl [command] [TYPE] [NAME] -o <output_format>
```

Here are some of the commonly used formats:

1. `-o json` Output a JSON formatted API object.
2. `-o name` Print only the resource name and nothing else.
3. `-o wide` Output in the plain-text format with any additional information.
4. `-o yaml` Output a YAML formatted API object.

Here are some useful examples:

**JSON format:**

```bash
kubectl create namespace test-123 --dry-run -o json
```
- **Output**
```json
{
    "kind": "Namespace",
    "apiVersion": "v1",
    "metadata": {
        "name": "test-123",
        "creationTimestamp": null
    },
    "spec": {},
    "status": {}
}
```

**YAML format:**

```bash
kubectl create namespace test-123 --dry-run -o yaml
```

- **Output**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test-123
spec: {}
status: {}
```

**Output with wide (additional details):**

Probably the most common format used to print additional details about the object:
```bash
kubectl get pods -o wide
```
- **Output**
```bash
NAME      READY   STATUS    RESTARTS   AGE     IP          NODE     NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          3m39s   10.36.0.2   node01   <none>           <none>
ningx     1/1     Running   0          7m32s   10.44.0.1   node03   <none>           <none>
redis     1/1     Running   0          3m59s   10.36.0.1   node01   <none>           <none>
```

# **Find options for commands easily in cli**


```bash
kubectl explain po --recursive | less
```
Inside less terminal press `/` and type the pattern that you want to search top to bottom

```bash
# Get the n lines from matching pattern
kubectl explain po --recursive | grep -A<number of lines> <pattern>
```

# **Resources**

## **Practice Questions**

1. [CKAD-exercises - dgkanatsios GitHub](https://github.com/dgkanatsios/CKAD-exercises)
2. [Kubernetes Tutorials with CKA and CKAD Prep Guide - School Of Devops](https://kubernetes-tutorial.schoolofdevops.com/)
3. [ckad-labs - kensipe GitHub](https://github.com/kensipe/ckad-labs)
4. [Practice Enough With These 150 Questions for the CKAD Exam - Bhargav Bachina](https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552)
5. [Kubernetes CKAD Example Exam Questions Practical Challenge Series - Kim Wuestkamp](https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681)
6. [Practice Exam for Certified Kubernetes Application Developer (CKAD) Certification - Matthew Palmer](https://matthewpalmer.net/kubernetes-app-developer/articles/ckad-practice-exam.html)
7. [Answers to Five Certified Kubernetes Application Developer CKAD Practice Questions (2021)](https://thospfuller.com/2020/11/09/answers-to-five-kubernetes-ckad-practice-questions-2021/)
8. [CKAD Self-Study Course - rx-m.com](https://rx-m.com/ckad-online-training/) 

## **Review and Tips** 

1. [Passing CKAD with flying colours - Damian Fiłonowicz](https://blog.datumo.io/updated-q3-2021-passing-ckad-with-flying-colours-e82ccf42fa3a)
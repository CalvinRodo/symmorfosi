## GitHub Issues Check container

The purpose of this container is to check if a GitHub repo has issues. It does this by queryin the GitHub API if it returns any issues for a given project. It will output a JSON file to the `/checks` directory, or whatever is specified in `OUT_PATH` environment variable.

To use the docker container effectively you need to specify the following environmental variables

| VARIABLE    | DEFAULT VALUE       | DESCRIPTION                                                                   |
| ----------- | ------------------- | ----------------------------------------------------------------------------- |
| ORIGIN      | Missing origin      | The origin of the check, for example a container name                         |
| COMPONENT   | Missing component   | The component the check is running on. Example: "Source code"                 |
| DESCRIPTION | Missing description | A narrative description of the check run                                      |
| OUT_PATH    | "/checks"           | Where the JSON files gets written to                                          |
| SATISFIES   | _None_              | A comma delimited string of all the ex: `AU 2, AU 3`                          |
| REPO_URL    | _None_              | The URL of the GitHub repo: https://github.com/cds-snc/vac-benefits-directory |

IMPORTANT: For this image to work you need to give the `default` user in the `default` namespace permissions to read the pod list. Please check the example YAML files in this repo.

Your Kubernetes init container configuration should look something like this:

```
spec:
  containers:
    -
      image: cdssnc/compliance-api
      imagePullPolicy: Always
      name: compliance-api
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
        -
          name: compliance-checks
          mountPath: /checks
  initContainers:
    - image: 'cdssnc/github-issues-check-compliance:latest'
        imagePullPolicy: Always
        name: 'github-issues-check-compliance-si-2
        env:
          - name: ORIGIN
            value: 'cdssnc/github-issues-check-compliance:latest'
          - name: COMPONENT
            value: 'Infrastructure'
          - name: DESCRIPTION
            value: 'The GitHub repo shows issues which indicate a policy of flaw remediation.'
          - name: SATISFIES
            value: 'SI-2'
          - name: REPO_URL
            value: 'https://github.com/cds-snc/vac-benefits-directory'
      volumeMounts:
        - name: compliance-checks
          mountPath: /checks
  volumes:
    -
      name: compliance-checks
      hostPath:
        path: /checks
```

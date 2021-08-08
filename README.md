
# Following kubernetes logs for all containers in all pods of a deployment.

Running ```./follow-kube-logs.y -n my-namespace -p my-deployment -d logdir``` will have the following effect.

1. Will create directory logdir, 
2. It will create a subdirectory in logdir for each pod running in the deployment my-deployment in namepace my-namespace 
3. Spawns a process that follows the logs of each of the containers for each pod of the deployment; this process gathers the logs for that container into a file in the log directory of the pod, while the script is running. 
4. The script then waits and asks for the user to press enter, whereas it will kill the spawned processes and stop the logging.
5. While the logging is proceeding: once a second the program scans the deployment for new pods and stopped pods. For new pods we also create a subdirectory with the name of the pod in logdir. A log file for each container will be created that follows the logs of the container.
6. Events like starting/stopping of a pod are displayed on standard output.

The purpose of this script is to be a more lightweight solution then to use prometheus/graphana for viewing your deployment logs, as it is sometimes easier to grep through the logs, as compared to writing elaborate prometheus queries. Also by following the logs for a set time period you will have all of them, and you will not have to deal with log rotation.

# Installation.

1. Download the script by following [this link](https://raw.githubusercontent.com/MoserMichael/follow-kube-logs/master/follow-kube-logs.py) and make it executable; you need to have python3 on the system.
2. You can optionally install argument autocompletion for the bash shell by running the following command: ```  follow-kube-logs.py -b >>$HOME/.bashrc ```

# help file of the script

The scripts help text:
```
usage: follow-kube-logs.py [-h] [--namespace NAMESPACE] [--deployment DEPLOYMENT] [--out OUTDIR] [--kubectl KUBECMD] [--trace] [--complete-bash] [--complete]

This program starts to follow the logs of containers in all pods of a kubernetes deployment. The output is written to a file per container. The script then waits for user input, logging
is stopped once the user has pressed enter.  optional arguments: -h, --help            show this help message and exit

log  pods/containers in deployment:
  --namespace NAMESPACE, -n NAMESPACE
                        optional: specify namespace of deployment (default: )
  --deployment DEPLOYMENT, -d DEPLOYMENT
                        mandatory: name of deployment (default: )
  --out OUTDIR, -o OUTDIR
                        mandatory: name of output directory (default: )
  --kubectl KUBECMD, -k KUBECMD
                        optional: name of kubectl command (default: kubectl)
  --trace, -x           optional: enable tracing (default: False)

suport for bash autocompletion of command line arguments:
  --complete-bash, -b   show bash source of completion function (default: False)
```


# What I learned from this

1. It is best to extract some items from the kubectl with  in-built [json path querries](https://kubernetes.io/docs/reference/kubectl/jsonpath/), shorter and more robust than grepping for the output or running jq to filter stuff out.
2. For deployments: the deployment controller adjusts the set of running pods based on the pods labels; i am using the following jsonpath expression to extract the labels ``` kubectl -n NAMESPACE get deployment DEPLOYMENT_NAME -o jsonpath='{.spec.selector.matchLabels} ```
3. Using the following command to list all the pods of a deployment, and to make a listing that includes the pod name and all of its containers; 
``` kubectl -n  NAMSPACE get pods -l POD_SELECTORS -o jsonpath="{range .items[*]}{' '}{.metadata.name}{range .spec.containers[*]}{' '}{.name}{end}{'\\n'}{end}" ```

# Similar projects

It turns out there is [Stern](https://github.com/wercker/stern) - but that one is dumping all logs to the same terminal.

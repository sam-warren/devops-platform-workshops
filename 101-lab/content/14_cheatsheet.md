# Cheatsheet

This is a placeholder to work with the students and determine what they find valuable in this sheet.

## Deleting your lab

- Don't delete your lab work yet if you're planning to complete the 'pod lifecycle' and 'templates' sections of the course! Move on to those first.

**WARNING**: You should ALWAYS validate the output before using `oc delete`. You can do that by replacing `oc delete` with `oc get`, or if using with xargs prefix with `echo`.

**WARNING**: Always double check, and triple check before running `oc delete`!!!

**WARNING**: Be very careful when copying and pasting directly into a terminal!!!

```
# List/validate resources to be deleted by labels
oc -n d8f105-tools get all -l build=rocketchat-samwarren

# Delete by labels
oc -n d8f105-tools delete all -l build=rocketchat-samwarren

# List/validate resources to be deleted by get+grep+delete
oc -n d8f105-dev get all,pvc,secret,configmap -o name --no-headers | grep -i -F -e '-samwarren'

# Delete resources by using get+grep+delete
oc -n d8f105-dev get all -o name --no-headers | grep -i -F -e '-samwarren' | xargs -I {} oc  -n d8f105-dev delete '{}'

# Delete data/unrecoverable resources (not covered by 'all') by using get+grep+delete
oc -n d8f105-dev get pvc,secret,configmap -o name --no-headers | grep -i -F -e '-samwarren' | xargs -I {} oc -n d8f105-dev delete '{}'

```

Next page - [Pod Lifecycle](./15_pod_lifecycle.md)

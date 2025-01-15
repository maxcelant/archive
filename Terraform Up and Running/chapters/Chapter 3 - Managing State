### Summary


### Notes
- The output of the `plan` is a **diff** between your local terraform files and the tfstate file.
- The `tfstate` is the _source of truth_ so when you do `tf plan` or `tf apply`, it compares the state in the `tfstate` with the actual infrastructure.
- Terraform uses the current state of the cloud resources to detect _drift_.

```ad-note
If there is a resource in the cloud that the state file does not know about, it will propose to _destroy it_.
```

- Don't store your state files in version control because it's difficult to manage them with the current state. Someone could easily forget to push up the new state files and someone else will end up destroying infrastructure.
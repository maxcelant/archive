- `terraform init` is an _idempotent_ command — you can run it as many times as you want.
- **tfstate** file is to allow tf to keep track of the resources it manages.
	- Used to perform a _diff_ to see what has changed during the **plan** stage and to see if theres _config drift_.
	- Terraform uses this to decided which CRUD operations to take.
- **config drift** is the idea of changing something _not_ using terraform (like manipulating the resources in some other way), which then causes drift from what tf current knows.
- The **Local** provider acts as a teaching aid, allowing you to create local resources like files.
- Depending on the state and plan, terraform will invoke CRUD operations. (Reading first, obvs).


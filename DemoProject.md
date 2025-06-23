### Task
- Spin up 4 servers in the aws. Installed new Relic infrastructure agent on those servers using ansible plabook
#### Steps
- Installed  ansible on my wsl ubuntu.
-  Spin up 10 servers on aws. Created security group for them. created keypair.pem.
- copied downloaded keypair.pem to my ~/.ssh folder for the ssh connection with my ec2 instances.
- inventory.ini lists my host ip addresses.

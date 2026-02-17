
# Ansible Automation

-   Learn about Ansible network and GNS3 automation

# Toolkit

- Ansible VyOS module docs: <https://docs.ansible.com/ansible/latest/collections/vyos/vyos>
- YAML Lint: <https://www.yamllint.com> – YAML syntax checker
- JinjaFx: <https://jinjafx.io> - Jinja syntax checker
- For a basic Ansible overview read: <https://medium.com/@denot/ansible-101-d6dc9f86df0a>
- Overview of the JSON data structure: <https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON>

# Task 1 - Clone the class GitHub repo and install Ansible

The foundations of this lab are an accumulation of a number of guides. There was no single guide that directly explained this process. Using these lab write‑ups as a starting point in other ecosystems may not yield a working environment. Various aspects of the systems are intentionally designed to be flexible. Many of the decisions for where to put files and what content goes in what files are subjective.

1. Authenticate the gHost’s CLI Git client to GitHub using the guide at: [GitHub CLI Authentication](https://github.com/OHIO-ECT/ECT-Lab-Introduction/blob/main/tasks/GitHub-Auth.md)

2. The remainder of the homeworks will assume a directory structure like `~/Cloud/<GITHUB_REPO_NAME>`. So the directory for this assignment is `~/Cloud/ITS-4900-Cloud-Release`. Use the following commands to create and begin that directory structure and clone this homework’s Git repo. If prompted for the fingerprint, answer “yes”. That prompt occurs the first time you use the key.

```
mkdir ~/cloud
cd ~/cloud
gh repo clone OHIO-ECT/ITS-4900-Cloud-Release
```

# Task 2 - Build an SSH key

Skip this task if you already have an SSH key.

3. Generate the key. Use the default file location when prompted. Give it a passphrase that you can remember and recall in the future. Choose a reasonable security level. Professor Saunders is storing this in my KeePass file.

 ```ssh-keygen -t ed25519 -C "itsclass"```

4. Start `ssh-agent` and set environment variables. This command doesn’t prompt for anything:

```eval $(ssh-agent -s)```

5. Add your key

```ssh-add ~/.ssh/id_ed25519```

You will be prompted for the passphrase once.

6. Verify the key is loaded

```ssh-add -l```

7. *(Optional)* Have this happen each time you log in to the gHost

```
cat >> ~/.bashrc << 'EOF'

# Auto-start ssh-agent if not running
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_ed25519
fi
EOF
```

# Task 3 - New Tofu configurations

8. Inspect the Tofu `main.tf` in `~/Cloud/ITS-4900-Cloud-Release/Deliverable_3/Task3`. Document how each Tofu configuration block relates to the options used to configure the VM in Deliverable 1.

9. Identify any innovations you developed during Deliverable 2 that are not present in this assignment’s example. “Be bold. Be brave. Be courageous.” – Capt. Pike

10. Update the subscription ID on Line 14 and test this code.

11. Be sure to delete any servers created by the task.

# Task 4 - Ansible initialization

12. Install additional software on the gHost. During class we often add packages and work with system files. **Do not** run full system updates on the gHost machines; that can make them unstable and force instructors to reset your gHost to a known good state.

```
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt update
sudo apt install -y ansible software-properties-common python-is-python3 python3-pip python3-tabulate python3-lxml

pip install pydantic==1.9 --break-system-packages
```

13. Test the Ansible configuration.

```
cd ~/Cloud/ITS-4900-Cloud-Release/Deliverable_3/Task_4/
ansible-playbook azure-subscription-info.yml
```

14. Blue text during an Ansible run comes from “turning up the number of v’s” (see previous step). These are sometimes helpful at understanding what is happening and are useful during development and debugging.

15. Red text during a run usually means something is wrong. Ask questions if you receive an error message or if you don’t understand the output. Automation is great when it works; when it goes awry, it can get pretty ugly.

16. The ansible-playbook command will accept a verbosity switch `-v`. Verbosity can be increased with more v's, `-vv` or `-vvv`. The choice of `-vvv` will provide an overwhelming amount of screen output.

17. Inspect the YAML playbook and identify three key programming concepts: variables, variable manipulation, and conditional statements. Propose a new task that includes a loop.

# Task 5 - Automation from start to finish.

18. The files in `Deliverable_3/Task_5` contain a partial Tofu/Ansible project. You’ll need to personalize these files with the appropriate authentication and region information.

```cd ~/Cloud/ITS-4900-Cloud-Release/Deliverable_3/Task_5```

19. Rename the example files.

```
mv account.auto.tfvars.example account.auto.tfvars
mv project.auto.tfvars.example project.auto.tfvars
```

20. Put the proper SSH key into the variables file.

```sed -i "s|admin_ssh_public_key = \".*\"|admin_ssh_public_key = \"$(cat ~/.ssh/id_ed25519.pub)\"|" account.auto.tfvars```

21. Load subscription ID from Azure CLI

```sed -i "s|subscription_id.*=.*|subscription_id      = \"$(az account show --query id -o tsv)\"|" account.auto.tfvars```

22. Manually edit the account variables file and set a region compatible with your Azure account.

23. Inspect the main.tf file to understand what infrastructure the code will create.

```
tofu init
tofu plan
tofu apply
```

24. Observe the inventory and the configuration.yml playbooks.

```cat inventory.yml```

25. View inventory structure

```ansible-inventory -i inventory.yml --graph```

26. Test ansible's configuration and connectivity to the webserver

```ansible -i inventory.yml webservers -m ping```

27. Run the configuration playbook.

```ansible-playbook -i inventory.yml configuration.yml```

28. Try to access the web server from your browser.  (It shouldn't work, ...prove it.)

29. Add a new `security_rule` to the `azurerm_network_security_group` in this project to allow HTTP access to the server and apply the change with the tools. Include the code changes and proof that it works.

30. Use tofu to plan and apply changes.

31. Improve SSH security by limiting access to Ohio University’s IP ranges (`132.235.0.0/16` and `64.247.64.0/18`) and an address space for your residence.

32. If you are **not** attempting Task 6, clean up your lab space.

```tofu destroy```

# Task 6 - Extending the network (Grad students - Undergrad bonus)

33. Enhance the automation scripts to install PHP on the web server and demonstrate success with a simple `index.php` “hello world” file.

34. Now cleanup for real.

```tofu destroy```
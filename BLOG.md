# Custom Ansible Collection Demo in Ansible Automation Platform: A Comprehensive Tutorial

In the ever-evolving landscape of IT automation, Ansible has emerged as a powerful and flexible tool. As organizations grow and their automation needs become more complex, the ability to create, manage, and distribute custom automation content becomes crucial. This is where Ansible collections come into play. In this tutorial, we'll walk through a comprehensive demo of creating, deploying, and using a custom Ansible collection within Ansible Automation Platform (AAP).

## Understanding the Basics

Before we dive into the demo, let's clarify some key concepts:

- **Ansible Collection**: A collection is a format for distributing Ansible content. It can contain playbooks, roles, modules, and plugins.
- **Ansible Automation Platform (AAP)**: A suite of tools provided by Red Hat that helps scale IT automation, manage complex deployments, and speed productivity.
- **Private Automation Hub**: A central repository for Ansible content within your organization, provided as part of AAP.
- **Execution Environment (EE)**: A containerized environment that includes Ansible and all necessary dependencies to run your automation.

## Prerequisites

To follow this tutorial, you'll need:

1. Access to an Ansible Automation Platform installation, including a Private Automation Hub
2. Ansible installed on your local machine
3. Ansible Galaxy (typically installed with Ansible)
4. Ansible Builder (`pip install ansible-builder`)
5. Podman (or Docker) for container operations
6. Git for version control

Ensure all these tools are properly installed and configured before proceeding.

## Our Demo Workflow

We'll cover the full lifecycle of a custom Ansible collection:

1. Creating and building the collection
2. Pushing it to a Private Automation Hub
3. Creating a custom Execution Environment
4. Configuring and running a playbook in AAP
5. Updating the collection and related artifacts

Let's dive into each step!

### Step 1: Create and Build a Custom Ansible Collection

In this step, we'll create a simple collection named `demo.demo_collection`. This collection will include a basic role that sets and displays a demo message.

1. Open your terminal and navigate to your workspace directory.

2. Create the collection structure:
   ```bash
   ansible-galaxy collection init demo.demo_collection
   cd demo/demo_collection
   ```
   This command creates a directory structure for your new collection.

3. Create a simple role within your collection:
   ```bash
   ansible-galaxy role init roles/demo_role
   ```
   This creates a new role named `demo_role` with a standard directory structure.

4. Edit the main tasks file of your role. Open `roles/demo_role/tasks/main.yml` in your text editor and add the following content:
   ```yaml
   ---
   - name: Set demo message
     ansible.builtin.set_fact:
       demo_message: "Hello from the demo_role in demo.demo_collection!"

   - name: Display demo message
     ansible.builtin.debug:
       msg: "{{ demo_message }}"
   ```
   This simple role sets a fact (variable) with a message and then displays it.

5. Edit the `galaxy.yml` file in the root of your collection directory. Update the metadata (version, author, description, etc.) to reflect your collection's information.

6. Build the collection:
   ```bash
   ansible-galaxy collection build --output-path ../../
   ```
   This command builds your collection and creates a tarball in the parent directory.

### Step 2: Push to Private Automation Hub

Now that we have our collection built, we need to publish it to our Private Automation Hub. This makes the collection available for use within our organization.

1. Obtain a Private Automation Hub API token:
   - Log in to your Private Automation Hub web interface
   - Navigate to 'Collections' > 'API Token'
   - Generate a new token

2. Create an `ansible.cfg` file in your working directory with the following content:
   ```ini
   [galaxy]
   server_list = staging

   [galaxy_server.staging]
   url=https://<your-private-automation-hub-url>/api/galaxy/content/staging/
   token=<your-private-automation-hub-token>
   ```
   Replace `<your-private-automation-hub-url>` and `<your-private-automation-hub-token>` with your actual values.

3. Publish the collection to the staging repository:
   ```bash
   ansible-galaxy collection publish ./demo-demo_collection-1.0.2.tar.gz --server staging
   ```

4. Approval Process:
   - Log in to the Private Automation Hub web interface
   - Navigate to 'Collections' > 'Staging'
   - Find your uploaded collection
   - Click on the collection to view details
   - Click 'Approve' to move the collection from staging to production
   - Provide a reason for approval if prompted

### Step 3: Create a Custom Execution Environment

An Execution Environment is a containerized environment that includes Ansible and all necessary dependencies. We'll create a custom EE that includes our new collection.

1. Create a new directory for the execution environment:
   ```bash
   mkdir custom-ee && cd custom-ee
   ```

2. Create `execution-environment.yml` with the following content:
   ```yaml
   ---
   version: 3

   build_arg_defaults:
     ANSIBLE_GALAXY_CLI_COLLECTION_OPTS: '-vvvv'

   dependencies:
     galaxy: requirements.yml
     python: requirements.txt
     system: bindep.txt

   images:
     base_image: 
       name: registry.redhat.io/ansible-automation-platform-24/ee-minimal-rhel8:latest

   options:
     package_manager_path: /usr/bin/microdnf

   additional_build_steps:
     prepend_galaxy:
       - ARG PAH_TOKEN
       - ENV ANSIBLE_GALAXY_SERVER_LIST=pah,community
       - ENV ANSIBLE_GALAXY_SERVER_PAH_URL=https://hub.sandbox342.opentlc.com/api/galaxy/
       - ENV ANSIBLE_GALAXY_SERVER_PAH_TOKEN=${PAH_TOKEN}
       - ENV ANSIBLE_GALAXY_SERVER_COMMUNITY_URL=https://galaxy.ansible.com
   ```
   This configuration file defines how your Execution Environment will be built.

3. Create `requirements.yml`:
   ```yaml
   ---
   collections:
     - name: demo.demo_collection
       version: 1.0.2
   ```
   This file specifies which collections should be included in your EE.

4. Create empty `requirements.txt` and `bindep.txt` files for future use:
   ```bash
   touch requirements.txt bindep.txt
   ```

5. Build the execution environment:
   ```bash
   export PAH_TOKEN=<Your PAH token>
   ansible-builder build -t custom-ee:latest --container-runtime podman --build-arg=PAH_TOKEN=$PAH_TOKEN
   ```
   This command builds your custom EE as a container image.

6. Push the execution environment to your Private Automation Hub:
   ```bash
   podman login <your-private-automation-hub-url>
   podman tag custom-ee:latest <your-private-automation-hub-url>/custom-ee:latest
   podman push <your-private-automation-hub-url>/custom-ee:latest
   ```
   Replace `<your-private-automation-hub-url>` with your actual Private Automation Hub URL.

### Step 4: Configure and Run in Ansible Automation Platform

Now we'll set up and run our automation in Ansible Automation Platform.

1. Log in to your Ansible Automation Platform web interface.

2. Add the Execution Environment:
   - Navigate to 'Administration' > 'Execution Environments'
   - Click 'Add'
   - Fill in the details:
     - Name: "Custom Demo EE"
     - Image: "<your-private-automation-hub-url>/custom-ee:latest"
     - Pull: "Always pull container before running"
   - If authentication is required for pulling images, create and select a Container Registry credential
   - Click 'Save'

3. Create a new Project:
   - Go to 'Resources' > 'Projects' > 'Add'
   - Name: "Demo Collection Project"
   - SCM Type: Git
   - SCM URL: (Your Git repository URL containing the playbook)
   - Update on Launch: Yes
   - Click 'Save'

4. Create a new Inventory:
   - Go to 'Resources' > 'Inventories' > 'Add'
   - Name: "Demo Inventory"
   - Click 'Save'
   - Add a host (e.g., localhost) in the 'Hosts' tab

5. Create a new Job Template:
   - Go to 'Resources' > 'Templates' > 'Add' > 'Job Template'
   - Name: "Run Demo Collection"
   - Inventory: "Demo Inventory"
   - Project: "Demo Collection Project"
   - Playbook: Select your playbook (e.g., `playbook/main.yml`)
   - Credentials: Select appropriate credentials
   - Execution Environment: Select "Custom Demo EE"
   - Click 'Save'

6. Run the Job Template:
   - Go to the Job Template you created
   - Click "Launch"
   - Monitor the job output

### Step 5: Update the Custom Ansible Collection

As your automation needs evolve, you may need to update your collection. Here's how to do that:

1. Make changes to your collection (e.g., update the role, add new roles)

2. Update the version in `galaxy.yml`

3. Rebuild the collection:
   ```bash
   ansible-galaxy collection build --output-path ../../
   ```

4. Publish the new version to the staging repository:
   ```bash
   ansible-galaxy collection publish ./demo-demo_collection-1.0.3.tar.gz --server staging
   ```

5. Go through the approval process in Private Automation Hub as described in Step 2.

6. Update the Execution Environment:
   - Update `requirements.yml` with the new version
   - Rebuild and push the execution environment as in Step 3

7. Update the Job Template in Ansible Automation Platform:
   - Go to the Job Template
   - Update the Execution Environment to the new version
   - Save changes

8. Run the updated Job Template to verify the changes

## Conclusion

By following this tutorial, you've learned how to create a custom Ansible collection, publish it to a Private Automation Hub, create a custom Execution Environment, and use it all within Ansible Automation Platform. This workflow enables you to create, manage, and deploy organization-specific automation content at scale.

Remember, the key to successful automation at scale is not just in writing good playbooks, but in how you organize, distribute, and manage your automation content over time. Custom collections and the workflow demonstrated here provide a solid foundation for achieving these goals.

As you become more comfortable with these concepts, you can expand your collections to include more complex roles, custom modules, and plugins. You can also start to think about how to structure your automation content to best serve your organization's needs.

Happy automating!
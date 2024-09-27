# Custom Ansible Collection Demo in Ansible Automation Platform

This guide provides detailed instructions for creating, deploying, and using a custom Ansible collection within Ansible Automation Platform.

## Purpose of this Demo

This demo serves as a guide to working with custom Ansible collections in the context of Ansible Automation Platform. It demonstrates the full lifecycle of a custom collection, including:

1. Creating and building a custom Ansible collection
2. Pushing the custom collection to an Ansible Automation Platform Private Automation Hub
3. Creating an Ansible Execution Environment that includes this custom collection
4. Configuring and running an Ansible playbook in Ansible Automation Platform to demonstrate the usage of this custom collection
5. Updating the custom Ansible collection, pushing the changes to the Private Automation Hub, and updating the related artifacts

By following this demo, you'll gain practical experience with key concepts in enterprise-scale Ansible automation, including custom content creation, private content management, and execution environment customization.

## Prerequisites

Before starting this demo, ensure you have the following:

1. **Access to Ansible Automation Platform**: You need access to an Ansible Automation Platform installation, including a Private Automation Hub.

2. **Ansible**: Ensure Ansible is installed on your local machine or control node. [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

3. **Ansible Galaxy**: This is typically installed with Ansible.

4. **Ansible Builder**: This tool is used for building Execution Environments. Install it using pip:
   ```
   pip install ansible-builder
   ```

5. **Podman**: This demo uses Podman for container operations. [Podman Installation Guide](https://podman.io/getting-started/installation)

   Note: While this demo uses Podman, Docker can be used as an alternative. Podman and Docker are largely interchangeable for the purposes of this demo. We've chosen Podman for this demonstration, but if you prefer to use Docker, you can replace `podman` commands with `docker` in most cases.

6. **Git**: For version control and interacting with your project repository.

Ensure all these tools are properly installed and configured before proceeding with the demo.

## Demo Workflow

1. [Create and Build a Custom Ansible Collection](#step-1-create-and-build-a-custom-ansible-collection)
2. [Push the Custom Ansible Collection to Private Automation Hub](#step-2-push-the-custom-ansible-collection-to-private-automation-hub)
3. [Create an Ansible Execution Environment Including the Custom Collection](#step-3-create-an-ansible-execution-environment-including-the-custom-collection)
4. [Configure and Run the Ansible Playbook in Ansible Automation Platform](#step-4-configure-and-run-the-ansible-playbook-in-ansible-automation-platform)
5. [Update the Custom Ansible Collection](#step-5-update-the-custom-ansible-collection)

## Step 1: Create and Build a Custom Ansible Collection

1. Create the collection structure:
   ```
   ansible-galaxy collection init demo.demo_collection
   cd demo/demo_collection
   ```

2. Create a simple role:
   ```
   ansible-galaxy role init roles/demo_role
   ```

3. Edit `roles/demo_role/tasks/main.yml`:
   ```yaml
   ---
   - name: Set demo message
     ansible.builtin.set_fact:
       demo_message: "Hello from the demo_role in demo.demo_collection!"

   - name: Display demo message
     ansible.builtin.debug:
       msg: "{{ demo_message }}"
   ```

4. Edit `galaxy.yml` to update metadata (version, author, description, etc.)

5. Build the collection:
   ```
   ansible-galaxy collection build --output-path ../../
   ```

## Step 2: Push the Custom Ansible Collection to Private Automation Hub

1. Obtain a Private Automation Hub API token:
   - Log in to your Private Automation Hub web interface
   - Go to 'Collections' > 'API Token' and generate a new token

2. Create an `ansible.cfg` file:
   ```ini
   [galaxy]
   server_list = staging

   [galaxy_server.staging]
   url=https://<your-private-automation-hub-url>/api/galaxy/content/staging/
   token=<your-private-automation-hub-token>
   ```

3. Publish the collection to the staging repository:
   ```
   ansible-galaxy collection publish ./demo-demo_collection-1.0.2.tar.gz --server staging
   ```

4. Approval Process:
   - Log in to the Private Automation Hub web interface
   - Navigate to 'Collections' > 'Staging'
   - Find your uploaded collection
   - Click on the collection to view details
   - In the collection detail view, you should see an 'Approve' button
   - Click 'Approve' to move the collection from staging to production
   - You may need to provide a reason for approval or additional metadata
   - Once approved, the collection will be available in the published repository

## Step 3: Create an Ansible Execution Environment Including the Custom Collection

1. Create a new directory for the execution environment:
   ```
   mkdir custom-ee && cd custom-ee
   ```

2. Create `execution-environment.yml` using version 3:
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
       # SEE: https://ansible.readthedocs.io/projects/builder/en/latest/scenario_guides/scenario_custom/
       - ARG PAH_TOKEN
       - ENV ANSIBLE_GALAXY_SERVER_LIST=pah,community
       - ENV ANSIBLE_GALAXY_SERVER_PAH_URL=https://hub.sandbox342.opentlc.com/api/galaxy/
       - ENV ANSIBLE_GALAXY_SERVER_PAH_TOKEN=${PAH_TOKEN}
       - ENV ANSIBLE_GALAXY_SERVER_COMMUNITY_URL=https://galaxy.ansible.com
   ```

3. Create `requirements.yml`:
   ```yaml
   ---
   collections:
     - name: demo.demo_collection
       version: 1.0.2
   ```

4. Create `requirements.txt` (even if empty, for future use):
   ```
   # Add any Python package dependencies here
   ```

5. Create `bindep.txt` (even if empty, for future use):
   ```
   # Add any system package dependencies here
   ```

6. Build the execution environment:
   ```
   export PAH_TOKEN=<Your PAH token>
   ansible-builder build -t custom-ee:latest --container-runtime podman --build-arg=PAH_TOKEN=$PAH_TOKEN
   ```

7. Push the execution environment to your Private Automation Hub:
   - First, ensure you're logged in to your Private Automation Hub's registry:
     ```
     podman login <your-private-automation-hub-url>
     ```
   - Tag your image for the Private Automation Hub:
     ```
     podman tag custom-ee:latest <your-private-automation-hub-url>/custom-ee:latest
     ```
   - Push the image:
     ```
     podman push <your-private-automation-hub-url>/custom-ee:latest
     ```

## Step 4: Configure and Run the Ansible Playbook in Ansible Automation Platform

1. Log in to your Ansible Automation Platform web interface.

2. Add the Execution Environment from Private Automation Hub to the Controller:
   - Navigate to 'Administration' > 'Execution Environments'
   - Click 'Add'
   - Fill in the following details:
     - Name: "Custom Demo EE"
     - Image: "<your-private-automation-hub-url>/custom-ee:latest"
     - Pull: "Always pull container before running"
   - If your Private Automation Hub requires authentication for pulling images:
     - Go to 'Resources' > 'Credentials'
     - Add a new Credential of type 'Container Registry'
     - Fill in the details for your Private Automation Hub registry
     - In the Execution Environment configuration, select this new credential in the 'Container Registry Credential' field
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
   - Go to the 'Hosts' tab of this inventory and add a host (e.g., localhost)

5. Create a new Credential (if needed):
   - Go to 'Resources' > 'Credentials' > 'Add'
   - Select the appropriate credential type and fill in the details
   - Click 'Save'

6. Create a new Job Template:
   - Go to 'Resources' > 'Templates' > 'Add' > 'Job Template'
   - Name: "Run Demo Collection"
   - Inventory: "Demo Inventory"
   - Project: "Demo Collection Project"
   - Playbook: Select your playbook (e.g., `playbook/main.yml`)
   - Credentials: Select the credential you created
   - Execution Environment: Select "Custom Demo EE" (the one you added in step 2)
   - Click 'Save'

7. Run the Job Template:
   - Go to the Job Template you created
   - Click "Launch"
   - Monitor the job output to ensure it runs successfully

8. Troubleshooting:
   - If the job fails due to missing collections or modules, ensure that your custom collection is correctly included in the Execution Environment
   - Check the job's "Details" tab for any error messages
   - Verify that the Execution Environment is being pulled correctly from Private Automation Hub

## Step 5: Update the Custom Ansible Collection

1. Make changes to your collection (e.g., update the role, add new roles)

2. Update the version in `galaxy.yml`

3. Rebuild the collection:
   ```
   ansible-galaxy collection build --output-path ../../
   ```

4. Publish the new version to the staging repository:
   ```
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


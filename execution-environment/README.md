# Custom Execution Environment for demo.demo_collection

This directory contains the necessary files to build a custom Execution Environment (EE) for use with the `demo.demo_collection` in Ansible Automation Platform.

## Contents

- [Custom Execution Environment for demo.demo_collection](#custom-execution-environment-for-demodemo_collection)
  - [Contents](#contents)
  - [Files](#files)
  - [Building the Execution Environment](#building-the-execution-environment)
  - [Pushing to Private Automation Hub](#pushing-to-private-automation-hub)
  - [Using the Execution Environment](#using-the-execution-environment)
  - [Customizing the Execution Environment](#customizing-the-execution-environment)

## Files

- `execution-environment.yml`: The main configuration file for the Execution Environment.
- `requirements.yml`: Specifies the collections to be included in the EE.
- `requirements.txt`: Lists Python package dependencies (if any).
- `bindep.txt`: Lists system-level dependencies (if any).

## Building the Execution Environment

To build the Execution Environment, follow these steps:

1. Ensure you have `ansible-builder` installed:
   ```
   pip install ansible-builder
   ```

2. Set your Private Automation Hub token as an environment variable:
   ```
   export PAH_TOKEN=<Your PAH token>
   ```

3. Run the build command:
   ```
   ansible-builder build -t custom-ee:latest --container-runtime podman --build-arg=PAH_TOKEN=$PAH_TOKEN
   ```

This will create a container image named `custom-ee:latest`.

## Pushing to Private Automation Hub

After building, push the Execution Environment to your Private Automation Hub:

1. Log in to your Private Automation Hub:
   ```
   podman login <your-private-automation-hub-url>
   ```

2. Tag the image:
   ```
   podman tag custom-ee:latest <your-private-automation-hub-url>/custom-ee:latest
   ```

3. Push the image:
   ```
   podman push <your-private-automation-hub-url>/custom-ee:latest
   ```

## Using the Execution Environment

To use this Execution Environment in Ansible Automation Platform:

1. In the Automation Platform web interface, go to "Administration" > "Execution Environments".
2. Click "Add" and fill in the details:
   - Name: "Custom Demo EE"
   - Image: "<your-private-automation-hub-url>/custom-ee:latest"
   - Pull Policy: "Always pull container before running"
3. If authentication is required, create a Container Registry credential and associate it with this Execution Environment.
4. In your Job Templates, select "Custom Demo EE" as the Execution Environment.

## Customizing the Execution Environment

To customize this Execution Environment:

1. Modify `requirements.yml` to add or remove collections.
2. Edit `requirements.txt` to change Python dependencies.
3. Update `bindep.txt` for system-level dependencies.
4. Adjust `execution-environment.yml` for structural changes to the EE.

After making changes, rebuild and push the Execution Environment as described above.
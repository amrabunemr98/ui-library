<div align="center">
  <h1 style="color: red;"> CI/CD Automation and Deployment of Internal React UI Library </h1>
</div>

## :star2: Overview:-
- This document outlines the automation strategy for developing and deploying an internal React UI library using GitHub Actions for CI/CD and a private npm registry for secure distribution. The library is built using TypeScript and is designed to provide reusable UI components for internal projects, ensuring consistency and ease of integration across various applications.
To address challenges such as synchronizing updates and avoiding issues like broken components or missing dependencies, this guide provides a comprehensive, step-by-step approach for automating the CI/CD pipeline, version control, and library publishing. The CI/CD pipeline is configured to run on a RedHat environment using Node.js version 18.6.0, leveraging GitHub Actions with a self-hosted runner for optimal performance and control.

## 🎯 Objective:-
- The objective of this document is to define a robust CI/CD pipeline for the development and deployment of an internal React UI library using GitHub Actions. This strategy automates build and deployment processes while securely publishing the library to a private npm registry. By doing so, it ensures efficient version control, consistency across projects, and reduces manual intervention, ultimately streamlining the development lifecycle and minimizing integration issues.

## :gear: Requirements:-
- Github
- Github Actions
- Operating System: RedHat Enterprise Linux
- npm Registry

## :scroll: Project Structure:-
- **GitHub**: Version control and repository management for source code and collaboration.  
- **GitHub Actions**: Automate CI/CD pipeline for build and deployment processes.  
- **Operating System: RedHat Enterprise Linux**: Environment for running the CI/CD pipeline.  
- **npm Registry**: Secure storage and distribution of the React UI library.

## :diamond_shape_with_a_dot_inside: Quick Start Guide:-
### Step 1:Create virtual machine on any cloud provider or using VMware Worksation
- **Operating System**: RedHat Enterprise Linux 9 (RHEL 9)
- **CPU**: Minimum 2 CPU cores
- **RAM**: Minimum 4 GB of RAM
- **Storage**: Minimum 20 GB free space for the runner and Node.js installations

### Step 2: Install Required Dependencies on RHEL
- If you want to run the GitHub Actions workflow on RedHat instead of Ubuntu, you'll need to make some changes to ensure compatibility. Currently, GitHub Actions does not provide a pre-built runner specifically for RedHat distributions. However, you can still achieve this by using self-hosted runners.
  
### What is a Self-Hosted Runner?
- A **self-hosted runner** allows you to run GitHub Actions on your own infrastructure, including RedHat. Instead of using GitHub's cloud runners (like `ubuntu-latest`), the job will be executed on your RedHat server. This setup gives you full control over the environment, but you'll need to configure and maintain it.

### Steps to Use GitHub Actions with RedHat

1. **Set Up a Self-Hosted Runner** on RedHat.
2. **Register the Self-Hosted Runner** with your GitHub Repository.
3. **Modify the Workflow Configuration** to use the self-hosted runner.

Let's go through these steps in detail:

---
Before you install the self-hosted runner, you’ll need to ensure that your RHEL 9 environment has the necessary dependencies:

1. **Update the System**:
   - First, make sure your RHEL system is up-to-date:
     ```bash
     sudo dnf update -y
     ```
2. **Install Node.js**:
   - If you don't have Node.js installed, set it up using the NodeSource repository or use the Node.js module from RHEL’s AppStream repository:
     ```bash
     curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
     sudo dnf install -y nodejs-18.6.0
     ```
     - Verify the Node.js installation:
     ```bash
     node -v
     npm -v
     ```
3. **Install Required Tools and Libraries**:
   - Install the following dependencies to ensure compatibility with the runner:
     ```bash
     sudo dnf install -y gcc gcc-c++ libicu libicu-devel curl libcurl libcurl-devel libunwind libunwind-devel tar
     sudo dnf install -y libicu
     ```

4. **Install additional dependencies for .NET Core 6.0:**
   If there are other dependencies required by your .NET project, you can install them using:

   ```bash
   sudo dnf install -y dotnet-runtime-6.0
   ```

### Step 3: Download and Configure the GitHub Actions Runner
1. **Create a Directory for the Runner**:
   - Create a directory to install the GitHub Actions runner:
     ```bash
     mkdir -p ~/actions-runner && cd ~/actions-runner
     ```
2. **Download the GitHub Actions Runner**:
   - Download the latest version of the GitHub Actions runner for Linux:
     ```bash
     curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
     ```
   - Extract the runner:
     ```bash
     tar xzf ./actions-runner-linux-x64.tar.gz
     ```
   ![Screenshot from 2024-09-29 03-38-42](https://github.com/user-attachments/assets/cc08fe14-b7e1-4c4e-8723-9710efb200a5)

3. **Configure the Runner**:
   - Get the registration token from your GitHub repository:
     - Go to **Settings** > **Actions** > **Runners** and click on **New self-hosted runner**.
     - Copy the generated registration token.
   - Configure the runner using the token:
     ```bash
     ./config.sh --url https://github.com/your-username/your-repo --token YOUR_GITHUB_RUNNER_TOKEN
     ```
### Step 4: Start the Runner on RHEL

1. **Run the GitHub Actions Runner**:
   - You can start the runner interactively using:
     ```bash
     ./run.sh
     ```
   - Alternatively, you can set it up as a service to run in the background:
     ```bash
     sudo ./svc.sh install
     sudo ./svc.sh start
     ```
   ![Screenshot from 2024-09-29 03-41-56](https://github.com/user-attachments/assets/4e1a436d-9783-4e35-b0f7-7b922cd3a6fa)

2. **Verify the Runner is Active**:
   - Go to your GitHub repository’s **Settings** > **Actions** > **Runners**.
   - You should see your new self-hosted runner listed as **Idle** or **Active**.
   ![Screenshot from 2024-09-29 03-39-11](https://github.com/user-attachments/assets/4a265afd-561c-4778-8e63-209491bf2e2b)
   
### Step 5: Modify the GitHub Actions Workflow to Use the Self-Hosted Runner
Modify the `ci.yml` file in your `.github/workflows` directory as follows:

```yaml
name: CI/CD Pipeline on RHEL

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: self-hosted  # Ensure this matches your runner's label

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.6.0'  # Use the desired Node.js version

      - name: Remove package-lock.json (if it exists)
        run: |
          if [ -f package-lock.json ]; then
            rm package-lock.json
          fi

      - name: Install dependencies
        run: npm install  # This will regenerate package-lock.json

      - name: Build the project
        run: npm run build

      - name: Set package version
        run: npm version 1.0.${{ github.run_number }}


      # Optional: Publish the library to npm
      - name: Publish to npm
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        run: npm publish --access public
        env:
          GITHUB_TOKEN: ${{ secrets.G_TOKEN }}  
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}    
```
### Step 6: Create Tokens
1. **Create Github Token**
![Screenshot from 2024-01-31 14-46-59_Original](https://github.com/amrabunemr98/Multi-Cloud-DevOps-Project/assets/128842547/5112996c-1e49-413b-b217-294dc9f76210)

2. **Create npm registry Token**
![Screenshot from 2024-09-29 03-26-45](https://github.com/user-attachments/assets/6c346fd3-a7a2-4639-8f9d-267c5cd3b7d2)

3. **Creating an Organization in GitHub and Configuring Tokens**

  To securely access GitHub and publish versions to the npm registry within your CI/CD pipeline, you need to create an organization in GitHub and configure the necessary tokens.
  
  - **Create a GitHub Organization**:  
     Set up a GitHub organization to manage your repositories and collaborators. This allows for better access control and organization of your projects.
  
  - **Add GitHub Token**:  
     Within your organization settings, navigate to the "Secrets" section to create a new secret. Name it `G_TOKEN` and input your GitHub personal access token. This token will allow your CI/CD pipeline to interact with the GitHub API, facilitating actions such as accessing repositories and managing workflows.
  
  - **Add npm Registry Token**:  
     Similarly, create another secret in the "Secrets" section and name it `NPM_TOKEN`. Enter your npm access token, which will be used by the CI/CD pipeline to publish the React UI library securely to the npm registry.
    ![Screenshot from 2024-09-29 03-28-29](https://github.com/user-attachments/assets/8d61e21a-8106-4bb5-b052-de47946341d4)

    ![Screenshot from 2024-09-29 03-27-41](https://github.com/user-attachments/assets/6eb19c0a-55f7-4647-93d5-a91850e4a628)

- **Utilize Tokens in `ci.yml`**:  
   In your GitHub Actions workflow file (`ci.yml`), reference these tokens under the `env` section:
   ```yaml
   env:
     GITHUB_TOKEN: ${{ secrets.G_TOKEN }}  
     NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
   ```
 ### Step 7: Logging into the npm Registry
- To publish and manage packages in the npm registry, you need to authenticate your npm client.
- Launch your terminal or command prompt where you want to execute the npm commands.
- Use the following command to initiate the login process. The `--auth-type=legacy` flag ensures that you use the legacy authentication method, which may be necessary for certain npm registries.
   ```bash
   npm login --auth-type=legacy
   ```
- After running the command, you will be prompted to enter your npm credentials:
   - **Username**: Your npm username.
   - **Password**: Your npm password.
   - **Email**: The email associated with your npm account.
  
   > [!IMPORTANT]
   > Make sure to enter the correct credentials.
  
   ![Screenshot from 2024-09-29 03-30-38](https://github.com/user-attachments/assets/4666f391-0d56-44d7-b21e-533ee752a442)
   ![Screenshot from 2024-09-29 03-32-45](https://github.com/user-attachments/assets/3a73e877-29ca-4810-9816-b09ad398a207)

---
### 📈 Results

When a developer uploads changes to the GitHub repository for the UI library, the following automated processes are triggered:

1. **GitHub Actions Workflow**: Any commit to the main branch automatically triggers the GitHub Actions workflow defined in the `ci.yml` file.

2. **Dependency Installation**: The workflow begins by installing all necessary dependencies to ensure that the project is ready for building.

3. **Project Build**: After successful installation, the project is built, preparing the React UI library for distribution.

4. **Library Publishing**: Finally, the built library is published to the npm registry, making it available for use in internal applications.

![Screenshot from 2024-09-29 03-34-11](https://github.com/user-attachments/assets/aff16d38-6ff4-43dd-a7df-81b5feca3283)
![Screenshot from 2024-09-29 03-35-17](https://github.com/user-attachments/assets/11c82606-f729-4336-a1c4-255c27a2da7a)
![Screenshot from 2024-09-29 03-35-48](https://github.com/user-attachments/assets/1762b870-1fa7-4006-b217-b8664ad2cbd4)

## :rocket: Conclusion:-
- In conclusion, this document outlines a streamlined approach for automating the development and deployment of an internal React UI library using GitHub Actions and a private npm registry. By implementing a self-hosted runner on RedHat Enterprise Linux, the CI/CD pipeline efficiently installs dependencies, builds the project, and publishes the library automatically upon code changes. This automation reduces manual errors, ensures consistency, and enhances collaboration among developers, making the latest versions of the UI library readily available for internal applications.

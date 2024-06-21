# Landscape Lab 1

## Setup Landscape Server

### Minimum Requirements for Landscape Server

Refer to the official installation guide for Landscape: [Ubuntu Landscape Install Guide](https://ubuntu.com/landscape/install)

**Supported Operating Systems:**
- Ubuntu 22.04 LTS (Jammy Jellyfish)
- Ubuntu 24.04 LTS (Noble Numbat) (AWS Marketplace)

**Hardware Requirements:**
- Dual core 2 GHz processor
- 4 GB of RAM
- 30 GB of disk space (recommended instance type: t2.xlarge)

**Networking Requirements:**
- TCP communication allowed for:
  - SSH (typically port 22)
  - HTTP (port 80)
  - HTTPS (port 443)
  - gRPC (port 6554)

## Install Landscape Server

1. **Set the hostname for the Landscape server:**
    ```sh
    sudo hostnamectl set-hostname landscape-server
    ```

    ## Add repo
   ```
   sudo add-apt-repository -y ppa:landscape/self-hosted-24.04
   ```

3. **Update the system and install the Landscape server:**
    ```sh
    sudo apt update && sudo apt-get install -y landscape-server-quickstart
    ```
    - During the installation, when prompted for configuration options, select "no configuration".

4. **Connect to the Landscape server from your browser:**
    - Open your browser and navigate to `https://<public-ip-address>`
    - You might encounter a security warning (processed unsafe). Proceed to continue.

5. **Complete the setup in the browser:**
    - **Name:** YourName
    - **Email:** abc@gmail.com
    - **Passphrase:** Password@123


# Landscape Lab 2

## Setup Landscape Client on a New EC2 Instance

### Step 1: Launch a New EC2 Instance

1. **Instance Type:**
   - t2.micro

2. **Operating System:**
   - Ubuntu 24.04 LTS (Noble Numbat)

### Step 2: Install Landscape Client

1. **Update Package List and Install Landscape Client:**
   ```sh
   sudo apt update && sudo apt install -y landscape-client
   ```
   - `sudo apt update`: Updates the list of available packages and their versions.
   - `sudo apt install -y landscape-client`: Installs the `landscape-client` package without prompting for confirmation (`-y` flag).

### Step 3: Copy the Landscape Server Certificate to the Client

1. **On the Landscape Server:**
   - Open the certificate file with a text editor:
     ```sh
     sudo vi /etc/ssl/certs/landscape_server_ca.crt
     ```
     - `vi /etc/ssl/certs/landscape_server_ca.crt`: Opens the file in the `vi` editor to view and copy the contents.

2. **On the Landscape Client:**
   - Create and paste the certificate into a new file:
     ```sh
     sudo vi landscape_server_ca.crt
     ```
     - `vi landscape_server_ca.crt`: Opens a new file in the `vi` editor where you will paste the copied certificate contents.

### Step 4: Move and Set Permissions for the Certificate

1. **Set the ownership of the certificate file:**
   ```sh
   sudo chown root:root landscape_server_ca.crt
   ```
   - `sudo chown root:root landscape_server_ca.crt`: Changes the file owner and group to `root`.

2. **Move the certificate to the appropriate directory:**
   ```sh
   sudo mv landscape_server_ca.crt /etc/landscape/
   ```
   - `sudo mv landscape_server_ca.crt /etc/landscape/`: Moves the certificate file to the `/etc/landscape/` directory.

### Step 5: Update the Client Configuration

1. **Edit the client configuration file:**
   ```sh
   sudo vi /etc/landscape/client.conf
   ```
   - `sudo vi /etc/landscape/client.conf`: Opens the client configuration file in the `vi` editor.

2. **Add the following line at the end of the file:**
   ```
   ssl_public_key = /etc/landscape/landscape_server_ca.crt
   ```
   - This line specifies the path to the SSL public key.

### Step 6: Configure the Landscape Client

1. **Run the Landscape Client Configuration Command:**
   ```sh
   sudo landscape-config --silent --account-name='standalone' --computer-title="My Computer" --url https://<Landscape-server-privateIP>/message-system --ping-url https://<Landscape-server-privateIP>/ping --include-manager-plugins=ScriptExecution --script-users=ALL
   ```
   - `sudo landscape-config --silent`: Runs the Landscape configuration command without interactive prompts.
   - `--account-name='standalone'`: Specifies the account name.
   - `--computer-title="My Computer"`: Sets the title for the computer.
   - `--url https://<Landscape-server-privateIP>/message-system`: Specifies the URL for the message system on the Landscape server.
   - `--ping-url https://<Landscape-server-privateIP>/ping`: Specifies the URL for the ping system on the Landscape server.

### Step 7: Approve the Computer

1. **On the Landscape Server:**
   - Approve the new computer from the notification in the Landscape web interface to complete the setup.



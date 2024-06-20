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

2. **Update the system and install the Landscape server:**
    ```sh
    sudo apt update && sudo apt-get install -y landscape-server-quickstart
    ```
    - During the installation, when prompted for configuration options, select "no configuration".

3. **Connect to the Landscape server from your browser:**
    - Open your browser and navigate to `https://<public-ip-address>`
    - You might encounter a security warning (processed unsafe). Proceed to continue.

4. **Complete the setup in the browser:**
    - **Name:** Ninad Samudre
    - **Email:** abc@gmail.com
    - **Passphrase:** Password@123

5. **Approve the computer in the notifications:**
    - After logging in, you will receive a notification to approve the computer. Approve it to complete the setup.

## Notes

- Ensure that all networking requirements are properly configured to allow seamless access to the Landscape server.
- DNS configuration is crucial if you plan to use LetsEncrypt for SSL certification.
- It's recommended to use strong, unique credentials for security purposes.

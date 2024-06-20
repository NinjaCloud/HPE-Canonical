# Landscape API Important Commands Cheat Sheet

## General Commands

## Download the landscape-api from snap
```
sudo snap install landscape-api
```

### Getting the API Key
```sh
sudo landscape-api setup
```
- **Explanation:** Generates the API key needed to use the Landscape API.

### Listing Available Methods
```sh
landscape-api list-methods
```
- **Explanation:** Lists all available methods that can be called using the Landscape API.

## Account Management

### Listing Accounts
```sh
landscape-api get accounts
```
- **Explanation:** Retrieves a list of all accounts managed by the Landscape server.

### Creating an Account
```sh
landscape-api create account --name "Account Name" --contact-email "email@example.com"
```
- **Explanation:** Creates a new account with the specified name and contact email.

## Computer Management

### Listing Registered Computers
```sh
landscape-api get computers
```
- **Explanation:** Retrieves a list of all computers registered with the Landscape server.

### Getting Computer Details
```sh
landscape-api get computer --id <computer-id>
```
- **Explanation:** Retrieves detailed information about a specific computer identified by `<computer-id>`.

### Approving a Computer
```sh
landscape-api approve computer --id <computer-id>
```
- **Explanation:** Approves a computer identified by `<computer-id>` to be managed by the Landscape server.

### Deleting a Computer
```sh
landscape-api delete computer --id <computer-id>
```
- **Explanation:** Deletes a computer identified by `<computer-id>` from the Landscape server.

## Group Management

### Listing Groups
```sh
landscape-api get groups
```
- **Explanation:** Retrieves a list of all groups managed by the Landscape server.

### Creating a Group
```sh
landscape-api create group --name "Group Name"
```
- **Explanation:** Creates a new group with the specified name.

### Adding a Computer to a Group
```sh
landscape-api add-to-group --group-id <group-id> --computer-id <computer-id>
```
- **Explanation:** Adds a computer identified by `<computer-id>` to a group identified by `<group-id>`.

## Script Management

### Listing Scripts
```sh
landscape-api get scripts
```
- **Explanation:** Retrieves a list of all scripts available on the Landscape server.

### Running a Script
```sh
landscape-api run script --id <script-id> --computer-id <computer-id>
```
- **Explanation:** Runs a script identified by `<script-id>` on a computer identified by `<computer-id>`.

### Creating a Script
```sh
landscape-api create script --name "Script Name" --content "echo 'Hello, World!'"
```
- **Explanation:** Creates a new script with the specified name and content.

## Tag Management

### Listing Tags
```sh
landscape-api get tags
```
- **Explanation:** Retrieves a list of all tags used on the Landscape server.

### Creating a Tag
```sh
landscape-api create tag --name "Tag Name"
```
- **Explanation:** Creates a new tag with the specified name.

### Adding a Tag to a Computer
```sh
landscape-api tag computer --computer-id <computer-id> --tag-id <tag-id>
```
- **Explanation:** Adds a tag identified by `<tag-id>` to a computer identified by `<computer-id>`.

### Removing a Tag from a Computer
```sh
landscape-api untag computer --computer-id <computer-id> --tag-id <tag-id>
```
- **Explanation:** Removes a tag identified by `<tag-id>` from a computer identified by `<computer-id>`.


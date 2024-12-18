# Coding Standards for Proxmox VE Helper-Scripts

**Welcome to the Coding Standards Guide!** 
📜 This document outlines the essential coding standards for all our scripts and JSON files. Adhering to these standards ensures that our codebase remains consistent, readable, and maintainable. By following these guidelines, we can improve collaboration, reduce errors, and enhance the overall quality of our project.

### Why Coding Standards Matter

Coding standards are crucial for several reasons:

1. **Consistency**: Consistent code is easier to read, understand, and maintain. It helps new team members quickly get up to speed and reduces the learning curve.
2. **Readability**: Clear and well-structured code is easier to debug and extend. It allows developers to quickly identify and fix issues.
3. **Maintainability**: Code that follows a standard structure is easier to refactor and update. It ensures that changes can be made with minimal risk of introducing new bugs.
4. **Collaboration**: When everyone follows the same standards, it becomes easier to collaborate on code. It reduces friction and misunderstandings during code reviews and merges.

### Scope of This Document

This document covers the coding standards for the following types of files in our project:

- **`APP-install.sh` Scripts**: These scripts are responsible for the installation of applications and are located in the `/install` directory.
- **`APP.sh` Scripts**: These scripts handle the creation and updating of containers and are found in the `/ct` directory.
- **JSON Files**: These files store structured data and are located in the `/json` directory.

Each section provides detailed guidelines on various aspects of coding, including shebang usage, comments, variable naming, function naming, indentation, error handling, command substitution, quoting, script structure, and logging. Additionally, examples are provided to illustrate the application of these standards.

By following the coding standards outlined in this document, we ensure that our scripts and JSON files are of high quality, making our project more robust and easier to manage. Please refer to this guide whenever you create or update scripts and JSON files to maintain a high standard of code quality across the project. 📚🔍

Let's work together to keep our codebase clean, efficient, and maintainable! 💪🚀

---

# **APP<span></span>-install.sh Scripts**
 `APP-install.sh` scripts found in the `/install` directory. These scripts are responsible for the installation of the desired Application. For this guide we take `/install/snipeit-install.sh` as example.

## 1. **File Header**

### 1.1 **Shebang**
- Use `#!/usr/bin/env bash` as the shebang for portability across systems.

```bash
#!/usr/bin/env bash
```

### 1.2 **Comments**
- Add clear comments for script metadata, including author, copyright, and license information.
- Use meaningful inline comments to explain complex commands or logic.

Example:
```bash
# Copyright (c) 2021-2024 community-scripts ORG
# Author: [YourUserName]
# License: MIT
# Source: [SOURCE_URL]
```
### 1.3 **Variables and Function import**
- This sections adds the support for all needed functions and variables.
```bash
source /dev/stdin <<<"$FUNCTIONS_FILE_PATH"
color
verb_ip6
catch_errors
setting_up_container
network_check
update_os
```
---

## 2. **Variable Naming and Management**

### 2.1 **Naming Conventions**
- Use uppercase names for constants and environment variables.
- Use lowercase names for local script variables.

Example:
```bash
DB_NAME=snipeit_db    # Environment-like variable (constant)
DB_USER="snipeit"     # Local variable
```

### 2.2 **Avoid Hardcoding Values**
- Dynamically generate sensitive values, like passwords, using tools like `openssl`.

Example:
```bash
DB_PASS=$(openssl rand -base64 18 | tr -dc 'a-zA-Z0-9' | head -c13)
```

---

## 3. **Dependencies**

### 3.1 **Install all at once**
- Install all dependencies with a single command if possible

Example:
```bash
$STD apt-get install -y \
  curl \
  composer \
  git \
  sudo \
  mc \
  nginx 
```

### 3.2 **Collaps Dependencies**
- Collaps dependencies to keep the Code readable.

Example: <br>
Use
```bash
php8.2-{bcmath,common,ctype}
```
instead of
```bash
php8.2-bcmath php8.2-common php8.2-ctype
```
---
## 4. **Paths to applications**
- If possible install the App and all nessesery files in `/opt/`

## 5. **Version Management**

### 5.1 **Install the latest Release**
- Always try and install the latest Release if possibly
- Do not Hardcode any Version if not absolutly nessesery

Example for a git Release:
```bash
RELEASE=$(curl -s https://api.github.com/repos/snipe/snipe-it/releases/latest | grep "tag_name" | awk '{print substr($2, 3, length($2)-4) }')
wget -q "https://github.com/snipe/snipe-it/archive/refs/tags/v${RELEASE}.zip"
```
### 5.2 **Store the Version in a File for later Updates**
- Write the installed Version into a file.
- This is used for the Update function in app.sh to check if we need to update or not

Example:
```bash
echo "${RELEASE}" >"/opt/${APPLICATION}_version.txt"
```

## 6. **Input and Output Management**

### 6.1 **User Feedback**
- Use standard functions like `msg_info` and `msg_ok` to print status messages.
- Display meaningful progress messages at key stages.

Example:
```bash
msg_info "Installing Dependencies"
$STD apt-get install ...
msg_ok "Installed Dependencies"
```
### 6.2 **Verbosity**
- Use the appropiate flag (**-q** in the examples) for a command to suppres its output
Example:
```bash
wget -q
unzip -q
```
- If a command dose not come with such a functionality use `$STD` (a custom standard redirection variable) for managing output verbosity.

Example:
```bash
$STD apt-get install -y nginx
```


## 7. **String/File Manipulation**

### 7.1 **File Manipulation**
- Use `sed` to replace placeholder values in configuration files.

Example:
```bash
sed -i -e "s|^DB_DATABASE=.*|DB_DATABASE=$DB_NAME|" \
       -e "s|^DB_USERNAME=.*|DB_USERNAME=$DB_USER|" \
       -e "s|^DB_PASSWORD=.*|DB_PASSWORD=$DB_PASS|" .env
```

---

## 8. **Security Practices**

### 8.1 **Password Generation**
- Use secure tools (e.g., `openssl`) to generate random passwords.
- Use only Alphanumeric Values to not introduce unknown behaviour.

Example:
```bash
DB_PASS=$(openssl rand -base64 18 | tr -dc 'a-zA-Z0-9' | head -c13)
```

### 8.2 **File Permissions**
- Explicitly set secure ownership and permissions for sensitive files.

Example:
```bash
chown -R www-data: /opt/snipe-it
chmod -R 755 /opt/snipe-it
```

---

## 9. **Service Configuration**

### 9.1 **Configuration Files**
- Use `cat <<EOF` to write configuration files in a clean and readable way.

Example:
```bash
cat <<EOF >/etc/nginx/conf.d/snipeit.conf
server {
    listen 80;
    root /opt/snipe-it/public;
    index index.php;
}
EOF
```
### 9.2 **Credential Management**
- Store the generated credentials in a file
Example:
```bash
USERNAME=username
PASSWORD=$(openssl rand -base64 18 | tr -dc 'a-zA-Z0-9' | head -c13)
{
    echo "Application-Credentials"
    echo "Username: $USERNAME"
    echo "Password: $PASSWORD"
} >> ~/application.creds
```
### 9.3 **Enviromental Files**
- Use `cat <<EOF` to write enviromental files in a clean and readable way.
```bash
cat <<EOF >/path/to/.env
VARIABLE="value"
PORT=3000
DB_NAME="${DB_NAME}"
EOF
```

### 9.4 **Reload Services**
- Enable affected services after configuration changes and start it right away.

Example:
```bash
systemctl enable -q --now nginx
```

---

## 10. **Cleanup**

### 10.1 **Remove Temporary Files**
- Remove temporary files or unnecessary downloads after use.

Example:
```bash
rm -rf /opt/v${RELEASE}.zip
```

### 10.2 **Autoremove and Autoclean**
- Clean up unused dependencies to reduce disk space usage.

Example:
```bash
apt-get -y autoremove
apt-get -y autoclean
```

---

## 11. **Consistency and Style**

### 11.1 **Indentation**
- Use 2 spaces for indentation for better readability.

---

## 11. **Best Practices Checklist**

- [ ] Shebang is correctly set (`#!/usr/bin/env bash`).
- [ ] Metadata (author, license) is included at the top.
- [ ] Variables follow naming conventions.
- [ ] Sensitive values are dynamically generated.
- [ ] Files and services have proper permissions.
- [ ] Script cleans up temporary files.

---

### Example: High-Level Script Flow

1. **Dependencies Installation**
2. **Database Setup**
3. **Download and Configure Application**
4. **Service Configuration**
5. **Final Cleanup**

--- 

# **APP<span></span>.sh Scripts**
 `APP.sh` scripts found in the `/ct` directory. These scripts are responsible for the installation of the desired Application. For this guide we take `/ct/snipeit.sh` as example.


## 1. **File Header**

### 1.1 **Shebang**
- Use `#!/usr/bin/env bash` as the shebang for portability across systems.

```bash
#!/usr/bin/env bash
```
### 1.2 **Import Functions**
- Import the build.func File.
- When developing your own Script, change the link to your own repository.

> [!CAUTION]
> Before opening a Pull Request change the link to point to the community-scripts repo.

Example for development:
```bash
source <(curl -s https://raw.githubusercontent.com/[USER]/[REPO]/refs/heads/[BRANCH]/misc/build.func)
```

Example for final Script: 
```bash
source <(curl -s https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/misc/build.func)
```

### 1.3 **Metadata**
- Add clear comments for script metadata, including author, copyright, and license information.

Example:
```bash
# Copyright (c) 2021-2024 community-scripts ORG
# Author: [YourUserName]
# License: MIT | https://github.com/community-scripts/ProxmoxVE/raw/main/LICENSE
# Source: [SOURCE_URL]
```

## 2 **Variables and Function import**
> [!IMPORTANT]
> You need to have all this set in Your Script, otherwise it will not work!

### 2.1 **Default Values**
- This sections sets the Default Values for the Container.
- `APP` needs to be set to the Application name and must represent the filenames of your scripts.
- `var_tags`: You can set Tags for the CT wich show up in the Proxmox UI. Don´t overdo it! 
Example:
```bash
APP="SnipeIT"
var_tags="assat-management;foss"
var_cpu="2"
var_ram="2048"
var_disk="4"
var_os="debian"
var_version="12"
var_unprivileged="1"
```
### 2.2 App Output & Base Settings
- `header_info "$APP` sets the ASCII header.
- `base_settings` sets the values for container creation.
- `variables`,`color`,`catch_error` are helper functions importet from `build.func`.


## 3 **Update Function**

### 3.1 **Function Header**
- if applicable write a function wich updates the Application and the OS in the container.
- Each update function starts with a standardised Header:
```bash
function update_script() {
  header_info
  check_container_storage
  check_container_resources
```

### 3.2 **Check APP**
- Befor doing anything updatewise, check if the App is installed in the Container.

Example:
```bash
if [[ ! -d /opt/snipe-it ]]; then
    msg_error "No ${APP} Installation Found!"
    exit
  fi
```
### 3.3 **Check Version**
- The last step befor the update is to check if ther is a new version. 
- For this we use the `${APPLICATION}_version.txt` file created in `/opt` during the install.

Example with a Github Release:
```bash
 RELEASE=$(curl -s https://api.github.com/repos/snipe/snipe-it/releases/latest | grep "tag_name" | awk '{print substr($2, 3, length($2)-4) }')
  if [[ ! -f /opt/${APP}_version.txt ]] || [[ "${RELEASE}" != "$(cat /opt/${APP}_version.txt)" ]]; then
    msg_info "Updating ${APP} to v${RELEASE}"
    #DO UPDATE STUFF
  else
    msg_ok "No update required. ${APP} is already at v${RELEASE}."
  fi
  exit
}
```
### 3.4 **Verbosity**
- Use the appropiate flag (**-q** in the examples) for a command to suppres its output.
Example:
```bash
wget -q
unzip -q
```
- If a command dose not come with such a functionality use `&>/dev/null` for suppresinf output verbosity.

Example:
```bash
php artisan migrate --force &>/dev/null
php artisan config:clear &>/dev/null
```

### 3.5 **Backups**
- Backup userdata if nessesary.
- Move all userdata back in the Directory when the update is finnished.
>[!WARNING]
>This is not meant to be a permantent backup
Example backup:
```bash
  mv /opt/snipe-it /opt/snipe-it-backup
```
Example config restore:
```bash
  cp /opt/snipe-it-backup/.env /opt/snipe-it/.env
  cp -r /opt/snipe-it-backup/public/uploads/ /opt/snipe-it/public/uploads/
  cp -r /opt/snipe-it-backup/storage/private_uploads /opt/snipe-it/storage/private_uploads
```

### 3.6 **Cleanup**
- Do not forget to remove any temporary files/folders such as zip-files or temporary backups.
Example:
```bash
  rm -rf /opt/v${RELEASE}.zip
  rm -rf /opt/snipe-it-backup
```

### 3.7 **No update function**
- In case you can not provide a update function use the following code to provide user feedback.
```bash
function update_script() {
    header_info
    check_container_storage
    check_container_resources
    if [[ ! -d /opt/snipeit ]]; then
        msg_error "No ${APP} Installation Found!"
        exit
    fi
    msg_error "Ther is currently no automatic update function for ${APP}."
    exit
}
```

## 4 **End of the Script##
- The script ends with a few function calls and a success Message.
- With `echo -e "${TAB}${GATEWAY}${BGN}http://${IP}${CL}"` you can point the user to the IP:PORT/folder needed to access the App.

```bash
start
build_container
description

msg_ok "Completed Successfully!\n"
echo -e "${CREATING}${GN}${APP} setup has been successfully initialized!${CL}"
echo -e "${INFO}${YW} Access it using the following URL:${CL}"
echo -e "${TAB}${GATEWAY}${BGN}http://${IP}${CL}"
```

## 5. **Consistency and Style**

### 5.1 **Indentation**
- Use 2 spaces for indentation for better readability.

---

## 5. **Best Practices Checklist**

- [ ] Shebang is correctly set (`#!/usr/bin/env bash`).
- [ ] Correct link to *build.func*
- [ ] Metadata (author, license) is included at the top.
- [ ] Variables follow naming conventions.
- [ ] Update function exists.
- [ ] Update functions checks if App is installed an for new Version.
- [ ] Update function up temporary files.
- [ ] Script ends with a helpfull message for the User to reach the App.

---

# **APP<span></span>.json Files**
 `APP.json` files found in the `/json` directory. These files are used to provide informations for the frontend. For this guide we take `/json/snipeit.json` as example.

 ## 1 **Json Generator**
 - To spare you some headache creating the json file, use the [Json-Editor](https://community-scripts.github.io/ProxmoxVE/json-editor) 


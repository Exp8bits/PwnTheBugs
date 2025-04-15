## User Account Management

### Display users on the system
- **Command**: `users`  
  *This will display the users on the system.*

### Add, Delete, Modify User accounts

- **Add a new user**:  
  - **Command**: `useradd -m -s /bin/bash -G sudo [username]`  
    *`-m` = Create home directory | `-G` = Adding the user to specific group, `sudo` is required.*  
    *`-s` = The shell (bash or zshrc) that will be assigned to this user.*

- **Switch user**:  
  - **Command**: `su [username]`  
    *To switch from the current user to another user, type the username and then enter the password.*

- **View all users**:  
  - **Command**: `cat /etc/shadow`  
    *Displays the users on the system.*

### Change user password
- **Change password**:  
  - **Command**: `passwd [username]`  
    *To change the password for the specified user.*

- **Force user to change password**:  
  - **Command**: `passwd -e [username]`  
    *This forces the user to change their password upon first login (must be executed as root).*

### Delete a user
- **Delete user**:  
  - **Command**: `userdel -r [username]`  
    *`-r` = When you delete it, remove the home directory as well.*

### Modify a user
- **Change user home directory**:  
  - **Command**: `usermod -d /change/to/default/path [username]`  
    *Set this path as the default home directory for this user.*

- **Change username**:  
  - **Command**: `usermod -l newUser oldUser`  
    *Change a specific username.*

- **Change user shell**:  
  - **Command**: `usermod -s /bin/zsh [username]`  
    *Change the shell of a specific user.*

- **Change user groups (remove last groups)**:  
  - **Command**: `usermod -G group1,group2 [username]`  
    *Change the user groups with removing the last groups.*

- **Change user groups (without removing last groups)**:  
  - **Command**: `usermod -aG newGroup [username]`  
    *Change the user groups without removing the last groups.*

- **View groups in the system**:  
  - **Command**: `cat /etc/group`  
    *To display the groups in the system.*

- **View user's groups**:  
  - **Command**: `groups [username]`  
    *To view the groups of the specified user.*

### Lock/Unlock a user
- **Lock user account**:  
  - **Command**: `passwd -l [username]`  
    *To lock a specific user from logging in.*

- **Unlock user account**:  
  - **Command**: `passwd -u [username]`  
    *To unlock a specific user from logging in.*

### Group Management
- **Add a new group**:  
  - **Command**: `groupadd [groupName]`

- **Add a new group with id**:  
  - **Command**: `groupadd [groupName] -g [GID]`  
    *Create a group with a specific GID.*

- **Delete group**:  
  - **Command**: `groupdel [groupName]`  
    *Delete a specific group.*

- **Rename group**:  
  - **Command**: `groupmod oldName -n newName`  
    *To rename the group.*

- **Change the GID of a group**:  
  - **Command**: `groupmod -g [num] [groupName]`  
    *Replace `[num]` with the desired GID (greater than 1001).*

- **Remove user from group**:  
  - **Command**: `gpasswd -d [username] [groupName]`  
    *Remove a specific user from a specific group.*

- **Get information about a group**:  
  - **Command**: `getent group [groupName]`  
    *To get more information about a specific group.*

- **Change the primary group for a user**:  
  - **Command**: `usermod -g [newGroup] [username]`  
    *The primary group is the group that is created when you create the user.*

- **Put a password for a group**:  
  - **Command**: `gpasswd [groupName]`

- **Join the group with a password**:  
  - **Command**: `newgrp [groupName]`  
    *Type the group password.*

---

### Explanation of Shadow and Passwd Files

#### /etc/passwd
- **Example**:  
  `root:x:0:0:root:/root:/usr/bin/zsh`
  - **1st field**: Username.
  - **2nd field**: The password is stored in the shadow file.
  - **3rd field**: UID (user id):GID (group id).
  - **4th field**: User's group.
  - **5th field**: Default path.
  - **6th field**: The shell being used.

#### /etc/shadow
- **Example**:  
  `admin:$y$j9T$QbCTLnmDFJaWMDz6EaxEF0$wgVb2ruLvNfM0kvAqLglnnQb/K2VgzEDxqeRD4kRCS4:19957:0:99999:3:::`
  - **1st field**: Username.
  - **2nd field**: User's password hash (SHA-512).
  - **3rd field**: The number of days the password has not been changed since 1970-01-01.
  - **4th field**: The minimum number of days password can be changed.
  - **5th field**: The maximum number of days password needs to be changed.
  - **6th field**: Days to warn the user to change the password (before 7 days).
  - **7th field**: Days after password expired and before account is locked.
  - **8th field**: The account will be locked (expired) after the specified period (number of days since 1970).

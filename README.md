# Dual Accounts

**General**

This script enables fully automatic configuration for the dual account feature.

**Requirements**
- This scripts can run against Vault and PVWA v12.1 and up.
- This script is supported for CP v12.0 and up.
 
 **Limitations**
 
This script supports only platforms with the following properties:
   - Address
   - Username
   - Password
   - Account
   
If your platform has other mandatory properties, [configure dual account manually](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-CP/Latest/en/Content/CP%20and%20ASCP/cv_Automatic_dual_account.htm?tocpath=Administration%7CCredential%20Provider%7CAccounts%20and%20Safes%7CManage%20Dual%20Accounts%7C_____1)

### Step 1 - Fill in Policy-DualAccount-Creation.json
**Policy-DualAccount-Creation.json** is the configuration file for the script that creates dual accounts. 
It contains all mandatory and optional properties of your dual accounts.

Before you run the script fill in the required details as follows:

**The following table list the parameters in this file:**

| Parameter              | Description                                                                                                                                                                  | Required | Acceptable Value | Default Value    | 
| :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------| :------- | :--------------- | :--------------- |
| PVWAURL                | The URL address of your PVWA.                                                                                                                                                | **Yes**  | URL              | -                |
| PlatformSampleTemplate | The full path of the sample platform for Rotational Groups, including the zip file name (Rotational Group.zip).                                                              | **Yes**  | Full file path   | -                |
| PlatformID             | The ID of the platform for your dual account pair.                                                                                                                           | **Yes**  | String           | -                |
| VirtualUserName        | The name that the application uses to request access to the dual account pair secrets.                                                                                       | **Yes**  | String           | -                | 
| SafeName               | The Safe used to store the dual account pair.                                                                                                                                | **Yes**  | String           | -                |
| GroupName              | The group for the dual account pair - used for rotating the accounts.                                                                                                        | **Yes**  | String           |  -               |
| AccountDelimiter       | The character used to separate the properties of the accounts. Make sure this character is not used in any of the properties.                                                | No       | Char             |  @               |
| ListDelimiter          | The character used to separate the two accounts in the pair. Make sure this character is not used in either of the account properties.                                       | No       | Char             |  ;               |
| GracePeriod            | The number of minutes between the rotation of roles between the accounts (Active/Inactive) and the beginning of the password change process for the current inactive Account.| No       | Numeric          |  6               |
| LogFileFullPath        | The path where your log file is saved, including the log file name (Log-DualAccount.log). Make sure you have write permissions to this folder.                               | No       | Full file path   |  Script location |
| LogDebugLevel          | When enabled, more information will be written to the log file. This is useful when having issues with the script.                                                           | No       | Boolean          |  False           |
| LogVerboseLevel        | When enabled, more information will be written to the log file. This includes more information on the technical flow.                                                        | No       | Boolean          |  False           |
| DisableSSLVerify       | When enabled, SSL verification will be disabled and self-signed SSL certificates will be bypassed. It is not recommended.                                                    | No       | Boolean          |  False           |

**Note:**
If you want to use a different Rotational Group platform for your dual account pair than the one that already exist, do the following:
  - Configure the PlatformsSampleTemplate to be the default one you got from the script's folder (no need to configure GracePeriod value).
  - Continue with **Step 2 - Run the Dual Account Creation script** below
  - When the script has finished running, log on to the PVWA as admin.
  - Duplicate the Rotational Group platform and change **GracePeriod** as desired.
  - For each dual account pair that should work with the new Rotational Group, and update its Rotational Group Platform with the duplicated platform.

### Step 2 - Run the Dual Account Creation script
The Dual Account Creation script (DualAccount-Creation.ps1) creates and performs the necessary changes on the platform and in the account properties to enable the accounts to work as a dual accounts pair.

#### Usage
```powershell
DualAccount-Creation.ps1 -PASUserName <string> -PASPassword <string> -AccountList <string> -AuthenticationType <string> -ConfigFileFullPath <string>
```

For the **AccountList** combine the username, IP address and password or path of SSH key file including file name as follows:
```
[USER_NAME1]@[IP_ADDRESS]@[PASSWORD];[USER_NAME2]@[IP_ADDRESS]@[PASSWORD]
```
or
 ```
 [USER_NAME1]@[IP_ADDRESS]@[SSH_KEY_PATH];[USER_NAME2]@[IP_ADDRESS]@[SSH_KEY_PATH]
 ```

| Parameter              | Description                                                                                                                                                                  | Required | Acceptable Value | Default Value    |
| :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------| :------- | :--------------- | :--------------- | 
| PASUserName            | The Vault/PVWA username.                                                                                                                                                     | **Yes**  | String           | -                |
| PASPassword            | The Vault/PVWA password.                                                                                                                                                     | **Yes**  | String           | -                |
| AccountList            | The credentials of the accounts that you want to create.                                                                                                                     | **Yes**  | String           | -                |
| AuthenticationType     | The type of authentication type for logging on - available values: CyberArk, LDAP or RADIUS.                                                                                 | No       | String           | CyberArk         |              
| ConfigFileFullPath     | The full path of the configuration file, including file name (Policy-DualAccount-Creation.json). Make sure you have write permissions to this folder.                        | No       | Full file path   | Script location  |

**Note:**
You can view Dual Account creation script's progress and failures in the Console and in the Log file (Log-DualAccount.log). 

### Step 3 - Test your dual account pair
- In PVWA > Accounts > Accounts Details, select your dual account pair.
- Select one of the accounts and on the CPM tab, click Display. Both accounts appear in the same group, where one accounts is active and the other is inactive (see the Status field). 
- Using the dual account pair's virtual username, retrieve the dual account pair's password. The password retrieved should be the password of the active account.
- Trigger a CPM rotation for one of the accounts or wait for this to happen on it's own.
- Once again, using the dual account pair's virtual username, retrieve the dual account pair's password.
- Verify the currently active account is the account that was inactive before. 
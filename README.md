# WordPress Plugin WP All Import <= 3.6.7 - Remote Code Execution (RCE) (Authenticated)

```
Date: November 05 2022
Exploit Author: AkuCyberSec (https://github.com/AkuCyberSec)
Vendor Homepage: https://www.wpallimport.com/
Software Link: https://wordpress.org/plugins/wp-all-import/advanced/ (scroll down to select the version)
Version: <= 3.6.7 (tested: 3.6.7)
Tested on: WordPress 6.1 (os-independent since this exploit does NOT provide the payload)
CVE: CVE-2022-1565
```

## VULNERABILITY DESCRIPTION
The plugin WP All Import is vulnerable to arbitrary file uploads due to missing file type validation via the wp_all_import_get_gz.php file in versions up to, and including, 3.6.7. 
This makes it possible for authenticated attackers, with administrator level permissions and above, to upload arbitrary files on the affected sites server which may make remote code execution possible. 

## HOW THE EXPLOIT WORKS
### 1. Prepare the zip file:
  - create a PHP file with your payload (e.g. reverse shell)
  - set the variable "payload_file_name" with the name of this file (e.g. "shell.php")
  - create a zip file with the payload
  - set the variable "zip_file_to_upload" with the PATH of this file (e.g. "/root/shell.zip")
### 2. Login using an administrator account:
  - set the variable "target_url" with the base URL of the target (do NOT end the string with the slash /)
  - set the variable "admin_user" with the username of an administrator account
  - set the variable "admin_pass" with the password of an administrator account
### 3. Get the wpnonce using the get_wpnonce_upload_file() method
  - there are actually 2 types of wpnonce:
      - the first wpnonce will be retrieved using the method retrieve_wpnonce_edit_settings() inside the PluginSetting class.
          This wpnonce allows us to change the plugin settings (check the step 4)
      - the second wpnonce will be retrieved using the method retrieve_wpnonce_upload_file() inside the PluginSetting class.
          This wpnonce allows us to upload the file
  
### 4. Check if the plugin secure mode is enabled using the method check_if_secure_mode_is_enabled() inside the PluginSetting class
  - if the Secure Mode is enabled, the zip content will be put in a folder with a random name.
      The exploit will disable the Secure Mode.
      By disabling the Secure Mode, the zip content will be put in the main folder (check the variable payload_url).
      The method called to enable and disable the Secure Mode is set_plugin_secure_mode(set_to_enabled:bool, wpnonce:str)
  - if the Secure Mode is NOT enabled, the exploit will upload the file but then it will NOT enable the Secure Mode.
### 5. Upload the file using the upload_file(wpnonce_upload_file: str) method
  - after the upload, the server should reply with HTTP 200 OK but it doesn't mean the upload was completed successfully.
      The response will contain a JSON that looks like this:
          {"jsonrpc":"2.0","error":{"code":102,"message":"Please verify that the file you uploading is a valid ZIP file."},"is_valid":false,"id":"id"}
      As you can see, it says that there's an error with code 102 but, according to the tests I've done, the upload is completed
### 6. Re-enable the Secure Mode if it was enabled using the switch_back_to_secure_mode() method
### 7. Activate the payload using the activate_payload() method
  - you can define a method to activate the payload.
      There reason behind this choice is that this exploit does NOT provide any payload.
      Since you can use a custom payload, you may want to activate it using an HTTP POST request instead of a HTTP GET request, or you may want to pass parameters

## WHY DOES THE EXPLOIT DISABLE THE SECURE MODE?
According to the PoC of this vulnerability provided by WPSCAN, we should be able to retrieve the uploaded files by visiting the "Managed Imports page".

I don't know why but, after the upload of any file, I couldn't see the uploaded file in that page (maybe the Pro version is required?).

I had to find a workaround and so I did, by exploiting this option.

WPSCAN Page: https://wpscan.com/vulnerability/578093db-a025-4148-8c4b-ec2df31743f7

## ANY PROBLEM WITH THE EXPLOIT?
In order for the exploit to work please consider the following:
1. check the target_url and the admin credentials
2. check the path of the zip file and the name of the payload (they can be different)
3. if you're testing locally, try to set verify_ssl_certificate on False
4. you can use print_response(http_response) to investigate further

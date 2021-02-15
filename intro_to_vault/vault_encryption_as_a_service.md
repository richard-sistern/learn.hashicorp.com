# Vault Encryption as a Service

Show how Vault's Transit secrets engine provides encryption as a service.

## Enable the Transit Secrets Engine

The Transit secrets engine allows Vault to function as an encryption-as-a-service.

In this track, you will use the Transit secrets engine with a Python web app that talks to a MySQL server. Both of these run on the same VM as the Vault server, which is run in "dev" mode with root token set to root.

The web app can be viewed on the "Web App" tab. You don't actually need to use it until the third challenge, but we exposed it to satisfy the curious.

All secrets engines must be enabled before they can be used. Check which secrets engines are currently enabled on the "Vault CLI" tab.

```shell
vault secrets list
```

Note that the Transit secrets engine is not enabled. Please enable an instance of it at the path "lob_a/workshop/transit" so that the members of Line of Business A can use it.

```shell
vault secrets enable -path=lob_a/workshop/transit transit
```

Other lines of business could create their own instances of the Transit engine.

## Create a Key for the Transit Secrets Engine

In order to use the Transit secrets engine, you need to create at least one encryption key to use with it.

Create the "customer-key" encryption key:

```shell
vault write -f lob_a/workshop/transit/keys/customer-key
```

That was easy!

## Use the Web App Without Vault

The web app can be viewed on the "Web App" tab. You'll want to click the icon in the upper right corner of the tabs area to hide the assignment window so that you can see the entire UI of the web app. Click it again to display the assignment window.

The web app has two sections:

Records View: This displays what a real user would see after any encrypted data has been decrypted.
Database View: This displays raw records from the database. If any items are encrypted in the database, they will not be decrypted.
In its initial state, no records have been encrypted by Vault. So, you will essentially see the same data in both views. (The headers and order of the columns are different.) Go ahead and confirm that.

The web app was already started by the track's first challenge.

Click the "Add Record" button and add a new record with some fake data. Then check that the new record is not encrypted in the Database View.

Without Vault, the web app gets a username and password for talking to the MySQL database from a file "config.ini" in the directory /home/vault/transit-app-example/backend. Specifically, the "User" field specifies "root" as the database user while the "Password" field gives the original password for the root user, "sJ2w*8NX". In the next challenge, the web app will retrieve a username and password dynamically generated from Vault's Database secrets engine.

In the next challenge, you'll enable Vault for the application and see the Transit engine in action.

## Use the Web App With Vault

In this challenge, you will stop the Python web app, edit its configuration file to use Vault, and see how new records are encrypted by Vault's Transit engine.

As a bonus, you'll see that the web app is now logging into the MySQL database that stores its data with short-lived credentials retrieved from Vault's Database secrets engine. These credentials are dynamically generated on the fly when the app is started. Instead of using insecure, long-lived database credentials, the app will now get a new set of credentials every time it is started.

You first need to determine the process ID (PID) of the running web app and stop it. The PID of the web app can be found by running this command:

```shell
ps -ef | grep app.py
```

It can then be stopped by running this command against the first process ID you see for the running application:

```shell
kill -9 <PID>
```

Be sure to pick the PID from the line with the command, python3 app.py, rather than the line with the command, grep --color=auto app.py.

Now, you should edit the "config.ini" file on the "Config File" tab, doing the following:

```ini
[DEFAULT]
LogLevel = INFO
[DATABASE]
Address=localhost
Port=3306
User=root
Password=sJ2w*8NX
Database=my_app
[VAULT]
Enabled = True
DynamicDBCreds = True
DynamicDBCredsPath = lob_a/workshop/database/creds/workshop-app-long
ProtectRecords=False
Address=http://localhost:8200
#Address=vault.service.consul
Token=root
KeyPath=lob_a/workshop/transit
KeyName=customer-key
```

Change the first line of the "[VAULT]" section that has Enabled = False to Enabled = True.
Change the next line that has DynamicDBCreds = False to DynamicDBCreds = True.
Then save the "config.ini" file.

You should now restart the web app in the background with the following commands:

```shell
cd /home/vault/transit-app-example/backend
python3 app.py &
```

After running those commands, press your <enter> or <return> key to get your prompt back.

After running those commands, press your <enter> or <return> key to get your prompt back.

Note that the log will show a message like "using dynamic credentials from Vault: v-token-workshop-a-LcIkgRzHwTF2m with password A1a-U9QfaBp6hQaynvJy". This shows that the web app got dynamic, short-lived credentials from Vault's Database secrets engine. It then uses these to login to the MySQL server.

Return to the "Web App" tab and add another record. (Remember to click Instruqt's assignment toggle icon to hide the assignment so that you can see the "Add Record" button.) Then check that sensitive fields of the new record is encrypted in the Database View. In particular, the birth_date, social_security_number, address, and salary fields are all encrypted for the new record.

Next, login to the Vault UI with token root and click on the "lob_a/workshop/transit" secrets engine on the Secrets tab of the Vault UI. Then click on the "customer-key" key. Then select the "Versions" tab. Rotate the key by clicking the "Rotate encryption key" button and confirming that you really want to rotate it by clicking "Rotate". You will now see two versions of the key.

Go back to the web app, add another record, and then look at the record in the Database View. Verify that is is also encrypted. Note that the first record you added after enabling Vault and the Transit engine has encrypted fields with encrypted values starting with "vault:v1" while the new record has fields with encrypted values starting with "vault:v2".

Vault used the new version of the key to encrypt the new record. Someone with the old version of the key could not decrypt this new record. But Vault can still decrypt the older record that was encrypted with the first version of the key. Finally, if desired, data encrypted with an older version of the key can be rewrapped with the newest version using the Transit engine's rewrap endpoint.

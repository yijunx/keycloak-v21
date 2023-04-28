# keycloak-v21

## SETUP REALM ON NEW KEYCLOAK WITH TOKEN-EXCHANGE AND ADMIN-FINE-GRAINED-AUTHZ

- to enable the token exchange and admin fine grained authz, please refer to the `keycloak/Dockerfile` and pay attention to the build part (last line of the builder), and the entrypoint.
- to make use of them.. (TO BE ADDED)


## EXPORT FROM OLD KEYCLOAK
bitnami keycloak export (keycloak 16.1.1). Here we focus on bitnami, because it is quite not clear how and there are not many clues out there on the internet.

- edit the keycloak statefulset to run forever without the server going up, using the command field below

    ```yaml
        containers:
        - command:
            - tail
            - -f
            - /dev/null
          env:
          - name: blahblah
          - value: blahblah

    ```

- exec into the pod

    ```bash
    kubectl exec -it keycloak-0 -n your-namespace -- bash
    ```

- now we are in the pod, lets export

    ```bash
    # create a folder for export
    mkdir -p tmp/exports/default

    # clean up the extra args with the ones just for export
    # for this part you can refer to the part below (A short list of extra args)
    # here I recommand using DIFFERENT_FILES for usersExportStrategy, because the realm is likely to be created by some other program, and we just need to import the users
    export KEYCLOAK_EXTRA_ARGS="-Dkeycloak.migration.action=export -Dkeycloak.migration.provider=dir -Dkeycloak.migration.dir=/tmp/exports/default -Dkeycloak.migration.usersExportStrategy=DIFFERENT_FILES -Dkeycloak.migration.realmName=default"
    # then there will be a few files in the folder for export

    # finally lets trigger it
    /opt/bitnami/scripts/keycloak/entrypoint.sh /opt/bitnami/scripts/keycloak/run.sh

- on your laptop, copy the files down

    ```bash
    # here we copy from a folder
    kubectl cp keycloak-0:tmp/exports/default -n your-namespace /Users/yourname//migrate-keycloak
    ```


- A short list of extra args 

    -Dkeycloak.migration.realmName
    This property is used if you want to export just one specified realm instead of all. If not specified, then all realms will be exported.

    -Dkeycloak.migration.usersExportStrategy
    This property is used to specify where users are exported. Possible values are:

    DIFFERENT_FILES - Users will be exported into different files according to the maximum number of users per file. This is default value.

    SKIP - Exporting of users will be skipped completely.

    REALM_FILE - All users will be exported to same file with the realm settings. (The result will be a file like "foo-realm.json" with both realm data and users.)

    SAME_FILE - All users will be exported to same file but different from the realm file. (The result will be a file like "foo-realm.json" with realm data and "foo-users.json" with users.)

    -Dkeycloak.migration.usersPerFile
    This property is used to specify the number of users per file (and also per DB transaction). It’s 50 by default. It’s used only if usersExportStrategy is DIFFERENT_FILES

    -Dkeycloak.migration.strategy
    This property is used during import. It can be used to specify how to proceed if a realm with same name already exists in the database where you are going to import data. Possible values are:

    IGNORE_EXISTING - Ignore importing if a realm of this name already exists.

    OVERWRITE_EXISTING - Remove existing realm and import it again with new data from the JSON file. If you want to fully migrate one environment to another and ensure that the new environment will contain the same data as the old one, you can specify this.

    now lets get the users into the new realm


## IMPORT THE USERS TO THE KEYCLOAK 21.0.2

- go to the ui
- go the realm of interest
- click the Realm settings of the lower half of the navigation panel
- click action at the top right corner
- in the drop down select partial import
- then select the json for users
- and its done!!
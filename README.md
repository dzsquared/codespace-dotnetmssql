# C# (.NET Core) and MS SQL
This project is a part of the vscode-dev-containers project here: https://github.com/microsoft/vscode-dev-containers/. 

## Summary

*Develop C# and .NET Core based applications. Includes all needed SDKs, extensions, dependencies and an MS SQL container for parallel database development. Adds an additional MS SQL container to the C# (.NET Core) container definion and deploys any .dacpac files from the mssql .devcontainer folder.*

| Metadata | Value |  
|----------|-------|
| *Contributors* | The VS Code and Azure Data Teams |
| *Definition type* | Docker Compose |
| *Published images* | mcr.microsoft.com/vscode/devcontainers/dotnetcore |
| *Available image variants* | 2.1, 3.1 |
| *Published image architecture(s)* | x86-64 |
| *Works in Codespaces* | Yes |
| *Container host OS support* | Linux, macOS, Windows |
| *Container OS* | Ubuntu |
| *Languages, platforms* | .NET Core, C#, Microsoft SQL |

## Description
This definition creates two containers, one for C# (.NET Core) and one for Microsoft SQL.  VS Code will attach to the .NET Core container, and from within that container the Microsoft SQL container will be available on **`localhost`** port 1433. By default, the `sa` user password is `P@ssw0rd`. For more on the configuration of MS SQL, see the section [MS SQL Configuration](#MS-SQL-Configuration)

## Using this definition with an existing folder

While this definition should work unmodified, you can select the version of .NET Core the container uses by updating the `VARIANT` arg in the included `devcontainer.json` (and rebuilding if you've already created the container).

```json
"args": { "VARIANT": "3.1" }
```


### Debug Configuration

Only the integrated terminal is supported by the Remote - Containers extension. You may need to modify your `.vscode/launch.json` configurations to include the following:

```json
"console": "integratedTerminal"
```

### Using the forwardPorts property

By default, ASP.NET Core only listens to localhost inside the container. As a result, we recommend using the `forwardPorts` property in `.devcontainer/devcontainer.json` (available in v0.98.0+) to make these ports available locally.

```json
"forwardPorts": [5000, 5001]
```

The `appPort` property [publishes](https://docs.docker.com/config/containers/container-networking/#published-ports) rather than forwards the port, so applications need to listen to `*` or `0.0.0.0` for the application to be accessible externally. This conflicts with ASP.NET Core's defaults, but fortunately the `forwardPorts` property does not have this limitation.

If you've already opened your folder in a container, rebuild the container using the **Remote-Containers: Rebuild Container** command from the Command Palette (<kbd>F1</kbd>) so the settings take effect.

### Enabling HTTPS in ASP.NET Core

To enable HTTPS in ASP.NET, you can mount an exported copy of your local dev certificate.

1. Export it using the following command:

    **Windows PowerShell**

    ```powershell
    dotnet dev-certs https --trust; dotnet dev-certs https -ep "$env:USERPROFILE/.aspnet/https/aspnetapp.pfx" -p "SecurePwdGoesHere"
    ```

    **macOS/Linux terminal**

    ```powershell
    dotnet dev-certs https --trust; dotnet dev-certs https -ep "${HOME}/.aspnet/https/aspnetapp.pfx" -p "SecurePwdGoesHere"
    ```

2. Add the following in to `.devcontainer/devcontainer.json`:

    ```json
    "remoteEnv": {
        "ASPNETCORE_Kestrel__Certificates__Default__Password": "SecurePwdGoesHere",
        "ASPNETCORE_Kestrel__Certificates__Default__Path": "/home/vscode/.aspnet/https/aspnetapp.pfx",
    }
    ```

3. Finally, make the certificate available in the container as follows:

    **If using GitHub Codespaces and/or Remote - Containers**

    1. Start the container/codespace
    2. Drag `~/.aspnet/https/aspnetapp.pfx` from your local machine into the root of the File Explorer in VS Code.
    3. Open a terminal in VS Code and run:
        ```bash
        mkdir -p /home/vscode/.aspnet/https && mv aspnetapp.pfx /home/vscode/.aspnet/https
        ```

    **If using only Remote - Containers with a local container**

    Add the following to `.devcontainer/devcontainer.json`:

    ```json
    "mounts": [ "source=${env:HOME}${env:USERPROFILE}/.aspnet/https,target=/home/vscode/.aspnet/https,type=bind" ]
    ```

If you've already opened your folder in a container, rebuild the container using the **Remote-Containers: Rebuild Container** command from the Command Palette (<kbd>F1</kbd>) so the settings take effect.

### Installing Node.js or the Azure CLI

Given how frequently ASP.NET applications use Node.js for front end code, this container also includes Node.js. You can change the version of Node.js installed or disable its installation by updating the `args` property in `.devcontainer/devcontainer.json`.

```json
"args": {
    "VARIANT": "3.1",
    "INSTALL_NODE": "true",
    "NODE_VERSION": "10",
}
```

If you would like to install the Azure CLI update you can set the `INSTALL_AZURE_CLI` argument line in `.devcontainer/devcontainer.json`:

```Dockerfile
"args": {
    "VARIANT": "3.1",
    "INSTALL_NODE": "true",
    "NODE_VERSION": "10",
    "INSTALL_AZURE_CLI": "true"
}
```

If you've already opened your folder in a container, rebuild the container using the **Remote-Containers: Rebuild Container** command from the Command Palette (<kbd>F1</kbd>) so the settings take effect.

### Adding the definition to your folder

1. If this is your first time using a development container, please follow the [getting started steps](https://aka.ms/vscode-remote/containers/getting-started) to set up your machine.

2. To use VS Code's copy of this definition:
   1. Start VS Code and open your project folder.
   2. Press <kbd>F1</kbd> select and **Remote-Containers: Add Development Container Configuration Files...** from the command palette.
   3. Select the C# (.NET Core) and MS SQL definition.

3. To use latest-and-greatest copy of this definition from the repository:
   1. Clone this repository.
   2. Copy the contents of `containers/dotnetcore/.devcontainer` to the root of your project folder.
   3. Start VS Code and open your project folder.

4. After following step 2 or 3, the contents of the `.devcontainer` folder in your project can be adapted to meet your needs.

5. Finally, press <kbd>F1</kbd> and run **Remote-Containers: Reopen Folder in Container** to start using the definition.

## MS SQL Configuration
A secondary container for MS SQL is defined in `devcontainer.json` with the Dockerfile and supporting scripts in the `mssql` folder.  This container is deployed from the latest developer edition of Microsoft SQL 2019.  The database(s) are made available directly in the Codespace/VS Code through the MSSQL extension with a connection labeled "mssql-container".  The default `sa` user password is set to `P@ssw0rd`. The default SQL port is mapped to port `1433` in `docker-compose.yml`.

### Changing the sa password
To change the `sa` user password, change the value in `docker-compose.yml` and `devcontainer.json`.

### Database deployment
By default, a blank user database is created titled "ApplicationDB".  To add additional database objects or data through T-SQL during Codespace configuration, edit the file `.devcontainer/mssql/setup.sql` or place additional `.sql` files in the `.devcontainer/mssql/` folder. *Large numbers of scripts may take a few minutes following container creation to complete, even when the SQL server is available the database(s) may not be available yet.*

Alternatively, .dacpac files placed in the `./bin/Debug` folder will be published as databases in the container during Codespace configuration. [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage) is used to deploy a database schema from a data-tier application file (dacpac), allowing you to bring your application's database structures into the dev container easily. *The publish process may take a few minutes following container creation to complete, even when the server is available the database(s) may not be available yet.*

## Testing the definition

This definition includes some test code that will help you verify it is working as expected on your system. Follow these steps:

1. If this is your first time using a development container, please follow the [getting started steps](https://aka.ms/vscode-remote/containers/getting-started) to set up your machine.
2. Clone this repository.
3. Start VS Code, press <kbd>F1</kbd>, and select **Remote-Containers: Open Folder in Container...**
4. Select the `containers/dotnetcore` folder.
5. After the folder has opened in the container, if prompted to restore packages in a notification, click "Restore".
6. After packages are restored, press <kbd>F5</kbd> to start the project.
7. Once the project is running, press <kbd>F1</kbd> and select **Remote-Containers: Forward Port from Container...**
8. Select port 8090 and click the "Open Browser" button in the notification that appears.
9. You should see "Hello remote world from ASP.NET Core!" after the page loads.
10. From here, you can add breakpoints or edit the contents of the `test-project` folder to do further testing.

## License

Copyright (c) Microsoft Corporation. All rights reserved.

Licensed under the MIT License. See [LICENSE](https://github.com/Microsoft/vscode-dev-containers/blob/master/LICENSE).

Licenses for [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage-download), [SQLCMD](https://docs.microsoft.com/sql/linux/sql-server-linux-setup-tools), and [SQL Server Developer Edition](https://go.microsoft.com/fwlink/?linkid=857698).
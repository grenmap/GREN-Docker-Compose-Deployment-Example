# GREN Docker Compose Deployment Example

This repo was created by the GRENMap authors to provide a [Docker Compose](https://docs.docker.com/compose/compose-file/) example for deploying a GREN Map instance in production. Please note this is an example to build off of, not a full how-to guide.

For information about the development environment for GREN Map project, please check [GREN Map DB Node](https://github.com/grenmap/GREN-Map-DB-Node/tree/main/docs).

[Shibboleth Single Sign-on (SSO)](https://incommon.org/software/shibboleth/) has been used for user login and authorization. Please contact your local [Federation/IdP provider](https://technical.edugain.org) to integrate.

## Required Skill and Knowledge

Although the installation process is intended to be accessible to a broad audience, the following skills and knowledge would be beneficial:

* [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/compose-file/)
* [Shibboleth Single Sign-on (SSO)](https://incommon.org/software/shibboleth/)
* [Shibboleth Service Provider](https://shibboleth.atlassian.net/wiki/spaces/SP3/overview)
* [PostgreSQL](https://www.postgresql.org/) database management
* Service and deployment management strategy
* Firewall configuration and management

## Configuration

You must complete all the configure settings (".env", Shibboleth SSO, Apache) before you can deploy your instance.

### Config the Environment Variables

Open the ".env" file and update the environment variables with your desired values.

Contact your local [Federation/IdP provider](https://technical.edugain.org) to get support for Shibboleth SSO related configurations.

### Prepare the Secret Files

Get all of the private keys and certificates for Apache ("host-key.pem", "host-cert.perm") and Shibboleth ("sp-key.pem", "sp-cert.pem"), the database password file ("db_password.txt"), the Django secret file ("secret_key.txt"), and put them under the "./config/" folder.

Note:

1. Any new changes or placements of these files will require restarting the related containers.
2. Make sure to secure these files and restrict access to them for improved security.

```bash
    secrets:
        websp-host-key:
            file: ./config/host-key.pem
        websp-host-cert:
            file: ./config/host-cert.pem
        websp-sp-key:
            file: ./config/sp-key.pem
        websp-sp-cert:
            file: ./config/sp-cert.pem
        db_password:
            file: ./config/db_password.txt
        secret_key:
            file: ./config/secret_key.txt
```

#### The Private Keys and Certificates for Shibboleth and Apache

Obtain the SSL/TLS certificates from a trusted certificate authority (CA). For a quick start with a self-signed certificate, use the following command to generate "sp-key.pem" and "sp-cert.pem" for Shibboleth (replace "map.example.com" with your GREN Map instance's domain name):

```bash
    ./config/shibboleth_keygen.sh -o ./config -f  -h map.example.com -y 10 -n sp
```

Using the following command to generate "host-key.pem" and "host-cert.pem" for Apache (replace "map.example.com" with your GREN Map instance's domain name):

```bash
    ./config/shibboleth_keygen.sh -o ./config -f -h map.example.com -y 10 -n host
```

#### Create the Password and secret_key Files

The GREN Map instance needs to know the password for the database, and the secret_key for Django application. They must be stored in files named "db_password.txt" and "secret_key.txt" under "confg/" folder.

Run the following commands to create the related files, replace "your_db_password" and "your_django_secret_key" with your real values.

```bash
    echo "your_db_password" > ./config/db_password.txt
    echo "your_django_secret_key" > ./config/secret_key.txt
```

### Use a Custom Logo for Embeded Discovery Service(EDS) (Optional)

This is an optional configuration. You only need to consider this step when you are using the default value of "IDP_SSO" in ".env" file.

When you choose to use EDS, it uses a generic GREN Map logo by default. To override the default logo with a custom image below, keep reading...

The image file of "CAFLogo.png" under "./config" is served as an example. Create your own customized logo with similar dimension and size, and save it to the same folder. Uncomment the following line in the "docker-compose.yaml" file, and replace "CAFLogo.png" with your logo file name.

```bash
    - ./config/CAFLogo.png:/var/www/html/shibboleth-ds/eds_logo.png
```

## Deployment

### Hosting Requirements

* Ensure that your deployment server meets the minimum requirements for running Docker and Docker Compose.
* Allocate sufficient resources (CPU, memory, storage) based on the expected usage and scale of your application.

### Firewall Configuration

You should have a firewall enabled on your deployment server, and ensure that the following ports are open:

* HTTPS: 443 (or the custom MAP_HOST_PORT specified in the ".env" file)

### Deploy the GREN Map Instance with a remote PostgreSQL database

It's a good idea to use a remote database when you require a more robust and scalable database solution or need to connect to an existing database infrastructure.

Before deploying the GREN Map instance, make sure the remote database is up and running and can accept connections from the containers.

1. Start the instance

    ```bash
        docker compose up -d
    ```

2. View logs of containers

    ```bash
        docker compose logs -f
    ```

3. Access your GREN Map instance

    When all of the containers are up and running, you can access your GREN Map instance by visiting "https://${MAP_HOST_NAME}" in your web browser.

4. Stop and remove containers/networks

    ```bash
        docker compose down
    ```

### Deploy the GREN Map Instance with a container-based PostgreSQL database

Note: If you are using database containers, it's important to keep the following in mind:

* Take precautions to ensure that the volume storing the database data is not accidentally destroyed.
* Implement regular backup and restore procedures for your database to avoid data loss.

1. Start the instance

    ```bash
        docker compose --profile container_db up -d
    ```

2. View logs of containers

    ```bash
        docker compose --profile container_db logs -f
    ```

3. Access your GREN Map instance

    When all of the containers are up and running, you can access your GREN Map instance by visiting "https://${MAP_HOST_NAME}" in your web browser.

4. Stop and remove containers/networks

    ```bash
        docker compose --profile container_db down
    ```

# Deploy Multiple Hummingbot Instances with different profiles

This guide explains how to install two [Hummingbot](https://github.com/hummingbot/hummingbot) instances. You can choose to configure the bots to use either a **master_account** or **sub_accounts** for credentials and API keys. This feature is particularly useful if you manage multiple API keys or have set up subaccounts on exchanges and wish for an easy method to switch between them.

## Prerequisites

This configuration requires [Docker Compose](https://docs.docker.com/compose/), a tool for defining and running multi-container Docker applications. The recommended way to get Docker Compose is to install [Docker Desktop](https://www.docker.com/products/docker-desktop/), which includes Docker Compose along with Docker Engine and Docker CLI which are Compose prerequisites.

See [Docker](../DOCKER.md) for more information about how to install and use Docker Compose, as well as helpful commands.

## Getting Started

Verify that Docker Compose is installed correctly by checking the version:

```bash
docker compose version
```

The output should be: `Docker Compose version v2.17.2` or similar. Ensure that you are using Docker Compose V2, as V1 is deprecated.


## 1. Clone the **deploy-examples** repo

Clone the repository to your machine and navigate to the folder:

```
git clone https://github.com/hummingbot/deploy-examples.git
cd deploy-examples
```

## 2. Initial Configuration

### Create sub_account folder

By default, both bots will utilize the **master_account**. However, to configure the first bot with the **master_account** and the second bot with a **sub_account**—using a different Hummingbot password and API keys from the **master account**—follow the instructions below:

Create a new folder named **sub_account** under the **/credentials** folder, resulting in two folders:

```
deploy-examples/
├── credentials/
│   ├── master_account/
│   └── sub_account/

```

### Modify the Docker Compose file

Edit the Docker Compose file, updating the **bot_2** section to redirect the credentials folder to the newly created **sub_account** folder. Also, comment out the **environment** and **CONFIG_PASSWORD** fields for now, as we will be updating the password.

```bash hl_lines="5-6 12"
  bot_2:
    container_name: bot_2
    image: hummingbot/hummingbot:development
    volumes:
      - ./credentials/sub_account:/home/hummingbot/conf
      - ./credentials/sub_account/connectors:/home/hummingbot/conf/connectors
      - ./instances/bot_2/logs:/home/hummingbot/logs
      - ./instances/bot_2/data:/home/hummingbot/data
      - ./conf/scripts:/home/hummingbot/conf/scripts
      - ./conf/controllers:/home/hummingbot/conf/controllers
#    environment:
#      - CONFIG_PASSWORD=a
#      - CONFIG_FILE_NAME=v2_generic_with_controllers.py
#      - SCRIPT_CONFIG=conf_v2_generic_with_contorllers_2.yml
    logging:
      driver: "json-file"
      options:
          max-size: "10m"
          max-file: 5
    tty: true
    stdin_open: true

```

Save your changes

### Launch Hummingbot

From the root folder, run the following command to download the image and start the instances:

```
docker compose up -d
```

Upon successful download, you should see an output similar to:
```
[+] Running 4/4
 ⠿ Network multiple_bots_setup                                Created
 ⠿ Container bot_1                                            Started
 ⠿ Container bot_2                                            Started
 
```

Both bots will be running but we will need to configure **bot_2** first so we will need to attach to it

```
docker attach bot_2
```

Set your preferred password for the **sub_account**, using **b** as an example. After setting the password, proceed to enter the API keys for your sub-accounts. Once completed, exit the Hummingbot client with:

```
exit
```

Then use **docker compose down** to exit out all the running instances


```
docker compose down
```

### Update Docker Compose configuration

Edit the Docker Compose file again to enable auto-start with the new password. Uncomment the **environment** section and the **CONFIG_PASSWORD**, setting the password for **bot_2** as "**b**":

```bash hl_lines="5-6 12"
  bot_2:
    container_name: bot_2
    image: hummingbot/hummingbot:development
    volumes:
      - ./credentials/sub_account:/home/hummingbot/conf
      - ./credentials/sub_account/connectors:/home/hummingbot/conf/connectors
      - ./instances/bot_2/logs:/home/hummingbot/logs
      - ./instances/bot_2/data:/home/hummingbot/data
      - ./conf/scripts:/home/hummingbot/conf/scripts
      - ./conf/controllers:/home/hummingbot/conf/controllers
    environment:
      - CONFIG_PASSWORD=b
#      - CONFIG_FILE_NAME=v2_generic_with_controllers.py
#      - SCRIPT_CONFIG=conf_v2_generic_with_contorllers_2.yml
    logging:
      driver: "json-file"
      options:
          max-size: "10m"
          max-file: 5
    tty: true
    stdin_open: true

```

### Relaunch Hummingbot

After saving the updates to the Docker Compose file, restart the bots by running:

```
docker compose up -d
```


To attach to any container use 

```
docker attach [container name]
```


### Adding more bots

Following this configuration, you can add more bots with different credentials by simply adjusting the **credentials** folder and **CONFIG_PASSWORD** field as needed. For instance, to add a third bot using **sub_account** credentials, append the Docker Compose file accordingly.

```bash hl_lines="1-2 5-6 12"
  bot_3:
    container_name: bot_3
    image: hummingbot/hummingbot:development
    volumes:
      - ./credentials/sub_account:/home/hummingbot/conf
      - ./credentials/sub_account/connectors:/home/hummingbot/conf/connectors
      - ./instances/bot_2/logs:/home/hummingbot/logs
      - ./instances/bot_2/data:/home/hummingbot/data
      - ./conf/scripts:/home/hummingbot/conf/scripts
      - ./conf/controllers:/home/hummingbot/conf/controllers
    environment:
      - CONFIG_PASSWORD=b
#      - CONFIG_FILE_NAME=v2_generic_with_controllers.py
#      - SCRIPT_CONFIG=conf_v2_generic_with_contorllers_2.yml
    logging:
      driver: "json-file"
      options:
          max-size: "10m"
          max-file: 5
    tty: true
    stdin_open: true

```

Here we added the name of the new bot to **bot_3**, made sure the credentials volume is mapped to the **sub_account** folder and set the autostart password for **sub_account** which is **b**


## Updating to the Latest Version of Hummingbot

Hummingbot and Hummingbot Gateway are updated on a monthly basis, with each new version marked by a code release on Github and DockerHub, accompanied by the publication of comprehensive release notes. To upgrade to the most recent version, you just need to pull the `latest` Docker images.

Follow the steps below to upgrade your Hummingbot system:

1. **Ensure no containers are running**

   Before you initiate the update process, it is crucial to verify that no Docker containers are currently running. Use the following command to shut down any active containers:

   ```
   docker compose down
   ```

2. **Fetch the latest Docker image**

   Once you have confirmed that no containers are running, proceed to pull the latest Docker image. Use the following command to accomplish this:

   ```
   docker pull hummingbot/hummingbot
   ```

3. **Start the updated containers**

   Having pulled the latest Docker image, you can now start up your containers. They will be running the latest version of Hummingbot. Use the following command to start the containers:

   ```
   docker compose up -d
   ```

With these steps, you will have successfully updated your Hummingbot to the latest version.

## Deleting unused Docker images

Use the below command to manually remove unused Docker images and free up space

```
docker rmi [image_name]
```

To remove all unused images, not just dangling ones, you can use:

```
docker image prune -a
```

This command removes all images without at least one container associated with them. Use it with caution, as it can remove images that you may wish to keep.





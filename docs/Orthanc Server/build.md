!!!info "Acknowledgement"
    This project was a collaborative effort of me and [Donggyun Park](https://ki.se/personer/donggyun-park).

## Overview

This project sets up a self-contained environment for medical image management using the Orthanc DICOM server and related components. Here's what the included files do:

1. **docker-compose.yml [[Source]](https://github.com/NeuroGranberg/orthanc_server/blob/3a0e78590fb61bd2dd6c633702dc893de161a597/docker-compose.yml):**  Defines containerized services for smooth deployment: 

    * **orthanc_postgres:** A PostgreSQL database for storing Orthanc's index of medical images.
    * **orthanc_server:** The core Orthanc DICOM server, responsible for receiving, storing, and querying medical images.
    * **orthanc_client:**  A separate Orthanc instance configured primarily to send images to the main Orthanc server.
    * **adminer:** Provides a web interface for easy PostgreSQL database management.

    ```yaml
    version: '3.9'
    services:
    orthanc_postgres:
        image: postgres
        restart: always
        environment:
        POSTGRES_HOST_AUTH_METHOD: "trust"
        POSTGRES_DB: ${POSTGRES_DB}
        POSTGRES_USER: ${POSTGRES_USER}
        POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
        volumes:
        - orthanc_postgres_index:/var/lib/postgresql/data
        ports:
        - "5432:5432"

    adminer:
        image: adminer
        restart: always
        ports:
        - "8080:8080"

    orthanc_server:
        image: orthancteam/orthanc
        restart: unless-stopped
        ports:
        - "8042:8042"
        volumes:
        - orthanc_server_data:/var/lib/orthanc/db
        - ./orthanc_server_config.json:/etc/orthanc/config.json
        environment:
        ORTHANC_CONFIGURATION: /etc/orthanc/config.json

    orthanc_client:
        image: orthancteam/orthanc
        restart: unless-stopped
        ports:
        - "8043:8043"
        volumes:
        - orthanc_client_data:/var/lib/orthanc/db
        - ./orthanc_client_config.json:/etc/orthanc/config.json
        environment:
        ORTHANC_CONFIGURATION: /etc/orthanc/config.json

    volumes:
    orthanc_postgres_index:
    orthanc_server_data:
    orthanc_client_data:
    ```

2. **orthanc_server_config.json [[Source]](https://github.com/NeuroGranberg/orthanc_server/blob/3a0e78590fb61bd2dd6c633702dc893de161a597/orthanc_server_config.json):**  Configuration for the main Orthanc server, outlining its settings and the connection to the PostgreSQL database.

    ```json
    {
        "Name": "orthanc-server",
        "HttpPort": 8042,
        "DicomAet": "${DICOM_HOST_SERVER}",
        "DicomPort" : "${DICOM_PORT}",
        "RemoteAccessAllowed" : true,
        "StorageCompression": false,
        "KeepAlive": true,
        "TcpNoDelay": true,
        "StorageAccessOnFind": "Never",
        "SaveJobs": false,
        "AuthenticationEnabled": true,
        "IngestTranscodingOfUncompressed": false,
        "IngestTranscodingOfCompressed": false,
        "OverwriteInstances": true,
        "DicomScuTimeout": 120,
        "DicomScpTimeout": 240,
        "RegisteredUsers": {
        "orthanc": "orthanc"
        },
        "PostgreSQL": {
        "EnableIndex": true,
        "EnableStorage": false,
        "IndexConnectionsCount": 2,
        "Port": 5432,
        "Host": "orthanc_postgres",
        "Database": "${POSTGRES_DB}",
        "Username": "${POSTGRES_USER}",
        "Password": "${POSTGRES_PASSWORD}",
        "EnableSsl": false,
        "Lock": false
        },
        "DicomModalities":{
        "orthanc-client": ["${DICOM_AET_CLIENT}", "${DICOM_HOST_CLIENT}", "${DICOM_PORT}"]
        }
    }
    ```
3. **orthanc_client_config.json [[Source]](https://github.com/NeuroGranberg/orthanc_server/blob/3a0e78590fb61bd2dd6c633702dc893de161a597/orthanc_client_config.json):** Configuration for the Orthanc client, specifying how to connect to the main Orthanc server.

    ```json
    {
        "Name": "orthanc-client",
        "HttpPort": 8043,
        "DicomAet": "${DICOM_HOST_CLIENT}",
        "DicomPort": "${DICOM_PORT}",
        "RemoteAccessAllowed": true,
        "AuthenticationEnabled": true,
        "RegisteredUsers" : {
            "orthanc": "orthanc"
        },
        "DicomModalities": {
            "orthanc-server": ["${DICOM_AET_SERVER}", "${DICOM_HOST_SERVER}", "${DICOM_PORT}"]
        }
    }
    ```
  
## Key Components

* **Orthanc:** A lightweight, open-source DICOM server designed for ease of use and rapid deployment. ([https://www.orthanc-server.com/](https://www.orthanc-server.com/))
* **PostgreSQL:** A robust database system powering the storage and indexing of image metadata within Orthanc.
* **Adminer:** A web-based database management tool. ([https://www.adminer.org/](https://www.adminer.org/))
* **Docker Compose:** Orchestrates the deployment and relationships between all services.



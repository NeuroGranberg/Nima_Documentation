## How to Use

1. **Prerequisites:**
   * Docker engine installed ([https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/))
   * Docker Compose installed ([https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/))

2. **Environment Variables:**
   * Create a `.env` file in the project directory. 
   * Define the required environment variables (example provided below).

3. **Run:**
   * From the project directory, execute: `docker-compose up -d`

## Environment Variables

```
POSTGRES_DB=orthanc
POSTGRES_USER=orthancuser
POSTGRES_PASSWORD=yoursecurepassword
DICOM_HOST_CLIENT=client
DICOM_HOST_SERVER=server
DICOM_AET_CLIENT=ORTHANC_CLIENT
DICOM_AET_SERVER=ORTHANC_SERVER
DICOM_PORT=4242
```

## Accessing the System

* **Orthanc Server:** http://localhost:8042
* **Orthanc Client:** http://localhost:8043
* **Adminer (Database Management):** http://localhost:8080

!!!info Additional Notes

    * Persisting image data is achieved through the volumes specified in `docker-compose.yml`. Modify accordingly for your storage needs. 
    * You can send DICOM images to the Orthanc client, which will automatically forward them to the primary Orthanc server.
    * For production use, ensure robust security measures for image data and network access.

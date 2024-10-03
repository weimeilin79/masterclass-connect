## Use Case:
Build a streaming pipeline that processes backend error data from Redpanda in real-time. You need to enrich your log entries by adding geo-location data based on the client's IP address. You’ll achieve this by sending HTTP requests to an external API and caching the results to optimize performance. And at the same time send the log to an error count service by making HTTP requests to another external services. Additionally, you must implement caching to avoid redundant API calls for frequently encountered IP addresses, optimizing the pipeline’s performance.


![Screenshot 2024-10-01 at 10.21.13 PM.png](../assets/Screenshot%202024-10-01%20at%2010.21.13%E2%80%AFPM.png)

This is what the input payload looks like from the `backend_error`  topic:
```copy
{
    "client_ip": "192.168.1.142",
    "context": {
        "method": "POST",
        "path": "/login",
        "status_code": 401,
        "user_id": "user142"
    },
    "errors": [
        {
            "code": "E401",
            "description": "Unauthorized",
            "details": "User tried to access restricted resource."
        },
        {
            "code": "E002",
            "description": "Invalid Credentials",
            "details": "Password does not match the user account."
        },
        {
            "code": "E403",
            "description": "Forbidden",
            "details": "User does not have access to this resource."
        },
        {
            "code": "E406",
            "description": "Not Acceptable",
            "details": "Content type or encoding not acceptable."
        },
        {
            "code": "E407",
            "description": "Proxy Authentication Required",
            "details": "User must authenticate with a proxy server."
        }
    ],
    "event_id": "a388b2f7-fa83-419c-b057-d9a7632f8aa5",
    "log_level": "ERROR",
    "message": "Unauthorized access attempt detected.",
    "metadata": [
        {
            "key": "browser",
            "value": "Firefox"
        },
        {
            "key": "os",
            "value": "macOS"
        }
    ],
    "priority": 2,
    "source_system": "payment_gateway",
    "timestamp": "2024-10-01T16:23:21Z"
}
```
## Objective  :
- Consume error data from the Redpanda topic `backend_error`.
- Enrich the data with geo-location and error count by querying external APIs:
    - **URL**: `http://localhost:8081/geo/lookup`
    - **verb**: `POST`
    - **payload**:
    ```nocopy
      {"ip":"192.168.5.190"}
    ```
- Implement caching for geo-location lookups to avoid repeated HTTP requests.
- Send data to an external error count service via HTTP requests in parallel:
    - **URL**: `http://localhost:8082/error/count`
    - **verb**: `POST`
    - **payload**:
    ```nocopy
    {
        "ip":"192.168.2.12",
        "message":"Page not found.",
        "priority":2,
        "source_system":"inventory_system"
    }
    ```
- Output the enriched data to the Redpanda topic `backend_error_geo`.

> [!IMPORTANT]
> Due to tooling constraints, make sure to run the following command in the Terminal whenever you refresh the browser:.
>  ```bash,run
>  source /root/.bash_profile
>   ```

## Steps:
- Cache configuration:
    - Implement caching for geo-location lookups using an in-**memory** cache.
- Geo-location enrichment:
        - Extract the `client_ip` from the incoming message.
        - Check if the `client_ip` has already been cached.
        - If the IP address is not cached, send an HTTP POST request to `http://localhost:8081/geo/lookup` to retrieve geo-location information.
- Parallel processing for error count :
    -  Use a workflow to allow parallel calls.
    - Extract necessary fields (`client_ip`, `source_system`, `message`, `priority`) from the message.
    - Send an HTTP POST request to `http://localhost:8082/error/count` to update the error count for the system.
- Output Configuration:
    - After enrichment, output the processed data to the Redpanda topic `backend_error_geo`.

In the [button label="Editor"](tab-1), under the working directory (`~/masterclass-connect/lab-03`), you should see a `rpcn.yaml` file. Go ahead and create your pipeline in it. To test and run the pipeline, simply go to the [button label="Terminal"](tab-0) and run:

```bash,run
source /root/.bash_profile
rpk connect run -e .env rpcn.yaml
```


## Test your pipeline:
### Start Redpanda
Check if your Redpanda streaming platform is running by going to the [button label="Redpanda Console"](tab-2), If it’s not running, please execute the following command:
```bash,run
cd /root/masterclass-connect
docker-compose up -d
cd /root/masterclass-connect/lab-03
```

### Start HTTP external service
Start the external services needed for data enrichment.  . Go to [button label="Terminal A"](tab-3) and run the following command:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-03
rpk connect streams -r cache-config.yaml error-count.yaml  geo-location.yaml
```
These services are also written in Redpanda Connect—feel free to take a look at how they are implemented in the [button label="Editor"](tab-1)

### Feeding more data
You can feed the topic with more data by re-running the previous pipeline. Go to  [button label="Terminal B"](tab-4) run:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-03/solution
rpk connect streams -e .env rpcn-01.yaml rpcn-02.yaml
```

## Expected Outcome:
For each message, geo-location data will be enriched using an external API, with caching applied to avoid redundant lookups. The caching mechanism will significantly reduce the number of API calls for frequently encountered IPs, improving the pipeline’s efficiency. The enriched messages will be produced to the `backend_error_geo` topic, ready for further processing or analysis.

In  [button label="Redpanda Console"](tab-2), under topic `backend_error_geo`, you should see data ingested, with geo_location data added to each entry
![Screenshot 2024-10-02 at 12.01.56 AM.png](../assets/Screenshot%202024-10-02%20at%2012.01.56%E2%80%AFAM.png)

This setup ensures efficient, real-time enrichment of streaming data while optimizing performance with caching for repeated geo-location lookups.


Solution
===
Here is one possible solution:
```copy,yaml
input:
  kafka_franz:
    seed_brokers: [ ${RP_ADDRESS} ]
    topics: ["backend_error"]
    consumer_group: "rpcn-lab3"
pipeline:
  processors:
    - workflow:
        meta_path: meta.workflow
        branches:
          geo_location:
            request_map: |
              root.ip = this.client_ip
            processors:
              - cached:
                  key: '${! ip }'
                  cache: ip_cache
                  processors:
                    - http:
                        url: http://localhost:8081/geo/lookup
                        verb: POST
            result_map: |
              root.ip = deleted()
              root.ip_continent = deleted()
              root.geo_info = this.geo_info
          error_count:
            request_map: |
              root.ip = this.client_ip
              root.source_system = this.source_system
              root.message = this.message
              root.priority = this.priority
            processors:
              - http:
                  url: http://localhost:8082/error/count
                  verb: POST
output:
  kafka_franz:
    seed_brokers: [ ${RP_ADDRESS} ]
    topic: "backend_error_geo"
cache_resources:
  - label: ip_cache
    memory:
      default_ttl: 360s
```

### Feeding more data
Make sure you are feeding new data into the system. In  [button label="Terminal A"](tab-3) run:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-03/solution
rpk connect streams -e .env rpcn-01.yaml rpcn-02.yaml
```

### Running the solution
To run this simply go to the solution folder, and in  [button label="Terminal"](tab-0) run
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-03/solution
rpk connect run -e .env rpcn.yaml
```

This pipeline does not have any logs, simply go to [button label="Redpanda Console"](tab-2) and the result in `backend_error_geo` topic.
![Screenshot 2024-10-02 at 12.01.56 AM.png](../assets/Screenshot%202024-10-02%20at%2012.01.56%E2%80%AFAM.png)

Type `ctrl+c` to shutdown the pipeline.
## Use Case:
You are tasked with building a streaming pipeline to handle error data by grouping and aggregating it based on the `source_system`. Additionally, you need to return to the original file consumption pipeline and control the rate at which log data is processed to make it easier to observe the results over time.
![Screenshot 2024-10-01 at 10.21.24 PM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/35fad0e2657427ede4afdf6a2367b2e9/assets/Screenshot%202024-10-01%20at%2010.21.24%E2%80%AFPM.png)

Here’s what the input payload looks like from the `backend_error_geo` topic:
```
{
    "client_ip": "192.168.2.205",
    "context": {
        "method": "POST",
        "path": "/login",
        "status_code": 403,
        "user_id": "user876"
    },
    "errors": [
        {
            "code": "E404",
            "description": "Not Found",
            "details": "The requested resource was not found."
        },
        {
            "code": "E500",
            "description": "Server Error",
            "details": "Internal server error occurred."
        },
        {
            "code": "E505",
            "description": "HTTP Version Not Supported",
            "details": "The server does not support the HTTP version used."
        },
        {
            "code": "E502",
            "description": "Bad Gateway",
            "details": "Received an invalid response from the upstream server."
        },
        {
            "code": "E503",
            "description": "Service Unavailable",
            "details": "The server is temporarily unavailable."
        }
    ],
    "event_id": "0049dc23-b78b-4bbf-af90-734f0ac2659a",
    "geo_info": {
        "city": "Accra",
        "country": "Ghana",
        "timezone": "GMT"
    },
    "log_level": "ERROR",
    "message": "Page not found.",
    "meta": {
        "workflow": {
            "succeeded": [
                "error_count",
                "geo_location"
            ]
        }
    },
    "metadata": [
        {
            "key": "browser",
            "value": "Safari"
        },
        {
            "key": "os",
            "value": "Linux"
        }
    ],
    "priority": 3,
    "source_system": "auth_service",
    "timestamp": "2024-10-01T15:42:21Z"
}
```
## Objective:

- **Aggregation**: Group error data by `source_system` and calculate statistics such as the unique number of IP addresses and the occurrence of each priority level (1, 2, and 3). Use windowing to produce aggregated results at regular intervals.
- **Rate Limiting**: Slow down the consumption of log data to allow better visibility of the aggregation results over time.

```nocopy
{
   "error_count":36,
   "priority_01":17,
   "priority_02":10,
   "priority_03":9,
   "source_system":"inventory_system"
}
```
> [!IMPORTANT]
>  Due to tooling constraints, make sure to run the following command in the Terminal whenever you refresh the browser.
>  ```bash,run
>  source /root/.bash_profile
>   ```

## Steps:
- **Build the Aggregation Pipeline**:
  - Configure the pipeline to consume data from a Redpanda topic.
  - Use a grouping mechanism to group the data by the `source_system` field.
  - Implement logic to aggregate the data, calculating:
    - The unique number of IP addresses.
      ```nocopy
      {
        "source_system" : "inventory_system",
        "error_count" : 36
      }
      ```
    - The count of messages with priority levels 1, 2, and 3.
      ```nocopy
      {
        "priority_01" : 5,
        "priority_02" : 2,
        "priority_03" : 6
      }
      ```
  - Use the default tumbling window strategy to produce aggregated results every **3** seconds.

- **Implement the Rate Limiting Pipeline**:
  - Set the pipeline processes  `100` lines of log entries every `2` seconds, slowing down the consumption rate to observe the impact over time. (You'll find a file named `rpcn-01.yaml` that you can update.)

In the [button label="Editor"](tab-1), under the working directory (`~/masterclass-connect/lab-04`), you should see a `rpcn.yaml` file. Go ahead and create your pipeline in it. To test and run the pipeline, simply go to the [button label="Terminal"](tab-0) and run:

```bash,run
source /root/.bash_profile
rpk connect run -e .env rpcn.yaml
```

To test your rate limit pipeline :

```bash,run
source /root/.bash_profile
rpk connect run -e .env rpcn-01.yaml
```


## Test your pipeline:
### Start Redpanda
Check if your Redpanda streaming platform is running by going to the [button label="Redpanda Console"](tab-2), If it’s not running, please execute the following command:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect
docker-compose up -d
cd /root/masterclass-connect/lab-04
```

### During development
Sometimes it’s easier to control the input before configuring it. You can either do that by defining a unit test or simply replace the input with `stdin`:

```yaml,copy
input:
  stdin:
    scanner:
       lines: {}
output:
  stdout: {}
```

And here is a log for you to test the mapping:
```json,run
{"client_ip":"192.168.1.95","context":{"method":"DELETE","path":"/profile","status_code":403,"user_id":"user476"},"errors":[{"code":"E401","description":"Unauthorized","details":"User tried to access restricted resource."},{"code":"E002","description":"Invalid Credentials","details":"Password does not match the user account."},{"code":"E403","description":"Forbidden","details":"User does not have access to this resource."},{"code":"E406","description":"Not Acceptable","details":"Content type or encoding not acceptable."},{"code":"E407","description":"Proxy Authentication Required","details":"User must authenticate with a proxy server."}],"event_id":"597c85b9-ef83-42a0-ae94-20c69e0684bf","geo_info":{"city":"Beijing","country":"China","timezone":"CST"},"log_level":"ERROR","message":"Unauthorized access attempt detected.","meta":{"workflow":{"succeeded":["error_count","geo_location"]}},"metadata":[{"key":"browser","value":"Safari"},{"key":"os","value":"macOS"}],"priority":3,"source_system":"auth_service","timestamp":"2024-09-30T20:47:11Z"}
{"client_ip":"192.168.1.54","context":{"method":"GET","path":"/profile","status_code":404,"user_id":"user513"},"errors":[{"code":"W001","description":"High Latency","details":"The server is experiencing high latency."},{"code":"E408","description":"Request Timeout","details":"The request timed out."},{"code":"W002","description":"Slow Response","details":"The server responded but took longer than expected."},{"code":"E504","description":"Gateway Timeout","details":"The upstream server failed to respond in time."},{"code":"E509","description":"Bandwidth Limit Exceeded","details":"The server has exceeded its bandwidth limits."}],"event_id":"168bee86-ea3e-4a4f-af4f-fc7f440f12bf","geo_info":{"city":"Beijing","country":"China","timezone":"CST"},"log_level":"ERROR","message":"Server latency detected.","meta":{"workflow":{"succeeded":["error_count","geo_location"]}},"metadata":[{"key":"browser","value":"Chrome"},{"key":"os","value":"Android"}],"priority":1,"source_system":"payment_gateway","timestamp":"2024-09-30T20:39:11Z"}
{"client_ip":"192.168.1.67","context":{"method":"POST","path":"/profile","status_code":401,"user_id":"user657"},"errors":[{"code":"W001","description":"High Latency","details":"The server is experiencing high latency."},{"code":"E408","description":"Request Timeout","details":"The request timed out."},{"code":"W002","description":"Slow Response","details":"The server responded but took longer than expected."},{"code":"E504","description":"Gateway Timeout","details":"The upstream server failed to respond in time."},{"code":"E509","description":"Bandwidth Limit Exceeded","details":"The server has exceeded its bandwidth limits."}],"event_id":"a162c648-50c3-4896-9e1f-390aa5c1ecd4","geo_info":{"city":"Beijing","country":"China","timezone":"CST"},"log_level":"ERROR","message":"Server latency detected.","meta":{"workflow":{"succeeded":["error_count","geo_location"]}},"metadata":[{"key":"browser","value":"Chrome"},{"key":"os","value":"macOS"}],"priority":2,"source_system":"inventory_system","timestamp":"2024-09-30T21:40:11Z"}
{"client_ip":"192.168.1.114","context":{"method":"PUT","path":"/profile","status_code":408,"user_id":"user411"},"errors":[{"code":"E404","description":"Not Found","details":"The requested resource was not found."},{"code":"E500","description":"Server Error","details":"Internal server error occurred."},{"code":"E505","description":"HTTP Version Not Supported","details":"The server does not support the HTTP version used."},{"code":"E502","description":"Bad Gateway","details":"Received an invalid response from the upstream server."},{"code":"E503","description":"Service Unavailable","details":"The server is temporarily unavailable."}],"event_id":"ff034c36-5bb8-417c-ab9d-1a39eb5cbd3e","geo_info":{"city":"Seoul","country":"South Korea","timezone":"KST"},"log_level":"ERROR","message":"Page not found.","meta":{"workflow":{"succeeded":["error_count","geo_location"]}},"metadata":[{"key":"browser","value":"Edge"},{"key":"os","value":"Android"}],"priority":2,"source_system":"inventory_system","timestamp":"2024-09-30T21:42:11Z"}
```

You should see something like below in the output:
```json,nocopy
{"error_count":1,"priority_01":0,"priority_02":0,"priority_03":1,"source_system":"auth_service"}
{"error_count":1,"priority_01":1,"priority_02":0,"priority_03":0,"source_system":"payment_gateway"}
{"error_count":2,"priority_01":0,"priority_02":2,"priority_03":0,"source_system":"inventory_system"}
```

### Feeding more data
You can feed new data into the system, In  [button label="Terminal A"](tab-3) run:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-04/solution
rpk connect streams -e .env -r cache-config.yaml geo-location.yaml error-count.yaml rpcn-03.yaml rpcn-02.yaml rpcn-01.yaml
```


## Expected Outcome:
- **Aggregation Pipeline**: The pipeline will continuously process error data from Redpanda, group it by source_system, and output an aggregated summary at regular intervals using windowing. The summary will include:
  - A count of unique IP addresses.
  - The number of messages with different priority levels.
- **Updated rate limiting pipeline**: The log file entries in the first pipeline will be consumed at a controlled rate, this  slowed processing will allow you to observe the effects of rate limiting in real-time.

When running the pipeline,  you should see it continuously printout error summary for all source system every 3 seconds.
![Screenshot 2024-10-02 at 12.04.10 AM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/3bc1a58a238d8ae68c8cc08987698c9b/assets/Screenshot%202024-10-02%20at%2012.04.10%E2%80%AFAM.png)

This setup allows for efficient, real-time aggregation of streaming data while applying rate limits to control log consumption, optimizing both visibility and performance.

Solution
===

Here is one possible solution:
```yaml,copy
input:
  kafka_franz:
    seed_brokers: [ ${RP_ADDRESS} ]
    topics: ["backend_error_geo"]
    consumer_group: "rpcn-lab4"
pipeline:
  processors:
    - group_by_value:
        value: '${! json("source_system") }'
    - mapping: |
        map aggregate {
          root.source_system = json("source_system")
          root.error_count = json("client_ip").from_all().unique().length()
          root.priority_01 = json("priority").from_all().find_all(1).length()
          root.priority_02 = json("priority").from_all().find_all(2).length()
          root.priority_03 = json("priority").from_all().find_all(3).length()
        }

        root = if batch_index() != 0 {
          deleted()  } else {
          root.apply("aggregate") }
output:
  stdout: {}

#output:
#  kafka_franz:
#    seed_brokers: [ ${RP_ADDRESS} ]
#    topic: "error_summary"

buffer:
  system_window:
    timestamp_mapping: root = now()
    size: 3s
    allowed_lateness: 500ms
```


Here is one possible solution for updating the rate limit pipeline:
```copy,yaml
input:
  file:
    paths: [ ${LOG_FILE} ]
    scanner:
      lines: {}
  processors:
    - rate_limit:
        resource: slower
pipeline:
  processors:
    - log:
        level: INFO
        message: Logging message from file -> ${! message }
    - mapping: |
        root = this
        root.machine_code = deleted()
        root.reference = deleted()
        root.request_id = deleted()
output:
  switch:
    cases:
      - check: this.log_level == "ERROR"
        output:
          kafka_franz:
            seed_brokers: [ ${RP_ADDRESS} ]
            topic: "raw_error"
      - check: this.log_level == "WARNING"
        output:
          kafka_franz:
            seed_brokers: [ ${RP_ADDRESS} ]
            topic: "raw_warning"
      - output:
          kafka_franz:
            seed_brokers: [ ${RP_ADDRESS} ]
            topic: "raw_info"
rate_limit_resources:
  - label: slower
    local:
      count: 100
      interval: 2s
```

### Feeding more data
You can feed new data into the system, In  [button label="Terminal A"](tab-3) run:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-04/solution
rpk connect streams -e .env -r cache-config.yaml geo-location.yaml error-count.yaml rpcn-03.yaml rpcn-02.yaml rpcn-01.yaml
```

### Running the solution
To run this simply go to the solution folder, and in  [button label="Terminal"](tab-0) run
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-04/solution
rpk connect run -e .env rpcn.yaml
```
Type `ctrl+c` to shut down the pipeline.
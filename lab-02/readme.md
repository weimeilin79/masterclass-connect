## Use Case:
Your organization is processing logs from various systems such as:
-  payment_gateway,
- auth_service
- inventory_system, and
- web_frontend.

Logs originating from the **payment_gateway** system, especially those related to latency errors, should be given higher priority. Based on the source system, you'll assign priorities to each log and remove redundant details for streamlined processing.

![Screenshot 2024-10-01 at 10.13.02 PM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/0f89f3fc70947c8be7deb9e971faaeb2/assets/Screenshot%202024-10-01%20at%2010.13.02%E2%80%AFPM.png)

In this lab, you will categorize errors by priority, remove unnecessary details from the error codes, and send processed logs to different outputs (Redpanda topic, HTTP endpoint). Additionally, logs that fail will be handled by a Dead Letter Queue (`DLQ`).

Here’s what the incoming payload looks like:
```nocopy
{
   "timestamp":"2024-10-01T16:01:21Z",
   "event_id":"1632a866-a6ab-4f6c-85a8-417bd4550a48",
   "source_system":"auth_service",
   "reference":"ZTJMKDMBNRLOYJRZGFLDQAPY",
   "machine_code":"82807888136045504903204305261984150575659914781168",
   "client_ip":"192.168.1.151",
   "log_level":"ERROR",
   "message":"Page not found.",
   "request_id":"req3630",
   "context":{
      "user_id":"user612",
      "path":"/profile",
      "method":"DELETE",
      "status_code":401
   },
   "errors":[
      {
         "code":"E404",
         "description":"Not Found",
         "details":"The requested resource was not found."
      },
      {
         "code":"E500",
         "description":"Server Error",
         "details":"Internal server error occurred."
      },
      {
         "code":"E505",
         "description":"HTTP Version Not Supported",
         "details":"The server does not support the HTTP version used."
      },
      {
         "code":"E502",
         "description":"Bad Gateway",
         "details":"Received an invalid response from the upstream server."
      },
      {
         "code":"E503",
         "description":"Service Unavailable",
         "details":"The server is temporarily unavailable."
      }
   ],
   "metadata":[
      {
         "key":"browser",
         "value":"Chrome"
      },
      {
         "key":"os",
         "value":"Linux"
      }
   ]
}
```

## Objective:
- Gets individual log from a Redpanda topic `raw_error`.
- Categorize logs by source system and assign priority.
- Drop logs from **web_frontend** as they are not required for further processing.
- Remove all detailed information from error messages, keeping only the error code and description.
- Route the transformed logs to the correct outputs Redpanda topic and HTTP endpoint.
- Implement a Dead Letter Queue (DLQ) to handle failed outputs.

> [!IMPORTANT]
>  Due to tooling constraints, make sure to run the following command in all the Terminal whenever you refresh the browser.
>  ```bash,run
>  source /root/.bash_profile
>   ```

## Steps:
- Consume data from topic `raw_error`
- Add  a `priority` field based on the `source_system`, assign a priority level to the logs:
  - `source_system` = `payment_gateway` with latency errors (`message` = `Server latency detected.`) : **Priority = 1**
  - `source_system` = `payment_gateway` or `source_system` = `inventory_system`: **Priority = 2**
  - `source_system` = `auth_service`: **Priority = 3**
  - others **Priority = 4**

For example, for a  `payment_gateway` log with a message `Server latency detected.`, you will assign a priority of 1.
Here's an example payload:
```nocopy
{
   ...
   "priority": 1,
   "source_system": "payment_gateway",
   "message":"Server latency detected."
   "timestamp": "2024-10-01T16:23:21Z"
   ...
}
```
- **Removing Detailed Error Information**: Simplify the error messages by removing the `details` field from each error object, while keeping only the `code` and `description`.
Before:
```nocopy
{
  "code": "E002",
  "description": "Invalid Credentials",
  "details": "Password does not match the user account."
}
```

After:
```nocopy
{
  "code": "E002",
  "description": "Invalid Credentials"
}
```
- Route the transformed logs,
  - If priority are smaller and equal to 3, send them to the `backend_error` topic
  - Send the rest to an non-existing HTTP Endpoint `http://localhost:1234/notexist`, since this does not exist, you will need to implement a Dead Letter Queue (DLQ), simply send the failed message to the `DLQ` topic in Redpanda.



In the [button label="Editor"](tab-1), under the working directory (`~/masterclass-connec/lab-02`), you should see a `rpcn.yaml` file. Go ahead and create your pipeline in it. To test and run the pipeline, simply go to the [button label="Terminal"](tab-0) and run:

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
cd /root/masterclass-connect/lab-02
```

### During development
Sometimes it'll be easier to control the input before you actually configure the input, you can either do that by defining a unit test or simply replace the input with `stdin`:

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
{"timestamp": "2024-09-30T20:55:11Z", "event_id": "cb323fed-610a-4115-b427-17a6b502320f", "source_system": "payment_gateway", "client_ip": "192.168.1.4", "log_level": "ERROR", "message": "Server latency detected.", "context": {"user_id": "user685", "path": "/profile", "method": "GET", "status_code": 403}, "errors": [{"code": "W001", "description": "High Latency", "details": "The server is experiencing high latency."}, {"code": "E408", "description": "Request Timeout", "details": "The request timed out."}, {"code": "W002", "description": "Slow Response", "details": "The server responded but took longer than expected."}, {"code": "E504", "description": "Gateway Timeout", "details": "The upstream server failed to respond in time."}, {"code": "E509", "description": "Bandwidth Limit Exceeded", "details": "The server has exceeded its bandwidth limits."}], "metadata": [{"key": "browser", "value": "Chrome"}, {"key": "os", "value": "Windows"}]}
```

You should see something like below in the output:
```json,nocopy
{
   ....
   "errors":[
      {
         "code":"W001",
         "description":"High Latency",
         "details":"The server is experiencing high latency."
      },
      {
         "code":"E408",
         "description":"Request Timeout",
         "details":"The request timed out."
      },
      {
         "code":"W002",
         "description":"Slow Response",
         "details":"The server responded but took longer than expected."
      },
      {
         "code":"E504",
         "description":"Gateway Timeout",
         "details":"The upstream server failed to respond in time."
      },
      {
         "code":"E509",
         "description":"Bandwidth Limit Exceeded",
         "details":"The server has exceeded its bandwidth limits."
      }
   ],
   .....
   "priority":1,
   "source_system":"payment_gateway",
   "timestamp":"2024-09-30T20:55:11Z"
}
```

### Feeding more data
You can feed the topic with more data by re-running the previous pipeline. Go to  [button label="Terminal A"](tab-3)
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-02/solution
rpk connect run -e .env rpcn-01.yaml
cd /root/masterclass-connect/lab-02
```


## Expected Outcome:
- **Frontend logs** will fail and are routed to a Dead Letter Queue(DLQ).
- In  [button label="Redpanda Console"](tab-2), under `backend_error` topic, you should see data ingested with the priority added and the error messages simplified.
![Screenshot 2024-10-01 at 11.50.26 PM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/d6bd593a411982ddd075ee4327162755/assets/Screenshot%202024-10-01%20at%2011.50.26%E2%80%AFPM.png)
- In the `DLQ`, you'll find a couple of logs with `web_frontend` as `source_system`.
![Screenshot 2024-10-01 at 11.50.59 PM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/dc4ad350c1d566fd830846ea888b3b53/assets/Screenshot%202024-10-01%20at%2011.50.59%E2%80%AFPM.png)

This configuration ensures that data can be transformed and routed dynamically based on its contents, providing flexibility and scalability in real-time data processing workflows.


Solution
===
Here’s one possible solution:
```copy
input:
  kafka_franz:
    seed_brokers: [ ${RP_ADDRESS} ]
    topics: ["raw_error"]
    consumer_group: "rpcn-lab2"
pipeline:
  processors:
    - mapping: |
        root = this
        root.priority = match this {
            this.source_system == "payment_gateway" && this.message == "Server latency detected."   => 1 ,
            this.source_system == "payment_gateway" ||  this.source_system == "inventory_system" => 2 ,
            this.source_system == "auth_service"  => 3 ,
            _ => 4,
        }
    - mapping: |
        root = this
        map simplify {
          root = this
          root.code = this.code
          root.description = this.description
        }
        root.errors = this.errors.map_each(error -> error.apply("simplify"))
    - log:
        level: INFO
        message: "Priority: ${! priority }"
output:
  switch:
    cases:
      - check: this.priority <= 3
        output:
          kafka_franz:
            seed_brokers: [ ${RP_ADDRESS} ]
            topic: "backend_error"
      - output:
          fallback:
            - reject_errored:
                kafka_franz:
                  seed_brokers: [ ${RP_ADDRESS} ]
                  topic: "DLQ"
            - http_client:
                url: http://localhost:1234/notexist
                retries: 1
```

### Feeding more data
Make sure you are feeding new data into the system.  In  [button label="Terminal A"](tab-3) run:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-02/solution
rpk connect run -e .env rpcn-01.yaml
```

### Running the solution
To run this, go to the solution folder and in  [button label="Terminal"](tab-0), run:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-02/solution
rpk connect run -e .env rpcn.yaml
```
Press `ctrl+c` to shut down the pipeline.
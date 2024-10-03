## Use Case:
Your organization needs to monitor server logs in real-time to detect issues, errors, and suspicious activities.
![Screenshot 2024-10-01 at 10.12.16 PM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/a9314ea8fff4a0eb4c6617124460583c/assets/Screenshot%202024-10-01%20at%2010.12.16%E2%80%AFPM.png)
Here's a sample error:
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
You need to monitor server logs in real-time to detect anomalies such as unauthorized access attempts, server errors, or suspicious activities. The logs will be ingested from a file, split, processed, and sent to Redpanda for further analysis.

Here's what the log file looks like
```nocopy
{"timestamp": "2024-10-01T16:01:21Z", "event_id": "1632a866-a6ab-4f6c-85a8-417bd4550a48", "source_system": "auth_service", "reference": "ZTJMKDMBNRLOYJRZGFLDQAPY", "machine_code": "82807888136045504903204305261984150575659914781168", "client_ip": "192.168.1.151", "log_level": "ERROR", "message": "Page not found.", "request_id": "req3630", "context": {"user_id": "user612", "path": "/profile", "method": "DELETE", "status_code": 401}, "errors": [{"code": "E404", "description": "Not Found", "details": "The requested resource was not found."}, {"code": "E500", "description": "Server Error", "details": "Internal server error occurred."}, {"code": "E505", "description": "HTTP Version Not Supported", "details": "The server does not support the HTTP version used."}, {"code": "E502", "description": "Bad Gateway", "details": "Received an invalid response from the upstream server."}, {"code": "E503", "description": "Service Unavailable", "details": "The server is temporarily unavailable."}], "metadata": [{"key": "browser", "value": "Chrome"}, {"key": "os", "value": "Linux"}]}
{"timestamp": "2024-10-01T15:46:21Z", "event_id": "4690a918-c7bd-49d5-98b0-a7d42c801454", "source_system": "web_frontend", "reference": "OXUXMDLCRIHMGIIIYODNQWIM", "machine_code": "14890859593576023560386059813420137650691499505283", "client_ip": "192.168.1.192", "log_level": "INFO", "message": "Unauthorized access attempt detected.", "request_id": "req5403", "context": {"user_id": "user907", "path": "/signup", "method": "GET", "status_code": 401}, "errors": [], "metadata": [{"key": "browser", "value": "Firefox"}, {"key": "os", "value": "Linux"}]}
{"timestamp": "2024-10-01T16:19:21Z", "event_id": "63193b86-42c6-4f3c-b18f-8783bf380554", "source_system": "payment_gateway", "reference": "CYQORMJMRJPZHTOXLRYHACWK", "machine_code": "59309362232180144714532699817581355358865884571789", "client_ip": "192.168.4.194", "log_level": "WARNING", "message": "Server latency detected.", "request_id": "req8492", "context": {"user_id": "user441", "path": "/signup", "method": "PUT", "status_code": 200}, "errors": [], "metadata": [{"key": "browser", "value": "Firefox"}, {"key": "os", "value": "macOS"}]}
{"timestamp": "2024-10-01T16:09:21Z", "event_id": "01dd11b2-e6a9-41e9-8900-23fbcd2b1e86", "source_system": "auth_service", "reference": "QFEXANSJXPAJPIGLRDONNICX", "machine_code": "41674976067860007992568402779809391983062091043895", "client_ip": "192.168.2.69", "log_level": "INFO", "message": "System running smoothly.", "request_id": "req3626", "context": {"user_id": "user815", "path": "/login", "method": "GET", "status_code": 500}, "errors": [], "metadata": [{"key": "browser", "value": "Chrome"}, {"key": "os", "value": "Windows"}]}
{"timestamp": "2024-10-01T16:53:21Z", "event_id": "0bd09b5e-c415-4fa1-86b9-21f836bf2ea2", "source_system": "payment_gateway", "reference": "XOBHGBLGFDZOYKPBZZCLJFYY", "machine_code": "73060981189949906453282115330844380127567476217866", "client_ip": "192.168.1.72", "log_level": "INFO", "message": "Page not found.", "request_id": "req5668", "context": {"user_id": "user871", "path": "/dashboard", "method": "DELETE", "status_code": 200}, "errors": [], "metadata": [{"key": "browser", "value": "Firefox"}, {"key": "os", "value": "Windows"}]}
{"timestamp": "2024-10-01T15:44:21Z", "event_id": "d1611598-7c14-4938-82d4-b88b4a6faba1", "source_system": "inventory_system", "reference": "KEDIWEAPVFFWOCUEWVCTUSDZ", "machine_code": "22197713276748067785721143584475923825026015216715", "client_ip": "192.168.5.173", "log_level": "INFO", "message": "Server latency detected.", "request_id": "req8622", "context": {"user_id": "user914", "path": "/settings", "method": "GET", "status_code": 401}, "errors": [], "metadata": [{"key": "browser", "value": "Chrome"}, {"key": "os", "value": "Linux"}]}
```

## Objective:
- Read server logs from a file (`log_entries.txt`).
- Log each entry.
- Extract specific fields and filter out unwanted ones.
- Dynamically route logs to Redpanda topics.

> [!IMPORTANT]
>  Due to tooling constraints, make sure to run the following command in all the Terminal whenever you refresh the browser.
>  ```bash,run
>  source /root/.bash_profile
>   ```

## Steps:

-  Read log entries from the file:
   -  Use the `file` input to read server logs from `log_entries.txt`.
-  Split log entries:
   -  Split the file content into individual log lines.
-  Log each entry:
   -  Add a `log` processor to monitor each log entry.
- Extract log data :
   - Use a `jq` or `mapping` to filter the unwanted field, `reference`, `machine_code` and `request_id`
- Route logs to Redpanda topics:
   - Route logs to Kafka topics based on their severity.
      - Logs with level `ERROR` to `raw_error`
      - Logs with level `WARNING` to `raw_warning`
      -  Logs with level `INFO` to `raw_info`

In the [button label="Editor"](tab-1), under the working directory (`~/masterclass-connect/lab-01`), you should see a `rpcn.yaml` file. Go ahead and create your pipeline in it. To test and run the pipeline, simply go to the [button label="Terminal"](tab-0) and run:

```bash,run
source /root/.bash_profile
rpk connect run -e .env rpcn.yaml
```

## Test your pipeline:
### Environment variables setup:
Try to use an environment variable file instead of hard-coding everything in the pipeline. You’ll find a `.env` file in the working directory that already contains the following variables:
```nocopy
LOG_FILE ="log_entries.txt"
RP_ADDRESS="localhost:19092"
```
### Start Redpanda
```bash,run
cd /root/masterclass-connect
docker-compose up -d
cd /root/masterclass-connect/lab-01
```

## Expected Outcome:
Logs are dynamically logged, split, and routed to Redpanda topic. The message  is rerouted based on the `log_level`.
In  [button label="Redpanda Console"](tab-2), under topic, you should see logs in the 3 newly created topics:
![Screenshot 2024-10-01 at 11.54.30 PM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/b8cb5f32719be7ffd306230e93b1ee96/assets/Screenshot%202024-10-01%20at%2011.54.30%E2%80%AFPM.png)
- raw_error
- raw_warning
- raw_info
![Screenshot 2024-10-01 at 11.49.54 PM.png](https://play.instruqt.com/assets/tracks/pw95vxordilg/4f03274e1c954a12e23e81443fe2316a/assets/Screenshot%202024-10-01%20at%2011.49.54%E2%80%AFPM.png)
This setup enables real-time ingestion of streaming data from Redpanda, ensuring seamless integration into the pipeline, and efficiently preparing the data for further transformation and processing.

Solution
===

Here is one possible solution:
```nocopy,yaml
input:
  file:
    paths: [ ${LOG_FILE} ]
    scanner:
      lines: {}
pipeline:
  processors:
    - log:
        level: INFO
        message: Logging message from file -> ${! message }
    - jq:
        query: 'del(.reference, .machine_code, .request_id)'
        raw: false
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
```
You can view the `log_entries.txt` file under the `lab-01` folder in the [button label="Editor"](tab-1)

### Running the solution
To run this, simply go to the solution folder, and in the [button label="Terminal"](tab-0) run:
```bash,run
source /root/.bash_profile
cd /root/masterclass-connect/lab-01/solution
rpk connect run -e .env rpcn.yaml
```
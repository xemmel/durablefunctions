# durablefunctions
# Integration-IT
## Morten la Cour


DURABLE FUNCTIONS


func init durabledemotemp --worker-runtime dotnet

func new -t DurableFunctionsOrchestration -n firstdurablefunction


Creates 3 functions in same file:

Normal (starter) in this case an HTTPTrigger

```csharp

   [FunctionName("firstdurablefunction_HttpStart")]
        public static async Task<HttpResponseMessage> HttpStart(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequestMessage req,
            [DurableClient] IDurableOrchestrationClient starter,
            ILogger log)
        {
            // Function input comes from the request content.
            string instanceId = await starter.StartNewAsync("firstdurablefunction", null);

            log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

            return starter.CreateCheckStatusResponse(req, instanceId);
        }

```

Note: [DurableClient] Parameter.


```csharp

  [FunctionName("firstdurablefunction")]
        public static async Task<List<string>> RunOrchestrator(
            [OrchestrationTrigger] IDurableOrchestrationContext context)
        {
            var outputs = new List<string>();

            // Replace "hello" with the name of your Durable Activity Function.
            outputs.Add(await context.CallActivityAsync<string>("firstdurablefunction_Hello", "Tokyo"));
            outputs.Add(await context.CallActivityAsync<string>("firstdurablefunction_Hello", "Seattle"));
            outputs.Add(await context.CallActivityAsync<string>("firstdurablefunction_Hello", "London"));

            // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
            return outputs;
        }

```

```csharp

        [FunctionName("firstdurablefunction_Hello")]
        public static string SayHello([ActivityTrigger] string name, ILogger log)
        {
            log.LogInformation($"Saying hello to {name}.");
            return $"Hello {name}!";
        }

```

Normal csharp code. Use Tasks (async /await) for logic




Start the HttpTrigger

```powershell

Clear-Host;
$functionAppName = "";
$functionName = "firstdurablefunction_HttpStart";

$url = "http://localhost:7071/api/firstdurablefunction_HttpStart";
$url = "http://localhost:7071/api/humandurable_HttpStart";

$response = invoke-webrequest -uri $url;

$responseObject = $response.Content | ConvertFrom-Json;
$responseObject;

```


### Get Status

```powershell

Clear-Host;
$response = invoke-webrequest -uri $responseObject.statusQueryGetUri;
$response.Content | ConvertFrom-Json;



```




testhubname (default)



### Change to parallel

```csharp

           List<Task<string>> tasks = new();
            tasks.Add(context.CallActivityAsync<string>("firstdurablefunction_Hello", "Tokyo"));
            tasks.Add(context.CallActivityAsync<string>("firstdurablefunction_Hello", "Seattle"));
            tasks.Add(context.CallActivityAsync<string>("firstdurablefunction_Hello", "London"));
            
            await Task.WhenAll(tasks);
            outputs.AddRange(tasks.Select(t => t.Result));

```


## Human interaction


```csharp

            using var timeoutCts = new CancellationTokenSource();
            DateTime dueTime = context.CurrentUtcDateTime.AddMinutes(2);
            Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);
            Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
            if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
            {
                timeoutCts.Cancel();
                //await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
                System.Console.WriteLine($"Back from approver: {approvalEvent.Result}");
            }
            else
            {
                //await context.CallActivityAsync("Escalate", null);
                System.Console.WriteLine("Timeout");
            }



```


### Give approval

```powershell

Clear-Host;

$baseUrl = "http://localhost:7071";
$instanceId = "5cb7f9248b5a4a218036513747b8ed74";
$eventName = "ApprovalEvent";

$body = "false";

$url = "$baseUrl/runtime/webhooks/durabletask/instances/$instanceId/raiseEvent/$eventName";

$response = invoke-webrequest -uri $url `
		-Body $body `
		-Method Post `
		-Headers @{ "Content-Type" = "application/json" };
$response;




```
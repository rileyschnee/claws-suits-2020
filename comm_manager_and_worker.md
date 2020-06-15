# Communications Manager and Worker
The communications manager is designed to help facilitate communications with the HoloLens and the MCC. This makes it easier to send and receive JSON. In the future, there will be added support for sending and receiving media. 

<h2> Using the Communications Manager</h2>

There is no need to use the communications worker, as this is called by the communications manager and the communications manager should be interfaces with.

To use the communications manager, you need a url you would like to interface with and a function to call after the communication has occurred. It is important to note that UnityWebRequests (which are what the Communications Manager is built off of), are asynchronous. Lets look at an example to understand this better and how to use the CommunicationsManager

```
private string MCC_Send_Heartbeat_Url = "[our_server]/Send_Heartbeat;
private string MCC_Get_Heartbeat_Url = "[our_server]/Get_Heartbeat";

private void SendMCCHeartbeat()
{
  private string json = "{\"Heartbeat\":\"HoloLens Happy\"}";
  // Send the above json to the MCC url specified and call the CheckMCCReceiveHeartbeatfunction afterwards
  CommunicationsManager.SendJSON(MCC_Send_Heartbeat_Url, json, CheckMCCReceiveHeartbeat);
  // Any code after this may execute before the request is sent to the MCC because of the asynchronous nature of UnityWebRequests. Put code you want to run after communications have occurred in the CheckMCCReceiveHeartbeat function instead.
}
  
// The status variable is the HTTP status received from the MCC
private void CheckMCCReceiveHeartbeat(long status)
{
  // Code run after MCC Communications have occurred 
  if (status == 200)
  {
    Debug.Log("Communications link all good!");
  }
  Debug.Log("An error! Oh no :(");
}

private void GetMCCHeartbeat()
{
  // Get the JSON from the MCC url specified and call the GetMCCStatus afterwards
  CommunicationsManager.GetJSON(MCC_Get_Heartbeat_Url, GetMCCStatus);
  // Any code after this may execute before the request is sent to the MCC because of the asynchronous nature of UnityWebRequests. Put code you want to run after communications have occurred in the GetMCCStatus function instead.
}

// The status variable is the HTTP status received from the MCC. THe JSON is the received JSON from the MCC
private void GetMCCStatus(long status, string json)
{
  // Code run after MCC Communications have occurred 
  if (status == 200)
  {
    Debug.Log("Communications link all good!");
    Debug.Log("Received JSON " + json);
  }
  Debug.Log("An error! Oh no :(");
}
```

When sending JSON to the MCC you must specify a function to call afterwards that takes a long as a parameter. This long will contain the HTTP response from the MCC. When receiving JSON from the MCC you must specify a function to call afterwards that takes a long and a JSON string as the parameters which will contrain the HTTP response and the received JSON.

<h2>Understanding the CommunicationsManager and Worker</h2>

The communications manager uses the MCC endpoint being hit and the function to be called afterwards as a unique identifier. This unique identifier is used to make sure to limit the number of requests made to the MCC, where the HoloLens will wait before making identical requests until the previous one has been served. The Manager will make a worker for each request that will then perform the send or get request and then call the desired function. Finally the unique identifier will be released so another identical request can be handled in the future

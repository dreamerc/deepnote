# Usage

## Second Life Chat to Discord Webhook Channel
```csharp
// Original from https://github.com/TBGRenfold/discord-lsl

key REQUEST_KEY;

string WEBHOOK_CHANNEL = "XXXXXXXXXXXXXXXXXXXXXXXXXXXX";
string WEBHOOK_TOKEN =  "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX";
string WEBHOOK_URL = "https://discordapp.com/api/webhooks/";
integer WEBHOOK_WAIT = TRUE;

string slurl(key AvatarID)
{
    string regionname = llGetRegionName();
    vector pos = llList2Vector(llGetObjectDetails(AvatarID, [ OBJECT_POS ]), 0);
 
    return "http://maps.secondlife.com/secondlife/"
        + llEscapeURL(regionname) + "/"
        + (string)llRound(pos.x) + "/"
        + (string)llRound(pos.y) + "/"
        + (string)llRound(pos.z) + "/";
}

key PostToDiscord(key AvatarID, string Message)
{
    list json = [ 
        "username", llGetObjectName() + "",
        "content", llGetUsername(AvatarID) + "" + Message
    ];
    string query_string = "";
    if (WEBHOOK_WAIT)
        query_string += "?wait=true";

    return llHTTPRequest(WEBHOOK_URL + WEBHOOK_CHANNEL + "/" + WEBHOOK_TOKEN + query_string, 
    [ 
        HTTP_METHOD, "POST", 
        HTTP_MIMETYPE, "application/json",
        HTTP_VERIFY_CERT,TRUE,
        HTTP_VERBOSE_THROTTLE, TRUE,
        HTTP_PRAGMA_NO_CACHE, TRUE ], llList2Json(JSON_OBJECT, json));
}

integer listenHandle;

default
{
    state_entry()
    {
        listenHandle = llListen(0, "", NULL_KEY, "");
    }

    listen(integer channel, string name, key id, string message)
    {
        PostToDiscord("", name + " : " + message);
//      llListenRemove(listenHandle);
    }

```
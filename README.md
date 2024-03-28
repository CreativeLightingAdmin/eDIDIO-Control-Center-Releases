# eDIDIO_Control_Center
A Windows-based Tray Application for Lighting Control via an eDIDIO Series 10

This application provides a small customisable system tray icon with sliders and buttons for controlling DALI lights. 

# Scripting Control
This application provides scripting control based on LUA. The commands are read line by line from a script located in "Appdata/Local/eDIDIO_Control_Center"

# Commands
You can call linked commands from the LUA script to run functions in the application. These are listed below.

| Command | Description |
| --- | --- |
| ShowNotification(string) | List all new or modified files |
| GetTime(time) | Retrieves a time String with format "MM/dd/yyyy HH:mm:ss" |
| DALIArcMessage(line, address, level) | Sends a DALI Arc Level (0-254) |
| DALIQueryStatus(line, address) | Queries the DALI Status of a device |
| DALIQueryLevel(line, address | Queries the DALI Level of a device |
| DALIQuery(line, address, query) | Sends any Query - Note 'packet >> 24' provides the packet type |
| SendDALIFrame(line, frame, is24bit_bool) | Sends a 16 or 24 bit DALI frame |
| SendDALIFrameWithReply(line, frame, is24bit_bool) | Sends a 16 or 24 bit DALI frame with Reply |
| DALIColour(line, address, red, green, blue) | Sends R,G,B colour to address-address + 2 |
| SendeDIDIOTrigger(line, target, query, type, zone, value) | See ProtoBufs |
| HttpPost(baseAddress, address, json) | Sends a HttpPost Request |
| HttpGet(baseAddress, address) | Sends a HttpGet Request |
| ParseLastGet(key) | Returns the value matching the key in the last HttpGet |
| SleepScript(time) | Sleeps for time (ms) |
| TaskSchedulerInterval(name, repeat_bool, interval_s, script) | Schedules a script to call every interval_s seconds. It will run now, and then wait interval_s. If not repeating, it will not run immediatly |
| TaskSchedulerCron(name, cron, script) | Schedules a script based on cron |


# Enums
* DALI Address - ADDRESS_0 to ADDRESS_63
* DALI Groups - GROUP_0 to GROUP_15
* DALI Broadcast - BROADCAST

# DALI Packet Types (packet >> 24)
* Received 8 Bit Frame - 0x03
* Received 16 Bit Frame - 0x04
* Received 24 Bit Frame - 0x05
* Received Partial Frame - 0x06
* Error or Waiting - All other values

Note - To extract the message in lua, use bit32.band(0xFFFFFF, variable)

# Examples
To send a standard DALI packet - Line 1, Address 0 to 254
```
DALIArcMessage(LINE_1, ADDRESS_0, 254)
```

You can use LUA logic to perform more advanced calls. Fade over 25 seconds
```
for i = 255,1,-10 do DALIArcMessage(LINE_1, ADDRESS_0, i) SleepScript(1000) end
```
or toggle a fitting between ON and OFF
```
variable = DALIQueryLevel(LINE_1, ADDRESS_0)
if bit32.band(0xFFFFFF, variable) > 0 then DALIArcMessage(LINE_1, ADDRESS_0, 0) else DALIArcMessage(LINE_1, ADDRESS_0, 254) end
```
For 3rd party connection, the HttpGet and HttpSet requests can be used. For Madrix Lighting Control Software Control
```
version = HttpGet(Madrix IP, "RemoteCommands/?GetVersion") -- Madrix Get Version Number
ShowNotification(version)
HttpPost(Madrix IP, "RemoteCommands/SetStorage=S1P25", "") -- Madrix Cue and Play S1P25
```
For Ethernal API connections - Example Weather API
```
weather = HttpGet("https://api.open-meteo.com/", "v1/forecast?latitude=-34.9112&longitude=138.7073&hourly=temperature_2m&timezone=Australia%2FSydney")
```

For Scheduling, create a new script "ExampleUserScript.Lua". This example will create a scheduled Task for every 10 seconds which will record the DALI level of Line 1 Address 0.

Script1.lua
```
TaskSchedulerCron("Test", "0 0/1 * * * ?", "[AppData]\\eDIDIO Control Center\\ExampleUserScript.lua")
```

ExampleUserScript.lua
```
file = io.open("LogLevel.txt","a") 
file:write("\n") -- Start a New Line
time = GetTime("MM/dd/yyyy HH:mm:ss")
level = DALIQueryLevel(LINE_1, ADDRESS_0)
file:write(time .. " DALI Level: " .. level)
file:close()
```

# Limitations
No LUA Library Support. You can't import additional LUA libraries

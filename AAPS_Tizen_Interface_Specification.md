# AAPS - Tizen specification

[TOC]

## SAP Settings (to be confirmed, not working today)

Android Application is Consumer

Watch application is Provider

Standard implementation as describe in [SDK](https://developer.samsung.com/galaxy-accessory/download.html) or available in provided [native examples](https://developer.samsung.com/galaxy-watch-develop/creating-your-first-app/native-companion/setup-sdk.html#)

accessoryservices.xml (Tizen side) is:

```
<?xml version="1.0" encoding="UTF-8"?>
<resources>
   <application name="org.tizen.androidapsprovider">
      <serviceProfile
         id="/androidaps/tizen"
         name="AAPS"
         autoLaunchAppId="org.tizen.androidapsprovider"
         role="provider"
         version="1.0">
         <supportedTransports>
            <transport type="TRANSPORT_BT" />
            <transport type="TRANSPORT_WIFI" />
         </supportedTransports>
         <serviceChannel id="104" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="105" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="110" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="115" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="120" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="125" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="200" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="205" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="210" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="220" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="225" dataRate="low" priority="low" reliability="enable" />
         <serviceChannel id="230" dataRate="low" priority="low" reliability="enable" />
      </serviceProfile>
   </application>
</resources>
```

**Channel list needs to be updated (104 only for testing, other channels will probably be necessary)**

| Channel | Use                                                          | Direction  |
| ------- | ------------------------------------------------------------ | ---------- |
| 104     | test and debug                                               | both       |
| 105     | Send Status                                                  | AAPS→Watch |
| 110     | Send BG datas                                                | AAPS→Watch |
| 115     | Send Preference (to watch) or send Preference mask (to AAPS) | both       |
| 120     | Resend BG history datas (to watch) or request resend (to AAPS) | both       |
| 125     | Send basals (+ all graph datas)                              | AAPS→Watch |
| 200     | Initiate action                                              | Watch→AAPS |
| 205     | Confirmation Request (to watch) or Confirm Action (to AAPS)  | both       |
| 210     | Send Cancel notification                                     | AAPS→Watch |
| 220     | Bolus Progress                                               | AAPS→Watch |
| 225     | Cancel Bolus                                                 | Watch→AAPS |
| 230     | Open settings                                                | AAPS→Watch |



## Data exchanged for Watchface update

- All informations for watch face are sent as string excepted informations concerning graph

- Settings for watch faces are not allowed by Samsung (API not available), only build in and some third parties watch faces have this functionality with a long press on watch face)
  - this point needs to be improved in current WearOS watch faces available with AAPS (watch face settings are split between AndroidAPS and Wear app)
  - I propose to manage all watch face settings in AndroidAPS (to be defined), I included a long value (allow 64 On/Off settings that should be enough for all data and all Watch faces) in preference Json string to send all settings. Later we can define a dedicated Json string sent by Watch face to androidAPS to define a "Mask" and watch face name (for showing only available settings in androidAPS)

### SendStatus (AndroidAPS→Watchface)

Channel: 105

Type of data: JSON string (Array of bytes)

(note, specification below needs to be updated for watchface setting management)

```java
{
	"externalStatusString": String,
	"wearsettings":Long,	// All settings for watch (WearControl and watch face settings), a long value allow 64 On/Off settings that should be enough...
	"iobSum": String,	// Total IOB converted to string with 2 decimal and localisation
	"iobDetail": String,	// Detailled IOB converted to string with 2 decimal "(xx|yy)"
	"cob": String,		// Cob value converted in string
	"currentBasal":String,	//percentage or basal rate ("U/h" according to activeTemp or not)
	"battery":String,	// phone battery in string
	"rigBattery":String,	// Synthesis of Phone Battery and sensor battery
	"openApsStatus":Long,	// -1  or LoopPlugin.lastRun.lastTBREnact                
	"bgi":String,		// BGI value formatted with sign, 1 decimal and localisation
	"batteryLevel":int	// 1 if level > 30, 0 if level < 30
}
```

Note: detailedIob and showBgi moved to [send preference Json String](#sendpreferences-androidapswatchface)

### SendData (AndroidAPS→Watchface)

Channel: 110

Type of data: JSON string (Array of bytes) (dataMapSingleBG)

```java
{
	"sgvString":String,	// conversion to unit and to string for correct rounding and localisation already done
	"glucoseUnits":String,	// "mg/dl" or "mmol"
	"timestamp":Long,	// lastBG.dat
	"slopeArrow":String,	// "" if glucoseStatus=null, "\u21ca", ↓ "\u2193", ↘ "\u2198", → "\u2192", ↗ "\u2197", ↑ "\u2191", "\u21c8" according to value
	"delta":String,		// deltastring function used to make string with rounded value according to units and localisation (decimal "." or ",")
	"avgDelta":String,	// idem delta (deltastring function)
	"sgvLevel":Long,	// 0=in range, 1>highLine, -1<lowline (color of BG point)
	"sgvDouble":Double,	// lastBG.value for graph updating
	"high":Double,		// High range (for graph)
	"low":Double		// Low range (for graph)
}
```



### Preferences (AndroidAPS→Watchface)

Channel: 115

Type of data: JSON string (Array of bytes)

```java
{
	"timestamp":Long,	// System.currentTimeMillis()
	"wearsettings":Long	// All settings for watch (WearControl and watch face settings), a long value allow 64 On/Off settings that should be enough...
}
```

This message request a "Setting Mask" answer from Tizen Watch



**Table for WearSettings value**

Note: just a proposal, done with settings available in WearOS watches (currently split in AndroidAPS and Wear settings in Watch)

| Setting               | definition                                                   | value                      | N°   |
| --------------------- | ------------------------------------------------------------ | -------------------------- | ---- |
| Wear Control          | Allow or not control of AAPS from Watch for treatments       | 0x00000000 0x00000001 (1)  | 00   |
|                       | **Blood Glucose Section** (one spare)                        |                            |      |
| Show BG value         | Show Blood Glucose Value                                     | 0x00000000 0x00000002 (2)  | 01   |
| Show BG arrow         | Show Blood Glucose Arrow                                     | 0x00000000 0x00000004 (4)  | 02   |
| Show delta            | Show delta                                                   | 0x00000000 0x00000008 (8)  | 03   |
| Show 15' delta        | Show 15' delta                                               | 0x00000000 0x00000010 (16) | 04   |
| Show 40' delta        | Show 40' delta (not available today in WearOS watches)       | 0x00000000 0x00000020 (32) | 05   |
| Show detailled delta  | Show delta(s) with one more decimal                          | 0x00000000 0x00000040 (64) | 06   |
|                       | **IOB / COB / BGI Section** (2 spares)                       |                            |      |
| Show IOB              | Show total Insulin On Board                                  | 0x00000000 0x00000100      | 08   |
| Show detailled IOB    | Show Bolus IOB and Basal IOB                                 | 0x00000000 0x00000200      | 09   |
| Show COB              | Show Carb On Board                                           | 0x00000000 0x00000400      | 10   |
| Show BGI              | Show Blood Glucose Impact                                    | 0x00000000 0x00000800      | 11   |
|                       | **Battery Section** (2 spares)                               |                            |      |
| Show phone battery    | Show Phone battery                                           | 0x00000000 0x00004000      | 14   |
| Show Pump Battery     | Show Pump battery (not available today in WearOS watches)    | 0x00000000 0x00008000      | 15   |
| Show Sensor Battery   | Show Pump battery (not available today in WearOS watches)    | 0x00000000 0x00010000      | 16   |
| Show rig battery      | Show Rig battery (synthesis of phone battery and sensor battery) => could be improved with also pump battery | 0x00000000 0x00020000      | 17   |
|                       | **Other settings Section** (1 spare)                         |                            |      |
| Show basal rate       | Show Basal rate                                              | 0x00000000 0x00100000      | 20   |
| Show loop status      | Show how many minutes since last loop run                    | 0x00000000 0x00200000      | 21   |
| Show ago              | Show minutes since last CGM reading                          | 0x00000000 0x00400000      | 22   |
|                       | **Graph Section** (0/2 spare(s))                             |                            |      |
| Show Prediction Lines | Show prediction lines in graph                               | 0x00000000 0x01000000      | 24   |
| Show Carbs            | Show Carb point in graph                                     | 0x00000000 0x02000000      | 25   |
| Show Bolus            | Show Bolus point in graph                                    | 0x00000000 0x04000000      | 26   |
| Show SMB as Bolus     |                                                              | 0x00000000 0x08000000      | 27   |
| Show Profile          |                                                              | 0x00000000 0x10000000      | 28   |
| Show Basal Rate       |                                                              | 0x00000000 0x20000000      | 29   |
| Chart Timeframe       | 2 last bits for Timeframe (from 2 hours to 5 hours)<br />not sure this is necessary (in WearOS watch double tap in graph allow changing time frame) |                            | 30   |
|                       | **Dedicated Watch face settings section** (proposal according to what existing currently in WearOS watches) |                            |      |
| Show date             |                                                              | 0x00000001 0x00000000      | 32   |
| White Background      | Switch Background from black to white                        |                            |      |
| Matching divider      |                                                              |                            |      |
|                       |                                                              |                            |      |
|                       |                                                              |                            |      |
|                       |                                                              |                            |      |
|                       |                                                              |                            |      |

### Setting Mask (Watch→AndroidAPS) (proposal, but hide / show preferences seems to be complicated in Android applications...)

Channel: 115

Note: same channel than sendPreferences 

Type of data: JSON string (Array of bytes)

```java
{
	"watchfaceName":String,		// Watch face name
	"settingsMask":Long		// define settings that must be shown in AndroidAPS menu
}
```

settings mask allow watch to define which settings can be set by user (default: 0 = none)



### Resend (AndroidAPS→Watchface)

SendBGHistory (note: currently sent in a separate thread (real time constraints I suppose))

Channel: 120

Type of data: JSON string (Array of bytes) (for dataMapSingleBG see [sendData](#senddata-androidapswatchface))

```java
{
	"entries":[dataMapSingleBG]	// JSON Array of dataMapSingleBG 
}
```

Note: when a resend action is done then all these json strings are sent to watch:

```java
sendBGHistory();
sendPreferences();
sendBasals();
senStatus();
```



### Resend request (Watch→AndroidAPS)

Channel: 120

Same channel than Resend (send anything to this channel to request a resend)



### SendBasals (AndroidAPS→Watchface)

Channel: 125

Type of data: JSON string (Array of bytes)

```java
{
	"basals":[basals], 		// JSON Array of basals
	"temps":[temps],		// JSON Array of temp basal
	"boluses":[boluses],		// JSON Array of bolus & carbs
	"predictions":[predictions]	// JSON Array of predictions lines
}
```

Contains Basals rate, profile rate, Carbs, Bolus, SMB and prediction lines (maybe easier if we split in 6 dedicated Json strings?)

#### basals (basalMap)

```java
{
	"starttime":long,	// start time of basal
	"endtime":long,		// end time of basal
	"amount",double	// amount
}
```

#### temps (tempDatamap)

```java
{
	"starttime":long,	// start time of basal
	"startBasal":double,	// basal level before
	"endtime":long,		// end time of basal temp
	"endbasal":double,	// basal level after
	"amount",double		// temp basal level
}
```

#### boluses (treatment Map)

```java
{
	"date":long,		// time of treatment
	"bolus":double,		// bolus level
	"carbs":double,		// carb level
	"isSMB":boolean,	// is SMB for bolus
	"isValid",boolean	// is bolus valid or not (ex prefill)
}
```

#### predictions (predictionMap)

```java
{
	"timestamp":long,	// time of prediction
	"sgv":double,		// level of prediction point (in mg/dl)
	"color":int		// color of point
}
```



## Data exchanged for Actions requests

Data exchanges between Wearable App and AAPS always start with a request from Wearable:

For all treatments, there is an "Initiate action" that requires a "Confirm Action"

### Initiate Action (watch→AndroidAPS)

Channel: 200

Type of data: String (Array of bytes) or JSON String, tbc

Initialize an action from watch is done threw a "command line" sent as String

This command line contain a keyword and from 0 to 4 parameters depending to action

For example:

new treatment for a meal: 10 UI and 50 g carbs

```
bolus 10 50
```

New wizard request for 50 g carbs and 80% percentage of injection:

```
wizard2 50 80
```



If json string (not decided yet):

```java
{
	InitiateActionString:String	// Command line
}
```



#### available commands

| Keyword (act[0])    | 1rst param (act[1])                     | 2nd param (act[2])               | 3rd param (act[3])               | 4th param (act[4])       |
| ------------------- | --------------------------------------- | -------------------------------- | -------------------------------- | ------------------------ |
| fillpreset          | "1"<br />"2"<br />"3"                   |                                  |                                  |                          |
| fill                | amount<br />stringToDouble              |                                  |                                  |                          |
| bolus               | insulin<br />stringToDouble             | carbs<br />stringToDouble        |                                  |                          |
| temptarget          | isMGDL<br />parseBoolean                | duration<br />stringToInt        | low<br />stringToDouble          | high<br />stringToDouble |
| status              | "pump"<br />"loop"                      |                                  |                                  |                          |
| wizard2             | carbsBeforeConstraints<br />stringToInt | percentage<br />Integer.parseInt |                                  |                          |
| opencpp             |                                         |                                  |                                  |                          |
| cppset              | timeshift                               | percentage                       |                                  |                          |
| tddstats            |                                         |                                  |                                  |                          |
| ecarbs              | carbs<br />stringToInt                  | starttime (min)<br />stringToInt | duration (hour)<br />stringToInt |                          |
| changeRequest       |                                         |                                  |                                  |                          |
| cancelChangeRequest |                                         |                                  |                                  |                          |



### Confirmation Request (AndroidAPS→Watch)

Channel: 205

Type of data: JSON string (Array of bytes)

```java
{
	rTitle:String, 		//Title of response (text)
	rMessage:String, 	//detailled answer (text)
	rAction:String 		//action requested for acknowledge
}
```



### Confirm Action (watch→AndroidAPS)

Channel: 205

Type of data: String (Array of bytes) or JSON String, tbc

```
rAction
```

rAction is memorized in AndroidAPS, to confirm initiate action, Watch should send (within 65s timeout) a confirmation string that match exactly Confirmation Request.



If json string (not decided yet):

```java
{
	ConfirmActionString:String	// Command line, must contain rAction String received in confirmation request
}
```



## 
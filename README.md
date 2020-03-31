# AAPS - Tizen specification (draft)

[TOC]

## Watchface and Wearable Settings

Current jobshare between AndroidAPS and WearOS watch is quite confusing and should be improved:

- Watchface settings are split between AAPS and WearOS
- Wear watch settings are too complicated with a mix of:
  - General watch settings (vibration, date, units)
  - Transverse Watchface settings (which information is shown or not)
  - Dedicated settings according to watchface (Circle settings, background colors...)
  - Wearable app settings (show or not some actions in mains menu, with interface layout, ...)

Samsung doesn't allow User Interfaces in watchfaces package, so my proposal would be to put all watchfaces settings in AAPS (sub-menu prepared for them), and to put all "Wearable app settings" in watch.

=> To be discussed

(The way we will share and distribute Tizen package could also have an impact because of Stores policies...)

## Data exchanged between AAPS and Watchface

### SendStatus (from AAPS to Watchface)

dataMap / json string

```java
{
	externalStatusString: String,
	iobSum: String,		// Total IOB converted to string with 2 decimal and localisation
	iobDetail: String,	// Detailled IOB converted to string with 2 decimal "(xx|yy)"
	detailedIob:Boolean, // Setting value done in AAPS => To be discussed*
	cob: String,		// Cob value converted in string
	currentBasal:String,//percentage or basal rate ("U/h" according to activeTemp or not)
	battery:String,		// phone battery in string
	rigBattery:String,	// Synthesis of Phone Battery and sensor battery
	openApsStatus:Long,	// -1  or LoopPlugin.lastRun.lastTBREnact                   
	bgi:String,			// BGI value formatted with sign, 1 decimal and localisation
    showBgi:Boolean,	// Setting value done in AAPS*
	batteryLevel:int		// 1 if level > 30, 0 if level < 30
}
```

*Setting management is quite confusing in WearOS plugin

### SendBasals (from AAPS to Watchface)



### SendData (From AAPS to Watchface)

dataMap / Json string

Note Unit conversion made in Plugin (mg/dl or mmol/l)

```java
{
    sgvString:String,		// conversion to unit and to string for correct rounding and localisation
    glucoseUnits:String,	// "mg/dl" or "mmol"
    timestamp:Long,			// lastBG.dat
    slopeArrow:String,		// "" if glucoseStatus=null, "\u21ca", ↓ "\u2193", ↘ "\u2198", → "\u2192", ↗ "\u2197", ↑ "\u2191", "\u21c8" according to value
    delta:String,			// deltastring function used to make string with rounded value according to units and localisation (decimal "." or ",")
    avgDelta:String,		// idem delta (deltastring function)
    sgvLevel:Long,			// 0=in range, 1>highLine, -1<lowline (color of BG point)
    sgvDouble:Double,		// lastBG.value for graph updating
    high:Double,			// High range (for graph)
    low:Double				// Low range (for graph)
}
```

### Resend (From AAPS to Watchface)

SendBGHistory in separate thread (real time constraints I suppose)



Then

```java
sendPreferences();

sendBasals();

senStatus();
```



### sendPreferences

dataMap / Json String

```java
{
	timestamp:Long,			// System.currentTimeMillis()
	wearcontrol:Boolean		// WearControl value defined in AAPS preferences
}
```



### Resend Request (Request from Watchface to AAPS)

Receiving something in resendData channel is enough to do the action.



## Between AAPS and Wearable App

Data exchanges between Wearable App and AAPS always start with a request from Wearable:

=> Different Path used for WearOS, I propose to use channel in Tizen Service to explain how it works

| Channel                                 | Action                                               |
| --------------------------------------- | ---------------------------------------------------- |
| WEARABLE_RESEND_CH = 110                | resendData()                                         |
| WEARABLE_CANCELBOLUS_CH = 115           | cancelBolus()                                        |
| WEARABLE_INITIATE_ACTIONSTRING_CH = 120 | ActionStringHandler.handleInitiate(actionstring)     |
| WEARABLE_CONFIRM_ACTIONSTRING_CH = 125  | ActionStringHandler.handleConfirmation(actionstring) |

Receiving something in resendData channel or cancelBolus channel is enough to do the action

(it's the result of code analysis because these messages are not implemented today in wearOS)



#### ActionStringHandler : Class of Wear plugin

methods:

1. **For build an answer after an Initiate action String sent by watch:**

```java
public synchronized static void handleInitiate(String actionstring)
```

Responses messages has always the same structure (3 strings with a title, a response and an action contains the action that watch should answer to confirm Action: 

```
WearPlugin.getPlugin().requestActionConfirmation(rTitle, rMessage, rAction);
```

Note: When an initiate Action String does not require a confirmation, rAction is set to statusmessage or info.

2. **For intent actions (only for fill, bolus, temptarget, wizard2, cppset, changeRequest and ecarb)**

```
public synchronized static void handleConfirmation(String actionString)
```



Note: action string is split in ActionStringHandler with regex "\\\s+"

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

Timeout : 65s

rTitle : Title of response (text)

rMessage : detailled answer (text)

rAction : action for acknowledge

```java
	WearPlugin.getPlugin().requestActionConfirmation(rTitle, rMessage, rAction)
```



```java
public synchronized static void handleConfirmation(String actionString)
```

action string split with regex "\\\s+"





## 
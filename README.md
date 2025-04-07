# vicidial_Carrier_Failover
vicidial Carrier Failover and Manual Round Robin


Process One Failover Carrier

First you need to Setup 2 Carrier in vicidial

You have two SIP trunks (carriers):

Carrier One (Carrier 1)

Carrier tow (Carrier 2)

SOLUTION 1: Failover/Backup Carrier (when Carrier one is full)

Concurrent calling — if Carrier one is full (busy or at max channels), calls should route to Carrier tow.


Carrier one 
Account Entry:

[Carrier_One]
disallow=all
allow=alaw
allow=ulaw
allow=g729
qualify=yes
type=friend
nat=force_rport,comedia
username=your_user_name
secret=your_secret
host=Carrier Host
dtmfmode=rfc2833
context=trunkinbound
canreinvite=no
insecure=port,invite

Globals String: Carrier_One= SIP/Carrier_One

you should Write Dial Plan main Carrier like Carrier One

Dialplan Entry:

exten => _101[2-9]XXXXXXXX,1,AGI(agi://127.0.0.1:4577/call_log)
exten => _101[2-9]XXXXXXXX,2,Set(CALLERID(num)=Carrier one did number)
exten => _101[2-9]XXXXXXXX,3,Dial(SIP/${EXTEN:1}@Carrier_One,,tTorR)
exten => _101[2-9]XXXXXXXX,4,GotoIf($["${DIALSTATUS}" = "CONGESTION"]?6)
exten => _101[2-9]XXXXXXXX,5,GotoIf($["${DIALSTATUS}" = "CHANUNAVAIL"]?6)
exten => _101[2-9]XXXXXXXX,6,Set(CALLERID(num)=09639143156)
exten => _101[2-9]XXXXXXXX,7,Dial(SIP/${EXTEN:1}@JTI_ICC,,tTorR)
exten => _101[2-9]XXXXXXXX,8,Hangup



Campaign Dial Prifix will be 10


Carrier Tow


Account Entry

[Carrier_Tow]
disallow=all
allow=alaw
allow=ulaw
allow=g729
qualify=yes
type=friend
nat=force_rport,comedia
username=your_user_name
secret=your_secret
host=your Carrier Host
dtmfmode=rfc2833
context=trunkinbound
canreinvite=no
insecure=port,invite

Globals String: Carrier_Tow= SIP/Carrier_Tow

dial plan Should be empty


----------------------------------------------------------------------------------------

SOLUTION 2: Manual Round Robin (basic call distribution logic)

You can create two different Campaigns or Lists or even use custom AGI script or Random Selector in dialplan, but here's a basic way using mod(${RAND()}) for 50-50 split:

Round-robin / call splitting — distribute a batch of calls (e.g., first 25 via Carrier one, next 25 via Carrier tow).


every this like Account entry and other option is Same like SOLUTION 1

Only Dial Plan will be below



Dial Plan:-

exten => _10XXXXXXXXXX,1,AGI(agi://127.0.0.1:4577/call_log)
exten => _10XXXXXXXXXX,2,Set(RANDOM_CALL=$[${RAND(0,1)}])
exten => _10XXXXXXXXXX,3,GotoIf($["${RANDOM_CALL}" = "0"]?carrierone:carriertow)

exten => _10XXXXXXXXXX,n(carrierone),Set(CALLERID(num)=Carrier one DID Number)
exten => _10XXXXXXXXXX,n,Dial(SIP/${EXTEN:1}@Carrier_one,,tTorR)
exten => _10XXXXXXXXXX,n,Hangup

exten => _10XXXXXXXXXX,n(carriertow),Set(CALLERID(num)=Carrier tow DID Number)
exten => _10XXXXXXXXXX,n,Dial(SIP/${EXTEN:1}@Carrier_tow,,tTorR)
exten => _10XXXXXXXXXX,n,Hangup

This gives approximate 50/50 call distribution, though not exact sequencing like “first 25” and “next 25”.

Thank you


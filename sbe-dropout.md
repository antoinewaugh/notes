
# Exploration into MD outages with new SBE Adapter

It has been observed that we are receiving repeated market data outages across multiple machines.

These outages started since we updated the apama CME SBE adapter. 

Outages appear to be on a whole channel (i.e. we saw symbols PL and PA which both are on the same channel 384), outages are _not limited to a single channel_ in the sense that we have observed multiple channels depth frozen.

Before the adapter upgrade, we may have had one outage in a year. Since the SBE adapter upgrade we have now seen 5 separate occurences across two days.

# Exploring data inbound to correlator

## Identify Channel - engine_inspect

Using engine_inspect, we can view the channels currently subscribed, of interest are symbols PLV8, using engine_inspect we can see the channel data is being published on is:

`BBA_PLV8_1533694577477`
`T_PLV8_1533694577477`
`D_PLV8_1533694577424`

``` 
[apama52@RTVAURCMESERV02 logs]$ engine_inspect -x -p 15910 | less

Contexts
========
Name                                  Queue Size   Num Sub Monitors   Channels
----                                  ----------   ----------------   --------
FIX_CME_Support_Extensions                   0               29        BBA_6AU8_1533694577464 BBA_6BU8_1533694577465 BBA_6CU8_1533694577459 BBA_6EU8_1533694577481 BBA_6JU8_1533694577453 BBA_6SU8_1533694577474 BBA_CLU8_1533694577455 BBA_ESU8_1533694577457 BBA_GCZ8_1533694577454 BBA_HGU8_1533694577456 BBA_HOU8_1533694577458 BBA_NIYU8_1533694577461 BBA_NQU8_1533694577476 BBA_PAU8_1533694577467 BBA_PLV8_1533694577477 BBA_RBU8_1533694577475 BBA_RTYU8_1533694577466 BBA_SIU8_1533694577472 BBA_YMU8_1533694577483 BBA_ZBU8_1533694577469 BBA_ZCZ8_1533694577462 BBA_ZFU8_1533694577463 BBA_ZLZ8_1533694577478 BBA_ZMZ8_1533694577486 BBA_ZNU8_1533694577479 BBA_ZSX8_1533694577470 BBA_ZTU8_1533694577484 BBA_ZWU8_1533694577482 

....

MDR:CME Prices:fut-PL_201810                 0               1        D_PLV8_1533694577424 T_PLV8_1533694577425 

....

```

## Check inbound data is being received on all input queues

At a macro level, we can see inbound com.apama.md.BBA events hitting correlator for `BBA`, `Trade` and `DD` types.

```

[apama52@RTVAURCMESERV02 logs]$ engine_receive -c com.apama.input -p 15910

com.apama.md.BBA(1,1533694577456,"HGU8",27665,7,27670,1,{},{})
BATCH 978
com.apama.md.T(1,1533694577402,"HGU8",27670,1,"","","",{},{})
BATCH 978
com.apama.md.DD(1,1533694576940,"HGU8",[],[com.apama.md.DI(0,27670,1,2,{}),com.apama.md.DI(4,27695,5,0,{})],{"CumulativeTradeVolume":"209"},{})
BATCH 979
com.apama.md.BBA(1,1533694577456,"HGU8",27665,7,27675,6,{"CumulativeTradeVolume":"209"},{})
BATCH 980
com.apama.md.DD(1,1533694576940,"HGU8",[],[com.apama.md.DI(0,27670,1,0,{}),com.apama.md.DI(5,27695,5,2,{})],{},{})
BATCH 980
com.apama.md.BBA(1,1533694577456,"HGU8",27665,7,27670,1,{},{})
BATCH 980
com.apama.md.DD(1,1533694576940,"HGU8",[],[com.apama.md.DI(0,27670,2,1,{})],{},{})
BATCH 980

```

## Check inbound data is being received queues specific to symbol PLV8

First, check queue filtering command works for a liquid contracts like ES which doesn't currently have a price issue

```
^C[apama52@RTVAURCMESERV02 logs]$ engine_receive -c com.apama.input.D_ESU8_1533694576937 -p 15910
Control-C to exit...
BATCH 610
com.apama.md.DD(1,1533694576937,"ESU8",[],[],{},{})
BATCH 611
com.apama.md.DD(1,1533694576937,"ESU8",[com.apama.md.DI(4,284950,67,1,{})],[],{},{})
BATCH 2054
com.apama.md.DD(1,1533694576937,"ESU8",[com.apama.md.DI(0,285050,51,1,{})],[],{},{})

```

The Same command run on the PL BBA, Trade and Depth channels respectively yield zero updates.

*NB:* During the period that we measured inbound events on the below channels, I actually triggered trades in the live market. This confirms that although real trading and MD events are being triggered by CME the SBE adapter is not emitting any events to the correlator.

```
^C[apama52@RTVAURCMESERV02 logs]$ engine_receive -c com.apama.input.T_PLV8_1533694577425 -p 15910                                                                                                                  
Control-C to exit...
^C[apama52@RTVAURCMESERV02 logs]$ engine_receive -c com.apama.input.D_PLV8_1533694577425 -p 15910                                                                                                                  
Control-C to exit...
^C[apama52@RTVAURCMESERV02 logs]$ engine_receive -c com.apama.input.BBA_PLV8_1533694577425 -p 15910                                                                                                                
Control-C to exit...

```

# CME SBE Logs

Whilst one of the SBE instances had two log lines regarding RptSeq jump, the others did not, and all correlators experienced the SAME outage...

```
[apama52@RTVAURCMESERV02 logs]$ cat CME_MARKETDATA_20180808_021334.log  | grep RptSeq
2018-08-08 14:10:47.280 INFO  [140039925249792] - Channel : 310, MsgSeqNum : 6468644, SecurityID : 57287 Got unexpected message RptSeqNum : 6437867 expecting 6437865, attempting to recover
2018-08-09 18:23:08.401 INFO  [140039933642496] - Channel : 382, MsgSeqNum : 140451902, SecurityID : 517851 Got unexpected message RptSeqNum : 5392875 expecting 5392874, attempting to recover

```

# Conclusion

As it stands, we have

* PLV8 currently trading and open on CME
* No events being emitted from SBE adapter to the correlator on ANY of the PLV8 nmda channels 
* 5 outages similar to this across 2 days
* We are seeing a CHANNEL drop out, so if PA drops on channel 382 as does PL.
* We are seeing multiple CHANNELS drop out at times, and other times its just one of the many channels we have subscribed

We would like you to roll back the adapter code to the original , functioning version of apama.9.12 without any of the new config options, and add simply the template change(s).

If you could do this, and send a pre-release we may be able to prevent a production outage this Sunday evening (as the template file goes live 12th August), and also if no more issues occur we have isolated which change(s) may have caused the issues seen here.

What are your thoughts?


# Separately: 

In addition to the above, we have seen mass outages on the adapter where the Watchdog appears to state the correlator<->SBE adapter has dropped.

I was reluctant to give you this additional information in case it confuses the above because they occur at separate times, but that being said it may be related and help your understanding of the situation?

```
[apama@RTVAURCMESERV04 logs]$ less CME_MARKETDATA_20180809_141634.log

2018-08-09 22:34:47.446 INFO  [140357139633920] - Status: ApEvRx=17951 ApEvTx=11910 TrEvRx=2892 TrEvTx=0 vm=4797036 pm=240580 si=0.0 so=0.0 oq=0
2018-08-09 22:34:51.786 INFO  [140352341993216] - Session::HeartbeatWatchDog: heartbeat thread finished
2018-08-09 22:34:51.786 INFO  [140352341993216] - [CME_SBE - CME_FAST_TRANSPORT] : Correlator Watchdog Timedout
2018-08-09 22:34:51.786 INFO  [140352341993216] - Session::HeartbeatWatchDog: stopping the heartbeat thread
2018-08-09 22:34:51.786 INFO  [140352341993216] - Session::SessionMgr: capability : [com.apama.md.BBA] added for session : [CME]
2018-08-09 22:34:51.786 INFO  [140352341993216] - Session::SessionMgr: capability : [com.apama.md.T] added for session : [CME]
2018-08-09 22:34:51.786 INFO  [140352341993216] - Session::SessionMgr: capability : [com.apama.md.D] added for session : [CME]
2018-08-09 22:34:51.786 INFO  [140352341993216] - Session::SessionMgr: capability : [com.apama.md.EP] added for session : [CME]
2018-08-09 22:34:51.786 INFO  [140352333600512] - Session::HeartbeatWatchDog: thread started for the heartbeat watching
2018-08-09 22:34:52.446 INFO  [140357139633920] - Status: ApEvRx=17953 ApEvTx=11912 TrEvRx=2892 TrEvTx=0 vm=4805232 pm=240592 si=0.0 so=0.0 oq=0
2018-08-09 22:34:57.446 INFO  [140357139633920] - Status: ApEvRx=17956 ApEvTx=11914 TrEvRx=2892 TrEvTx=0 vm=4805232 pm=240592 si=0.0 so=0.0 oq=0
....

2018-08-09 22:36:47.446 INFO  [140357139633920] - Status: ApEvRx=18022 ApEvTx=11958 TrEvRx=2892 TrEvTx=0 vm=4805232 pm=241096 si=0.0 so=0.0 oq=0
2018-08-09 22:36:51.799 INFO  [140352333600512] - Session::HeartbeatWatchDog: heartbeat thread finished
2018-08-09 22:36:51.799 INFO  [140352333600512] - [CME_SBE - CME_FAST_TRANSPORT] : Correlator Watchdog Timedout
2018-08-09 22:36:51.799 INFO  [140352333600512] - Session::HeartbeatWatchDog: stopping the heartbeat thread
2018-08-09 22:36:51.799 INFO  [140352325207808] - Session::HeartbeatWatchDog: thread started for the heartbeat watching
2018-08-09 22:36:51.799 INFO  [140352333600512] - Session::SessionMgr: capability : [com.apama.md.BBA] added for session : [CME]
2018-08-09 22:36:51.799 INFO  [140352333600512] - Session::SessionMgr: capability : [com.apama.md.T] added for session : [CME]
2018-08-09 22:36:51.799 INFO  [140352333600512] - Session::SessionMgr: capability : [com.apama.md.D] added for session : [CME]
2018-08-09 22:36:51.799 INFO  [140352333600512] - Session::SessionMgr: capability : [com.apama.md.EP] added for session : [CME]
2018-08-09 22:36:52.446 INFO  [140357139633920] - Status: ApEvRx=18025 ApEvTx=11960 TrEvRx=2892 TrEvTx=0 vm=4813428 pm=241108 si=0.0 so=0.0 oq=0
2018-08-09 22:36:57.446 INFO  [140357139633920] - Status: ApEvRx=18029 ApEvTx=11962 TrEvRx=2892 TrEvTx=0 vm=4813428 pm=241108 si=0.0 so=0.0 oq=0
2018-08-09 22:37:02.446 INFO  [140357139633920] - Status: ApEvRx=18031 ApEvTx=11964 TrEvRx=2892 TrEvTx=0 vm=4813428 pm=241108 si=0.0 so=0.0 oq=0
2018-08-09 22:37:07.446 INFO  [140357139633920] - Status: ApEvRx=18034 ApEvTx=11966 TrEvRx=2892 TrEvTx=0 vm=4813428 pm=241108 si=0.0 so=0.0 oq=0



```

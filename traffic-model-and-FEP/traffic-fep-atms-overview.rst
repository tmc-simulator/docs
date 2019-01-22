====================================
Traffic Model / FEP / ATMS Overview
====================================

#. `Terminology </wiki/TrafficModelATMSOverview#terminology>`__
#. `Purpose </wiki/TrafficModelATMSOverview#purpose>`__
#. `System Architecture </wiki/TrafficModelATMSOverview#architecture>`__
#. `Overview of System </wiki/TrafficModelATMSOverview#overview>`__

   #. `Traffic
      Model </wiki/TrafficModelATMSOverview#trafficmodeloverview>`__
   #. `FEP Simulator </wiki/TrafficModelATMSOverview#fepsimoverview>`__
   #. `ATMS </wiki/TrafficModelATMSOverview#atmsoverview>`__
   #. `System
      Behavior </wiki/TrafficModelATMSOverview#systembehavior>`__
   #. `Why do we need FEP
      Sim? </wiki/TrafficModelATMSOverview#whyfep>`__

#. `Detailed Component
   Overviews </wiki/TrafficModelATMSOverview#detailedcomponentoverviews>`__

   #. `Traffic
      Model </wiki/TrafficModelATMSOverview#trafficmodeldetail>`__

      #. `Behavior </wiki/TrafficModelATMSOverview#trafficmodelbehavior>`__
      #. `Traffic Model
         Classes </wiki/TrafficModelATMSOverview#trafficmodelclasses>`__
      #. `Different Abstractions. FEP Line vs
         Highway </wiki/TrafficModelATMSOverview#differentabstractions>`__

   #. `FEP Simulator </wiki/TrafficModelATMSOverview#fepsimdetail>`__

      #. `The Real FEP </wiki/TrafficModelATMSOverview#realfep>`__
      #. `Our FEP Simulator </wiki/TrafficModelATMSOverview#ourfep>`__

   #. `ATMS </wiki/TrafficModelATMSOverview#atmsdetail>`__

#. `Protocols </wiki/TrafficModelATMSOverview#protocols>`__

   #. `Traffic Model to FEP
      Simulator </wiki/TrafficModelATMSOverview#traffictofep>`__

      #. `Example Traffic Conditions
         Message </wiki/TrafficModelATMSOverview#exampletraffic>`__

   #. `FEP Simulator to
      ATMS </wiki/TrafficModelATMSOverview#feptoatms>`__

      #. `fep\_reply
         structs </wiki/TrafficModelATMSOverview#fepreply>`__
      #. `Traffic Conditions Message
         Structure </wiki/TrafficModelATMSOverview#trafficmessage>`__

.. rubric:: Terminology\ `¶ <#terminology>`__
   :name: terminology

-  FEP = Front End Processor
-  ATMS = Advanced Traffic Management Server
-  TMC = Traffic Management Control
-  RPC = Remote Procedural Calls

.. rubric:: Purpose\ `¶ <#purpose>`__
   :name: purpose

An important function of the TMC simulator is to simulate the traffic
flow that results from a simulated traffic incident. The traffic flow is
shown to the Cal Trans dispatcher as a road map with sensor locations
displayed in different colors that correspond to stopped, slowed, or
free flowing traffic. In the real world, data from sensors embedded in
highways is transmitted in real time to a computer called the ATMS which
then determines the flow of traffic along a network of highways. For the
TMC Simulator there is no actual sensor data so this must be simulated
with a traffic model. In previous versions of the TMC Simulator the
traffic model used was a commercial product called Paramics. For a
variety of technical, legal, and economic reasons the use of Paramics
became infeasible and we choose to abandon it and develop our own
solution.

This document provides an overview of the Traffic Model, the FEP
Simulator, and the ATMS. Each of these components work together in the
TMC Simulation System to simulate traffic conditions, and display them
to trainees. This document will explain to the reader, at a high level,
the system architecture and how the components work together to simulate
the traffic conditions, an overview of each component, and lastly, the
protocols between each component.

.. rubric:: System Architecture\ `¶ <#architecture>`__
   :name: architecture

The `System (Traffic Model / FEP / ATMS) Deployment/Component
Diagram </wiki/DeploymentDiagram>`__ is a UML Deployment / Component
diagram which helps to describe the system architecture.

.. rubric:: Overview of System\ `¶ <#overview>`__
   :name: overview

.. rubric:: What is the Traffic Model?\ `¶ <#trafficmodeloverview>`__
   :name: trafficmodeloverview

The Traffic Model is a loose term that describes the components of the
Java CAD Server that model the simulated traffic conditions of the
highways during the simulation. It consists of the Traffic Model Manager
and the Highways model.

.. rubric:: What is the FEP Simulator?\ `¶ <#fepsimoverview>`__
   :name: fepsimoverview

The FEP Simulator simulates the actual FEP that is used in production
systems at Cal Trans. The actual FEP gathers the real time highway
sensor data and transmits it to the ATMS. The FEP Simulator will gather
simulated highway sensor data from the traffic model and transmit it to
the ATMS. It acts as a mediator between the Traffic Model and the ATMS.
Its purpose is to receive the traffic condition data over a socket from
the Traffic Model, parse and reformat this received data into fep\_reply
structs, and send the fep\_reply structs to the ATMS via RPC. The FEP
Simulator is a C++ program that runs on a Linux VM on the TMC network.

.. rubric:: What is the ATMS?\ `¶ <#atmsoverview>`__
   :name: atmsoverview

The ATMS is a production server, purchased from Cal Trans, that accepts
the fep\_reply structs from the FEP Simulator via RPC. The ATMS accepts
fep\_reply structs, which contain traffic condition data, to feed the
ATMS Client. The ATMS Client displays traffic condition data on an
interactive map GUI to the trainees.

.. rubric:: System Behavior / How do these components work
   together?\ `¶ <#systembehavior>`__
   :name: systembehavior

Reference the `Traffic Model / FEP / ATMS Activity
Diagram </wiki/TrafficFEPATMSActivityDiagram>`__ while reading this
section. It helps to describe system behavior.

When the CAD Server starts, the Traffic Model Manager and the Highways
model are instantiated. The Traffic Model Manager spawns a thread, the
WriteToFEPThread, which creates a client socket connection with the FEP
Simulator. The traffic conditions message is sent over this socket, to
the FEP Simulator, on every 30 second interval. To do so, the
WriteToFEPThread calls upon the Highways class' writeToFEP() method.

The FEP Simulator acts as a socket server, and awaits the traffic conditions message from the CAD Server's Traffic Model Manager. Upon
receipt of this message, the FEP Simulator parses and reconfigures the
data into fep\_reply structs. It then sends the fep\_reply structs to
the ATMS via RPC. Once this process is finished, the FEP Simulator
awaits another traffic conditions message from the CAD Server, and the
process repeats.

The ATMS receives the fep\_reply structs from the FEP Simulator, and
then communicates the fep\_reply data, which contains the traffic
conditions, to the ATMS Client. The end result is the traffic conditions
being displayed to trainees on the interactive map GUI.

.. rubric:: Why do we need the FEP Simulator?\ `¶ <#whyfep>`__
   :name: whyfep

Why do we need the FEP Simulator as a mediator? Can't we just send the
fep\_replys directly to the ATMS via RPC? The short answer to this
question is no. Java does not support RPC natively, whereas C and C++
do. It is possible to use RPCs with Java, but to do so it would require
a great deal of work or money. Read more about this decision in the
`Lessons Learned Document </wiki/LessonsLearned>`__.

.. rubric:: Detailed Component
   Overviews\ `¶ <#detailedcomponentoverviews>`__
   :name: detailedcomponentoverviews

This section will describe each component individually in more detail,
to give a better understanding of how they work.

.. rubric:: Traffic Model\ `¶ <#trafficmodeldetail>`__
   :name: trafficmodeldetail

Reference the `Traffic Model Class
Diagram </wiki/TrafficModelClassDiagram>`__ while reading this section.

.. rubric:: Behavior\ `¶ <#trafficmodelbehavior>`__
   :name: trafficmodelbehavior

As previously stated, the Traffic Model consists of the Traffic Model
Manager and the Highways model. The Traffic Model Manager contains an
instance of the Highways class. Both the Traffic Model Manager and the
Highways model and instantiated upon CAD Server start up. Once
instantiated, the Traffic Model Manager spawns an instance of the
WriteToFEPThread, which sends the current traffic conditions of the
Highways to the FEP Simulator by calling the Highways class'
writeToFEP() method, on every 30 second interval.

The Traffic Model Manager also loads Traffic Events from a batch file
into a queue, the eventsQueue, when instantiated. The Traffic Events
batch file is specified in the Traffic Model Manager's properties file.
A Traffic Event changes the current traffic conditions at a specified
location. As the simulation runs, the Traffic Model Manager queries the
Coordinator object for the current simulation time, and then checks the
queue to see if there are Traffic Events that should be applied to the
Highways model at the current simulation time.

.. rubric:: Traffic Model Classes\ `¶ <#trafficmodelclasses>`__
   :name: trafficmodelclasses

The Highways and Highway classes were developed to give developers an
abstraction of the traffic network that was feasible to work with. A
Highway is undirected, meaning it does not have an associated (N/S/W/E)
direction and represents traffic in both directions on the highway. A
Highway is an aggregation of all of the Stations on a highway, sorted by
postmile value. Stations do have an associated direction. Stations may
or may not have Loop Detectors associated with the specified station
direction, or the opposite specified direction. Hence, the undirected
Highway abstraction.

The Highways, Highway, and Station classes are all immutable. None of
them contain dynamic data. The only class in the Highways model that
contains dynamic values is the `LoopDetector? </wiki/LoopDetector>`__
class. The dynamic values in the `LoopDetector? </wiki/LoopDetector>`__
class are volume and occupancy. The volume field is an int, and the
occupancy field is a float in the [0,1] range. The DOTCOLOR enum
(specified in the `LoopDetector? </wiki/LoopDetector>`__ class)
represents the colors shown on the ATMS Client
(`Green/Yellow/Red? </wiki/Green/Yellow/Red>`__). This enum supplies the
necessary volume and occupancy values to achieve the desired display
color for a Loop Detector.

.. rubric:: Different Abstractions. FEP Line vs
   Highway\ `¶ <#differentabstractions>`__
   :name: differentabstractions

It should be noted that the ATMS has no concept of the Highways and
Highway abstraction, only FEPLines, Stations, and Loop Detectors.
FEPLines, like the Highway Class, are also composed of Stations but are
not a convenient abstraction because they only represent serial
communication lines used to communicate with a geographic group of field
stations. The FEPLines need to be represented because they contain
necessary attributes that must be passed on to the FEP Simulator to
populate the fep\_reply structs. As a developer you will work within the
Highways, Highway, Station, and `LoopDetector? </wiki/LoopDetector>`__
abstraction. The Traffic Model communicates with the FEP Simulator /
ATMS in the context of the FEPLine, Station,
`LoopDetector? </wiki/LoopDetector>`__ abstraction.

The next section "FEP Simulator" will give a better understanding of
FEPLines and why their representation is necessary in the simulation
system.

.. rubric:: FEP Simulator\ `¶ <#fepsimdetail>`__
   :name: fepsimdetail

Reference the `FEP Simulator Class Diagram </wiki/FEPSimClassDiagram>`__
while reading this section. It is not quite as helpful as the Traffic
Model Class Diagram, because the FEP Simulator is really just a
functional program.

.. rubric:: The Real FEP\ `¶ <#realfep>`__
   :name: realfep

To understand how the FEP Simulator works, it helps to understand how
the actual FEP works. The actual FEP "polls" field stations, over serial
communication lines (FEPLines), for traffic data via RPC. For clarity,
"polling" is the act of the FEP requesting traffic data from a field
station. When "polled", the field station sends traffic data back to the
FEP via RPC. Upon receipt of the traffic data from the field stations,
the FEP then reformats the data into an fep\_reply struct and sends it
to the ATMS via RPC.

.. rubric:: Our FEP Simulator\ `¶ <#ourfep>`__
   :name: ourfep

In contrast, our FEP Simulator is not actually "polling" field stations,
and instead just reads all traffic data over the socket from the CAD
Server's Traffic Model. It is similar, however, in that it reformats
traffic condition data into fep\_reply structs and sends them to the
ATMS via RPC.

The FEP Simulator runs persistently on the Linux Virtual Machine on the
TMC network.The FEP Simulator acts as a socket server, and awaits the
traffic conditions message from the CAD Server's Traffic Model Manager.

Upon receipt of this message, the FEP Simulator parses and reconfigures
the data into fep\_reply structs. To do so, it uses the
``HighwaysParser`` class to parse the message from the Traffic Model
into FEP\_LINE and STATION structs. Once parsed, it reformats the data
into fep\_reply structs. fep\_replys contain a Traffic Conditions
Message, a char[] array that contains traffic condition data. This
message is "packed" or created by using the static packData() method
from the ``DataPacker`` class. See the Protocols section for more
information. It then sends the fep\_reply structs to the ATMS via RPC.
Once this process is finished, the FEP Simulator awaits another traffic
conditions message from the CAD Server, and the process repeats.

.. rubric:: ATMS\ `¶ <#atmsdetail>`__
   :name: atmsdetail

The ATMS is a complete production server purchased from
`CalTrans? </wiki/CalTrans>`__. We were unable to find detailed
documentation about ATMS features other than its ability to receive
fep\_reply structs via RPC and feed the received data to the ATMS Client
interactive map GUI.

.. rubric:: Protocols\ `¶ <#protocols>`__
   :name: protocols

.. rubric:: Traffic Model to FEP Simulator\ `¶ <#traffictofep>`__
   :name: traffictofep

The Traffic Model communicates with the FEP Simulator by sending the
traffic conditions message over a socket connection.

The traffic conditions message is produced by the Highways class method
"toCondensedFormat(false)".

.. rubric:: Example Traffic Conditions Message\ `¶ <#exampletraffic>`__
   :name: exampletraffic

Example traffic conditions message (toCondensedFormat(false)) output:

.. code:: wiki

    43 // "number of lines"
    32 0 13 // "line id" "count num" "number of stations"
    1210831 1 5 S 0.9 8 // "station id" "drop num" "route num" "direction" "postmile" "number of loops"
    1210832  0.0 0  ML_1 // "loop id" "occ" "vol" "loopLocation"
    1210833  0.0 0  ML_2 // ..
    1210834  0.0 0  ML_3 // ..
    1210835  0.0 0  ML_4 // ..
    1210836  0.0 0  PASSAGE // ..
    1210837  0.0 0  DEMAND // ..
    1210838  0.0 0  QUEUE // ..
    1210839  0.0 0  RAMP_OFF // ..
    ...

.. rubric:: FEP Simulator to ATMS\ `¶ <#feptoatms>`__
   :name: feptoatms

The FEP Simulator communicates with the ATMS via RPC. It sends
fep\_reply structs which contain traffic condition data messages. Each
fep\_reply struct corresponds to a single FEPLine.

The FEP Simulator includes the generated rpc files: "fep.h",
"fep\_clnt.c", and "fep\_xdr.c". These files need to be included to
utilize RPC.

The method used to transfer the fep\_replys, from the included rpc
files, is named fep\_reply\_xfer\_32(fep\_reply \*reply, CLIENT \*clnt).

fep\_reply structs contain metadata and an fep\_answer\_list struct. An
fep\_answer\_list struct contains a fep\_shortanswer\_list struct. An
fep\_shortanswer\_list struct contains an fep\_shortanswer struct. An
fep\_shortanswer struct contains an fep\_shortanswer\_msg struct. The
fep\_shortanswer\_msg contains a char array, which is the actual message
that contains traffic condition data. All of these structs are defined
in the "fep.h" file. The struct definitions are as follows:

.. rubric:: fep\_reply struct\ `¶ <#fepreply>`__
   :name: fepreply

.. raw:: html

   <div class="wikipage" style="font-size: 80%">

.. raw:: html

   <div class="code">

::

    struct fep_reply {
            int reply;
            int schedule;
            int lineinfo;
            polltype kind;
            replykind flag;
            int schedule_sequence;
            int global_sequence;
            long schedule_time;
            int user_info1;
            int user_info2;
            int system_key;
            struct fep_answer_list answers;
    };
    typedef struct fep_reply fep_reply;

.. raw:: html

   </div>

.. raw:: html

   </div>

.. rubric:: fep\_answer\_list struct\ `¶ <#fep_answer_liststruct>`__
   :name: fep_answer_liststruct

.. raw:: html

   <div class="wikipage" style="font-size: 80%">

.. raw:: html

   <div class="code">

::

    struct fep_answer_list {
            int size;
            union {
                    fep_shortanswer_list shortp;
                    fep_longanswer_list longp;
            } fep_answer_list_u;
    };
    typedef struct fep_answer_list fep_answer_list;

.. raw:: html

   </div>

.. raw:: html

   </div>

.. rubric:: fep\_shortanswer\_list\ `¶ <#fep_shortanswer_list>`__
   :name: fep_shortanswer_list

.. raw:: html

   <div class="wikipage" style="font-size: 80%">

.. raw:: html

   <div class="code">

::

    struct fep_shortanswer_list {
            int count;
            fep_shortanswer answers[MAXSHORTPOLLS];
    };
    typedef struct fep_shortanswer_list fep_shortanswer_list;

.. raw:: html

   </div>

.. raw:: html

   </div>

.. rubric:: fep\_shortanswer\ `¶ <#fep_shortanswer>`__
   :name: fep_shortanswer

.. raw:: html

   <div class="wikipage" style="font-size: 80%">

.. raw:: html

   <div class="code">

::

    struct fep_shortanswer {
            fep_answer_info info;
            fep_answer_short_msg msg;
    };
    typedef struct fep_shortanswer fep_shortanswer;

.. raw:: html

   </div>

.. raw:: html

   </div>

.. rubric:: fep\_answer\_short\_msg\ `¶ <#fep_answer_short_msg>`__
   :name: fep_answer_short_msg

.. raw:: html

   <div class="wikipage" style="font-size: 80%">

.. raw:: html

   <div class="code">

::

    struct fep_answer_short_msg {
            int message_len;
            char message[MAXSHORTREPLYLEN];
    };

.. raw:: html

   </div>

.. raw:: html

   </div>

.. rubric:: Traffic Conditions Message
   Structure\ `¶ <#trafficmessage>`__
   :name: trafficmessage

As seen above in the fep\_anser\_short\_msg struct, the traffic
conditions message is a char[] array. The FEP Simulator uses the static
packData(STATION\*) method from the ``DataPacker`` class to pack the
traffic condition data into a message.

The message contains static data, such as station drop numbers, and
dynamic data, like volume and occupancy.

+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Byte #                | Value                                                             | Description                                                                                                                                                                                                                                               |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1                     | VARIES                                                            | Station drop number                                                                                                                                                                                                                                       |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 2                     | # of loops \* 2 + Fixed\_Byte\_To\_Checksum                       | Describes number of loops at station                                                                                                                                                                                                                      |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 3                     | high = # of opp loops, low = # of ml loops, value = high \| low   | Describes number of ML and OP loops                                                                                                                                                                                                                       |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 4                     | 0xAO                                                              | UNKNOWN                                                                                                                                                                                                                                                   |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 5 to 8                | VARIES                                                            | Lane configuration                                                                                                                                                                                                                                        |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 9                     | 0                                                                 | 0 = no metering                                                                                                                                                                                                                                           |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10 to 13              | 0                                                                 | Lane malfunction flags. We assume all are functional and set to 0                                                                                                                                                                                         |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 14 to 22              | Values are constants                                              | Although constant, values differ between metering lanes and regular lane types. UNKNOWN?                                                                                                                                                                  |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 23                    | VARIES                                                            | ML Tot Volume                                                                                                                                                                                                                                             |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 24                    | VARIES                                                            | OP Tot Volume                                                                                                                                                                                                                                             |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 25                    | 0x03                                                              | UNKNOWN                                                                                                                                                                                                                                                   |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 26                    | 0x84                                                              | UNKNOWN                                                                                                                                                                                                                                                   |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 27 to Last Byte - 1   | VARIES                                                            | Dynamic data for loop detectors. If there is data for the loop detector, pack volume and occupancy into a two byte data packet using the packVOLOCC(vol, occ) method, insert at the current position, and increase the current position after doing so.   |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Last Byte             | VARIES                                                            | check sum value                                                                                                                                                                                                                                           |
+-----------------------+-------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

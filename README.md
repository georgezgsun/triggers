The CopTrax automation triggers simulator
This is a project that uses raspberry pi to simulate triggers, RADAR signals, and audio signals for CopTrax automation test. The triggers used for CopTrax automations are Siren, Light bar, Aux 4, Aux 5, Aux 6, LightSwitch, Mic1Trigger, Mic2Trigger and ignition. The simulated RADAR signals have two purposes. 1 to tell the firmware on the CopTrax unit to disable the charging of the Lithium Battery. 2 to simulate the target vechicle speed.

The main.cpp is the main module which implements the central control of all the simulations. In brief, it get the instructions from the remote automation server on the intra networks over TCP/IP. Then it executes the instructions by output the required signals over the I2C chips and reports the results to the server. The radar module simulates the RS232 signals from a real RADAR. The audio module will play the pre-saved audio files. 

The proposal on the communication protocol between the automation server and raspberry pi are listed as followings.

  The Raspberry Pi simulator acts as the server in the TCP/IP Socket communication. It listens on the port 8080 for any connection from the automation server.
  The automation server tries to connect the Raspberry Pi 1 on it IP address and port 10.0.9.199:8080 or Raspberry Pi 2 on 10.0.9.198:8080.
  When the connection is accepted by Raspberry Pi, both Raspberry Pi and the automation server store the socket for future communication.
  The commands the automation server sending to Raspberry Pi and the replies the Raspberry Pi sending back will always be three bytes, where the first two bytes are command and parameter, the third byte is the id. Any space character in the beginning will be consider as the separation between commands. For example, t11 m02 t33 …
  The command can be sent individually or in sequence in a TCP/IP packet. For example, TCPSend “t11” and TCPSend “t11 m02 t33” are all acceptable.
  The server sends the commands and does not wait for any instant replies. It will read the replies later.
  Raspberry Pi receives a packet, parses the command sequence from it, executes the commands in sequence one by one, and sends a reply after executing any commands.  
  The automation server will send heart beat command to Raspberry Pi every minute.
  In case the automation server does not receive any replies from Raspberry Pi for more than 2 mins, the automation server will consider the connection lost. It will try to setup another connect immediately.

Supported commands and parameters from the automation server to Raspberry Pi are as followings: 
a. t1 - trigger the siren button 
b. t3 - trigger the light bar button 
c. t4 – trigger the Aux 4 button 
d. t5 – trigger the Aux 5 button 
e. t6 – trigger the Aux 6 button 
f. t7 – trigger the lightswitch recording button 
g. m1 – trigger the button on the microphone 1
h. m2 – trigger the button on the microphone 2
i. h0 – heart beat handshake signal only

The replies from the Raspberry Pi to automation server will be as followings: 
a. OK – The corresponding command (with the same ID)has been executed successfully. 
b. NO - The corresponding command (with the same ID)was not executed or something wrong.

The raspberry pi will output power and ignition on signal at the very beginning.

The raspberry pi will keep output the radar signals by a seperated thread.

There are three I2C chips on board.
Chip1, named U, has a address 001 or 0x19, its configuration is 0xC0, 
	Trigger 1 -> t1 -> Siren;
	Trigger 2 -> t3 -> LightBar;
	Trigger 3 -> t4 -> Aux4;
	Trigger 4 -> t5 -> Aux5;
	Trigger 5 -> t6 -> Aux6;
	Trigger 6 -> t7 -> LightSwitch;
	Audio_Bank;
	unused
Chip2, named U, has a address 010 or 0x1A, its configuration is 0x00
	Trigger 8:1 <- input from UUT 1;

	Trigger 8:8 <- input from UUT 8;
Chip3, named U, has a address 011 or 0x1B, its configuration is 0x03,
	Trigger 8:9 <- input from UUT 9;
	Trigger 8:10 <- input from UUT 10;
	Power -> output of power;
	Ignition -> output of Ignition;
	Mic_Trigger_1 -> m1 -> Mic1Trigger
	Mic_Trigger_2 -> m2 -> Mic2Trigger
	unused
	unused
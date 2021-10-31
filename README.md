##IOT Fall Detection + MuleSoft + Twilio

###WHATS IT ABOUT
"Save Me" is a health domain IOT project which is specifically meant to detect a person's fall during an accident, slip or a sudden trip movement. "Fall Detection" is relatively a simple term to think about but in reality it is a complex task to achieve. It includes sensorial data from acclerometer and gyroscope from your health device/equipment on which we need to perform calculation considering about the trajectory of the sensor data changes and differential movements.

###HOW WE ACHIEVED THAT
####ENTER ARDUINO
Our project, for example, takes an arduino board (uno wifi rev2) as the health device. The embedded IMU unit in the arduino consists of the accelerometer and gyroscope sensor to determine the readings. Arduino is continuously sending sensorial data over HTTP to the mulesoft APIs.

{
"fallDetection": true,
"pedometer": 0,
"accelerometer": "-0.45,0.04,1.98"
}

The event is a json object consisting of three critical pieces:
1. **fallDetection** (determines if a fall has been detected or not)
2. **pedometer** (contains the steps taken by the person)
3. **accelerometer** (contains data about the accelerometer readings)

IOT Fall Detection + MuleSoft + Twilio 2
####ENTER MULESOFT
Once the event has been recieved from arduino via an HTTP POST request, mulesoft
Publishes the original event to a FIFO Messaging Queue so that the arduino
microcontroller code can quickly get the http response back and continue to loop over
the code to send us precise data readings.

![image](https://user-images.githubusercontent.com/49921179/139592249-134ccd7c-c55d-4494-b748-a60553111355.png)

Mulesoft Subscribes to those event and determines if the event has fallDetection as
true and sends it over to the Group Based Aggregator. The aggregator collects events
for that user till it either aggregates 180 events OR 3 minute timeout for that group.

Once either the aggregated size or the timeout has been reached, mulesoft takes all
those events and processes them by comparing the latter reading to the prior.

"-0.45,0.04,1.98" => this reading is in the format of "x_axis,y_axis,z_axis"

If the difference between the axis is greater than 0.30 (its a callibrated baseline for
considering the differene as an actual movement) for any two dimension , then that
event is considered as a movement. Based on all the events received we calculate the
movement% like this :

movement% = ( eligible movement events / total no. of events received ) * 100

IOT Fall Detection + MuleSoft + Twilio 3
We also extract the maximum no. of steps taken from the pedometer values.

The movement% and the pedometer count are the factors which determine what kind
of physical activity is the person capable to doing (can he walk , can he move , is he
unconscious) and mulesoft then sends sms and call based on the severity (LOW, HIGH,
MED)


####ENTER TWILIO
Mulesoft Uses the below mentioned twilio APIs to send sms and make voice calls.

SMS API : https://www.twilio.com/docs/sms/api

POST REQUEST BODY
output application/x-www-form-urlencoded
---
{
Body: "YourSampleBody",
From: "your_twilio_number",
To: "recipient_number"
}

VOICE API : https://www.twilio.com/docs/voice/api

POST REQUEST BODY
output application/x-www-form-urlencoded
---
{
From: "your_twilio_number",
To: "recipient_number"
Url: "post endpoint serving response in TwiML format"
}

IOT Fall Detection + MuleSoft + Twilio 4
NOTE: Please refer to this twilio doc to learn more about TwiML Responses
https://www.twilio.com/docs/messaging/twiml

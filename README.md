
const int buzzer     = 9;
byte statusLed       = 13;
byte sensorInterrupt = 0; 
byte sensorPin       = 2;
float calibrationFactor = 4.5;             
volatile byte pulseCount;  
float flowRate;
unsigned int flowMilliLitres;
unsigned long totalMilliLitres;
unsigned long oldTime;

void setup()
{
  Serial.begin(9600);                                                                 
  pinMode(buzzer,OUTPUT);                                                             
  pinMode(statusLed, OUTPUT);                                                        
  digitalWrite(statusLed, HIGH);                                                      
  pinMode(sensorPin, INPUT);
  digitalWrite(sensorPin, HIGH);                                                      // make waterflow sensor pin active

  pulseCount        = 0;
  flowRate          = 0.0;
  flowMilliLitres   = 0;
  totalMilliLitres  = 0;
  oldTime           = 0;
  
  attachInterrupt(sensorInterrupt, pulseCounter, FALLING);                             
}

/**
 * Main program loop
 */
void loop()
{
   
   if((millis() - oldTime) > 1000)                                                     // Only process counters once per second
  { 
    detachInterrupt(sensorInterrupt);                                                  // Disable the interrupt while calculating flow rate 
    flowRate = ((1000.0 / (millis() - oldTime)) * pulseCount) / calibrationFactor;     // Measure the number of pulses per second per units of measure coming from the sensor.
    oldTime = millis();                                                                // Measure the number of pulses per second per units of measure coming from the sensor.
    flowMilliLitres = (flowRate / 60) * 1000;                                          // Divide the flow rate in litres/minute by 60 to determine how many litres have passed through sensor
    totalMilliLitres += flowMilliLitres;                                               // Add the millilitres passed in this second to the cumulative total
      
    unsigned int frac;
   
    Serial.print("Flow rate: ");                                                       // Print the flow rate for this second in litres / minute
    Serial.print(int(flowRate));                                                       // print integer part of variable
    Serial.print("L/min");
    Serial.print("\t");      
 while(flowRate> 10){                            
    Serial.println("**************** \t\t LEAK \t\t ****************");                // If its greater than 10l then print leak in serial monitor
    delay(1000);
    tone(buzzer,1000);
    delay(1000);
    }
    
    Serial.print("Output Liquid Quantity: ");                                        // Print the cumulative total of litres flowed since starting   
    Serial.print(totalMilliLitres);
    Serial.println("mL");       
    Serial.print(totalMilliLitres/1000);
    Serial.print("L");
    pulseCount = 0;                                                                  // Reset the pulse counter so we can start incrementing again
    attachInterrupt(sensorInterrupt, pulseCounter, FALLING);                         // Enable the interrupt again now that we've finished sending output
  }
}

/*
Interrupt Service Routine
 */
void pulseCounter()
{
  pulseCount++;                                                                      // Increment the pulse counter
}

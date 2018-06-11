# LG-PACK
Lg pack paralleling
#include <mcp_can.h>
#include <SPI.h>

const int SPI_CS_PIN1 = 9; // select pin 9
const int SPI_CS_PIN2 = 10 ; // select pin 10



//Variables
int buttonState1= 0; // Button state : not pressed = 0, pressed = 1
int buttonState2= 0;
int buttonState3= 0;
int buttonState4= 0;

MCP_CAN CAN1 (SPI_CS_PIN1); 
MCP_CAN CAN2 (SPI_CS_PIN2);   
                                     // Set both CS pin

void setup()
{
    
   Serial.begin(115200);

    while (CAN_OK != CAN1.begin(CAN_500KBPS) || CAN_OK != CAN2.begin(CAN_500KBPS) )              // init can bus 1 & 2 with baudrate = 500k
    {
        Serial.println("CAN BUS Shield init fail");
        Serial.println(" Init CAN BUS Shield again");
        delay(100);
    }
    Serial.println("CAN BUS Shield init ok!");
}


unsigned char data1[8] = {68,00,00,00,00,00,00,00}; // Turn on contactor
unsigned char data2[8] = {00,00,00,00,00,00,00,00}; // Turn off contactor
unsigned char data3[8] = {04,00,00,00,00,00,00,00}; // Wake up can
unsigned char data4[8] = {20,00,00,00,00,00,00,00}; // Reset 

void loop()
{
    unsigned char len1 = 0;
    unsigned char buf1[8];
    unsigned char len2 = 0;
    unsigned char buf2[8];
    float Voltage1;
    float Voltage2;
    
    CAN1.readMsgBuf(&len1, buf1); 
    CAN2.readMsgBuf(&len2, buf2);  // read data,  len: data length, buf: data buf
    unsigned int canId1 = CAN1.getCanId();
    unsigned int canId2 = CAN2.getCanId();
    
if (canId1==768)
      {
            unsigned int x1= buf1[2]; //Select bit
            unsigned int y1= buf1[3];
            unsigned int z1= x1*256 + y1; //concatenate
            Voltage1 = z1*0.002; //Convert obtained decimal value to voltage
            Serial.println(Voltage1);
            delay(1000);
      }
 if (canId2==768)
      {
            unsigned int x2= buf2[2]; //Select bit
            unsigned int y2= buf2[3];
            unsigned int z2= x2*256 + y2; //concatenate
            Voltage2 = z2*0.002; //Convert obtained decimal value to voltage
            Serial.println(Voltage2);
            delay(1000);
     }

float difference = Voltage1-Voltage2;
  if (difference < 3 && difference > -3)
      {
            CAN1.sendMsgBuf(0x700,0,8,data1);
            CAN2.sendMsgBuf(0x700,0,8,data1);
            Serial.println("Open both contactors");
      }

      if(Voltage1>Voltage2) 
      {
        Serial.println("Error : Volatge difference above 3V");
        CAN1.sendMsgBuf(0x700,0,8,data2);
        CAN2.sendMsgBuf(0x700,0,8,data2);
      }      
 if(Voltage2>Voltage1)
    {
        Serial.println("Error : Volatge difference above 3V");
        CAN2.sendMsgBuf(0x700,0,8,data2);
        CAN1.sendMsgBuf(0x700,0,8,data2);
      }
     
 if(canId1==1146)
 {

  Serial.println("Error message received from pack 1");
  unsigned char a1= buf1[0]; //Select bit
  unsigned char b1= buf1[1];
  unsigned char c1= buf1[2]; 
  unsigned char d1= buf1[3];
  unsigned char e1= buf1[4]; 
  unsigned char f1= buf1[5];
  unsigned char g1= buf1[6]; 
  unsigned char h1= buf1[7];
  unsigned char errordata1[8]= {a1,b1,c1,d1,e1,f1,g1,h1};
  CAN2.sendMsgBuf(0x701,0,8,errordata1);
  delay(1000);
 }
  if(canId2==1146)
 {

  Serial.println("Error message received from pack 2");
  unsigned char a2= buf2[0]; //Select bit
  unsigned char b2= buf2[1];
  unsigned char c2= buf2[2]; 
  unsigned char d2= buf2[3];
  unsigned char e2= buf2[4]; 
  unsigned char f2= buf2[5];
  unsigned char g2= buf2[6]; 
  unsigned char h2= buf2[7];
  unsigned char errordata2[8]= {a2,b2,c2,d2,e2,f2,g2,h2};
  CAN1.sendMsgBuf(0x701,0,8,errordata2);
  delay(1000);
 }
 
}

// END FILE

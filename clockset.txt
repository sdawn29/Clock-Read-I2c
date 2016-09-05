#include<Wire.h>
#define slave_add 0x68
int m,s,h,date,month,year,day,f=0;
String ti;
#define _pin 13

void blink(void){
  digitalWrite(_pin, HIGH);
  delay(50);
  digitalWrite(_pin, LOW);
  delay(50);
}

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Wire.begin();
  pinMode(_pin, OUTPUT);

  int sec,mi,hrs,da;
  Serial.println("Enter ss (ss:mm:hh dd:mm:yy): ");  
  while(Serial.available()==0);
  delay(20);
  sec=Serial.parseInt();
  Serial.println(sec);
  
  int r=sec % 10;
  int n=sec/10;
  unsigned char ensec=((n & 0x7) <<4) | (r & 0xf) | (1<<7); // ss + start clock

  blink();

  Serial.println("Enter mm (ss:mm:hh dd:mm:yy): ");  
  while(Serial.available()==0);
  delay(20);
  mi=Serial.parseInt();
  Serial.println(mi);

  int tmin=mi;
  int r1=tmin % 10;
  int n1=tmin/10;
  unsigned char enmin=((n1 & 0x7) <<4) | (r1 & 0xf);

  blink();

  Serial.println("Enter hh (ss:mm:hh dd:mm:yy): ");  
  while(Serial.available()==0);
  delay(20);
  hrs=Serial.parseInt();
  Serial.println(hrs);
  
  int thrs=hrs;
  int r2=thrs % 10;
  int n2=thrs/10;
  unsigned char enhrs=((n2 & 0x3) <<4) | (r2 & 0xf);

  blink();

  Serial.println("Enter dd (ss:mm:hh dd:mm:yy): ");  
  while(Serial.available()==0);
  delay(20);
  da=Serial.parseInt();
  Serial.println(da);

  unsigned char enday=(da & 0x7);

  blink();

  Wire.beginTransmission(slave_add);
  Wire.write((unsigned char)0);
  Wire.write(ensec);
  Wire.write(enmin);
  Wire.write(enhrs);
  Wire.write(enday);
  Wire.endTransmission();

  blink();
  blink();

  delay(200);
}


void loop() {
  // put your main code here, to run repeatedly:
  Wire.beginTransmission(slave_add);
  Wire.write((unsigned char)0);
  Wire.endTransmission();


  Wire.requestFrom(slave_add,7);
  unsigned char x1=Wire.read();
  unsigned char x2=Wire.read();
  unsigned char x3=Wire.read();
  unsigned char x4=Wire.read();
  unsigned char x5=Wire.read();
  unsigned char x6=Wire.read();
  unsigned char x7=Wire.read();

  s=(x1 & 0xf) + (10*(x1 & 0x70)>>4);
  m=(x2 & 0xf) + (10*(x2 & 0x70)>>4);

  h |= (x3 & 0xf);
  if (x3 & (1 << 6)) { // 12 hour mode
    h += (x3 & (1 << 4)) ? 10 : 0;
    ti = (x3 & (1 << 5)) ? " PM" : " AM";
  } else { // 24 hour mode
      h += ((x3 & 0x30) >> 4) * 10;
      ti = " HH";
    }
h = x3;
day= x4 & 0x7;
date= (x5 & 0xf)+(10 * (x5 & 0x30)>>4);
month=(((x6 & (0x1)<<4)>>4)*10) + (x6 & 0xf);
year=((x7 & 0xf0)>>4)*10 + (x7 & 0xf);
Serial.println("time=");
Serial.print(month);
Serial.print("\\");
Serial.print(date);
Serial.print("\\");
Serial.print(year);
Serial.print("  ");
Serial.print(day);
Serial.print("  ");
Serial.print(h);
Serial.print(":");
Serial.print(m);
Serial.print(":");
Serial.print(s);
Serial.print(ti);
Serial.println();


if (x1 & (1 << 7)){
Wire.beginTransmission(slave_add);
Wire.write(0);

Wire.write(x1 & 0x7f);
Wire.endTransmission();
}


  delay(1000);
}
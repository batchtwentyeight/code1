#include<reg52.h>
idata unsigned char rcg,i,count=0,chr='x',chr1='x';
unsigned char pastnumber[11];
struct s_time
{
unsigned char sec;
unsigned char min;
unsigned char hour;
unsigned char date;
unsigned char month;
unsigned char year;
};

struct alrm_time
{
unsigned char min;
unsigned char hour;
};

struct alrm_time alm[20];

#define NUM_OF_ALRMS 20//set number of alarms


sbit ent=P3^4;
sbit inc=P3^3;
sbit dec=P3^2;


sbit m1a = P1^2;
sbit m1b = P1^3;

sbit m2a = P1^4;
sbit m2b = P1^5;

sbit m3a = P1^6;
sbit m3b = P1^7;

sbit voice1 = P0^0;
sbit voice2 = P0^1;
sbit voice3 = P0^2;
sbit voice4 = P0^3;
sbit voice5 = P0^4;
sbit voice6 = P0^5;


sbit ir1   = P3^5;
sbit ir2   = P3^6;
sbit ir3   = P3^7;


#include"lcd.h"
#include"rtc.h"
//#include"serial.h"

//#include"other.h"

struct s_time p_time;//present
struct s_time u_time;//update
struct s_time a_time;//alarm

#define buzzer_on buz=0;
#define buzzer_off buz=1;
  
void serinit()
  {
    TMOD=0x20;
    TH1=0xFD;	  //9600
    SCON=0x50;
    TR1=1;
  }
  
 unsigned char receive()
 {
  unsigned char rx; 
   while(RI == 0);
    rx=SBUF;
	RI=0;
	return rx;
 }

void sertxs(unsigned char *tx)
 {
    //unsigned char v;
   for(;*tx != '\0';tx++)
     {
	   SBUF=*tx;
	  while(TI == 0);
	   TI=0;
	  // v= receive();
	   //delay(2);
	   
	 }
 }

void sertx(unsigned char tx)
 {
  	   ///unsigned char v;
	   SBUF=tx;
	  while(TI == 0);
	   TI=0;
	     //v= receive();
		 //delay(2);
 }

void okcheck()
  {
  	do{
	
	rcg=receive();
	}	while(rcg!='K');
  }	


void okc()
  {
  	do{
	
	rcg=receive();
	}	while(rcg!='K');
  
  }

void gsm_send(unsigned char *rchr)
{	
   	     sertxs("AT+CMGS=\"");
		 sertxs(pastnumber);							
		 sertxs("\"\r\n");	  delay(500);
		 for(;*rchr != '\0';rchr++)
		    {sertx(*rchr);}
   		 sertx(0x1A);delay(700);
}
int main()
{
//struct s_time *temp_time;
char temp_sec=0,alrm_count=0;
unsigned char rec,cnt=0,k=0,jj=0;
//code unsigned char mobile[13]="9703798193\0",mobile1[13]="8142761296\0",mobile2[13]="7674916567\0";

ent=inc=dec=1;
voice1=voice2=voice3=1;
voice4=voice5=voice6=1;
ir1=ir2=ir3=1;
m1a=m1b=0;m2a=m2b=0;m3a=m3b=0;
	  /*
m1a=1;m1b=0;delay(500);m1a=m1b=0;
delay(500);
m1a=0;m1b=1;delay(500);m1a=m1b=0;


m2a=1;m2b=0;delay(500);m2a=m2b=0;
delay(500);
m2a=0;m2b=1;delay(500);m2a=m2b=0;


m3a=1;m3b=0;delay(500);m3a=m3b=0;
delay(500);
m3a=0;m3b=1;delay(500);m3a=m3b=0;
		*/

P2 = 0xff;

initlcd();
serinit(); 

		/*
			 while(1){
			           m1a=1;m1b=0;   delay(500); m1a=m1b=0;  delay(700);
					   m1a=0;m1b=1;   delay(500); m1a=m1b=0;  delay(700);
			         }	   */
 //wifiinit();
 //delay(800);

  
  sertxs("AT\r\n");okcheck();
  sertxs("ATE0\r\n");okcheck();
  sertxs("AT+CMGF=1\r\n");okcheck(); 
  sertxs("AT+CNMI=1,2,0,0\r\n");okcheck();  
  sertxs("AT+CSMP=17,167,0,0\r\n");okcheck(); 

  clcd(0x01);
  clcd(0x80);
  stringlcd(0x80,"SEND A MESSAGE TO STORE MOB. NO:");

     do{
    	 rcg=receive();
       }while(rcg != '*'); 

	  for(count=0;count<10;count++)
	      {
		   pastnumber[count]=receive();
		  }

		  clcd(1);
		  stringlcd(0x80,pastnumber);	  

		 sertxs("AT+CMGS=\"");
		 sertxs(pastnumber);							
		 sertxs("\"\r\n");	  delay(500);
   		 sertxs("Mobile no. registered\r\n");
   		 sertx(0x1A);delay(700);
	


for(k=0;k<NUM_OF_ALRMS;k++)
   {
	alm[k].hour=read((k+5)*2); delay(100);
	alm[k].min=read(((k+5)*2)+1);delay(100);
   }
//txs("welcome");
clcd(1);
stringlcd(0x80,"time>   hh:mm:ss");
stringlcd(0xc0,"date>   dd/mm/yy");

		   
		   temp_sec=read(0);
		   save(0x00,(temp_sec&0x7f));

//read alrm



		   if(ent==0)//edit mode
		   {   clcd(1);
		   stringlcd(0x80,"release enter");
		   while(ent==0);
		   cnt=1;
		   //stringlcd(0x80,"serial_edit.");
		   /*while(1)
		        {
		   	     if(inc==0){while(inc==0);cnt=0;stringlcd(0x80,"serial_edit.");delay(400);}
 		   	     if(dec==0){while(dec==0);cnt=1;stringlcd(0x80,"switch_edit.");delay(400);}
			     if(ent==0){while(ent==0);break;}
		        }
		   */

		   if(cnt==1)
		     { //switch mode
		   		  p_time=readtime();
				  cnt=0; clcd(1);
				  stringlcd(0x80,"time_edit.");
				   while(1){
				   	 if(inc==0){delay(200);while(inc==0);cnt=0;stringlcd(0x80,"time_edit.");delay(400);}
		 		   	 if(dec==0){delay(200);while(dec==0);cnt=1;stringlcd(0x80,"alrm_edit.");delay(400);}
					 if(ent==0){delay(200);while(ent==0);break;}
					 }
					 	if(cnt==0){	//time edit

						 
						 stringlcd(0x80,"time>   hh:mm:ss");
						 stringlcd(0xc0,"date>   dd/mm/yy");
						 time_display(p_time);
						u_time.sec=0;
						u_time.min=time_edit(p_time.min,0,60,0x8b);
						u_time.hour=time_edit(p_time.hour,0,24,0x88);

						u_time.year=time_edit(p_time.year,0,100,0xce);
						u_time.month=time_edit(p_time.month,1,13,0xcb);
						u_time.date=time_edit(p_time.date,1,32,0xc8);
						
						settime(u_time);
						goto oc;
						}//end of time edit
						if(cnt==1){//alrm edit

								stringlcd(0x80,"alrm count:");							
								alrm_count = time_edit(1,1,20,0xc0);
							 			 delay(300);
							stringlcd(0x80,"time>   hh:mm:00");

							for(k=0;k < alrm_count; k++){
									rec=read((k+5)*2); delay(100);
									cnt=read(((k+5)*2)+1);
									clcd(0x88);conv(rec);
									clcd(0x8b);conv(cnt);
									stringlcd(0xc0,"alarm No: ");conv(k+1);
									alm[k].min=time_edit(cnt,0,60,0x8b);
									alm[k].hour=time_edit(rec,0,24,0x88);
							}
							stringlcd(0xc0,"          ");
						    mem_update(alrm_count);
			   				goto oc;
						}//end of alrm edit
		   } //end of switch mode
		   
		   } //end of edit mode
		   else
		   {	
		   for(jj=0;jj<NUM_OF_ALRMS;jj++)
		      {
		       if(p_time.hour<=alm[jj].hour)	
			     {
		   			if(p_time.min<alm[jj].min)	
					  {
		   				almnxt_display(alm[jj]);
					  }
					goto oc; 
				 }
		  	  }

						almnxt_display(alm[0]);

		 		  while(1)
				   {
			oc:	   p_time=readtime();
				   time_display(p_time);
				   delay(300);


				   //if(sw1 == 0){stringlcd(0x86,"1");while(sw1 == 0);stringlcd(0x86," ");voice4=0;delay(700);voice4=1;}
				   for(jj=0;jj<NUM_OF_ALRMS;jj++)
						{
							if((p_time.hour==alm[jj].hour)&&(p_time.min==alm[jj].min)&&((p_time.sec>0)&&(p_time.sec<50)))
							{
									if(jj<NUM_OF_ALRMS-1)
									{
									almnxt_display(alm[jj+1]);
									}
									else
									{almnxt_display(alm[0]);
									} 
									 
								if(jj == 0 && ((p_time.sec>0)&&(p_time.sec<3)))
								  {
								  if(ir1 == 0)
								     {
									  clcd(1);stringlcd(0x80,"Med-1 Avilable");
									  m1a=1;m1b=0;  delay(500); m1a=m1b=0;  delay(900);
									  voice1=0;     delay(900); voice1=1;
									  m1a=0;m1b=1;  delay(500); m1a=m1b=0;  delay(200);
									  gsm_send("Medicine-1 Reminder - Avilable\r\n");
									 }
								  if(ir1 == 1)
								     {
									  clcd(1);stringlcd(0x80,"Med-1 Notavilble");
									  voice4=0; delay(900);voice4=1;
									  gsm_send("Medicine-1 Reminder - Not avilable\r\n");
									 }	delay(900);clcd(1);
								  }
								if(jj == 0 && ((p_time.sec>10)&&(p_time.sec<50)))
								  {
								   if(ir1 == 0){clcd(1);stringlcd(0x80,"Med-1 Didn't");
								   stringlcd(0xc0," Collect ");
								   gsm_send("Med-1 Did't Collect\r\n");
								   }
								  }

							 
								if(jj == 1 && ((p_time.sec>0)&&(p_time.sec<3)))
								  {
								  if(ir2 == 0)
								     {
									  clcd(1);stringlcd(0x80,"Med-2 Avilable");
									  m2a=1;m2b=0;  delay(500); m2a=m2b=0;  delay(900);
									  voice2=0;     delay(900); voice2=1;
									  m2a=0;m2b=1;  delay(500); m2a=m2b=0;  delay(200);
									  gsm_send("Medicine-2 Reminder - Avilable\r\n");
									 }
								  if(ir2 == 1)
								     {
									  clcd(1);stringlcd(0x80,"Med-2 Notavilble");
									  voice5=0; delay(900);voice5=1;
									  gsm_send("Medicine-2 Reminder - Not avilable\r\n");
									 }	delay(900);clcd(1);
								  }									 
								if(jj == 1 && ((p_time.sec>10)&&(p_time.sec<50)))
								  {
								   if(ir2 == 0){clcd(1);stringlcd(0x80,"Med-2 Didn't");
								   stringlcd(0xc0," Collect ");
								   gsm_send("Med-2 Did't Collect\r\n");
								   }
								  }

								
								if(jj == 2 && ((p_time.sec>0)&&(p_time.sec<3)))
								  {
								  if(ir3 == 0)
								     {
									  clcd(1);stringlcd(0x80,"Med-3 Avilable");
									  m3a=1;m3b=0;  delay(500); m3a=m3b=0;  delay(900);
									  voice3=0;     delay(900); voice3=1;
									  m3a=0;m3b=1;  delay(500); m3a=m3b=0;  delay(200);
									  gsm_send("Medicine-3 Reminder - Avilable\r\n");
									 }
								  if(ir3 == 1)
								     {
									  clcd(1);stringlcd(0x80,"Med-3 Notavilble");
									  voice6=0; delay(900);voice6=1;
									  gsm_send("Medicine-3 Reminder - Not avilable\r\n");
									 } delay(900);clcd(1);
								  }								 
								if(jj == 2 && ((p_time.sec>10)&&(p_time.sec<50)))
								  {
								   if(ir2 == 0){clcd(1);stringlcd(0x80,"Med-3 Didn't");
								   stringlcd(0xc0," Collect ");
								   gsm_send("Med-3 Did't Collect\r\n");
								   }
								  }

								  delay(900);clcd(1);
									
							}
						}
				   }
		   }
		  
/*while(1)
{
p_time=readtime();
time_display(p_time);

delay(400);

}*/

return 0;
}

# hostingraja.in-Enable-port26-in-postfix

The code enables the port 26 in postfix

#!/usr/bin/env bash

smtp_log=/var/log/smtp_log;

if [ "$#" -lt 1 ]

then

echo " " >/dev/null;



else

date_data=$(date -u);

echo -e "SMTP  port enable and disable process start at $date_data(UTC)..........." >>$smtp_log;

ver_d=$(rpm -qa \*-release | grep -Ei "oracle|redhat|centos" | cut -d"-" -f3);

pos_status=0;

path_status=1;

syml_status=1;

csf_status=0;

if  yum list installed  |  grep -F 'postfix' >>/dev/null;

then

echo "">/dev/null;

else

pos_status=1;

fi

master_file_path="/etc/postfix/master.cf";

ext_d="back_g";

c_path="/etc/postfix/master.cf";

d_path="/etc/postfix/master.cf";

csf_path="/etc/csf/csf.conf";

if [ -f  "$master_file_path" ]

then

path_status=0;



if [ -h  "$master_file_path" ]

then

syml_status=0;

echo "File is symlink ,lets handle with care " >>$smtp_log;

fi

else

echo "Postfix master.cf file is not exist" >> $smtp_log;

fi

if [ -f  "$csf_path" ]

then

csf_status=0;

else

csf_status=1;

#echo  "Please install csf  ........">>$smtp_log;

fi



if [ "$path_status" == 0 ] &&  [ "$pos_status"  == 0 ]

then

case  $1 in

enable)

if [ "$csf_status" == 0 ]

then



if [ "$syml_status" == 0 ]

then

c_path=$(readlink $master_file_path);

fi

d_path=$c_path$ext_d;

          if [ -f "$d_path" ];

then

yes | cp -avr  $c_path  $d_path >>$smtp_log;

else

cp -avr $c_path  $d_path >>$smtp_log;
            

fi



n_csf_path=$csf_path$ext_d;

if [ -f "$n_csf_path" ];

then

yes | cp -avr  $csf_path  $n_csf_path >>$smtp_log;

else

cp -avr $csf_path  $n_csf_path >>$smtp_log;



fi

#sed -i '/^26.*smtpd$/d'  $c_path

sed -i '/^26.*smtpd/d'  $c_path

echo "26     	inet  n       -       n       -       -       smtpd" >>$c_path;

port_detail=$(grep -o "TCP_IN.*\""  $csf_path  | sed -e 's/ //g' | sed 's/\"//g' | sed -e "s/\'//g" | sed -e 's/TCP_IN=//g');

port_detail1=$(echo $port_detail | sed  -e "s/\,/ /g");



port_odetail=$(grep -o "TCP_OUT.*\""  $csf_path  | sed -e 's/ //g' | sed 's/\"//g' | sed -e "s/\'//g" | sed -e 's/TCP_OUT=//g');

port_odetail1=$(echo $port_detail | sed  -e "s/\,/ /g");





port_exist1=0;

port_exist2=0;

for a in $port_detail1

do



if [ "$a" == 26 ]

then

port_exist1=1;

fi



done



for b in $port_odetail1

do



if [ "$b" == 26 ]

then

port_exist2=1;

fi

done

if [ "$port_exist1" == 0 ]

then

line1=$(grep -n "TCP_IN.*\""  $csf_path | awk '{print $1}' FS=":");

if [ -z "$line1" ]

then

echo "" >/dev/null;

else

sed -i "${line1}d"  $csf_path

fi



if [ -z "$port_detail" ]

then

echo -e "TCP_IN = \"26\" " >> $csf_path;

else

echo -e "TCP_IN = \"$port_detail,26\" " >> $csf_path;

fi

fi



if [ "$port_exist2" == 0 ]

then



line2=$(grep -n "TCP_OUT.*\""  $csf_path | awk '{print $1}' FS=":");

if [ -z "$line2" ]

then

echo "" >/dev/null;

else

sed -i "${line2}d"  $csf_path

fi



if [ -z "$port_odetail" ]

then

echo -e "TCP_OUT = \"26\" " >> $csf_path;

else

echo -e "TCP_OUT = \"$port_odetail,26\" " >> $csf_path;

fi

fi

csf -r >/dev/null;



echo "------------------------------postfix restarted --------------------------" >>$smtp_log;

if [ "$ver_d" == 7 ]

then

systemctl restart postfix ;

else

service postfix restart ;

fi



echo 	"26 port enabled for SMTP" >>$smtp_log;

else

echo  "Please install csf  ........">>$smtp_log;

fi



;;

disable)

if [ "$csf_status" == 0 ]

then

if [ "$syml_status" == 0 ]

then

c_path=$(readlink $master_file_path);

fi

d_path=$c_path$ext_d;

if [ -f "$d_path" ];

then

yes | cp -avr  $c_path  $d_path >>$smtp_log;

else

cp -avr $c_path  $d_path >>$smtp_log;



fi

n_csf_path=$csf_path$ext_d;

if [ -f "$n_csf_path" ];

then

yes | cp -avr  $csf_path  $n_csf_path >>$smtp_log;

else

cp -avr $csf_path  $n_csf_path >>$smtp_log;



fi



port_detail=$(grep -o "TCP_IN.*\""  $csf_path  | sed -e 's/ //g' | sed 's/\"//g' | sed -e "s/\'//g" | sed -e 's/TCP_IN=//g');

port_detail1=$(echo $port_detail | sed  -e "s/\,/ /g");


port_odetail=$(grep -o "TCP_OUT.*\""  $csf_path  | sed -e 's/ //g' | sed 's/\"//g' | sed -e "s/\'//g" | sed -e 's/TCP_OUT=//g');

port_odetail1=$(echo $port_detail | sed  -e "s/\,/ /g");

          

port_exist1=0;

port_exist2=0;

p1="";

p2="";

for a in $port_detail1

do

if [ "$a" == 26 ]

then

port_exist1=1;

else

p1+="$a "

fi

done

for b in $port_odetail1

do



if [ "$b" == 26 ]

then

port_exist2=1;

else

p2+="$b ";

fi


done



#echo $outarray;

#echo 	$p1 |sed  -e "s/ /\,/g"

          

if [ "$port_exist1" == 1 ]

then

line1=$(grep -n "TCP_IN.*\""  $csf_path | awk '{print $1}' FS=":");

if [ -z "$line1" ]

then

echo "" >/dev/null;

else

sed -i "${line1}d"  $csf_path

fi



if [ -z "$p1" ]

then

echo -e "TCP_IN = \"\" " >> $csf_path;

else

p1=$(echo   $p1 |sed  -e "s/ /\,/g");

echo -e "TCP_IN = \"$p1\" " >> $csf_path;

fi

fi



if [ "$port_exist2" == 1 ]

then

line2=$(grep -n "TCP_OUT.*\""  $csf_path  | awk '{print $1}' FS=":");

if [ -z "$line2" ]

then

echo "" >/dev/null;

else

sed -i "${line2}d"  $csf_path

fi

if [ -z "$p2" ]

then

echo -e "TCP_OUT = \"\" " >> $csf_path;

else

p2=$(echo   $p1 |sed  -e "s/ /\,/g");

echo -e "TCP_OUT = \"$p2\" " >> $csf_path;

fi

fi



csf -r >/dev/null;

#				echo $p1;

#		echo $p2;

line3=$(grep -n "26.*smtpd"  $c_path | awk '{print $1}' FS=":");

if [ -z "$line3" ]

then

echo "" >/dev/null;

else

sed -i "${line3}d"  $c_path

fi



echo "------------------------------postfix restarted --------------------------" >>$smtp_log;

if [ "$ver_d" == 7 ]

then

systemctl restart postfix ;

else

service postfix restart ;

fi



echo "26 port  has disabled for SMTP " >>$smtp_log;

else

echo  "Please install csf  ........">>$smtp_log;

fi

;;

getsmtp)



if [ "$syml_status" == 0 ]

then

c_path=$(readlink $master_file_path);

fi

smtp_text=$(grep -n "^26.*smtpd" $c_path)

if [ -z "$smtp_text" ]

then

echo "26 port not enabled " >> $smtp_log;

echo "0";



else



echo "26 port enabled " >> $smtp_log;

echo "1";
          

fi



echo "Get smtp detail part" >>$smtp_log;



;;

*)

echo "Please use  valid data" >>$smtp_log;

;;

esac



fi
  

fi



->For mail we are using 25 port port but some client demand for 26 port . That's why smtp port enable and disable shell script 
came into existance

->In this shell script we are using three method to satisfy smtp port hunger client 

->These three methods are enable,disable,getsmtp

->We are using getsmtp to check , if port is already added to mail server or not .

-> If port is not added in server we are calling enable port method to enable in server .

-> To enable port in server we are going through some porcedure.First we are checking port is enable in firewall or not if not 
then we are enabling port in firewall  server.Then we are enabling in our mail server to send mail using 26 port.

->After enable 26 port,client can disable 26 port when ever he want by calling disable method



To enable 26 port in server client has to excute shell script like this 

Ex sh shellscript.sh enable 

To disable 26 port in server client has to run disable instead of enable 

Ex sh shellscript.sh disable

After run this command, rest of the operation part will be taken care by script

If client want to know what is happen when  he is running shell script he can check all detail in /var/log/smtp_log


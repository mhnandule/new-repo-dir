cat issue_script.sh
#! /bin/bash

loginDetails='/var/lib/backup/.sqlLogin.txt'
USRNM=`cat ${loginDetails}|awk '{print $1}'`
USRPWD=`cat ${loginDetails}|awk '{print $2}'`

# Set filename
outfile="Analysis-$(hostname)-$(date +%Y%m%d)-server-info.out"
date>>$outfile
echo>>$outfile

# Global Status x2 60s apart
echo >>$outfile
echo "Global Status x2:">>$outfile
mysql -ABNe "select version(); show global status; select sleep(60); show global status; show status like '%quer%';" | sort >>$outfile

# Global Variables
echo "Global Variables:">>$outfile
mysql -ABNe "show global variables;" | sort | sed 's/; /;~\&/g' | tr -s '~' '\n' | tr -s '\&' '\t' >>$outfile

# Plugins
echo >>$outfile
echo "Plugins:">>$outfile
mysql -ABNe "show plugins;">>$outfile

# Data size
echo >>$outfile
echo "Dataset Size:">>$outfile
mysql -ABNe "select ifnull(B.engine,'Total') \"Storage Engine\", concat(lpad(format( \
B.DSize/power(1024,pw),3),17,' '),' ',substr(' KMGTP',pw+1,1),'B') \"Data Size\", \
concat(lpad(format(B.ISize/power(1024,pw),3),17,' '),' ', \
substr(' KMGTP',pw+1,1),'B') \"Index Size\",concat(lpad(format(B.TSize/ \
power(1024,pw),3),17,' '),' ',substr(' KMGTP',pw+1,1),'B') \"Table Size\" \
from (select engine,sum(data_length) DSize, \
sum(index_length) ISize,SUM(data_length+index_length) TSize from information_schema.tables \
where table_schema not in ('mysql','information_schema','performance_schema') AND \
engine is not null group by engine with rollup) B,(SELECT 2 pw) A order by TSize;" >>$outfile

# Service relevant values
echo >>$outfile
echo "systemctl service timeouts:">>$outfile
systemctl show mariadb | grep "^Timeout" >>$outfile

# Kernel
echo >>$outfile
uname -r >>$outfile

# Available RAM
echo >>$outfile
echo "MemInfo and CPU Core Count">>$outfile
cat /proc/meminfo | grep MemTotal >>$outfile

# CPU cores
echo >>$outfile
echo "CPU cores" >>$outfile
cat /proc/cpuinfo | egrep "core id|physical id" | tr -d "\n" | sed s/physical/\\nphysical/g | grep -v "^$" | sort | uniq | wc -l >>$outfile

# Disk space
echo >>$outfile
echo "Disk space" >>$outfile
df -h >>$outfile

hi guys

## Linux 常用指令（日志分析篇）



grep 'turnFinishnull' info.log | awk '{print $1}' | sort | uniq -c

grep 'turnFinishnull' info.log | awk -F'=' '{print $2}'|awk '{print $1}'|sort -nr |head -20


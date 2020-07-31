FROM telegraf:1.15.1

ENV DB_ADDRESS="http://mdt_influxdb:8086"
EXPOSE 57000
COPY ./new.conf /etc/telegraf/telegraf.conf
RUN mkdir /home/data

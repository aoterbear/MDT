FROM telegraf:1.15.1

ENV DB_URL="http://mdt_influxdb:8086"
ENV DB_USERNAME="admin"
ENV DB_PASSWORD="admin"
ENV DB_DATABASE="mdt_db"
ENV SERVICE_ADDRESS="mdt_telegraf:57000"

EXPOSE 57000
RUN mkdir /home/data
COPY ./new.conf /etc/telegraf/telegraf.conf

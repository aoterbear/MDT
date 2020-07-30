FROM influxdb:1.8.1

EXPOSE 8086
COPY ./influxdb.conf /etc/influxdb/

ENV INFLUXDB_DB="mdt_db"
ENV INFLUXDB_ADMIN_USER="admin"
ENV INFLUXDB_ADMIN_PASSWORD="admin"

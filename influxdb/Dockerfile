FROM influxdb:1.8.1
#EXPOSE is not necessary here, just for accessing influxdb api,
EXPOSE 8086

#ENV INFLUXDB_DB="mdt_db"
#ENV INFLUXDB_ADMIN_USER="admin"
#ENV INFLUXDB_ADMIN_PASSWORD="admin"

COPY ./influxdb.conf /etc/influxdb/

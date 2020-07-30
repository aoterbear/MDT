# MDT

## Environment

Even though this is container, it should run anywhere, but I still want to say **It works on my environment.** :)

My environment is below:

`NAME="Ubuntu"
VERSION="18.04.4 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.4 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic`

`Docker version 19.03.9, build 9d988398e7`

`docker-compose version 1.26.2, build eefe0d31`

## How To Use?

1. Use git clone to copy this project on your own laptop
2. cd this folder
3. Excute cmd `docker-compose up -d`
4. Open Grafana UI with  http://localhost:3000, or change the "localhost" to your own specific IP address
5. The datastore is already been added, so, test the datastore connectivity first and then you can begin to configure the Grafana Dashboard

## Configuration on IOS-XR

Below is configuration on IOS-XRv 9000, version 7.1.1, this configuration works with this project default telegraf config located in `./telegraf/data/new.conf`, you need t




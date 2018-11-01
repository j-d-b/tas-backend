FROM ubuntu:latest

ARG port
ARG secret
ARG hour

RUN apt-get update
RUN apt-get -y install cron
RUN apt-get -y install curl
RUN apt-get -y install openssl
RUN apt-get -y install tzdata

RUN ln -fs /usr/share/zoneinfo/UTC /etc/localtime

COPY ./create-jwt /create-jwt
RUN chmod +x /create-jwt

RUN (crontab -l ; echo "0 $hour * * * curl 'http://server:${port}/graphql' -H 'Content-Type: application/json' -H 'Authorization: Bearer $(/create-jwt $secret)' --data-binary '{\"query\":\"mutation { sendApptReminders(input: {}) }\"}' > /proc/1/fd/1 2> /proc/1/fd/2") | crontab

CMD ["cron", "-f"]

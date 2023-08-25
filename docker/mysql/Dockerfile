FROM mysql:8.0-debian

RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29
RUN apt update
RUN apt install -y rsyslog less vim locales
RUN sed -i -e 's/# \(ja_JP.UTF-8\)/\1/' /etc/locale.gen
RUN locale-gen
RUN update-locale LANG=ja_JP.UTF-8

ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8

ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 3306
CMD ["mysqld"]
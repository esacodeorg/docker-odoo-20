FROM odoo:20
LABEL MAINTAINER="support@esacode.org"
LABEL org.opencontainers.image.vendor="esacode - solutions numeriques"
LABEL org.opencontainers.image.url="https://www.esacode.org/erp-services/"
LABEL org.opencontainers.image.source="https://github.com/esacodeorg/docker-odoo-20"
LABEL org.opencontainers.image.description="Odoo 20 Community, pret a l'emploi en Docker"
LABEL org.opencontainers.image.licenses="MIT"

USER root

RUN apt-get update      && \
    apt-get upgrade -y  && \
    apt-get install -y build-essential sox python3 python3-dev python3-pip

# COPY addons/requirements.txt /requirements.txt
# RUN pip install --break-system-packages -r /requirements.txt

# USER odoo

# install-odoo

Install developement / production [odoo](https://www.odoo.com/) from [git](https://github.com/odoo/odoo) with / without using [docker](https://www.docker.com/).

## Preparation

    apt-get update | grep "Hit http" -C 10000 && echo "Some packages are not loaded"
    apt-get install git -y
    git clone https://github.com/it-projects-llc/install-odoo.git
    cd install-odoo

    # if you got error after apt-get update, you probably need to update source list:
    sed -i 's/\/\/.*\.ec2\.//g' /etc/apt/sources.list
    apt-get update | grep "Hit http" -C 10000 && echo "Some packages are not loaded"
    

## Basic usage

    # run script with parameters you need
    # (list of all parameters with default values can be found at install-odoo-saas.sh)
    INSTALL_DEPENDENCIES=yes \
    INIT_POSTGRESQL=yes \
    INIT_BACKUPS=yes \
    INIT_NGINX=yes \
    INIT_START_SCRIPTS=yes \
    INIT_ODOO_CONFIG=yes \
    INIT_USER=yes \
    INIT_DIRS=yes \
    CLONE_ODOO=yes \
    CLONE_IT_PROJECTS_LLC=yes \
    CLONE_OCA=yes \
    UPDATE_ADDONS_PATH=yes \
    /bin/bash -x install-odoo-saas.sh

## After installation

    # show settings (admin password, addons path)
    head /etc/openerp-server.conf
    # show odoo version
    grep '^version_info ' $ODOO_SOURCE_DIR/openerp/release.py

    # PGTune: http://pgtune.leopard.in.ua/"

    # log
    tail -f -n 100 /var/log/odoo/odoo-server.log
    
    # start from console (for ODOO_USER=odoo):
    sudo su - odoo -s /bin/bash -c  "/usr/local/src/odoo-source/openerp-server -c /etc/openerp-server.conf"
    
    # psql (use name of your database)
    sudo -u odoo psql DATABASE
    
    # some common issues:
    # https://www.odoo.com/forum/help-1/question/dataerror-new-encoding-utf8-is-incompatible-with-the-encoding-of-the-template-database-sql-ascii-52124



## Installation in Docker

    # Install docker
    # see https://docs.docker.com/engine/installation/
    apt-get install -y apt-transport-https ca-certificates
    apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

    # Ubuntu 12.04
    echo "deb https://apt.dockerproject.org/repo ubuntu-precise main" > /etc/apt/sources.list.d/docker.list

    # Ubunto 14.04
    echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" > /etc/apt/sources.list.d/docker.list

    apt-get update

    apt-get install -y linux-image-extra-$(uname -r)

    apt-get install -y docker-engine

    # create postgres container
    docker run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo --name db-odoo postgres:9.5


Simplest way to create odoo container is as following:

    # run (create) container
    docker run \
    -p 8069:8069 \
    --name odoo \
    --link db-odoo:db
    -t itprojectsllc/install-odoo

For more specific installation check following links:

* [Docker for development](docs/dev.rst)
* [SaaS dockers](docs/saas.rst)
* [Odoo versions](docs/odoo-versions.rst)


Finish docker installation:

    # start
    docker start odoo

    # update source
    docker exec GIT_PULL=yes /bin/bash /install-odoo-saas.sh

    # restart
    docker restart odoo

    # prepare nginx (apache will be removed if installed)
    INIT_NGINX=yes \
    install-odoo-saas.sh

    # add start scripts
    INIT_START_SCRIPTS=docker-host \
    install-odoo-saas.sh

## SaaS Tools

To prepare [saas tools](https://github.com/it-projects-llc/odoo-saas-tools) specify params for ``saas.py`` script, e.g.:

    INIT_SAAS_TOOLS_VALUE="\
    --portal-create \
    --server-create \
    --plan-create \
    --odoo-script=/usr/local/src/odoo-source/openerp-server \
    --odoo-config=/etc/openerp-server.conf \
    --admin-password=${ODOO_MASTER_PASS} \
    --portal-db-name=${ODOO_DOMAIN} \
    --server-db-name=server-1.${ODOO_DOMAIN} \
    --plan-template-db-name=template-1.${ODOO_DOMAIN} \
    --plan-clients=demo-%i.${ODOO_DOMAIN} \
    "

Then run script.

    # for base installation
    INIT_SAAS_TOOLS=$INIT_SAAS_TOOLS_VALUE bash -x install-odoo-saas.sh

    # for docker installation:
    docker exec INIT_SAAS_TOOLS=$INIT_SAAS_TOOLS_VALUE /bin/bash /install-odoo-saas.sh
    

# Contributors

* [@yelizariev](https://github.com/yelizariev) - original semi-automated [script](https://gist.github.com/yelizariev/2abdd91d00dddc4e4fa4) for odoo installation
* [@bassn](https://github.com/bassn) - fully auto-automated [script](https://gist.github.com/bassn/996f8b168f0b1406dd54) for odoo installation

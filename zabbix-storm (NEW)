#!/bin/bash
#
# ZABBIX‑STORM – versão Oracle Linux (8/9)
# Instala: MySQL‑Community, Zabbix Server 7.2, Apache + PHP, Grafana
# Autor original: BUG IT – Adaptado por ChatGPT (2024‑05)

# ─── CORES ────────────────────────────────────────────────────────────────
RED='\033[1;31m'; GREEN='\033[1;32m'; YELLOW='\033[1;33m'; BLUE='\033[1;34m'
WHITE='\033[1;37m'; NC='\033[0m'

ok ()  { echo -e "${GREEN}✅ Concluído${NC}\n"; }
fail(){ echo -e "${RED}❌ Falhou${NC}\n";  }

check(){ "$@" &>/dev/null && ok || fail; }

[[ $EUID -ne 0 ]] && { echo -e "${RED}❌ Execute como root!${NC}"; exit 1; }

clear

# ASCII ART
echo -e "${RED}"
cat <<'EOF'
 ######     ##     #####    #####     ####    ##  ##             ####    ######    ####    #####    ##   ##
     ##    ####    ##  ##   ##  ##     ##     ##  ##            ##  ##     ##     ##  ##   ##  ##   ### ###
    ##    ##  ##   ##  ##   ##  ##     ##       ###             ##         ##     ##  ##   ##  ##   #######
   ##     ######   #####    #####      ##       ##               ####      ##     ##  ##   #####    ## # ##
  ##      ##  ##   ##  ##   ##  ##     ##      ####                 ##     ##     ##  ##   ####     ##   ##
 ##       ##  ##   ##  ##   ##  ##     ##     ##  ##            ##  ##     ##     ##  ##   ## ##    ##   ##
 ######   ##  ##   #####    #####     ####    ##  ##             ####      ##      ####    ##  ##   ##   ##
EOF
echo -e "${NC}\n:: Iniciando instalação do MySQL + Zabbix + Grafana... Aguarde...\n"

OL_VER=$(rpm -E %{ol_release})

##############################################################################
echo -e "${YELLOW}📥 Configurando repositórios do Zabbix (Oracle Linux $OL_VER)...${NC}"
dnf -y install https://repo.zabbix.com/zabbix/7.2/rhel/$OL_VER/x86_64/zabbix-release-7.2-1.el$OL_VER.noarch.rpm \
               epel-release &>/dev/null
dnf clean all &>/dev/null
check true   # só exibe ✅

##############################################################################
echo -e "${YELLOW}📦 Instalando pacotes Zabbix...${NC}"
dnf -y install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf \
               zabbix-sql-scripts zabbix-agent &>/dev/null
check true

##############################################################################
echo -e "${YELLOW}📦 Instalando MySQL Community Server...${NC}"
dnf -y install https://dev.mysql.com/get/mysql80-community-release-el$OL_VER-1.noarch.rpm &>/dev/null
dnf -y module disable mysql -y &>/dev/null
dnf -y install mysql-community-server &>/dev/null
systemctl enable --now mysqld &>/dev/null
check true

# pega senha temporária
TMPPASS=$(grep 'temporary password' /var/log/mysqld.log | awk '{print $NF}')

echo -e "${YELLOW}📦 Configurando banco de dados Zabbix...${NC}"
mysql -uroot -p"$TMPPASS" --connect-expired-password <<MYSQL  &>/dev/null
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root123!';
CREATE DATABASE IF NOT EXISTS zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER IF NOT EXISTS 'zabbix'@'localhost' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
MYSQL

zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | \
      mysql -uzabbix -p123456 zabbix  &>/dev/null
mysql -uroot -pRoot123! -e "SET GLOBAL log_bin_trust_function_creators = 0;" &>/dev/null
ok

##############################################################################
echo -e "${YELLOW}📦 Ajustando zabbix_server.conf...${NC}"
sed -i 's/^# DBPassword=.*/DBPassword=123456/' /etc/zabbix/zabbix_server.conf
check true

##############################################################################
echo -e "${YELLOW}📦 Configurando locale PT-BR...${NC}"
localectl set-locale LANG=pt_BR.utf8 &>/dev/null
check true

##############################################################################
echo -e "${YELLOW}📦 Instalando Grafana OSS...${NC}"
cat >/etc/yum.repos.d/grafana.repo <<'GRAFANA'
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/oss/rpm/gpg.key
GRAFANA
dnf -y install grafana &>/dev/null
check true

##############################################################################
echo -e "${YELLOW}🔁 Habilitando e iniciando serviços...${NC}"
systemctl enable --now zabbix-server zabbix-agent httpd grafana-server &>/dev/null
check true

##############################################################################
echo -e "${GREEN}🎉 Instalação Finalizada com Sucesso!${NC}\n"

IP=$(hostname -I | awk '{print $1}')
echo -e "${WHITE}🔗 Zabbix:  ${YELLOW}http://${BLUE}${IP}${YELLOW}/zabbix${NC}  (Admin / zabbix)"
echo -e "${WHITE}🔗 Grafana: ${YELLOW}http://${BLUE}${IP}${YELLOW}:3000${NC}      (admin / admin)"
echo -e "\n${WHITE}Script adaptado para Oracle Linux por ChatGPT${NC}"

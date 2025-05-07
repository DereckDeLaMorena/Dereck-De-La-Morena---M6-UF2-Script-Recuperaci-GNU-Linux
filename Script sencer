#!/bin/bash

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # Sense Color

# Comprovar si l'script s'executa com a root
if [ "$EUID" -ne 0 ]; then
  echo -e "${RED}ERROR: Aquest script s'ha d'executar amb permisos de root (sudo).${NC}"
  exit 1
fi

# Funció per instal·lar Docker
install_docker() {
  if ! command -v docker &> /dev/null; then
    echo -e "${YELLOW}Instal·lant Docker...${NC}"
    apt-get update
    apt-get install -y apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    apt-get update
    apt-get install -y docker-ce docker-ce-cli containerd.io
    systemctl enable docker
    echo -e "${GREEN}Docker instal·lat correctament.${NC}"
  else
    echo -e "${GREEN}Docker ja està instal·lat.${NC}"
  fi
}

# Funció per instal·lar Docker Compose
install_docker_compose() {
  if ! command -v docker-compose &> /dev/null; then
    echo -e "${YELLOW}Instal·lant Docker Compose...${NC}"
    curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    echo -e "${GREEN}Docker Compose instal·lat correctament.${NC}"
  else
    echo -e "${GREEN}Docker Compose ja està instal·lat.${NC}"
  fi
}

# Funció per configurar WordPress amb Docker
setup_wordpress() {
  echo -e "${YELLOW}Configurant WordPress amb Docker...${NC}"
  mkdir -p /opt/wordpress && cd /opt/wordpress

  # Demanar configuració a l'usuari
  read -p "Introdueix el port per a WordPress (per defecte: 8080): " WP_PORT
  WP_PORT=${WP_PORT:-8080}

  read -p "Introdueix la contrasenya root de MySQL (per defecte: rootpass): " DB_ROOT_PASSWORD
  DB_ROOT_PASSWORD=${DB_ROOT_PASSWORD:-rootpass}

  read -p "Introdueix el nom de la base de dades (per defecte: wordpress): " DB_NAME
  DB_NAME=${DB_NAME:-wordpress}

  read -p "Introdueix l'usuari de la base de dades (per defecte: wpuser): " DB_USER
  DB_USER=${DB_USER:-wpuser}

  read -p "Introdueix la contrasenya de l'usuari de la base de dades (per defecte: wppass): " DB_PASSWORD
  DB_PASSWORD=${DB_PASSWORD:-wppass}

  # Crear docker-compose.yml
  cat > docker-compose.yml <<EOF
version: '3'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - "${WP_PORT}:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${DB_USER}
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
      WORDPRESS_DB_NAME: ${DB_NAME}

volumes:
  db_data:
  wp_data:
EOF

  # Iniciar els contenidors
  docker-compose up -d

  echo -e "${GREEN}WordPress s'ha instal·lat correctament.${NC}"
  echo -e "Accedeix a: ${YELLOW}http://$(hostname -I | awk '{print $1}'):${WP_PORT}${NC}"
}

# Menú principal
echo -e "${GREEN}\n==== Script d'Automatització per a WordPress amb Docker ====${NC}"
install_docker
install_docker_compose
setup_wordpress

echo -e "${YELLOW}\nNota: Per gestionar els contenidors, utilitza les comandes:${NC}"
echo -e "  - Aturar: ${YELLOW}cd /opt/wordpress && docker-compose down${NC}"
echo -e "  - Reiniciar: ${YELLOW}cd /opt/wordpress && docker-compose restart${NC}"

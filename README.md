# vaultwarden_renewal.sh

🛠️ Skripta: vaultwarden_renewal.sh
```bash
#!/bin/bash
set -e

# Konfiguracija
COMPOSE_DIR="/home/rade/docker/vaultwarden"
BACKUP_BASE="/home/rade/docker_backups"
DATE_TAG=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_DIR="$BACKUP_BASE/vaultwarden_backup_$DATE_TAG"
LOG_FILE="/home/rade/vaultwarden_renewal.log"

echo "===== $(date) =====" >> $LOG_FILE

# 1. Backup citavog compose_dir-a
echo "[INFO] Pravimo backup citavog compose_dir u $BACKUP_DIR" | tee -a $LOG_FILE
mkdir -p "$BACKUP_DIR"

# Kopiranje citavog compose_dir
cp -r "$COMPOSE_DIR" "$BACKUP_DIR/" 2>> $LOG_FILE || echo "[WARN] Backup direktorijuma nije uspeo" >> $LOG_FILE

# 2. Idemo u docker-compose direktorijum
cd "$COMPOSE_DIR"

# 3. Stopiranje i uklanjanje kontejnera
echo "[INFO] Stopiranje i uklanjanje kontejnera..." | tee -a $LOG_FILE
docker compose down >> $LOG_FILE 2>&1

# 4. Povuci nove slike (Treba proveriti da li je neophodno u ovom koraku da se stavi docker-compose pull)
echo "[INFO] Povlačenje novih slika..." | tee -a $LOG_FILE
docker compose pull >> $LOG_FILE 2>&1

# 5. Pokreni kontejnere
echo "[INFO] Pokretanje kontejnera..." | tee -a $LOG_FILE
docker compose up -d >> $LOG_FILE 2>&1

# 6. Restartuj nginx (ako postoji nginx servis)
echo "[INFO] Restart nginx servisa" | tee -a $LOG_FILE
docker compose restart nginx >> $LOG_FILE 2>&1 || echo "[WARN] nginx servis nije pronađen u docker-compose" >> $LOG_FILE

echo "[SUCCESS] Vaultwarden obnova završena." | tee -a $LOG_FILE
```

⏰ Cronjob primer (pokretanje na primer svakog 1. u mesecu u 3:00)

0 3 1 * * /bin/bash /home/rade/scripts/vaultwarden_renewal.sh

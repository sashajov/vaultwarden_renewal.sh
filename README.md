# vaultwarden_renewal.sh

🛠️ Skripta: vaultwarden_renewal.sh
# Pocetak skripte
bash
Copy
Edit
#!/bin/bash
set -e

# Konfiguracija
COMPOSE_DIR="/home/rade/docker/vaultwarden"
BACKUP_BASE="/home/rade/docker_backups"
DATE_TAG=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_DIR="$BACKUP_BASE/vaultwarden_backup_$DATE_TAG"
LOG_FILE="/home/rade/vaultwarden_renewal.log"

echo "===== $(date) =====" >> $LOG_FILE

# 1. Backup vault fajlova
echo "[INFO] Pravimo backup Vaultwarden fajlova u $BACKUP_DIR" | tee -a $LOG_FILE
mkdir -p "$BACKUP_DIR"

# Pretpostavljamo da su podaci u folderu 'vw-data' (prilagodi ako je drugačije)
cp -r "$COMPOSE_DIR/vw-data" "$BACKUP_DIR/" 2>> $LOG_FILE || echo "[WARN] vw-data direktorijum ne postoji" >> $LOG_FILE

# 2. Idemo u docker-compose direktorijum
cd "$COMPOSE_DIR"

# 3. Stopiranje i uklanjanje kontejnera
echo "[INFO] Stopiranje i uklanjanje kontejnera..." | tee -a $LOG_FILE
docker compose down >> $LOG_FILE 2>&1

# 4. Očisti zaostale kontejnere (opciono)
docker container prune -f >> $LOG_FILE 2>&1

# 5. Povuci nove slike
echo "[INFO] Povlačenje novih slika..." | tee -a $LOG_FILE
docker compose pull >> $LOG_FILE 2>&1

# 6. Pokreni kontejnere
echo "[INFO] Pokretanje kontejnera..." | tee -a $LOG_FILE
docker compose up -d >> $LOG_FILE 2>&1

# 7. Restartuj nginx (ako postoji nginx servis)
echo "[INFO] Restart nginx servisa (ako postoji)..." | tee -a $LOG_FILE
docker compose restart nginx >> $LOG_FILE 2>&1 || echo "[WARN] nginx servis nije pronađen u docker-compose" >> $LOG_FILE

echo "[SUCCESS] Vaultwarden obnova završena." | tee -a $LOG_FILE
# Kraj skripte

⏰ Cronjob primer (pokretanje svakog 1. u mesecu u 3:00)
Otvorite cron editor:

bash
Copy
Edit
crontab -e
Dodaj liniju:

bash
Copy
Edit
0 3 1 * * /bin/bash /home/rade/scripts/vaultwarden_renewal.sh
Naravno, prilagodi putanju ako čuvaš skriptu na nekom drugom mestu.

🛡️ Za dodatnu sigurnost
Možemo proširiti skriptu da napravi i snapshot Docker volume-a ako koristiš volume umesto lokalnih fajlova. Hoćeš da proverimo koji storage backend koristi tvoj vaultwarden (volume vs bind mount)?

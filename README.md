import time
import requests
import random

# ==========================
# Configuration Ubidots
# ==========================
TOKEN = "BBUS-vFMys8yflUN2qQUBLphNia3F5gmB89"
DEVICE_LABEL = "Raspberry"

VARIABLE_LABEL_1 = "temperature"
VARIABLE_LABEL_2 = "humidite"
VARIABLE_LABEL_3 = "position"

# ==========================
# Variables globales (mémoire)
# ==========================
last_temp = None
last_hum = None

# Seuil de changement minimum (optionnel)
SEUIL_TEMP = 2
SEUIL_HUM = 2

# ==========================
# Création du payload
# ==========================
def build_payload(var1, var2, var3):
    value_1 = random.randint(30, 81)   # Température
    value_2 = random.randint(50, 71)   # Humidité

    lat = 34.757358
    lng = 10.772115

    payload = {
        var1: value_1,
        var2: value_2,
        var3: {
            "value": 34.10,
            "context": {"lat": lat, "lng": lng}
        }
    }

    return payload, value_1, value_2

# ==========================
# Envoi des données
# ==========================
def post_request(payload):
    url = "http://industrial.api.ubidots.com/api/v1.6/devices/{}".format(DEVICE_LABEL)
    headers = {
        "X-Auth-Token": TOKEN,
        "Content-Type": "application/json"
    }

    status = 400
    attempts = 0

    while status >= 400 and attempts < 5:
        try:
            req = requests.post(url=url, headers=headers, json=payload)
            status = req.status_code
            attempts += 1
            time.sleep(1)
        except Exception as e:
            print("[ERROR] Exception:", e)
            attempts += 1
            time.sleep(1)

    if status >= 400:
        print("[ERROR] Failed after 5 attempts")
        return False

    print("[INFO] Data sent successfully")
    return True

# ==========================
# Logique principale
# ==========================
def main():
    global last_temp, last_hum

    payload, temp, hum = build_payload(
        VARIABLE_LABEL_1,
        VARIABLE_LABEL_2,
        VARIABLE_LABEL_3
    )

    print(f"[INFO] New values → Temp: {temp}, Hum: {hum}")

    # Première exécution → envoyer directement
    if last_temp is None or last_hum is None:
        print("[INFO] First send")
        if post_request(payload):
            last_temp = temp
            last_hum = hum
        return

    # Vérifier changement avec seuil
    temp_changed = abs(temp - last_temp) >= SEUIL_TEMP
    hum_changed = abs(hum - last_hum) >= SEUIL_HUM

    if temp_changed or hum_changed:
        print("[INFO] Significant change detected → Sending data...")
        if post_request(payload):
            last_temp = temp
            last_hum = hum
    else:
        print("[INFO] No significant change → Not sending")

    print("[INFO] Finished\n")

# ==========================
# Boucle infinie
# ==========================
if __name__ == "__main__":
    while True:
        main()
        time.sleep(10)  # toutes les 10 secondes

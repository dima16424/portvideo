import os
import requests
import time

# === Настройки ===
BOT_TOKEN = "7062038688:AAHMVNwtLwxdybRw1mogJaVOKKR8KWHjMJ0"
CHAT_ID = "6227457637"
FOLDER_PATH = "/sdcard/DCIM/Camera"  # Путь к папке с медиафайлами
DELAY_BETWEEN_FILES = 1  # Задержка между отправкой файлов в секундах
MAX_RETRIES = 3  # Максимальное количество попыток отправки при ошибках

def send_files():
    if not os.path.exists(FOLDER_PATH):
        print(f"⚠️ Ошибка: Папка {FOLDER_PATH} не существует!")
        return

  count = 0
    for filename in os.listdir(FOLDER_PATH):
        file_path = os.path.join(FOLDER_PATH, filename)

  # Определяем тип файла
  if filename.lower().endswith((".jpg", ".jpeg", ".png")):
            url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendPhoto"
            file_key = "photo"
        elif filename.lower().endswith((".mp4", ".mov")):
            url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendVideo"
            file_key = "video"
        else:
            continue

  # Пытаемся отправить файл с возможностью повтора
  for attempt in range(MAX_RETRIES):
            try:
                with open(file_path, "rb") as f:
                    response = requests.post(
                        url,
                        files={file_key: f},
                        data={"chat_id": CHAT_ID}
                    )

   if response.status_code == 200:
                    print(f"✅ Успешно отправлено: {filename}")
                    count += 1
                    break
                elif response.status_code == 429:
                    wait_time = 5
                    print(f"⏳ Слишком много запросов. Ждём {wait_time} секунд... (Попытка {attempt + 1}/{MAX_RETRIES})")
                    time.sleep(wait_time)
                else:
                    print(f"⚠️ Ошибка отправки {filename}: {response.status_code} — {response.text}")
                    if attempt == MAX_RETRIES - 1:
                        print(f"❌ Не удалось отправить {filename} после {MAX_RETRIES} попыток")
                    time.sleep(DELAY_BETWEEN_FILES)
                    
   except requests.exceptions.RequestException as e:
                print(f"⚠️ Ошибка сети при отправке {filename}: {str(e)}")
                if attempt == MAX_RETRIES - 1:
                    print(f"❌ Не удалось отправить {filename} из-за проблем с сетью")
                time.sleep(DELAY_BETWEEN_FILES)
            except Exception as e:
                print(f"⚠️ Неожиданная ошибка при обработке {filename}: {str(e)}")
                break

  time.sleep(DELAY_BETWEEN_FILES)

  print(f"\n📊 Итог: успешно отправлено {count} файлов")

# === Запуск ===
if __name__ == "__main__":
    print("=== Запуск отправки файлов ===")
    send_files()

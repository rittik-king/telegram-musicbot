# telegram-musicbot
import os import logging from uuid import uuid4 from telegram import Update from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes from yt_dlp import YoutubeDL

Ensure downloads folder exists

DOWNLOAD_DIR = 'downloads' os.makedirs(DOWNLOAD_DIR, exist_ok=True)

Configure logging

logging.basicConfig(level=logging.INFO) logger = logging.getLogger(name)

BOT_TOKEN = '7046442086:AAG3Db4xqES08PX3hvPRQtDl7cJ44Fh-UBY'  # Replace with your bot token

--- Helpers ---

def download_audio(query: str) -> str: output_path = os.path.join(DOWNLOAD_DIR, f"{uuid4()}.mp3") ydl_opts = { 'format': 'bestaudio/best', 'noplaylist': True, 'quiet': True, 'outtmpl': output_path, 'postprocessors': [{ 'key': 'FFmpegExtractAudio', 'preferredcodec': 'mp3', 'preferredquality': '192' }] } with YoutubeDL(ydl_opts) as ydl: logger.info(f"Downloading: {query}") ydl.download([f"ytsearch:{query}"]) return output_path

async def cleanup(file_path): try: os.remove(file_path) logger.info(f"Removed: {file_path}") except Exception as e: logger.error(f"Cleanup failed: {e}")

--- Handlers ---

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE): await update.message.reply_text("\U0001F3B5 Welcome to Song Player Bot! Use /play <song name> to get music.")

async def play(update: Update, context: ContextTypes.DEFAULT_TYPE): if not context.args: await update.message.reply_text("Please provide a song name. Example: /play Shape of You") return

query = ' '.join(context.args)
msg = await update.message.reply_text(f"\U0001F50A Searching for '{query}'...")

try:
    file_path = download_audio(query)
    await update.message.reply_audio(audio=open(file_path, 'rb'), title=query)
    await msg.edit_text(f"✅ Sent: {query}")
except Exception as e:
    logger.error(f"Error: {e}")
    await msg.edit_text(f"❌ Failed to get song: {e}")
finally:
    await cleanup(file_path)

--- Main ---

if name == 'main': app = ApplicationBuilder().token(BOT_TOKEN).build()

app.add_handler(CommandHandler('start', start))
app.add_handler(CommandHandler('play', play))

print("Bot is running...")
app.run_polling()


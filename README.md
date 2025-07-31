# BOT.py

import logging
import wikipedia
import requests
from telegram import Update
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    MessageHandler,
    ContextTypes,
    filters,
)

# Enable logging
logging.basicConfig(
    format="%(asctime)s - %(levelname)s - %(message)s", level=logging.INFO
)
logger = logging.getLogger(__name__)

# /start command handler
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text(
        "👋 Hello! Send me the name of a famous person or topic, and I’ll get a short summary and image from Wikipedia!"
    )

# Wikipedia fetch handler
async def fetch_wikipedia_details(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    query = update.message.text
    wikipedia.set_lang("en")

    try:
        # Get summary
        summary = wikipedia.summary(query, sentences=3, auto_suggest=True, redirect=True)
        page = wikipedia.page(query, auto_suggest=True)
        image_url = ""

        # Find a suitable image (jpg or png)
        for img in page.images:
            if img.lower().endswith((".jpg", ".jpeg", ".png")):
                image_url = img
                break

        # Send summary
        await update.message.reply_text(f"📚 *{page.title}*\n\n{summary}", parse_mode="Markdown")

        # Send image if found
        if image_url:
            response = requests.get(image_url)
            if response.status_code == 200:
                await update.message.reply_photo(photo=response.content)
            else:
                await update.message.reply_text("⚠️ Couldn't load the image.")
        else:
            await update.message.reply_text("ℹ️ No suitable image found on the page.")

    except wikipedia.DisambiguationError as e:
        options = "\n".join(e.options[:5])
        await update.message.reply_text(f"🔍 Your query is ambiguous. Did you mean:\n{options}")
    except wikipedia.PageError:
        await update.message.reply_text("❌ I couldn't find a page for your query.")
    except Exception as e:
        logger.error(f"Error: {e}")
        await update.message.reply_text("⚠️ Something went wrong.")

# Main bot startup
def main():
    BOT_TOKEN = "8212392425:AAHRAQAXjez5Xz0-NJjwSYSiagn_OypqgUw"  # Replace with your actual bot token

    app = ApplicationBuilder().token(BOT_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, fetch_wikipedia_details))

    print("✅ Bot is running...")
    app.run_polling()

if __name__ == "__main__":
    main()

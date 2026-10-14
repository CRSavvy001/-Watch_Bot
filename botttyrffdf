import logging
import os
import re

from telegram import Update
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    ContextTypes,
    MessageHandler,
    filters,
)

logging.basicConfig(
    format="%(asctime)s | %(levelname)s | %(message)s", level=logging.INFO
)
log = logging.getLogger("ca-forwarder")

# ---------- Config (set these as Railway Variables) ----------
BOT_TOKEN = os.environ["BOT_TOKEN"]
TARGET_CHAT_ID = int(os.environ["TARGET_CHAT_ID"])

# Optional: only watch these source chats (comma-separated IDs).
# Leave empty to watch every chat the bot is in.
SOURCE_CHAT_IDS = {
    int(x)
    for x in os.getenv("SOURCE_CHAT_IDS", "").replace(" ", "").split(",")
    if x
}

# 0x + 40 hex chars, not part of a longer hex string (e.g. 64-char tx hashes)
ADDRESS_RE = re.compile(r"(?<![A-Za-z0-9])0x[a-fA-F0-9]{40}(?![A-Za-z0-9])")


async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    msg = update.effective_message
    if msg is None:
        return

    chat_id = msg.chat_id
    if chat_id == TARGET_CHAT_ID:
        return  # never loop back
    if SOURCE_CHAT_IDS and chat_id not in SOURCE_CHAT_IDS:
        return

    text = msg.text or msg.caption or ""
    # unique addresses, keep original order
    addresses = list(dict.fromkeys(ADDRESS_RE.findall(text)))

    for addr in addresses:
        try:
            await context.bot.send_message(
                chat_id=TARGET_CHAT_ID, text=f"/watch {addr}"
            )
            log.info("Forwarded %s from chat %s", addr, chat_id)
        except Exception as e:
            log.error("Failed to forward %s: %s", addr, e)


async def get_id(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Send /id in any chat to see its chat ID."""
    await update.effective_message.reply_text(f"Chat ID: {update.effective_chat.id}")


def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("id", get_id))
    app.add_handler(
        MessageHandler(
            (filters.TEXT | filters.CAPTION) & ~filters.COMMAND, handle_message
        )
    )
    log.info("Bot started. Target chat: %s", TARGET_CHAT_ID)
    app.run_polling(allowed_updates=Update.ALL_TYPES)


if __name__ == "__main__":
    main()

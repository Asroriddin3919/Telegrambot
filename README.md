from telegram import Update, ReplyKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, CallbackContext

# Bot tokeni va admin ID
BOT_TOKEN = "7580852244:AAFv1W2tNGnzyGcHHAakjAznAWH9q6CAO00"
ADMIN_ID = 1026816624  

# Guruh va foydalanuvchilar ro‘yxati
group_ids = set()
user_ids = set()

# Admin uchun tugmalar
admin_keyboard = ReplyKeyboardMarkup(
    [["📊 Statistika", "📢 Reklama"]],
    resize_keyboard=True
)

# /start buyrug‘i
async def start(update: Update, context: CallbackContext) -> None:
    user_id = update.effective_user.id
    chat_id = update.effective_chat.id

    if update.effective_chat.type in ["group", "supergroup"]:
        group_ids.add(chat_id)
    else:
        user_ids.add(user_id)

    if user_id == ADMIN_ID:
        await update.message.reply_text(
            "👋 Assalomu alaykum, Admin!\nGuruhni qoriqlashga tayyorman.",
            reply_markup=admin_keyboard
        )
    else:
        await update.message.reply_text(
            "👋 Assalomu alaykum!\nMen guruhlarni qoriqlash uchun yaratilgan botman.\nMeni admin qiling, shunda ishlashni boshlayman."
        )

# Guruhda reklama yoki @ ni to‘sish (faqat oddiy foydalanuvchilar uchun)
async def monitor_group(update: Update, context: CallbackContext) -> None:
    chat = update.effective_chat
    message = update.message
    user = update.effective_user

    # Anonim xabarlarni tekshirmaslik
    if user is None or message.forward_origin:  # Kanaldan yuborilgan xabarlarni o'tkazib yuborish
        return

    # Foydalanuvchi admin emasligini tekshirish
    admins = [admin.user.id for admin in await chat.get_administrators()]
    if user.id not in admins:
        if "http" in message.text or "www" in message.text or ".com" in message.text or "@" in message.text:
            try:
                await message.delete()
                await context.bot.send_message(
                    chat_id=chat.id,
                    text=f"⚠️ @{user.username}, reklama va @ ishlatish taqiqlangan!\n"
                         f"📌 Guruh qoidalari:\n"
                         f"🚫 Reklama taqiqlangan.\n"
                         f"🚫 Link va spam yuborish mumkin emas.\n"
                         f"⚠️ Takrorlansa, guruhdan chiqarilasiz!"
                )
            except:
                pass

# Statistika funksiyasi
async def show_statistics(update: Update, context: CallbackContext) -> None:
    if update.effective_user.id == ADMIN_ID:
        group_count = len(group_ids)
        user_count = len(user_ids)
        await update.message.reply_text(f"📊 **Statistika:**\n\n🛡 Guruhlar soni: {group_count}\n👤 Foydalanuvchilar soni: {user_count}")

# Reklama funksiyasi
async def send_ad_request(update: Update, context: CallbackContext) -> None:
    if update.effective_user.id == ADMIN_ID:
        await update.message.reply_text("📢 Reklama matnini yuboring, men uni barcha foydalanuvchilarga va guruhlarga yuboraman.")

# Tugmalar bosilganda ishlaydigan funksiya
async def button_handler(update: Update, context: CallbackContext) -> None:
    text = update.message.text

    if text == "📊 Statistika":
        await show_statistics(update, context)
    elif text == "📢 Reklama":
        await send_ad_request(update, context)

# Asosiy funksiyani ishga tushirish
def main() -> None:
    application = ApplicationBuilder().token(BOT_TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT & filters.ChatType.GROUPS, monitor_group))
    application.add_handler(MessageHandler(filters.TEXT & filters.ChatType.PRIVATE, button_handler))

    application.run_polling()

# Botni ishga tushirish
if __name__ == "__main__":
    main()

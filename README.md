import asyncio
from telegram import Update, InlineKeyboardMarkup, InlineKeyboardButton
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes
from telegram.helpers import escape_markdown  # Para evitar problemas con Markdown

# Token del bot
BOT_TOKEN = '8192753383:AAExTTthTzcLypf9p62FlwVeBAQDugr7sTk'
CHANNEL_ID = '-1002094815052'  # ID del canal de referencias

# ID del usuario autorizado a aprobar grupos
OWNER_ID = 7353070970  # Tu ID de usuario
APPROVED_GROUPS = {-1002322559468}  # Lista de grupos aprobados

# Función para aprobar grupos dinámicamente
async def approve_group(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != OWNER_ID:
        await update.message.reply_text("❌ No tienes permiso para aprobar grupos.")
        return
    group_id = update.effective_chat.id
    APPROVED_GROUPS.add(group_id)
    await update.message.reply_text(f"✅ Grupo aprobado. ID: `{escape_markdown(str(group_id), version=2)}`", parse_mode="MarkdownV2")

# Función para manejar el comando /refe
async def refe_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_chat.id not in APPROVED_GROUPS:
        await update.message.reply_text("❌ Este grupo no está aprobado para usar este comando.")
        return

    user_message = update.message.reply_to_message
    if not user_message or not (user_message.photo or user_message.video or user_message.animation):
        await update.message.reply_text("❌ Responde a un mensaje con una imagen, video o GIF válido.")
        return

    # Extraer información del mensaje
    comment = escape_markdown(user_message.caption if user_message.caption else "Sin comentario", version=2)
    user_reference = escape_markdown(f"@{user_message.from_user.username}" if user_message.from_user.username else "Usuario_desconocido", version=2)
    formatted_time = escape_markdown(update.message.date.strftime('%Y-%m-%d %H\\:%M\\:%S'), version=2)
    user_id = escape_markdown(str(user_message.from_user.id), version=2)

    # Links correctos en las estrellas
    star_link = "https://t.me/gojoproyect/1680"
    
    # Mensaje con enlaces en las estrellas y referencias correctamente asignadas
    message_format = (
        f"[★ᵎᵎ⭒⋮ 𝐌𝐎𝐁 𝐕𝐈𝐏 ⋮⭒ᵎᵎ★]({star_link})\n"
        "▬▬▬▬▬▬▬✦▬▬▬▬▬▬▬▬\n"
        f"[⌞✶⌝]({star_link}) ⋮ 𝖱𝖾𝖿𝖾𝗋𝖾𝗇𝖼𝗂𝖺 𝖽𝖾: `{user_reference}`\n"
        f"[⌞✶⌝]({star_link}) ⋮ 𝖢𝗈𝗆𝖾𝗇𝗍𝖺𝗋𝗂𝗈: `{comment}`\n"
        f"[⌞✶⌝]({star_link}) ⋮ 𝖳𝗂𝗆𝖾 𝗌𝖾𝗇𝖽: `{formatted_time}`\n"
        f"[⌞✶⌝]({star_link}) ⋮ 𝖨𝖣: `{user_id}`\n"
        "▬▬▬▬▬▬▬✦▬▬▬▬▬▬▬▬"
    )

    # Botones interactivos corregidos
    keyboard = [
        [InlineKeyboardButton("⌗ 𝗠𝗢𝗕 𝗩𝗜𝗣", url="https://t.me/gojoproyect/1680")],
        [InlineKeyboardButton("⌗ 𝗢𝗪𝗡𝗘𝗥", url="https://t.me/Marxk09"),
         InlineKeyboardButton("⌗ 𝗢𝗪𝗡𝗘𝗥", url="https://t.me/Erickfanzz")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)

    # Enviar imagen, video o GIF con texto y botones
    if user_message.photo:
        file_id = user_message.photo[-1].file_id
        await context.bot.send_photo(
            chat_id=CHANNEL_ID,
            photo=file_id,
            caption=message_format,
            parse_mode="MarkdownV2",
            reply_markup=reply_markup
        )
    elif user_message.video:
        file_id = user_message.video.file_id
        await context.bot.send_video(
            chat_id=CHANNEL_ID,
            video=file_id,
            caption=message_format,
            parse_mode="MarkdownV2",
            reply_markup=reply_markup
        )
    elif user_message.animation:
        file_id = user_message.animation.file_id
        await context.bot.send_animation(
            chat_id=CHANNEL_ID,
            animation=file_id,
            caption=message_format,
            parse_mode="MarkdownV2",
            reply_markup=reply_markup
        )

    await update.message.reply_text("✅ ¡Referencia enviada correctamente!")

# Función principal
def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("approvegroup", approve_group))
    app.add_handler(CommandHandler("refe", refe_command))
    print("El bot está activo. Presiona Ctrl+C para detenerlo.")
    app.run_polling()

# Ejecuta el bot
if __name__ == "__main__":
    main()

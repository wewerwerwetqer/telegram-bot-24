import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, ContextTypes, MessageHandler, filters

# Включим логирование
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# Токен бота (получите у @BotFather)
BOT_TOKEN = "7921197293:AAG8RUCMOtKVBLVvLfrlDA1tsIc5I8SJ96c"

# Список администраторов
ADMIN_CHAT_IDS = [6258590428]

# База данных друзей (редактируйте этот список в коде)
friends_list = [
    {"username": "ivanov", "nickname": "Иван Петров"},
    {"username": "petrov", "nickname": "Лучший друг"},
    {"username": "sidorov", "nickname": "Коллега по работе"},
    {"username": "smirnov", "nickname": "Помощник"},
    {"username": "kuznetsov", "nickname": "Наставник"}
]

# Команда для установки администратора
async def setup_admin(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    chat_id = update.effective_chat.id
    
    if not ADMIN_CHAT_IDS:
        ADMIN_CHAT_IDS.append(chat_id)
        await update.message.reply_text(
            f"✅ Вы теперь администратор!\n"
            f"🆔 Ваш Chat ID: {chat_id}\n\n"
            f"Теперь вы будете получать все анонимные сообщения от пользователей."
        )
    else:
        await update.message.reply_text(
            f"❌ Администратор уже установлен."
        )

# Команда /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    
    welcome_text = f"""
👋 Привет, {user.first_name}!

Я информационный бот с функцией анонимных сообщений.

Основные команды:
📊 /info - Основная информация
📞 /contacts - Контактные данные
👥 /friends - Список друзей
🆘 /help - Помощь по командам

Анонимные сообщения:
💬 /send_anonymous - Отправить анонимное сообщение
"""
    
    keyboard = [
        [InlineKeyboardButton("📊 Информация", callback_data="info")],
        [InlineKeyboardButton("📞 Контакты", callback_data="contacts")],
        [InlineKeyboardButton("👥 Друзья", callback_data="friends")],
        [InlineKeyboardButton("💬 Анонимное сообщение", callback_data="send_anon")],
        [InlineKeyboardButton("🆘 Помощь", callback_data="help")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    
    await update.message.reply_text(welcome_text, reply_markup=reply_markup)

# Команда для отправки анонимного сообщения
async def send_anonymous(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not ADMIN_CHAT_IDS:
        await update.message.reply_text(
            "❌ Администратор еще не настроил бот.\n\n"
            "Пожалуйста, попробуйте позже."
        )
        return
    
    instruction_text = """
💬 Отправка анонимного сообщения

Напишите ваше сообщение, и я перешлю его администратору полностью анонимно.

Ваше имя, username и любая другая информация не будет раскрыта.

✍️ Пожалуйста, напишите ваше сообщение:
"""
    
    context.user_data['waiting_for_anonymous_msg'] = True
    await update.message.reply_text(instruction_text)

# Обработчик текстовых сообщений для анонимных сообщений
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if context.user_data.get('waiting_for_anonymous_msg'):
        user_message = update.message.text
        
        # Отправляем сообщение всем администраторам
        for admin_id in ADMIN_CHAT_IDS:
            try:
                admin_message = f"""
🔔 Новое анонимное сообщение

📝 Сообщение:
"{user_message}"

⏰ Время: {update.message.date.strftime('%Y-%m-%d %H:%M:%S')}
"""
                
                await context.bot.send_message(
                    chat_id=admin_id,
                    text=admin_message
                )
                
            except Exception as e:
                logging.error(f"Ошибка отправки сообщения администратору {admin_id}: {e}")
        
        # Отправляем подтверждение пользователю
        success_text = """
✅ Сообщение успешно отправлено!

Ваше сообщение было доставлено администратору полностью анонимно.

💬 Вы можете отправить еще одно сообщение командой /send_anonymous
"""
        
        await update.message.reply_text(success_text)
        context.user_data['waiting_for_anonymous_msg'] = False
    
    else:
        await update.message.reply_text(
            "Используйте команды из меню или /help для получения списка команд."
        )

# Команда /contacts
async def contacts(update: Update, context: ContextTypes.DEFAULT_TYPE):
    contacts_text = """
📞 Контактная информация

Основные контакты:
• Discord: wenr1ck
• Telegram: [ваш username]
• Email: [ваш email]

Режим связи:
Пн-Пт: 9:00 - 18:00
Сб-Вс: По договоренности

💬 Предпочитаете анонимность? Используйте /send_anonymous
"""
    await update.message.reply_text(contacts_text)

# Команда /friends
async def friends(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not friends_list:
        friends_text = "👥 Список друзей пуст."
    else:
        friends_text = "👥 Список друзей:\n\n"
        for i, friend in enumerate(friends_list, 1):
            friends_text += f"{i}. @{friend['username']} - {friend['nickname']}\n"
    
    await update.message.reply_text(friends_text)

# Команда /info
async def info(update: Update, context: ContextTypes.DEFAULT_TYPE):
    info_text = """
📊 Основная информация

• Название: Пример бота
• Создатель: [Ваше Имя]
• Назначение: Информационный бот с анонимными сообщениями
• Статус: Активно развивается

Возможности бота:
- Предоставление информации
- Анонимные сообщения
- Список друзей
- Обратная связь

💬 Есть вопрос? Отправьте анонимное сообщение!
"""
    await update.message.reply_text(info_text)

# Команда /help
async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    help_text = """
🆘 Помощь по командам

Основные команды:
/start - Запустить бота
/info - Основная информация
/contacts - Контактные данные
/friends - Список друзей

Анонимные сообщения:
/send_anonymous - Отправить анонимное сообщение

💡 Также вы можете использовать кнопки меню для навигации.
"""
    await update.message.reply_text(help_text)

# Обработчик нажатий на кнопок
async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    
    if query.data == "info":
        text = """
📊 Основная информация

• Название: Пример бота
• Создатель: [Ваше Имя]
• Назначение: Информационный бот
"""
    elif query.data == "contacts":
        text = """
📞 Контактная информация

• Discord: wenr1ck
• Telegram: [ваш username]
• Email: [ваш email]
"""
    elif query.data == "friends":
        if not friends_list:
            text = "👥 Список друзей пуст."
        else:
            text = "👥 Список друзей:\n\n"
            for i, friend in enumerate(friends_list, 1):
                text += f"{i}. @{friend['username']} - {friend['nickname']}\n"
    elif query.data == "send_anon":
        text = """
💬 Анонимное сообщение

Используйте команду /send_anonymous для отправки анонимного сообщения.

Сообщение будет доставлено полностью анонимно!
"""
    elif query.data == "help":
        text = """
🆘 Помощь

Используйте команды меню или кнопки для навигации.
Для анонимных сообщений: /send_anonymous
Для просмотра друзей: /friends
"""
    else:
        text = "Раздел не найден"
    
    await query.edit_message_text(text=text)

# Основная функция
def main():
    application = Application.builder().token(BOT_TOKEN).build()
    
    # Добавляем обработчики команд
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("setup_admin", setup_admin))
    application.add_handler(CommandHandler("info", info))
    application.add_handler(CommandHandler("contacts", contacts))
    application.add_handler(CommandHandler("friends", friends))
    application.add_handler(CommandHandler("help", help_command))
    application.add_handler(CommandHandler("send_anonymous", send_anonymous))
    
    # Добавляем обработчик текстовых сообщений
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    
    # Добавляем обработчик кнопок
    application.add_handler(CallbackQueryHandler(button_handler))
    
    # Запускаем бота
    print("Бот запущен...")
    print("Для настройки администратора используйте команду /setup_admin в чате с ботом")
    application.run_polling()

if __name__ == '__main__':
    main()

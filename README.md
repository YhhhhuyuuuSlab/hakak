import telebot
from telebot import types

API_TOKEN = '7267339380:AAEsZiY5_K0FyPFGzxdanw-9_xjgf7X_KxA'
CHANNEL_USERNAME = '@texriz_tg'  # Канал, на который нужно подписаться

bot = telebot.TeleBot(API_TOKEN)

# Структура для хранения данных о пользователях
users_data = {}

# Главное меню
def main_menu():
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    profile_button = types.KeyboardButton("Мой профиль")
    get_points_button = types.KeyboardButton("Получить поинты")
    referral_program_button = types.KeyboardButton("Реферальная программа")
    markup.add(profile_button, get_points_button, referral_program_button)
    return markup

# Команда /start
@bot.message_handler(commands=['start'])
def start(message):
    user_id = message.from_user.id
    if user_id not in users_data:
        users_data[user_id] = {'points': 0, 'referrals': 0}

    # Генерация реферальной ссылки
    referral_link = f"https://t.me/@leshka_coinbot?start={user_id}"
    
    bot.send_message(message.chat.id, f"Привет! Нажми 'Получить поинты', чтобы начать фармить поинты.\n"
                                      f"Твоя реферальная ссылка: {referral_link}", reply_markup=main_menu())

# Обработка реферальной системы
@bot.message_handler(func=lambda message: message.text.startswith('/start') and len(message.text.split()) > 1)
def handle_referral(message):
    referrer_id = int(message.text.split()[1])
    user_id = message.from_user.id

    # Проверка, был ли пользователь уже зарегистрирован
    if user_id not in users_data:
        users_data[user_id] = {'points': 0, 'referrals': 0}  # Новый пользователь

    # Если реферальная ссылка принадлежит другому пользователю и он существует
    if referrer_id != user_id and referrer_id in users_data:
        if users_data[user_id]['referrals'] == 0:  # Если пользователь не использовал реферальную ссылку ранее
            users_data[user_id]['points'] += 50  # 50 поинтов новому пользователю
            users_data[referrer_id]['points'] += 50  # 50 поинтов рефереру
            users_data[referrer_id]['referrals'] += 1
            users_data[user_id]['referrals'] += 1  # Отмечаем, что пользователь использовал реферальную ссылку
            bot.send_message(message.chat.id, f"Ты получил 50 поинтов за использование реферальной ссылки.\n"
                                              f"Твой баланс: {users_data[user_id]['points']} поинтов.")
            
            # Уведомляем реферера
            bot.send_message(referrer_id, f"Кто-то использовал твою реферальную ссылку! Ты получил 50 поинтов.")
        else:
            bot.send_message(message.chat.id, "Ты уже использовал реферальную ссылку!")
    else:
        bot.send_message(message.chat.id, "Ты не можешь использовать свою собственную реферальную ссылку или неверная ссылка!")

# Обработка кнопки "Получить поинты"
@bot.message_handler(func=lambda message: message.text == "Получить поинты")
def get_points(message):
    markup = types.InlineKeyboardMarkup()
    farm_button = types.InlineKeyboardButton("Фармить", callback_data="farm")
    markup.add(farm_button)
    bot.send_message(message.chat.id, "Нажми 'Фармить', чтобы начать зарабатывать поинты!", reply_markup=markup)

# Обработка нажатия на кнопку "Фармить"
@bot.callback_query_handler(func=lambda call: call.data == "farm")
def farm(call):
    markup = types.InlineKeyboardMarkup()
    subscribe_button = types.InlineKeyboardButton("Подписаться на канал", url="https://t.me/texriz_tg")
    check_subscription_button = types.InlineKeyboardButton("Проверить подписку", callback_data="check_subscription")
    markup.add(subscribe_button, check_subscription_button)

    bot.send_message(call.message.chat.id, "Подпишись на канал, чтобы получить 150 поинтов!", reply_markup=markup)

# Проверка подписки на канал
def check_user_subscription(user_id):
    try:
        member = bot.get_chat_member(chat_id=CHANNEL_USERNAME, user_id=user_id)
        if member.status in ['member', 'administrator', 'creator']:
            return True
        else:
            return False
    except Exception as e:
        print(f"Ошибка проверки подписки: {e}")
        return False

@bot.callback_query_handler(func=lambda call: call.data == "check_subscription")
def check_subscription(call):
    user_id = call.from_user.id
    if check_user_subscription(user_id):
        users_data[user_id]['points'] += 150
        bot.send_message(call.message.chat.id, f"Подписка подтверждена! Ты получил 150 поинтов.\n"
                                               f"Твой баланс: {users_data[user_id]['points']} поинтов.")
        
        # Отправка изображения с текстом "Леша Коин"
        with open('lesha_coin.jpg', 'rb') as photo:
            bot.send_photo(call.message.chat.id, photo, caption="Ты заработал 'Леша Коин'!")
    else:
        bot.send_message(call.message.chat.id, "Ты ещё не подписался на канал!")

# Обработка кнопки "Мой профиль"
@bot.message_handler(func=lambda message: message.text == "Мой профиль")
def show_profile(message):
    user_id = message.from_user.id
    if user_id in users_data:
        user_data = users_data[user_id]
        bot.send_message(message.chat.id, f"Твой профиль:\n"
                                          f"Поинты: {user_data['points']}\n"
                                          f"Рефералы: {user_data['referrals']}")
    else:
        bot.send_message(message.chat.id, "Ты ещё не зарегистрирован. Нажми /start.")

# Обработка кнопки "Реферальная программа"
@bot.message_handler(func=lambda message: message.text == "Реферальная программа")
def referral_program(message):
    user_id = message.from_user.id
    referral_link = f"https://t.me/@leshka_coinbot?start={user_id}"
    bot.send_message(message.chat.id, f"Твоя реферальная ссылка: {referral_link}\n"
                                      f"Приглашай друзей и получай по 50 поинтов за каждого!")
    
bot.polling(none_stop=True)

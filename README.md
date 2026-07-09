import telebot
from telebot import types
import json
import os

TOKEN = "8893276565:AAHuvC4ENetNoVHv6Y40JtG5pcrXhF977Nw"

bot = telebot.TeleBot(TOKEN)

DB_FILE = "users.json"

if not os.path.exists(DB_FILE):
    with open(DB_FILE, "w") as f:
        json.dump({}, f)

def load_users():
    with open(DB_FILE, "r") as f:
        return json.load(f)

def save_users(users):
    with open(DB_FILE, "w") as f:
        json.dump(users, f, indent=4)

@bot.message_handler(commands=["start"])
def start(message):
    users = load_users()
    user_id = str(message.from_user.id)

    if user_id not in users:
        users[user_id] = {"balance": 0, "referrals": 0}
        save_users(users)

    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.row("💰 الاستثمار", "💳 الإيداع")
    markup.row("💸 السحب", "👥 الإحالات")
    markup.row("👤 حسابي", "⚙️ الإعدادات")

    bot.send_message(
        message.chat.id,
        "🤖 أهلاً بك في ProfitRiseAI",
        reply_markup=markup
    )

@bot.message_handler(func=lambda message: True)
def buttons(message):
    users = load_users()
    user_id = str(message.from_user.id)

    if message.text == "💰 الاستثمار":
        markup = types.InlineKeyboardMarkup()
        markup.add(
            types.InlineKeyboardButton("📅 الباقات الأسبوعية", callback_data="weekly")
        )
        markup.add(
            types.InlineKeyboardButton("🗓️ الباقات الشهرية", callback_data="monthly")
        )

        bot.send_message(
            message.chat.id,
            "💰 اختر نوع الباقة:",
            reply_markup=markup
        )

    elif message.text == "💳 الإيداع":
        bot.send_message(
            message.chat.id,
            "💳 الإيداع\n\nالحد الأدنى للإيداع: 10 USDT"
        )

    elif message.text == "💸 السحب":
        balance = users[user_id]["balance"]
        bot.send_message(
            message.chat.id,
            f"💸 السحب\n\nرصيدك الحالي: {balance} USDT"
        )

    elif message.text == "👤 حسابي":
        balance = users[user_id]["balance"]
        refs = users[user_id]["referrals"]

        markup = types.InlineKeyboardMarkup()
        markup.add(
            types.InlineKeyboardButton(
                "📜 سجل السحب",
                callback_data="withdraw_history"
            )
        )

        bot.send_message(
            message.chat.id,
            f"""👤 حسابي

🆔 ID: {user_id}

💰 الرصيد: {balance} USDT

👥 الإحالات: {refs}
""",
            reply_markup=markup
        )

    elif message.text == "👥 الإحالات":
        link = f"https://t.me/اسم_البوت?start={user_id}"

        bot.send_message(
            message.chat.id,
            f"👥 رابط الإحالة:\n\n{link}"
        )

    elif message.text == "⚙️ الإعدادات":
        bot.send_message(
            message.chat.id,
            "⚙️ الإعدادات\n\n📞 تواصل معنا\n@YourSupport"
        )
@bot.callback_query_handler(func=lambda call: True)
def callback(call):

    if call.data == "weekly":

        markup = types.InlineKeyboardMarkup(row_width=1)

        markup.add(types.InlineKeyboardButton("💎 باقة 5 USDT", callback_data="week5"))
        markup.add(types.InlineKeyboardButton("🥉 باقة 10 USDT", callback_data="week10"))
        markup.add(types.InlineKeyboardButton("🥈 باقة 15 USDT", callback_data="week15"))
        markup.add(types.InlineKeyboardButton("🥇 باقة 20 USDT", callback_data="week20"))
        markup.add(types.InlineKeyboardButton("🔙 رجوع", callback_data="back"))

        bot.edit_message_text(
            "📅 الباقات الأسبوعية\n\nاختر الباقة:",
            call.message.chat.id,
            call.message.message_id,
            reply_markup=markup
        )

    elif call.data == "monthly":

        markup = types.InlineKeyboardMarkup(row_width=1)

        markup.add(types.InlineKeyboardButton("💎 باقة 25 USDT", callback_data="month25"))
        markup.add(types.InlineKeyboardButton("🥉 باقة 50 USDT", callback_data="month50"))
        markup.add(types.InlineKeyboardButton("🥈 باقة 100 USDT", callback_data="month100"))
        markup.add(types.InlineKeyboardButton("🥇 باقة 150 USDT", callback_data="month150"))
        markup.add(types.InlineKeyboardButton("🔙 رجوع", callback_data="back"))

        bot.edit_message_text(
            "🗓️ الباقات الشهرية\n\nاختر الباقة:",
            call.message.chat.id,
            call.message.message_id,
            reply_markup=markup
        )

    elif call.data == "back":

        markup = types.InlineKeyboardMarkup(row_width=1)

        markup.add(types.InlineKeyboardButton("📅 الباقات الأسبوعية", callback_data="weekly"))
        markup.add(types.InlineKeyboardButton("🗓️ الباقات الشهرية", callback_data="monthly"))
        markup.add(types.InlineKeyboardButton("❌ إغلاق", callback_data="close"))

        bot.edit_message_text(
            "💰 اختر نوع الباقات:",
            call.message.chat.id,
            call.message.message_id,
            reply_markup=markup
        )

    elif call.data == "close":

        bot.delete_message(
            call.message.chat.id,
            call.message.message_id
        )

    elif call.data == "week5":
        bot.send_message(call.message.chat.id, "💎 باقة 5 USDT")

    elif call.data == "week10":
        bot.send_message(call.message.chat.id, "🥉 باقة 10 USDT")

    elif call.data == "week15":
        bot.send_message(call.message.chat.id, "🥈 باقة 15 USDT")

    elif call.data == "week20":
        bot.send_message(call.message.chat.id, "🥇 باقة 20 USDT")

    elif call.data == "month25":
        bot.send_message(call.message.chat.id, "💎 باقة 25 USDT")

    elif call.data == "month50":
        bot.send_message(call.message.chat.id, "🥉 باقة 50 USDT")

    elif call.data == "month100":
        bot.send_message(call.message.chat.id, "🥈 باقة 100 USDT")

    elif call.data == "month150":
        bot.send_message(call.message.chat.id, "🥇 باقة 150 USDT")

    elif call.data == "withdraw_history":
        bot.send_message(
            call.message.chat.id,
            "📜 لا توجد عمليات سحب حتى الآن."
        )

print("Bot Started...")
bot.infinity_polling()

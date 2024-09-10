# blackcat
black cat coin mining
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext

# Kullanıcıların bilgilerini saklayacağımız sözlük
users = {}

# Kullanıcı başlangıç parası
STARTING_BALANCE = 1000

def start(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id

    # Kullanıcıyı yeni bir oyuncu olarak ekle
    if user_id not in users:
        users[user_id] = {
            'balance': STARTING_BALANCE,
            'mining_power': 1,  # Başlangıç madencilik gücü
            'crypto_owned': 0   # Kullanıcının sahip olduğu kripto miktarı
        }

    update.message.reply_text(f'Hoş geldin! Başlangıç bakiyen: {STARTING_BALANCE}')

def balance(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id

    if user_id in users:
        balance = users[user_id]['balance']
        crypto_owned = users[user_id]['crypto_owned']
        update.message.reply_text(f'Bakiye: {balance}, Sahip olduğun kripto: {crypto_owned}')
    else:
        update.message.reply_text('Henüz oyuna başlamadın. /start komutunu kullan.')

def mine(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id

    if user_id in users:
        # Basit bir madencilik algoritması: Kullanıcıya 10 birim kripto kazandır
        crypto_earned = users[user_id]['mining_power'] * 10
        users[user_id]['crypto_owned'] += crypto_earned
        update.message.reply_text(f'Madencilik yaptın! Kazandığın kripto: {crypto_earned}')
    else:
        update.message.reply_text('Henüz oyuna başlamadın. /start komutunu kullan.')

def trade(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id
    args = context.args

    if user_id in users:
        if len(args) == 2 and args[0] == 'buy':
            amount = int(args[1])
            if users[user_id]['balance'] >= amount:
                users[user_id]['balance'] -= amount
                users[user_id]['crypto_owned'] += amount // 10  # Basit bir oran
                update.message.reply_text(f'Başarıyla {amount} birim kripto aldın!')
            else:
                update.message.reply_text('Yeterli paran yok!')
        elif len(args) == 2 and args[0] == 'sell':
            amount = int(args[1])
            if users[user_id]['crypto_owned'] >= amount:
                users[user_id]['crypto_owned'] -= amount
                users[user_id]['balance'] += amount * 10  # Basit bir oran
                update.message.reply_text(f'Başarıyla {amount} birim kripto sattın!')
            else:
                update.message.reply_text('Yeterli kripton yok!')
        else:
            update.message.reply_text('Yanlış komut girdin! /trade buy <miktar> ya da /trade sell <miktar> kullan.')
    else:
        update.message.reply_text('Henüz oyuna başlamadın. /start komutunu kullan.')

def main():
    # Bot API tokenini gir
    updater = Updater("TELEGRAM_BOT_API_TOKEN")

    dispatcher = updater.dispatcher

    # Komutlar
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("balance", balance))
    dispatcher.add_handler(CommandHandler("mine", mine))
    dispatcher.add_handler(CommandHandler("trade", trade, pass_args=True))

    # Botu çalıştır
    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
